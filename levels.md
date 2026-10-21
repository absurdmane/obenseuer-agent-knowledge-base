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
