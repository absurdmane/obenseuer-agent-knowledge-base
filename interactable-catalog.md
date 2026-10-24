# Catalog of Interactable subclasses
Scope: one line per `Interactable` subclass: what its `Interact` does and its key fields. Base classes, `Lock` and the picker are in doors-interaction.md. Bodies marked "not read" were not checked; asset values and data files were not searched.
Status: code-only
Related: [doors-interaction.md](doors-interaction.md), [containers-theft.md](containers-theft.md), [crafting.md](crafting.md), [economy.md](economy.md), [map-travel-trains.md](map-travel-trains.md)

`Interactable.Interact` itself is an empty virtual, and many overrides skip `base.Interact()` on failure branches, so patch the concrete class. `Door` has no subclasses; `DoorLockBypass` and `FakeDoor` are separate Interactables. `DoorLockBypass` calls `Door.UseDoor`, so patching `UseDoor` covers it. Classes that share the `Lock` class (`_Lock`): `Door`, `DoorChangelevel`, `BreakableLock`, `Button`, `MarketShop`.

## Doors, locks, level changes [code]

- `Door`: `UseDoor(BypassLock)`. `DoorLockBypass`: `door.UseDoor(bypassLock: true)`. `DoorChangelevel` (base `Changelevel`): own `UseDoor`; the use message checks `_Lock.PlayerCanOpenOrUnlock()`. `FakeDoor`: plays a sound, animator trigger "Locked", warning; never opens. `Changelevel`: `ChangeLevel()`; `TriggerChangelevel` inherits it.
- `Button`: warns if locked or items are missing, otherwise continues (rest not read). `BreakableLock`: `_Lock.Use(Unlocked, gameObject)`. `Keypad`: `Use()` (types Keypad and RotaryLock, `noPower`). `MarketShop`: if `marketLock.Locked`, warning and `marketLock.Use`.

## Seating, climbing, movement [code]

- `InteractableChair`: if the player sits elsewhere it forwards to `InteractObjects.savedInteractable`; else `SitToggle()`. `Toilet` (`Use()`), `ToiletBucket` (empty subclass). `InteractableLadder`: same forwarding pattern via `playerClimbingGlobal`, else `ClimbToggle()`. `Sleep`: `Use()`. `InteractableFastTravel`: `FastTravelController.ToggleMenu()`. `TrackTrainInteractableCoupler`: `ReleaseCoupler()` if coupled. `VendingMachine`: `EnterExternally()`.

## Shops, trade, money [code]

- `BaseShop`: `OpenMenu()` (virtual; `FurnitureShop`, `TrainShop` not read). `Trade`: `StartTrade()` if `isInteractable` (`TradeSpecial` not read). `InteractableMoney`: `Money.Buy(price, RM)` then `onPurchase`. `InteractablePay`: `Money.CheckMoney(price)`, `RemoveMoney`, `onPaid`, else warning. `InteractableCashRegister`: base only. `InteractableFurnitureDisplay`: `AddAsItem()` or `Purchase()` while `GetStock()` allows. `InteractableItemGiver` (warns on no stock, `!canGive`, full inventory, missing items; else gives), `InteractableItemRemover` (removes matching slots, `onInteractFailed`), `InteractableTicketMachine` (`TakeTicket()`). `PutItemSelection`: `Use()` (`InvoicePaymentMachine` not read). `ParcelLocker`: spawns parcels from `MailController.FetchPackages(dontEmpty: true)`.

## Storage, liquids, fuel [code]

- `Storage`, `LiquidStorage`: `UseStorage()` if `isInteractable`. `InteractableShower`: toggles `showerOn`. `MinitrainFuelTank`: adds an accepted fuel unless full. `FuelPump`: `StartFilling()` if `currentTank` is set, else warning when `locked`. `BottleRecycling`: `Use()`.

## Crafting, growing, upgrades [code]

- `BaseCrafting`: empty body. `FurnitureCrafting`: not read. `CraftingBase`: `Use()` (no `base.Interact()`; `Generator`, `Growing`, `manufacturing`, `manufacturingProcess` inherit it). `CraftingUpgrade`: `Toggle(!Active)` unless `Passive`, and it always shows the warning, even after a successful toggle. `grow`: `Uproot()` then `HoldToUseController.StartUpdate(Harvest)`. `GrowItem`: harvests one if enabled and there is room.

## Media, phone, waiting, dialogue [code]

- `InteractableComputer`: `UseToggle()`. `Radio`: `Use()`. `Phone`: warns if owned by an NPC and not ringing, else uses. `Mailbox`: receive or insert mail. `EditableSign`: `StartEditing()`. `InteractableTalk`: `StartDialogue()`. `InteractableWait`: `Wait(WaitInMinutes)` unless `CanWait()` fails (`onCantWait`); `InteractableWaitWork` adds an `interactDisabled` warning; `InteractableWaitResident` has no own Interact. `SelectionButton`: `Exit()` when a menu is visible. `Lantern`: extinguish a burning candle, or insert one from the inventory, else warning. `RatTrap`: picks up rat items. `SlotMachineGameplay` (Standby starts a round), `PajatsoGameplay`, `CardGameTable` (see minigames.md). `InteractableDynamiteCone`: place or remove dynamite (takes a fixed item id). `TelevisionNoise`: `locked` warning, then power branch (rest not read).

## Collectibles, no-op [code]

- `Collectible`: fires `onCollect`/`onCollected` then `Collect()`; `CollectibleItem`, `CollectibleCoin`, `CollectibleNote`, `CollectibleRecipeBook`, `CollectiblePackage`, `CollectibleRecipes`, `CollectibleRecipe`, `CollectibleTaskItem`, `FurnitureCollectable`, `WastelandGlow` inherit it (overrides not checked).
- No own `Interact`: `Meter`, `InteractableCalendar`, `InteractableDamageReceiver`, `TriggerChangelevel`.

## Unverified

Whether the collectible subclasses, `FurnitureShop`, `TrainShop`, `TradeSpecial`, `InvoicePaymentMachine` and `InteractableWaitResident` override `Interact` or its helpers; the remaining bodies of `InteractableRelay`, `TelevisionNoise`, `Button`, the slot machine and the coin pusher.
