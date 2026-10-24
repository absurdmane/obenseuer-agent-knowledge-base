# Levels (scenes) and entrypoints

Scope: Changing scenes: `SaveController.ChangeLevel`, entrypoints, where the player spawns.
Status: code-only; one item [verified]
Related: [saves.md](saves.md), [containers-theft.md](containers-theft.md)

- [code] `SaveController.ChangeLevel(level, entrypoint = "NONE")` is the level change doors use (`Changelevel.ChangeLevel`):
  waits while `SaveController.Loading`, autosaves (into the oldest autosave slot), loads the new scene with the loading screen.
  `SaveController.CheckIfLevelExists(level)` = `Application.CanStreamedLevelBeLoaded`.
- [code] After the scene loads, `MovePlayerToEntrypointDelayed(entrypoint)` moves the player 0.1 s (realtime) later, and
  `SaveController.LoadingDone` fires right after scheduling it. So a mod that wants to place the player itself waits a bit longer.
- [code] `"NONE"` = no move: the player keeps the saved position, which after a scene change is the old scene's coordinates.
- [code] An entrypoint name the scene doesn't have: `Debug.LogError("Entrypoint X not found!")`, then the first entrypoint without
  `dontUseAsDefaultSpawn`, i.e. the scene's default spawn. Usable to reach a scene whose entrypoint names are unknown.
- [code] Entrypoints of the loaded scene: `PlayerLevelEntrypoints.instance.Entrypoints` (`Name`, `Location`, `OtherEntyPoint`;
  `UnderMapDrop` is special). `Changelevel` doors add theirs in `Awake`; a door has `OtherLevel` + `OtherEntrypoint` = where it leads.
  Names of other scenes' entrypoints are only known from such doors or after visiting.
- [code] `Teleport.TeleportPlayerToTargetStatic(Transform)` = `PlayerLocator.instance.Teleport(target)`.
- [verified] Changing level while sitting (`InteractableChair.Sit` → `GameController.ControlsDisabledMove(chair)`) leaves the
  player unable to walk: `OpenSewerCharacterController.disabledList` keeps the destroyed chair as a holder, and the static
  `InteractableChair.playerSittingGlobal` stays true. Doors can't be used while sitting; a scripted level change must check
  `InteractableChair.PlayerIsSitting` and that `disabledList` has no live holder other than its own.

## Doors between levels, preloading, spawners, portals `[code]`

- `Changelevel` (Interactable) only checks `SaveController.CheckIfLevelExists(OtherLevel)`; `DoorChangelevel` adds the lock, an `OnOpen` relay, sounds and an NPC door-open coroutine. `OpenDoor` disables controls, hides the game menu, then changes level directly or after `Delay` (default: open-sound length + 0.1 s). Its lock state is stored in `GlobalState` under `OtherLevel + "_" + OtherEntrypoint`, so the destination door picks it up on load (`DontSyncLocks` turns this off, `LockAfterUse` locks after a use).
- Scene names seen in code: `"Main Menu"`, `"Open Sewer Intro"` (new game, via `LoadingScreen.LoadLevel`), `"Interior Player Tenement"` (`SleepEventController`). Scene names are also keys in save data (`JanitorController.processedLevels`, `WaterSourceController.scenes`). The full scene list is in the build settings, not in code.
- `LoadSceneAsyncTrigger` preloads a neighbour scene additively when the player enters a trigger (statics `currentAsyncOperations`, `currentAsyncScenes`). `OnTriggerExit` removes the component's name rather than `OtherLevel` from `currentAsyncScenes`, and the `else if` branch of `OnTriggerEnter` adds `onLoadSceneAsync` which may be null: check before relying on the static lists. `LoadOnLevelIni` is a DontDestroyOnLoad carrier that applies save data after a load.
- `BuildingArea` (trigger volumes, player layer 10) sets `BuildingArea.currentManager` and `BuildingSystem.activeManager`; `Exit` clears them and destroys the build preview.
- `TenementController` (SavableScript) subscribes to `TimeOfDayAzure.SecondsPassed` and `DayChanged` in `Start`; scene-side `TenementGeneralPrefabs`/`TenementApartmentPrefabs` register with it and receive saved state.
- `Spawner` (SavableScript): `Start` destroys child objects that are not in its saved `objects` list (pre-placed children vanish); after time is enabled and three frames it rolls `spawnChance`; `DeltaSeconds` asks `SpawnSettings.TimeToSpawn`; with `SpawnSettings.IsPlayerTooClose` the spawn is deferred and `OnMapChanged` retries it. Radius spawners respawn only while the count is at most half of `objectsToSpawn` or `removeOldItems` is set. `SaveController` calls `OnMapChanged` on every root object of a newly loaded scene.
- Occlusion: `Door.UpdateOcclusion` sets `occlusionPortal.open` and runs the public `onOcclusionUpdated` action list. `Areaportal.LateUpdate` (every frame) opens its `OcclusionPortal` while the player is within `distance` (default 25) and fades the mesh alpha near the limit; it reads `PlayerStats.instance.transform` without a null check. `RelayPlayerDistance` (default 40) fires its relay once per day and on `DayChanged` only when the player is far enough; spawners and relays measure from `PlayerLocator.instance.transform`, areaportals from `PlayerStats.instance.transform`.
- Addressables from code: `SavableScriptableObject` and `SavedAssets` call `LoadAssetAsync(...).WaitForCompletion()` (blocking); `LoadingScreen` loads and releases a hint sprite; no code instantiates prefabs through Addressables.

Unverified: whether each tenement building or apartment is its own scene; whether `LoadSceneAsyncTrigger` is used in any scene.
