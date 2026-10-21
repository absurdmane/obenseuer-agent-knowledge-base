# Built-in developer console

Scope: The F1 developer console: existing commands, the game calls under them, how to add your own commands (mod guidance), and the same actions without the console.
Status: code-only; that the console opens in the release build is not verified (`DeveloperConsole` is in the release assemblies and its name is in `globalgamemanagers.assets` / `resources.assets`)
Related: [inventory-storage.md](inventory-storage.md), [saves.md](saves.md), [levels.md](levels.md)

- `DeveloperConsole` (singleton `Instance`, `DontDestroyOnLoad`). Toggle with **F1** (`ShouldToggleDevConsole`), or View+Menu on a gamepad.
- Opening it sets `Achievements.instance.globalDisable = true` until it's closed, so achievements are off while it's open.
- `ExecuteCommand(TMP_Text)` → private `HandleInput(string)`. The input is split on spaces; quoted parts stay whole.
  It's a big `switch (array[0])`, and `default` prints "is not a valid debug menu command".
- Help text: private `_validCommandExamples : string[]`, filled in `Awake`, printed by `help`.
- Output: `DeveloperConsole.Instance.Print(string)` (rich text). Handlers live in `DeveloperConsoleInputHandler` and
  `DeveloperConsoleRoutines`.

## Existing commands (0.5.08)

- General: `god_mode`, `kill`, `noclip`, `unlock`, `show_unity_console`, `destroy_all_items`.
- Items: `add_item id amount`, `find_item name`, `add_task_item`, `add_blueprint`, `add_note`, `list_*`.
- Money: `add_money currency amount`, `remove_money`.
- Time and weather: `change_time h[:m]`, `set_ingame_time_scale value`, `change_weather`, `list_weathers`, `thunder_storm`.
- Stats: `player_stats get | reset_stats | clear_needs | disable_progress | <stat> add/subtract/set amount | skill name amount`.
- Tasks: `set_task_status`, `set_objective_status`, `list_tracked_tasks`, ...
- Tenement: `ignore_upgrade_prices_and_items`, `complete_upgrades_instantly`.
- Furniture: `add_furniture name amount skin`, `add_furniture_all amount`, `find_furniture name`.
- Scenes, crime, trains, locales.

## Mod guidance: adding your own commands

Prefix on `DeveloperConsole.HandleInput` (publicized): split the input the same way. If `array[0]` is one of ours, handle it
and return false; otherwise return true. Postfix `Awake` to append your lines to `_validCommandExamples` so `help` lists them.

## The same without the console (for a mod menu) `[code]`

`DeveloperConsoleRoutines` print through `DeveloperConsole.Instance`, so call the game code under them directly:
- Items: `ItemDatabase.database` (static list), `ItemOperations.AddItemsAndDropRemaining(item, amount: n)`.
- Key/quest items (passport): `Resources.LoadAll<TaskItem>("TaskItems")`, `TaskItemsManager.instance.AddTaskItem(item)`.
- Furniture: `DeveloperConsole.Instance.furniture.furnitures` (+ `Resources.FindObjectsOfTypeAll<Furniture>()`),
  `BuildingSystem.instance.AddFurniture(furniture, amount, meta)`; skin via `FurnitureInfo.Meta.currentSkin`.
- Player: `PlayerStats.instance.Godmode`, `.StatsProgressDisabled`, `.ClearAllNeeds()`, `.Skills[]` (`PlayerSkill.Skill`).
- Money: `Money.instance.AddMoney(float, Money.Currency)`.
- Time: read `TimeOfDayAzure.instance.CurrentTimelineValue` (hours); **its setter is empty** [verified]: set with `SetTimeline(hours)`.
  Speed: `.timeSpeedOverrideFactor`. Weekday: `GetDayOfWeek()` (0 = Monday) comes from the calendar date; change it with
  `SetDate(y, m, d)` (also updates `currentTimeAndDay`, so the day counter doesn't move). `GetDayOfWeekString()` for the name.
- Teleport: the player is `PlayerLocator.instance` (has a `Rigidbody`); `PlayerLocator.Teleport(Transform)` shows how.
- Freeing the cursor for your own UI: `GameController.instance.ControlsDisabled(owner, ShowCursor: true)` / `ControlsEnabled(owner)`.
- Load a save in game: fade with `LoadingScreen.instance.Fade(...)`, then `SaveController.LoadGameWithMigration(character, save, fromMenu: true)`
  (as `LoadMenu` does). `SaveController.SaveGame(name)` is synchronous. Current save: `SaveController.CharacterName` / `SaveName` (private static).
