# Inventory and storage core

Scope: how player slots, container storages and their UI relate — `Inventory`, `SlotController`, `Slot`, `Storage`, `ItemOperations`; which methods see which slots; moving, adding, sorting. Not covered: item data (see [items-recipes.md](items-recipes.md)), the backpack itself ([backpack.md](backpack.md)), opening/theft rules ([containers-theft.md](containers-theft.md)), trade pricing.
Status: code-only unless a heading says `[verified]`.
Related: [backpack.md](backpack.md), [containers-theft.md](containers-theft.md), [crafting.md](crafting.md), [imgui.md](imgui.md).

## Key classes

- `SlotController` = data + callbacks (`itemStack`, `OnDropped`/`OnTaken`/`OnReturn`, `Siblings`, allowed/forbidden categories); `slot` = its UI (`Slot`, prefab `inventoryPanelUI.inventorySlot`).
  `CaptureSlot(Transform)` instantiates the UI under a parent if there is none, `RelaseSlot(destroy)` drops/destroys it.
- `Inventory.instance`: `Slots` (40 in the prefab [verified], code default 35; under `inventoryPanelUI.inventorySlotPanel`), `CharacterSlots` (8, under `characterClothesPanel`; index 5 is the backpack,
  6/7 hands), `ForeignSlots` (the open container's slots, or null). Player slots set `slot.playerSlot = true` and `Siblings = Slots` (main only).
- `Storage` (container): `Slots` (`SlotController[]`, `Siblings` = its own slots), `storageSlots` (saved `ItemStack`/`StorageSlot` data). `Storage.GetStorage` (private) sets categories
  and callbacks, then captures UI into `slotPanel` (default `Inventory.instance.storagePanelUI.storageSlotPanel`; `Storage.ChangeSlotPanel` exists).
- Slot colour: `Slot.SetColorToNormal` sets white (grey if `noDrop`), highlights are orange/green/red (`Slot.color*`). Called from `OnEnable` and selection changes.

## Storage API pitfall: `Storage.ExitExternally` on a storage that isn't open

`ExitExternally` closes `Storage.currentStorage` / `Storage.active` if they are another storage, sets `Inventory.ForeignSlots = null` and destroys this storage's slot objects
(`StorageLeave` → `RelaseSlot(destroy: true)`). Calling it on a backpack storage while a fridge is open closes the fridge. To show a storage's slots elsewhere, don't use
`EnterExternally`/`ExitExternally`; capture the slots directly. `Storage.Update` fires `onUpdate` only while `Storage.active == this` and a slot changed.
`ForeignSlots` is set by `Storage.EnterExternally` / `SetAsForeignController` (also `manufacturingProcess`), cleared by `ExitExternally`, `GrowingPanel`, `LiquidStorage`.

## `Slot.playerSlot` — direction and trade side

`Slot.playerSlot` is a UI flag (default false for container slots; true for `Slots` and `CharacterSlots`):
- `SlotController.MoveToOtherInventory(out, amount)`: `playerSlot` → `MoveToForeignInventory` (to `Inventory.ForeignSlots[0]`, fails if null), else `MoveToInventory`
  (`AddItemAndReturnRemaining(slot: null)` → `GetFirstSlot`; "full" flash is `inventoryPanelUI.InventoryFull()` / `storagePanelUI.StorageFull()`).
- Drag-drop: `Slot.IsOtherStorage` → `SlotController.DropItem(..., otherStorage)`; siblings overflow only when `otherStorage` (flag differs between source and target).
  With equal flags both sides are "same inventory" (plain swap/merge, no sibling overflow).
- Trade: `Trade.SameInventorySlot`, `Trade.MakeTrade` (`newSlot.playerSlot` → Buy else Sell), `Trade.TradeAndMove` (`playerSlot` → `SellAndMove`), `Trade.GetItemStackPrice`,
  `Trade.CanSell/CanBuy/IsStolen/CheckIfFull`, `SlotController.CantMoveToTradeSlot`, the drop check in `Slot`, `ItemInfoPanel` (Sell/Buy button), `AmountSliderUI` (Sell/Buy label),
  `ItemData.DropItem` (during trade only player-side items can be dropped). **A slot with the flag false is treated as the trader's: "Buy" and costs money.**
- Gamepad `InventoryNavigationHandler`: uses panels/`ForeignSlots` (`StorageOpen`); no direct `playerSlot` use found. Registration of an extra grid
  (`OtherPanelsOfTypeInventory`) is UI wiring — not analysed.

## `Inventory.InPlayerSlot(slot)`

True only for `Inventory.Slots` (not character slots, not backpack). Callers: `ItemOperations.AddItems` / `AddItemAndReturnRemaining` (money items ID 0/1 → `Money.AddMoney` instead of
storing; `ItemSpecialTriggers`; `CheckForAddedStolenItems`), `SlotController.DropItem` (same money branch + triggers), `ItemData` auto-move (money).
Patch point: postfix on `InPlayerSlot` to also accept other slots.

## Which methods see which slots

- `Inventory.AllSlots(ignoreBackpack = false, ignoreCharacterSlots = false)` = main + backpack storage slots + character slots. Nobody outside `Inventory.cs` reads `Inventory.instance.Slots` directly.
- **Include the backpack** (via `AllSlots()` default): `Inventory.RemoveItem(s)`, `FindItem`, `FindAndChangeItemAmmo`, `FindAndGetTotalAmmo`, `CheckForItem*`, `CheckForCategoryItem`, `GetItemAmountInInventory`,
  `GetEmptySlotsAmount`, `InventoryIsNotFull`, `UpdateAllSlotsNow`, `ItemOperations.AddLiquid`, `InteractableItemRemover`, `Logic_Thief` / `TriggerThief`, `MarketShop`, `NotesPanel`,
  `CheckForAddedStolenItems`, building system (`AllSlots(false, ignoreCharacterSlots: true)`). Lua dialogue helpers (`CheckForItemDialogue`, `RemoveItemDialogue`, ...) use the same methods, so quest hand-ins count it.
- **`ignoreBackpack: true`** is passed by `CollectibleItem` (room check; pickup settings, so pickup never lands in the backpack), `InteractableItemGiver` (room check), `ShowTime` (clock must be in main/char slots).
  The first two are capacity checks: ignoring the backpack there shows "inventory full" while the backpack has room. Patch: prefix forcing `ignoreBackpack = false` on `Inventory.InventoryIsNotFull` / `CheckForCategoryItem`, or on `AllSlots`.
- **Main-only** (do not see the backpack): `Inventory.GetInventoryItemData` / `GetInventoryItemDataByCategory` (users: `PlayerAutoConsume`, `Depression`), `IsInventoryEmpty` (users: building-system menu,
  `SleepEventController` robbery checks), `RemoveStack(id)` (main + char), `SortSlots`. `GetInventoryItemData` uses `slot.GetItemItemData()`, which exists only while the slot UI is captured: a storage never shown gives null entries.
- No "drop inventory on death" or autoloot code found (grep DropAll/Clear*).

## Adding items — overflow order

`ItemOperations.AddItemAndReturnRemaining` into a player slot (`InPlayerSlot`): target → its `Siblings` (main) → all `CharacterSlots` (filtered by allowed categories) → backpack slots unless `settings.ignoreBackpack`.
If the target is a container slot, overflow is only its siblings (no character slots, no triggers). `AddItems(slot: null)` → `GetFirstSlot()` = `Slots[0]`.

## Quick move

`ItemData.QuickMove`: bottle recycling / liquid storage / trade (`Trade.TradeAndMove`) first; otherwise `SlotController.MoveToOtherInventory`: player slot → `Inventory.ForeignSlots[0]`, other → player (overflow as above).
With `ForeignSlots == null` the move fails and an `ItemEquipable` (namespace `OS.Items`) item is equipped instead.
Callers of `SlotController.MoveToOtherInventory(out int, int)`: `ItemData.QuickMove` and `AmountSliderUI` (move with slider; move without slider for amount 1). Trade is handled before both (`Trade.TradeAndMove`).
Two add paths with different side effects `[code]`:
- `SlotController.MoveToOtherInventory(target, amount, out moved)` goes through `ItemOperations.AddItemAndReturnRemaining`. For a player-slot target (`InPlayerSlot`) that adds: money items (id 0/1) become `Money.AddMoney` (no slot used; stolen-money crime if `owner > 1` and witnessed), overflow into `CharacterSlots`, then the backpack (unless `settings.ignoreBackpack`), and finally `Inventory.ItemSpecialTriggers` + `CheckForAddedStolenItems`.
- `SlotController.AddItemAndReturnRemaining(item, amount, owner, meta, dropItems, ...)` fills that slot (honours `OnDroppedCheck` and `CheckIfAllowed`, fires `OnDropped`), then every `Siblings` entry (`AddItem(isSibling: true)`). No triggers, no money handling, no character slots, no backpack. Handy for moving within the player's own slots. Items carrying `ItemOwnerPermission` meta merge with neither liquid nor quality mixing.

## Sorting

`Inventory.SortSlots(SlotController[] slotToSort = null)` (null = `Slots`); called by `InventoryPanelUI.SortInventory` (button), `SortStorage` (`ForeignSlots`), `Trade.StartTrade`.
Algorithm: copy every non-empty `ItemStack` (plus a backup), `slot.RemoveItem()` (fires `OnTaken`!); sort with private `SortItems` (on exception falls back to the unsorted backup); then
`ItemOperations.AddItems(stack, slotToSort.First(), notify: false, ignoreCrime: true)` for each — adding into the first slot lets `SlotController.AddItem` merge stacks and spill into `Siblings` in order. Empty liquid metas are reset first.
It is generic: `Inventory.instance.SortSlots(someStorage.Slots)` sorts that storage alone (overflow stays inside it).
`SortItems` order: category priority (Water, Soft drink, Food, Meat, Medical, Wieldable Primary/Secondary, Utility, Crafting Material, Hardware, Alcohol, Mash, Fermenter, Liquid, Smokes, Chemical, Seeds,
Valuable, Bottle, Liquid Container; via `CompareCategories`), then name (liquid inside a container prefixes the name), quality desc, integrity desc, sale price desc, amount desc.

## Swapping `InputOutputStorage.storageSlots` (shared trick, pitfall) [code; the slowdown was verified in game]

- `storageSlots` is a saved field (not `[fsIgnore]`). `CountItems` / `CompareRecipeToItems(items: null)` read it directly, so mods widen it for the call and restore it after (for example adding the player's stacks as copies, or the real `StorageSlot` objects of nearby containers).
- Harmony runs finalizers in the same priority order as prefixes, not in reverse. Two mods doing prefix-widen + finalizer-restore at equal priority don't nest: the first prefix's finalizer runs first.
  If that finalizer sees a changed array it may keep it, and then the other mod restores to the first mod's combined array: the containers' slots stay in the station for good. Every later call adds them again, so the array grows with every recipe check -> hangs, RAM growth, and the
  bloated array goes into the save (item duplication after load, stays after removing the mod).
- Rule: an outer swap must be outermost on both ends (prefix `Priority.First`, finalizer `Priority.Last`), or not swap at all.
- A mod that caches its combined array by the reference of the array it finds (`combinedCacheOwn == own`) rescans every container per recipe check if an outer swap builds a new array per call. Reuse the same widened array.
- Such container-sourcing only covers the open station (`currentCraftingBase`); planters other than the open one are planted through their own `StartGrow`, so with seeds only in containers just the open planter gets planted.

## Storage panel buttons [verified]

- `InventoryNavigationMovement.storageSortButton` (private, `.Instance` may still be null while stations `Start`; find it with `FindObjectsByType`): "Sort button" under "Storage Info", 40 x 40, `Image` sprite `button_round_001_red`,
  `Button`, `DefaultUIButton`; child "ButtonText" (TMP). `StoragePanelUI.takeAllButton` is the wide rectangular one; its RectTransform is stretched, so a copy with fixed anchors needs an explicit `sizeDelta`.

## UI pitfalls when moving or reparenting inventory slot grids

- [verified] Destroying or disabling ANY item icon (`ItemData`, e.g. via `Slot.SetSlotController` / `SlotController.CaptureSlot` on a slot that is already set up, which destroys and re-instantiates `ItemInSlot`) closes the item info panel:
  `ItemData.OnDisable` → `ItemInfo.instance.Deactivate()`, whether or not that item was the selected one. Re-capturing slots during a Use click makes Use/Eat just close the panel.
- [verified in effect, cause from code] Loading a save rebuilds the character slot UI (`Inventory.StartInit` captures new `Slot` objects), so UI a mod parents to a character slot (e.g. `CharacterSlots[5].slot`) is gone after load; re-check the parent on every use instead of creating it once.
- [code, trigger inferred] `Inventory.ForceCharacterItemsUpdate` re-invokes `OnDropped` of the character slot controllers (`BackpackStorage.OnDropped`, `HeadSlotController`, `WatchSlotController`, ...) for already-equipped items; callers include `CharacterItemManager`, `Crime`, `SleepEventController`, `BlackoutTrigger`, `CharacterSlotController`.
  So `OnDropped` on a character slot does not mean "just equipped": treat it as "state may have changed".

- Mouse click closes the item info before the Use/Eat button fires `[verified]` (2026-10-20; the cause and fix below were confirmed in game): `InventoryNavigationHandler.HandleNonMovementInput` calls the private `DeselectSlotIfNecessary` on every mouse-button **down** (buttons fire on up). It runs `EventSystem.RaycastAll` at the pointer, takes the first hit whose own GameObject has a `Slot` component (`GetComponent`, not parents; panels, buttons and other graphics under the pointer are in the list too), and if that `Slot` is active and `GetSelectedStatus()` is false it calls `UndoSelection`: `_data.CurrentSlot.SetSelectedStatus(false)`, `ItemInfo.instance.Deactivate()`, coordinates reset. `Slot.selected` is only set true by `Slot.OnPointerClick` (and by the handler for the controller default selection). So any not-selected slot that lies **behind** the item info panel (Use/Eat button included) deselects and closes the panel on mouse down. Vanilla layout keeps the panel off other slots; a mod that moves slot grids or the panel so they overlap breaks Use/Eat. Slots hidden by a `RectMask2D` are **still hit** by `RaycastAll` `[verified]`: in the mod, masked (scrolled-out) slots lying under the info panel caused the deselect. Fix that worked: prefixes on `DeselectSlotIfNecessary` and `GetSlotUnderMouse` that skip slots outside the `RectMask2D` viewport. Other options (not tried): prefix on `DeselectSlotIfNecessary` returning false when the pointer is over `ItemInfo.instance`'s rect, or keep the panel off the slot grid. Not examined: where `ItemInfo` places itself.

## Changes in 0.5.13 `[code]`

- `Slot.ShowTradeUI` now calls `CanSell(..., showNoMoney: false, checkMoney: false)` and then `CheckIfTraderHasMoney` to tint the trade label and edges (new serialized `canBuyColor`/`noMoneyColor`).
- `Storage` load resizes the saved `storageSlots` to the prefab's slot count (`ResizeLoadedSlots`): trailing empty slots are removed first, a non-empty surplus stays, so the array can stay longer than the prefab. `AddItemAndReturnRemaining` writes only to `Slots[0]` and respects allowed and forbidden categories.
- A crafting station's recipe code reads `CraftingBase.InputSlots` (device storage plus extra slots, which include the player's slots while a growing bed's panel is open) instead of only `InputOutputStorage.storageSlots`; see crafting.md.
