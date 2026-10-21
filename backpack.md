# Backpack

Scope: the worn backpack — items, where its contents live, how it opens, screen pieces, and what differs from main inventory slots. General slot/storage mechanics are in [inventory-storage.md](inventory-storage.md).
Status: code-only unless marked.
Related: [inventory-storage.md](inventory-storage.md), [containers-theft.md](containers-theft.md), [crafting.md](crafting.md).

## Items

| ID | Title | Categories |
|---|---|---|
| 14930 | Empty Backpack | Backpack, Backpack Empty |
| 15480 | Backpack | Backpack, Storage |
| 15300 | Empty Rucksack | Backpack, Backpack Empty |
| 15490 | Rucksack | Backpack, Storage |
| 15340 | Empty Cooler Bag | Backpack, Backpack Empty |
| 15500 | Cooler Bag | Backpack, Storage |

- Full and empty are different items. `"Storage"` is in `SlotController.DefaultForbiddenCategories`, so a non-empty
  backpack can only sit in the backpack character slot (it is removed there: `CharacterSlots[5].DefaultForbiddenCategories.Remove("Storage")`)
  or lie in the world. An empty one fits in any slot.
- `BackpackStorage.UpdateEmptyState()` swaps the item in `CharacterSlots[5]` between the full/empty variant by
  `Storage.IsStorageEmpty()`. Called from `CollectibleItem`, `SlotController.DropItem` (swap into slot 5),
  `ItemOperations.MoveStackToStorage`, `Inventory.EquipBackpack`, `BackpackStorage.Awake`. Probably also from the storage's
  `onUpdate` relay (asset data, not visible).

## Where the contents live

- `BackpackStorage` (singleton, `BackpackStorage.instance`) holds three scene `Storage`s on the player: `backpackStorage`,
  `rucksackStorage`, `coolerBagStorage`. Slot counts and refrigeration are prefab data (not in `StoragePrefabs.json`); see "Runtime data" below.
- `BackpackStorage.GetStorage()` → the storage of the item in `Inventory.CharacterSlots[5]` (index const `Inventory.backpack`), or null.
- `Equipped` / `currentBackpack` are set by `OnDropped` / `OnTaken` / `OnReturn`, hooked as callbacks of `CharacterSlots[5]` in `Inventory.StartInit`.
- Dropping the backpack into the world: `ItemOperations` (spawn of an item with a `Storage` in children) moves the equipped
  storage's contents into the world object's `Storage` (`MoveToOtherStorage`). Picking it up: `CollectibleItem.UpdateBackpack` moves them back.

## How it opens (vanilla)

- Backpack button (`BackpackButton.ToggleBackpack`) and key `3` (`GameController`) → `BackpackStorage.ToggleBackpack` →
  `OpenBackpack`: **closes** the open crafting station / trade / bottle recycling / vending machine / liquid storage
  (remembers it), then `storage.EnterExternally(forceOpen: true)` — the backpack takes the storage panel like any container.
  `CloseBackpack` reopens what was closed.
- Equipping the backpack while the inventory is visible opens it (`OnDropped`); taking it off closes it (`OnTaken`).
- Menu hide (`GameUIController.HideGameMenu`) → `ResetBackpack` → `ExitStorages` → `ExitExternally` on all three storages.
- Pitfall: `ExitExternally` on a storage that is not open closes whatever else is open — see [inventory-storage.md](inventory-storage.md#storage-api-pitfall-storageexitexternally-on-a-storage-that-isnt-open).

## Backpack slots vs main slots (checklist for "backpack as an inventory extension")

Backpack slots are plain `Storage.Slots` (`slot.playerSlot == false`, `Siblings` = the storage's own slots). Details and patch points for each item are in [inventory-storage.md](inventory-storage.md):
- set `slot.playerSlot = true` on the backpack slot UI (otherwise trade shows "Buy", shift-click goes to the main inventory, drag-drop treats it as another storage);
- `Inventory.InPlayerSlot` is false for backpack slots (money items would be stored as items; no achievement/steal triggers on direct add);
- `ignoreBackpack: true` callers: `CollectibleItem` (pickup never lands in the backpack), `InteractableItemGiver`, `ShowTime`;
- main-only methods: `GetInventoryItemData*` (auto-consume, `Depression`), `IsInventoryEmpty` (robbery checks, building menu), `RemoveStack`, `SortSlots`;
- already counted via `AllSlots()`: removal/lookup/ammo/theft/building/quest hand-ins.
Crafting stations read only their own input storage (see [crafting.md](crafting.md)); the player drags ingredients in from the inventory.

## Inventory screen pieces [code]

- "OPEN" arrow tab next to the backpack slot: `BackpackButton.buttonEnabled`, shown by `ChangeButtonStatus(true)` on equip.
- Hotkey badge "3" on the backpack slot: `Slot.SetHotKeyNumber` sets `isHotKeySlot`; `Slot.OnEnable` re-shows the badge for keyboard input.
- The empty striped block below the inventory: `GameUIController._InventoryNotSelectedPanel`, active only in
  `OpenPanel(MenuPanels.None)` (plain inventory). `Hide_Panels` hides it for every other panel.

## Open question

Sorting only the backpack via `Inventory.SortSlots(GetStorage().Slots)` calls `RemoveItem` (fires `OnTaken`) on each slot; `UpdateEmptyState` could flip the item in `CharacterSlots[5]` in between — verify in game.

## Runtime data [verified]

Seen in game 0.5.08 (runtime dump, 2026-10-18):
- Slots: Backpack Storage 24, Rucksack Storage 40, Cooler Bag Storage 24 (refrigeration factor 0.5).
- All three: `StorageSettings` 'Custom', `AllowedCategories` [], `ForbiddenCategories` [Backpack] (vanilla already forbids a backpack inside a backpack), `disableSpoil` false, `maxSlotStackSize` -1.
- Inventory slot grid ("Inventory Slot Panel"): cell 69x69, spacing 0, padding l10 r10 t15 b15, UpperLeft, 5 fixed columns, 40 slots (`Inventory.slotAmount` is 40 in the prefab, not the code default 35).
  Its sibling "Not available slots" (placeholder grid) is drawn under it; if the sibling order flips, the placeholders hide the item icons (slots stay clickable).
