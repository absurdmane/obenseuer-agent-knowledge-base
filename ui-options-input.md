# UI, settings files, input, notifications, localization
Scope: HUD and panel switchboard, settings persistence, key bindings and input type, notifications, menu localization. Not covered: gamepad navigation helpers, `ItemInfoPanel` internals, difficulty UI (main-menu.md and inventory-storage.md cover those menus).
Status: code-only
Related: [main-menu.md](main-menu.md), [inventory-storage.md](inventory-storage.md), [saves.md](saves.md), [player-stats.md](player-stats.md)

## Where it lives [code]

- HUD: `PlayerStatsUI` with one `PlayerStatUI` per bar (Health, Bleeding, Hunger, Thirst, Tiredness, Bowel, AlcoholNeed, MushroomNeed, SmokesNeed, Depression, Duck, Hygiene, Gambling), plus an SMV canvas and a damage overlay. `GameUIController` (`instance`, `currentPanel: MenuPanels`) is the panel switchboard: `ToggleGameMenu`/`ShowGameMenu`/`HideGameMenu`/`OpenPanel`, `CloseAllMenus(force)`, `IsAnyMenuOpen()`, static event `ExitPanels`. `PauseMenu` (one Animator per page).
- Options: `OptionsMenu` (shared with the title screen via a `mainMenu` flag), `OptionsController` (boot loader), `KeyBindingsMenu`. Persistence classes: `QualitySave`, `DisplaySave`, `ControlsSave`, `VolumeSave`, `KeyBindingsSave`.
- Input: `InputManager` (singleton; `keyBindings` ScriptableObject; `inputType` Keyboard or Controller; `InputTypeChanged` event). Notifications: `Notifications` (SavableScript), `Notification`, `NotificationBox`. Tooltips: `TooltipUI`, `ItemInfoPanel`. Hints: `HintOverlay`.

## How it works [code]

- Settings files in `Application.persistentDataPath`: `Quality.json`, `Display.json`, `Controls.json`, `KeyBindings.json` (LitJSON) and `Volume.json` (FullSerializer, UTF-16 via `Encoding.Unicode`, so not plain UTF-8). `Display.json` also stores the language (key `"Language"`, set in `OptionsMenu.applyDisplaySettingsMono` together with resolution, refresh rate, screen mode and VSync); display settings apply only when the player applies them. `OptionsController.Start` -> `GetAndSetOptions` loads quality, controls, display and volume, each in its own try/catch. `HintOverlay` stores its on/off flag in `PlayerPrefs` key `HintOverlayEnabled` (default on), outside the save folder.
- Key bindings: `InputManager.SetDefaultKeys` then `KeyBindingsSave.GetKeyBindings` overlays `KeyBindings.json`. Actions are string names ("Forward", "Primary Use", "Hide Hud", "Pause Menu", "Drag And Drop", ...). `GetKey`/`GetKeyDown`/`GetKeyUp`/`GetKeyWithHold` take the action name, check the keyboard key first, then the controller key. Rebinding: `KeyBindingsMenu.ChangeKey` arms a row (blocked in Controller mode); `KeyBindingsMenu.OnGUI` takes the next key event (Escape cancels) and `KeyBindingsSave.SaveKeyBindings(action)` writes the file at once. `InputManager.ChangeInputType` switches Keyboard/Controller, updates the cursor and fires `InputTypeChanged` (button icons subscribe).
- Notifications: `Notifications.CreateNotification(type, text, silent)` queues an entry; a coroutine shows it while fewer than 3 boxes are visible; a box stays about 8 s real time. Visible and queued notifications are saved with the global save. About 50 call sites (Inventory, Money, Mail, Crime, PlayerStats, Map, Notes, ...); `type` and `text` are free strings, no localization key. Static event `Notifications.EventFired`.
- Menu localization: `MenuLocalization.Localize(Locale)` walks TMP_Text children and calls `LocalizationSettings.StringDatabase.GetLocalizedString("Menu", key)`, with the English text as the key; results are cached in `SavedLocalizationDictionaryData/PersistentDictionary-Menu.json` (the cache is keyed by English text, so a changed string keeps its old translation until the file is removed). Tables seen in code: "Menu", "Items", a dev "Test Table".
- `TooltipUI.UpdateTooltip(title, details, inputItems?, resourceItems?)` shows an item grid for recipes and resources.

## Mod guidance

- To add a setting, follow the `QualitySave`/`DisplaySave` pattern: its own JSON file, loaded in `OptionsController.GetAndSetOptions` inside a try/catch.
- Patch `Notifications.CreateNotification` to change, silence or skip a notification (`silent` controls the sound only). Notifications persist across save and load.
- `InputManager.Start` sets defaults and then reads `KeyBindings.json`, and `KeyBindingsMenu.Start` reads it again, so keys set earlier are overwritten. `KeyBindingsMenu.OnGUI` captures any key while a row is armed, so a custom key listener also fires during rebinding. Letters, digits, punctuation and keypad keys are excluded from the `OnGUI` tracking (`IsLayoutSensitiveKey`) and work only through `Input.GetKey*`.
- A mod that changes the locale elsewhere must also write `Display.json` `"Language"`, or the next display apply reverts it.

## Unverified

Whether `Controls.json` and `Quality.json` are written on every change or only on apply; `PlayerStatsUI` colour rules.
