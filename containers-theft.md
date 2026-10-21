# Containers and theft

Scope: Opening a container (`Storage.EnterExternally`), taking items, what counts as theft, tenement scene name, item slot icons.
Status: code-only; the gold "marked" frame is [verified]
Related: [inventory-storage.md](inventory-storage.md), [backpack.md](backpack.md), [levels.md](levels.md)

- Opening a container: `Storage.Interact` → `UseStorage` → (lock check) `EnterExternally(forceOpen)`. It sets `Storage.active`,
  `Inventory.instance.ForeignSlots = Slots`, shows the menu (`MenuPanels.Storage`) and spawns/respawns contents (`GetStorage`).
  Stations' storages go through the same `Storage` class (exclude by `GetComponent<CraftingBase>()`).
- Taking: `SlotController.MoveToInventory(out moved, amount)` → `MoveToOtherInventory(null, ...)` →
  `ItemOperations.AddItemAndReturnRemaining`; what doesn't fit stays in the slot. `Storage.TakeAll` does this for every slot.
- Theft is decided per stack, not per container (`Inventory.CheckForAddedStolenItems`): `ownerId >= 2` and no
  `ItemOwnerPermission.ownerPermission` in the meta. Then `ItemStolenData.playerStolen` is added, and a crime is committed if
  `Witness.IsWitnessed()`. `CraftingBase.PermissionToTake` / `grow.PermissionToTake` give the permission meta to produce.
- The tenement interior is its own scene: `"Interior Player Tenement"` (`SceneManager.GetActiveScene().name`).
- Middle mouse has no game action on inventory slots; `InventoryNavigationHandler` only selects the slot under the mouse on it.
  Slot under the mouse: `EventSystem.RaycastAll` → first `Slot`; its stack is `slot.slotcontrol.itemStack`.
- Item slot icons [code]: `ItemData.UpdateItem` sets `stolenImage` (owner > 0), `ownedImage`, `quality`, `marked` (a gold frame [verified]; item needed by a marked tenement upgrade, `TenementController.CheckIfItemBelongsToMarkedUpgrade`). UI sprites with stars in `resources.assets`: `item_quality_stars`, `button_round_001_upgrade_star`.
