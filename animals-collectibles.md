# Animals, collectibles, resource nodes, ambient fish
Scope: animal base class and stats, breeding, following animals, collectible items and spawners, harvestable resource nodes, ambient fish, the rat trap. Damage and bites are in combat-npcs.md, plants in growing.md; animal movement and hunting internals and the `Cat`/`Chicken`/`Chick`/`Pig`/`Rat` subclasses are not covered.
Status: code-only
Related: [combat-npcs.md](combat-npcs.md), [growing.md](growing.md), [item-system.md](item-system.md), [levels.md](levels.md), [time-and-weather.md](time-and-weather.md)

## Where it lives [code]

- `Animal` (base; subclasses `Cat`, `Chicken`, `Chick`, `Pig`, `Rat`), `AnimalStats : Stats` (health, hunger, growth), `AnimalBreeder` (SavableScript), `AnimalController` (SavableScript, `instance`), `AnimalSpawner : CollectibleItemSpawner`, `AnimalFoodDetector`, `AnimalAttackTrigger`. Animal data is the item meta `OS.Items.ItemAnimalData` (`tame`, `currentHealth`, `currentHunger`, `currentAge`, `hungerEnabled`).
- `Collectible` (base, static registry `Collectible.allCollectibles`), `CollectibleItem`, `CollectibleItemSpawner : Spawner`, `SpawnItemOnBreak`. Harvest nodes: `WorkableResourceSource : DamageReceiver, IInteractable`. Fish: `Fishspawner`, `Fish` (not savable). `RatTrap : Interactable`.

## How it works [code]

- States: `AnimalState` (Default, Attacking, Eating, Sleeping, Sitting) and idle actions (Wandering, Sit, Sleeping), driven by Animator bools and triggers.
- Food: `AnimalFoodDetector` (on `TimeOfDayAzure.SecondsPassed`) scans `Collectible.allCollectibles` for a `CollectibleItem` in range whose ID is in the animal's `itemsToEat`; `Animal.OnEatingDone` removes one from the item's amount and resets `currentHunger` to 0.
- Hurt: `AnimalStats.Damage` lowers `currentHealth`; at 0 or less `Death` fires the `onDead` relay, then `SpawnItemOnBreak` or destroy; otherwise `Animal.OnHurt` sets `tame = false` and the animal flees. `Animal.ChangeTameStatus` is the clean hook to keep one tame (tameness is `Animal.tame`, mirrored in `ItemAnimalData.tame`).
- Growth: `AnimalStats.UpdateGrow` adds seconds to `currentAge` (only if `growingEnabled`); at `fullGrownAge` it spawns `onFullyGrownPrefab` with the same skin and destroys the young one.
- Breeding: `AnimalBreeder.Breed` runs on each in-game hour. It reads the `Growing` slots of its object, takes fully grown `grow` components (not `cantBreed`), needs two animals per pair (one with `doesntNeedTwoAnimals`), rolls `breedChancePerHour` (default 0.1) and plants the result with `Growing.PlantExternal`. On load it runs one `Breed` per elapsed hour.
- Following animals: `AnimalController.OnMapChanging` stores tame followers as item stacks with animal meta (untame ones are dropped); `OnMapChanged` respawns them near the player after 0.5 s. Population cap: `Animal.UpdateItemSpawnTimer` spawns `itemsToSpawn` only while `AnimalController.animalLimit` (default 30) is not below the animal count.
- Resource nodes: `WorkableResourceSource.Damage` counts toward a randomised threshold (`damageVartiationForYield`); each crossing takes 1 from `currentStock` and gives a random `outputItems` entry through `ItemOperations.AddItemsAndDropRemaining`. A skill raises damage (`1 + skill/3`) and yield and gains 0.005 per hit. Stock regrows via `SpawnSettings.TimeToSpawn` and `ReplenishStock` when `respawnEnabled`; `Stock` is the only saved field (0.5.13 marks `itemSpawnInfo` of `WorkableResourceSource` and `CollectibleItem` `[fsIgnore]`).
- Collectible spawns: `CollectibleItemSpawner` draws from `StorageSpawnCategoryDatabase` by `itemSpawnCategory` (`GetRandomItem(rareItemSpawnChance)`) or `customItemsToSpawn`; point and radius spawns save each item's ID, position, rotation and meta. `CollectibleItem.DeltaSeconds` runs on `MinutePassed` until its spawn timer is done.
- Ambient fish: `Fishspawner.DeltaSeconds` spawns scaled `FishPrefabs` up to `MaxFishes`; `Fish.Update` only swims. No fishing mechanic was found. `RatTrap` has Rat and GlowingRat states and catches a rat on trigger.

## Pitfalls

- `AnimalStats.UpdateHunger` divides by `(sec / 100000)` in integer arithmetic: for `sec` under 100000 the divisor is 0, so hunger saturates and damage applies every tick when `hungerEnabled` is true (whether any shipped animal has it enabled is unchecked).
- `Fishspawner.Start` never picks the last prefab (exclusive `Random.Range` bound) and its null cleanup can skip entries.
- Per-second handlers on `SecondsPassed` must be unsubscribed in `OnDestroy`, as `AnimalStats` and `WorkableResourceSource` do.

## Unverified

Spawn timer and breed chance values per prefab (asset data); whether `Fishspawner` is used in any scene; the relation of `RatTrap` to `RatFightArena`.

## Animal movement, hunting, attacks, subclasses `[code]`

- Movement is A* (`IAstarAI`, `aiDestinationSetter`); `ObstacleAvoidance` (plain helper per animal) samples progress every 0.2 s, retries every 0.75 s when stopped and tries escape angles (+-60, +-90, +-135, 180) for 2.5 s. It is switched off globally while the static count of enabled animals exceeds `AnimalController.obstacleAvoidanceAnimalLimit` (default 50). Random destinations come from a random node of a 25000-cost `ConstantPath`.
- `Animal.Update`: in states other than `Default` (Attacking, Eating, Sleeping) it only checks wake-up. In `Default` a non-tame animal (without `dontFleeFromPlayer`) within 1.5 m of the player flees (`FleeFromPlayer`); arrival either eats a target `CollectibleItem` or starts an idle timer of 20 to 1000 s; a hunt is dropped after `huntStuckTimeout` (5 to 12 s).
- Fleeing: `OnHurt` clears `tame`, sets `fleeing`, starts running (`runSpeed` 2.5 vs `walkSpeed` 1, `runningCooldownTime` 6 s) and heads for `escapeTarget` or a random point.
- Hunting: `OnHuntTargetDetected` (fed by `NPC_FOV.TargetDetected` and `OnTriggerEnter`) ignores targets while fleeing or avoiding an obstacle and targets whose `Stats.type` is not in `targetsToHunt`; for the player (type 0) the hunt happens only if a 0..99 roll is at most `playerHuntChance` (default 100). `HuntTarget` needs hunger above 0.25 and a random value in [0.25, 1] not above the hunger (hungrier animals hunt more), else a 6 s cooldown.
- Attack: `AnimalAttackTrigger.OnTriggerStay` runs only while `attacking` is false. A non-tame animal touching the player rolls `playerBiteChance` at most once per 10 s; otherwise it attacks `Animal.CurrentTarget()`. `Attack` calls `stats.Damage(type, Damage)` then `Animal.Attack` (state Attacking, `ai.maxSpeed = 0`, animator trigger "Attack"); `OnAttackDone` returns to `Default`. The `attacking` flag is cleared by the animation event `AttackDamageEnabled` (the name means the window opened). Cat or rat hits on another animal (for example a chicken) call `AnimalStats.Damage` with `Damage` (default 1): the only animal-on-animal damage path found.
- Bite damage to the player: no C# path applies health loss. `PlayerStats.Damage(...)` only drives `PlayerStatsUI.Damage` (red flash, sound), `AnimalAttackTrigger.Start` disables the prefab's `TriggerHurt` (the component that would call `PlayerStatsHealth.Damage`), and `TryBitePlayer` has no C# caller (only a string animation event could call it, UI-only too). A runtime check (log `health.currentStatus` around a bite) would settle it.
- Subclasses: `Cat` overrides only `Awake`/`OnDestroy` (static `allCats`); `Rat` overrides `OnDestination` (walks to an untriggered `RatTrap.bait` within 5 m) and adds `GetTrapNearRat`; `Chicken` has an egg timer on `SecondsPassed` (spawns `eggItemFertilized` with 70% default, else `eggItem`, only while `AnimalController.animalLimit` allows; ignores tameness and hunger); `Chick` and `Pig` are empty (behaviour comes from prefab data).
- `Animal.Start` adds a `GraphUpdateScene` with a penalty of 100 to every animal.
