# Achievements, platform hooks, boot order
Scope: the achievements class and its Steam calls, platform integration, version handling, build-time switches, the order in which things start. The developer console itself is in dev-console.md. Achievement ids, maximum progress and rewards are serialized data and not recorded.
Status: code-only
Related: [dev-console.md](dev-console.md), [saves.md](saves.md), [main-menu.md](main-menu.md), [ui-options-input.md](ui-options-input.md), [time-and-weather.md](time-and-weather.md)

## Where it lives [code]

- `Achievements` (SavableScript, `instance`) with nested `Achievements.Achievement` and `Achievements.Reward`. `SteamManager` (Steamworks wrapper); only `Achievements` calls Steam stats APIs. No analytics or telemetry type and no HTTP use were found in the game assembly.
- `Application.version` is read in `ShowVersion`, the save header (`SaveController`), `LoadMenu`, and one migration gate.

## How it works [code]

- `Achievements.TriggerAchievement(name)` (overloads: plain, `float` progress, `int` progress with `set`) finds the entry by `API_ID` and calls `Trigger`, `AddProgress` or `SetProgress`; it does nothing while `initialized` is false or `globalDisable` is true. The `triggered` setter calls `SteamUserStats.SetAchievement` once, then `StoreStats()`; progress goes through `SteamUserStats.SetStat` only when `maxprogress != 0`. `EnableDisableAchievementTrigger(name, enabled)` flips an entry (a no-op before `initialized`; scripts use it to chain achievements).
- `Reward.SendReward` handles only `Furniture` and `Item` (sent through `MailController`, optionally to the parcel locker); the `Task` and `Money` branches are empty. `Achievement.Trigger` sends rewards before checking whether Steam accepted the unlock, and `SendRewards` does not check `rewardsSent`.
- State is saved globally, not per save (`OnSavingGameSpecial`/`OnLoadingGameSpecial`, `global: true`). `PauseMenu` calls `Achievements.ResetValues()`, which clears `_triggered` locally and does not touch Steam. Day milestones trigger `SURVIVE_7/30/365` from `TimeOfDayAzure.IncreaseDayCounter`.
- `globalDisable` is `[fsIgnore]`; the developer console sets it (opening the console disables achievements) and the hidden command `enable_achievements` turns them back on.
- `SteamManager.Awake` returns early in the editor, destroys duplicates, calls `SteamAPI.RestartAppIfNecessary` (quits when it says so) then `SteamAPI.Init()`; `Update` runs `SteamAPI.RunCallbacks()`. No game code reads `SteamManager.Initialized`. The Steam app id in code is 951240.
- Save version: the header stores `Application.version`; `LoadMenu` warns when it differs; a dialogue-name migration runs only for saves older than 0.3.40.
- Build switches: the decompiled code has no preprocessor blocks; editor-only paths are runtime `Application.isEditor` checks (about 15 sites, for example `NPCDirector`, `StorageCategoryDatabase`), false in a release build.
- Hidden console commands (in the command switch, not in the help list): `list_scenes`, `new_scene`, `show_credits`, `test_unity_console`, `reset_train_position`, `reset_all_trains_positions`, `break_trains`, `reset_crime`, `enable_achievements`.

## Boot order (as far as code shows) [code]

1. `Animal.ResetActiveAnimalCount` at SubsystemRegistration, before any scene.
2. A `[RuntimeInitializeOnLoadMethod]` guard creates a DontDestroyOnLoad runner that validates facial-animation components on each scene load and again 1 s later.
3. Scene objects: `SteamManager.Awake`, `DeveloperConsole.Awake/Start`, `Achievements.Awake`, `SaveController.Awake` (DontDestroyOnLoad singleton), `DisplaySave.Awake`, `ReadSceneNames.Start` (reads `LevelNames.bytes` from StreamingAssets).
4. `MainMenu.Awake/Start`, then `OptionsController.Start` (settings, see ui-options-input.md), then `TimeOfDayAzure.Awake/Start`; `Achievements` becomes `initialized` a few frames later (it waits for `TimeOfDayAzure.updateTimeDisabled` to clear).

## Mod guidance

Patch `Achievements.TriggerAchievement` or `Achievement.Trigger` to change behaviour; do not touch `Achievements.instance` before `initialized`. Achievement state is global and rewards are sent by mail.

## Unverified

How `SteamManager` enters the scene (probably a scene object); whether the console opens in a release build; whether `SteamUserStats` throws when Steam is not running.
