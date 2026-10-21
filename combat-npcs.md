# Combat, damage, hostile NPCs, spawning

Scope: who can be damaged and by what, how the player takes damage and dies, the one existing human aggression (`Act_Robber`), animal attacks, how scene spawners work, and what is known about infected-human models. Not covered: NPC schedules in general (`NPCDirector`, `Schedule`), crime and police in detail, dialogue.
Status: partly verified
Related: [player-movement-tiredness-sleep.md](player-movement-tiredness-sleep.md), [setting.md](setting.md), [asset-bundles.md](asset-bundles.md), [levels.md](levels.md)

## Damage interfaces [code]

- `IDamageReceiver.Damage(int damage, Vector3 direction, Vector3 point, DamageReceiver.DamageType type)`; `DamageType` is `Any, Blunt, Chop, Point, Cut`. Implemented only by `Breakable`, `BreakableLock`, `SpawnItemOnBreak` and `WorkableResourceSource` (trees, rocks, locks). No human implements it. Animals do, through `SpawnItemOnBreak`: it has `objectType = Animal`, `health`, `allowedDamageTypes`; `AnimalStats.Death` itself looks for that component on the same object.
- [verified] Rats and cats can be killed with melee weapons (axe, sledgehammer; user, 0.5.08). Path [code]: `MeleeWeapon.Hit` -> `GetComponent(typeof(IDamageReceiver))` on the hit collider -> `SpawnItemOnBreak.Damage(int, ..., type)`: type check, `health -= damage`, at <= 0 spawns the item/prefab (`itemToSpawn`, `prefabToSpawn`), plays `breaksound`, fires `onBreak`; for `Type.Animal` witnesses within 10 m comment "Generic/Animal Kill Negative" and, if the animal has an owner NPC and the kill is witnessed, `Crime.CommitCrime(owner, value * 0.1, AnimalCruelty)`; with `destroy` the object is destroyed (`DestroyWithEffect`). Requires the `SpawnItemOnBreak` to sit on the GameObject that has the hit collider. `AnimalStats.Damage` is the other (non-melee) route and ends in the same component.
- `Stats.Damage(DamageType damageType, float effectfactor)` is the other path (`DamageType` here is the data class from `DamageController.GetDamageType(DamageTypeNames)`: sounds, message). `Stats.type` is `Player, Human, Chicken, Chick, Cat, Dog, Rat, Bird, Egg, Pig`. Subclasses: `AnimalStats` and `PlayerStats` only. There is no stats class for humans.
- `AnimalStats.Damage`: skipped when the type is in `immuneDamageTypes`; `animalData.currentHealth -= effectfactor`; at <= 0 `Death()` (fires `onDead`, `SpawnItemOnBreak.Destroy` or `Object.Destroy`), otherwise `Animal.OnHurt()`. Callers: `RatTrap` (100), `TriggerHurt`, `Env_Explosion`, `AnimalAttackTrigger`.
- `MeleeWeapon.Hit` (private, called from the swing): raycast result from `InteractObjects.instance.meleeRaycastHit`. A collider with `CharacterBarkStarter` counts as a human: plays the "human" hit sound; with `NPCInfo`: `onHit` relay, a "Generic/Hit" bark or dialogue (`dontCommentOnHit`, `startDialogueOnHit`), `Crime.CommitCrime(npc, 100, Violence)` + `Witness.IsWitnessed()` (unless `dontGiveCrimeOnHit`), animator trigger "hit", and `Act_Police.StartChase()` / `Act_Robber.AbortRobbing()` if the NPC is doing those. Then for any collider: rigidbody push, and `GetComponent(typeof(IDamageReceiver))?.Damage(primaryDamage, ...)`.
- **Humans cannot be hurt or killed.** `NPCStats` has `Health`, `MaxHealth`, `Damagefactor`, `Agressiveness`, `Coward`, `SMV`, but nothing outside the class reads them (checked by Grep). A melee hit on a human only does the reactions above. `NPCInfo.IsDead()` exists (not read further).

## Player health and death [code]

- `PlayerStats.Damage(DamageType, float)` only drives UI (`PlayerStatsUI.Damage`: red flash alpha, hit sound). It does **not** remove health.
- Real damage: `PlayerStats.instance.health.Damage(float damage, DamageTypeNames reason, bool silent = false)` (`PlayerStatsHealth`). Special cases: `Mushrooms` is ignored with wasteland effect or face mushroom protection and adds `mushrooms.SMVProgression`/`SMVProgressionRate`; `Gass` is ignored with gas protection; `Bleed` adds to `health.Bleeding` instead of `currentStatus`. Otherwise `currentStatus -= damage`, flash via `PlayerStats.Damage`, camera shake for `Violence`.
- `DamageTypeNames`: Unspecified, Violence, Hunger, Thirst, ShitSeizure, SMV, Drown, Fire, Radiation, Fall, Poison, Withdrawal, Suicide, Bleed, Mushrooms, Explosion, Gun, Gass, Force, Liquid, Electric. `DamageController.instance` maps each to sounds/message/death sound.
- `currentStatus < 0` -> `PlayerStats.death(reason)` -> `Death.PlayerDeath`: death sounds, `ShowDeathEffect` (hides the HUD bars, shows the reason message), `PauseMenu.PausemenuDeath` (pause menu opens, **Resume and Save are disabled**: the player can only load or quit), death message. So a lost fight costs the unsaved progress since the last save.
- Other `health.Damage` callers: `Act_Robber` (violence), `TriggerHurt` (areas), `DialogueEventSystem` (Lua/dialogue hits), `SleepEventController`, `FallDamage`, `Water`, `Minitrain`, `Env_Explosion`, hunger/thirst, radiation, sauna.

## Animal attacks [code]

`AnimalAttackTrigger` (on the attack collider of rats/cats; `OnTriggerStay`): non-tame animals bite the player with `playerBiteChance` once per 10 s, otherwise attack their `CurrentTarget`. `Attack(stats)` calls `stats.Damage(DamageController.GetDamageType(DamageType), Damage)`; for the player that is the UI-only overload, then `Animal.Attack` plays the animator trigger and stops the animal until `OnAttackDone`. `TryBitePlayer` also only calls `PlayerStats.Damage`.
So by code alone a bite is feedback only; where the health loss of cat/rat bites comes from is **not found in code**. `AnimalAttackTrigger.Start` copies `Damage`/`DamageType` from a `TriggerHurt` on the prefab and sets `triggerHurt.Enabled = false`. Open: needs a runtime check (log `health.currentStatus` around a bite) before relying on it. `RatFightArena` is the rat-betting game (`PlayerStats.gambling`), not combat.
Movement of animals: `Animal` with an A* `IAstarAI`, `aiDestinationSetter.target`, states Default/Attacking/Eating/Sleeping/Sitting, run/hunt timers.

## Existing human aggression: `Act_Robber` [code]

Humans are driven by activities (`NPC.Activity`, started with `NPCController.StartActivity(activity, interrupt)` / `QueueActivity`), movement by `NPCController.SetChaseDestination(pos, minDistance)` / `SetFollowDestination` over A*. `Act_Robber` states: `Idle, Stand, Patrol, Wander, Chase, Aggro`.
- `OnPlayerDetected`: player wields an item of category "Weapon" -> the robber backs off (`ChangeRobberAloofStatus`); player has neither 1000 OC nor 33 RM (`minimumMoney`, checked with `Money.CheckMoney`) -> backs off; otherwise `StartChase()`.
- `AggroPlayer` (each frame): follows the player (retargets when the player moved > 1 m); within 1.25 m and `graceTimeRemaining == 0` and not changing level -> `health.Damage(Random.Range(5, 33), Violence)`, random audio, grace +1 s; below 25 health `StartChase()` (stops hitting). Ends after 10 s without sight, when the player wields a weapon, or on police.
- Melee on a robber calls `AbortRobbing`. So the code for "NPC walks to the player and hits" exists and can be a template; it is per-NPC activity data (serialized state, `[fsIgnore]` for runtime fields).
- No aggression tied to mushrooms, SMV or "zombie": Grep over the decompiled code finds none (`NPCStats.Agressiveness` is unused), and `Items.json` has no text about aggressive effects. `MushroomEffect`/`OrangeMushroomEffect` are player-side effects (not read in detail). Treat "red mushrooms make aggressive zombies" as flavour (no mechanism found).

## Spawners [code]

- `Spawner : SavableScript` is a placed scene object: `locations`/`radius`, `objectsToSpawn` (+variation), re-spawns by game time (`SecondsPassed` -> `DeltaSeconds`, `TrySpawn`), saves `spawnedList` (item ids), `spawnPositions`, `spawnRotations`, `savedTimeAndDay`. `CollectibleItemSpawner` adds item meta (`savedMetaList`).
- `AnimalSpawner : CollectibleItemSpawner`: animals are **items** (prefab with `CollectibleItem` + `Animal` + `AnimalStats`); on save it refreshes positions from the live objects. So wild rats/cats are saved with the scene spawner state.
- `CharacterSpawner : Spawner`: instantiates unique character prefabs (`characters[]`) at `spawnPoints`, `IsCharacterUnique` against `CharacterManager.CharactersInScene`.
- `AnimalController` (global, `animalLimit` 30): tame followers are stored as `ItemStack`s in the global save (`savedFollowingAnimals`), destroyed in `OnMapChanging`, re-instantiated near the player in `OnMapChanged` via `NPCSpawnUtilities.FindValidSpawnPosition(position, name, radius, height)`.
- `NPCSpawnUtilities.FindValidSpawnPosition` finds a free spot near the given position (not read in detail). Humans that live in the world are `NPCData`/`NPCDataState` records handled by `NPCDirector` and moved between scenes by `NPCSpawnUtilities.ChangeNPCLevel`; those are saved.

## Infected-human models (assets) [data] — Spoilers (names of assets beyond the first explored areas)

Searched the Addressables bundles (`find_names.py`) for infect/zombie/shroom/fungus/mutant/corpse names. They hold furniture and item prefabs, not NPCs. Mushroom-person things found: `Shroom_man_001_hat` material, `Body Pillow Shroom Girl`, `painting_004_mushroom_person`, `crazy_neighbor_001_*` (materials, animator controllers `crazy_neighbor`, `crazy_neighbor_sitting`, `crazy_neighbor_door`; it is a door/sitting event NPC, not an enemy). Humans are assembled from body-part objects named `Body: arms.biceps`, `Body: head`, `Body: full`, ... (`CharacterAppearance`, `CharacterModel`). Infected NPCs with mushrooms on them, if they exist as models, are in scene files (`levelN`) or built from clothing parts; **not located**. Needs a scene scan (offline asset reading) or a runtime look at the NPC hierarchy.

## Mod guidance (hostile NPCs, feasibility) [code]

- Own spawns: instantiate your own objects outside any `Spawner`/`NPCData` so nothing about them is saved; destroy them on `OnMapChanging` (same pattern as `AnimalController`), re-create near the player on `OnMapChanged` if wanted. That keeps saves clean when the mod is removed.
- A simple enemy needs: a model + A* agent (`IAstarAI`/`AIPath`) or your own NavMesh movement, a chase loop like `Act_Robber.AggroPlayer`, and damage through `PlayerStats.instance.health.Damage(x, DamageTypeNames.Violence)` (not `PlayerStats.Damage`). Player death handling is done by the game.
- To make humans killable you would have to add your own health (no vanilla model): patch/replace `MeleeWeapon.Hit` behaviour or add a collider component implementing `IDamageReceiver` to your own NPC; vanilla humans keep the crime/bark reactions.
