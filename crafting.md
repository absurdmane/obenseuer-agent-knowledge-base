# Crafting devices (CraftingBase)

Scope: `CraftingBase` and its subclasses: recipe types, conditions, quality, upgrades, ingredient matching. Not covered: power ([power.md](power.md)), growing ([growing.md](growing.md)), data files ([items-recipes.md](items-recipes.md)).
Status: code-only
Related: [power.md](power.md), [growing.md](growing.md), [water.md](water.md), [inventory-storage.md](inventory-storage.md)

`CraftingBase` — base of every workstation and producer (`manufacturing`, `manufacturingProcess`, `Generator`, `Growing`; most likely water taps too).

## Key members

- `Recipes : RecipeType[]` (+ `RecipesWithUpgrades`) — which recipe types the device runs (`WaterSource`, `Machining`, `Distillery`, ...).
- `Modifiers : List<RecipeCondition>` (+ getter **`ModifiersWithUpgrades`** = own + `CraftingUpgrade.AddModifiers`) — the device's conditions
  (Clean, Dirty, Industrial, Steam, ...). Recipe availability: `RecipeInfoUI.CheckModifiers(recipe, ModifiersWithUpgrades)`;
  panels show them via `ModifiersUI.UpdateModifiers(...)`. `RecipeDatabase.FetchRecipesByType(type, conds, ...)` implements required/excluding matching.
- `EquipmentQuality` (+ getter **`EquipmentQualityWithUpgrades`** = own + `CraftingUpgrade.AddEquipmentQuality`) — feeds
  fail chance (`Recipe.CurrentFailChance`), output quality (`PlayerStats.SkillQuality`), process time (`manufacturing`: `recipe.GetProcessTime(..., quality)`),
  grow time (`Growing`), fuel burn time (`_Fuel.BurnTimeFactor`).
- `_Power : PowerSource`, `AlwaysOn`, `On`, `PoweredAndActive` — see [power.md](power.md).
- `CraftingType {Growing, ProcessManufacturing, Manufacturing}`.
- `OutputStorage : Storage`, `OutputTank : LiquidStorage`.

## CraftingUpgrade

Attachable add-on connected to a nearby `CraftingBase` (`MatchesRequirements`, `MaxDistance`): `RequiredType`, `RequiredRecipes`,
`RequiredPower : PowerSourceType?`, adds `AdditionalRecipes`, `AddEquipmentQuality`, `AddModifiers`.

## Patch points

- Postfix `CraftingBase.get_ModifiersWithUpgrades` — change a device's conditions at runtime without touching the saved `Modifiers`
  (affects both UI and which recipes can run).
- Postfix `CraftingBase.get_EquipmentQualityWithUpgrades` — scale speed/quality/fail chance.

## Ingredients and storage [code]

- A station reads only its own `InputOutputStorage.storageSlots` (`StorageSlot[]`): `CountItems` (private) → `GetRecipeMultiplier`
  (max batch); `CompareRecipeToItems(recipe, items = null)` → whether a recipe lists (all items present, or for a known recipe any
  of them); `RemoveItems` → consumption, quality from the slots, liquids drained from containers (the empty container stays in the slot).
- `RemoveItems(..., dontRemoveItems: true)` is a dry run, but its `usedItemReferences` list is never filled.
- Matching in `RemoveItems`: the input item itself; a member of an `ItemGroupData` group (for liquid groups, the container's
  `ItemLiquidData.liquidItem`); with `metaSettings.useLiquidContainers`, a container holding the liquid.
- Consumption happens at the start: `Growing.StartGrow(n)`, `manufacturingProcess.StartProcess()` (batch = `ActiveRecipe.Value`),
  `manufacturing.Craft()`.
- Moving items: player slots `Inventory.instance.Slots` (`SlotController[]`, `.itemStack`); a station's `Storage.Slots`.
  `SlotController.MoveToOtherInventory(targetSlot, amount, out moved)` moves into the target slot's storage;
  `MoveToInventory(out moved, amount)` moves to the player. `Storage.TakeAll()` empties a storage into the player's inventory.

## Swapping `storageSlots` and storage panel buttons

Moved to [inventory-storage.md](inventory-storage.md) (the `InputOutputStorage.storageSlots` widen/restore trick and its Harmony finalizer pitfall; the storage panel Sort/Take-all buttons).
