# Catalog of static events (hook points)
Scope: static and instance events on game classes other than `TimeOfDayAzure` (time-and-weather.md) and `CardGame` (minigames.md). Subscriber lists beyond the examples and firing order were not traced.
Status: code-only
Related: [time-and-weather.md](time-and-weather.md), [saves.md](saves.md), [tasks-relays.md](tasks-relays.md), [mail-invoices-phone.md](mail-invoices-phone.md), [ui-options-input.md](ui-options-input.md)

All are plain C# events invoked with `?.Invoke`; subscribe with `+=` and always remove in `OnDestroy`. Owners do not clear them on scene change, so a subscriber that is not destroyed cleanly can leave stale handlers. [code]

| Owner | Event | Signature | When it fires |
|---|---|---|---|
| `SaveController` | `PlayerWillChangeLevel` | void() | before a level change is saved |
| `SaveController` | `PlayerWillLoadGame` | void() | at the start of a game load |
| `SaveController` | `SavingStarted` / `SavingDone` | void() | around a save |
| `SaveController` | `LoadingStarted` / `LoadingDone` | void() | around a load |
| `MailController` | `MailSent` | (string mailName, NPCReference receiver, Mail) | a mail is sent |
| `MailController` | `MailReceived` | (string mailName) | a mail reaches the player |
| `MailController` | `PackageReceived` | (string packageName) | a parcel reaches the player |
| `MailController` | `PackageSent` | (string, NPCReference, Mail) | declared, no invoke site found |
| `TaskController` | `ObjectiveUpdated` | void() | an objective status changes (fires before `TaskUpdated`) |
| `TaskController` | `TaskUpdated` | void() | a task status changes |
| `ItemInfo` | `ItemDataChanged` | (ItemData) | an item info's data is set or changed |
| `Inventory` (instance event) | `ItemConsumed` | (Item, SlotController, int amount, int owner, object[] meta) | from `Inventory.OnItemConsumed` on every consumption |
| `PlayerAutoConsume` | `Consumed` | Action | after auto-consume of needs ran |
| `PlayerStats` | `PlayerSoiledPants` | Action | from `PlayerStats.OnSoiledPants` (hunger and thirst code) |
| `Crime` | `OnGoingToPrison` | void() | when the go-to-prison flow starts |
| `GameUIController` | `ExitPanels` | void() | in the force-close-all-panels routine |
| `HintOverlay` | `HintsToggled` | Action | hints toggled |
| `ThirdPersonCameraCollision` | `CameraModeChange` | (bool firstperson) | camera mode switch |
| `Notifications` | `EventFired` | void() (delegate named `TaskStatusUpdated`) | the visible-notification list changes; also in `Awake`/`OnDestroy` and after a load, so guard against a null `instance` |
| `WeatherManager` | `WeatherUpdated` / `WeatherTransitioning` | void() | a new weather is set / a transition starts |
| `WindowNaturalLight` | `SourcesChanged` | Action | window light sources change |

## Pitfalls

`Crime.OnGoingToPrison` and `GameUIController.ExitPanels` fire inside game logic, so a handler that changes the same state can race the game's own routine. `Notifications.EventFired` passes no data; read the list to see what changed.

## Unverified

Whether `MailController.PackageSent` is ever invoked; the invoke site of `CameraModeChange`.
