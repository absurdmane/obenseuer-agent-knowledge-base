# Map, fast travel, teleport, trains, pathfinding graphs
Scope: the map panel and landmarks, fast-travel entry, teleporting the player, train shop and tracks, NPC pathfinding graphs, sewer controller disable/water-exit parts. Player speeds are in player-movement-tiredness-sleep.md. The fast-travel panel's destination logic, `info_map` internals and train coupler/crash logic are not covered.
Status: code-only
Related: [player-movement-tiredness-sleep.md](player-movement-tiredness-sleep.md), [levels.md](levels.md), [npc-schedules.md](npc-schedules.md), [saves.md](saves.md)

## Where it lives [code]

- Player controller: `OpenSewerCharacterController` (`instance`). Map: `Map` (panel with zoom and markers), `MapController` (SavableScript, per-map landmark data with inner `MapInfo`/`LandmarkInfo`), `LandmarkTrigger`, `PlayerMarker`. Fast travel: `FastTravelController`, `InteractableFastTravel`. Teleport: `Teleport` (SavableScript), `PlayerLocator` (player transform, head and rig references).
- Trains: `Train` (furniture), `TrainShop : BaseShop`, `TrainShopUI`, `TrackTrain` (SavableScript, `TrackTrain.TrainData`), `TrainController` (`instance`, registry by GUID of trains, `Track_Segment`s and `MinitrainFuelTank`s), coupler and crash triggers.
- Pathfinding: `NPC/WaypointGraph`, `NPC/InterScenePathfindingGraph`, `NPC/NPCPathfinding`; path types come from a third-party A* library.

## How it works [code]

- `OpenSewerCharacterController.DisableControls(go, noclip)` adds `go` to a private list (with `noclip` it also makes the Rigidbody kinematic); `EnableControls` removes it; `ControlsDisabled()` is true while the list is non-empty. Call `EnableControls` with the same object or controls stay locked.
- `TryWaterLedgeExit` (from the jump path) raycasts forward for a ledge up to 2.2 m above the water; if found, a timed lift and push moves the player out, otherwise a normal jump impulse applies.
- `Map` reads the per-scene `info_map.instance` (map image, reveal material) and `info_game_logic.hideMainMap`; `AddLandmark` inserts sorted by `landmark.priority` and posts a "Discovered" notification unless suppressed. `MapController` keys its data by the current map id (`requiredTaskItem.Id` if set, else the map image) and skips a landmark whose id is already present. `OnLoadingGame` waits 0.7 s, then re-adds all landmarks silently; `LandmarkTrigger` waits two frames (and 0.8 s while the map loads). Code that runs earlier sees an empty map.
- `InteractableFastTravel.Interact` -> `FastTravelController.ToggleMenu` -> `GameUIController.ToggleGameMenu(..., MenuPanels.FastTravel, ...)`; the destination list lives in the panel (not read).
- `Teleport.TeleportPlayer()` moves the player to `location` unless `TeleportDisabled`/`playerTeleportDisabled`, then fires `onTeleport`; `TeleportTarget()` moves `target` (default the player) only when target and location are farther than their minimum distances; `TeleportPlayerToEntrypoint(name)` delegates to `PlayerLevelEntrypoints`. `PlayerLocator.Teleport(t)` copies position, rotation and scale to the transform and Rigidbody, zeroes velocity, resets the third-person camera yaw/pitch and updates a `PersistLocation`. Callers include `BlackoutTrigger`, `DoctorExaminationBed`, `PlayerLevelEntrypoints`, `Prison`, `ChangeScene` and `SaveController`.
- `TrainShop` owns a `trackSegment` and `defaultPositionOnSegment`; `availableTrains` (sellable stock, saved through `FurnitureInfo.SavePath`) and `currentTrainsOnTrack` (saved); `AddTrainToTrack` places a `TrackTrain` and couples it to the previous one. `TrackTrain` field defaults: `maxSpeed` 30, `maxReverseSpeed` -20, `acceleration` 0.65, `brake` 1, `drag` 0.01.
- `InterScenePathfindingGraph` loads `Assets/Scenes/Scenegraph.nav` (path from `Application.dataPath`), a list of scene connections; `FindScenePath` is a breadth-first search over scenes with entrypoint matching; `GenerateConnections` builds entries from `Waypoint_LevelChange` objects. `WaypointGraph.WaypointConnection` holds per-region nodes with cached `ABPath`s; `CalculateDistances` runs asynchronously.

## Mod guidance

`Teleport.TeleportPlayerToTargetStatic` is the funnel for player teleports; patch there to see all callers. `PlayerLocator.Teleport` uses `ThirdPersonCameraController.instance` without a null check. A mod-added landmark must use the id returned by the current map id lookup. Do not rename the saved `TrainShop.currentTrainsOnTrack`/`availableTrains` fields.

## Unverified

What feeds the fast-travel list (teleports or landmarks); whether `TryWaterLedgeExit` is called only from the jump path; scene and asset values (teleport distances, segment ids, map image ids).
