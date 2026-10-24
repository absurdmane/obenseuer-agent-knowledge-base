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

- Since 0.5.13 a station reads `InputSlots` (its own `InputOutputStorage.storageSlots` plus `ExtraInputSlots`, which for a growing bed include the player's slots while its panel is open; earlier: only `storageSlots`) (`StorageSlot[]`): `CountItems` (private) → `GetRecipeMultiplier`
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

## Recipes and the production flow `[code+data]`

- `RecipeDatabase` loads `Recipes.json` (UTF-16, wrapper `{"Recipes": [...]}`, `Type` is a `RecipeType` enum name). `RecipeType` values: Basic, Oven, Stove, Mashing, Fermenter, Distillery, WaterSource, Grinder, Machining, ChemicalLab, Woodworking, Microwave, CoffeeMaker, PaperWork, Grow*, Misc, SludgeTank, Engineering, Electronics, Inventory, Furnace, Printer3D, Assembly, Compost, Kitchen, Malting, Sewing, Crusher, Loom, Anvil, ChemicalRefinery, Press, Bottler (added in 0.5.13: 30 recipes, no C# consumer, handled by the generic data path). Rough recipe counts `[data]`: Stove ~108, Oven ~73, ChemicalLab 32, Kitchen 12.
- Recipe fields: `input`/`output`/`outputFail` (`ItemReference` to amount), `ProcessTime`, `FailChance` (code default 0.2), `RelevantSkill`, `MinimumSkill`, `HardMinimumSkill`, `SkillFactor`, `SkillGainFactor`, `RequiredConditions`, `ExcludingConditions`, `RecipeTags`, `metaSettings`, `variantItemCategory/Amount`, `keepItemOwnerOnFail`, `disableJunkSpawn`, `Appearance`, `RecipeRequired`, `Meta`. The runtime dictionaries `Recipe.Input/Output/OutputFail` are `fsIgnore` and stay null until `Recipe.SetItems()` runs (`manufacturing.LoadDelayProcess` does it).
- `manufacturing` (batch, timer-based): `Craft()` returns unless `PoweredAndActive` and `CraftingInProgress < AllowSimultaneous`; `CraftingBase.RemoveItems` consumes inputs at once and returns returned containers plus `CraftingInputData` (average ingredient quality, `processTimeFactor`); a positive `GetProcessTime` enqueues a `CraftInProgress` on a `TimeOfDayAzure.Timer`, otherwise it finishes instantly; `CraftDelayed` -> `OnCraftDone` -> `CreateItems`. `Craft()` also calls `RecipeController.AddRecipe(recipe, learned: true)`. `OnCraftDone` runs under try/catch, so exceptions there are swallowed.
- `manufacturingProcess` (continuous): `ActiveRecipe` is recipe plus batch multiplier, passed to `RemoveItems` in `StartProcess`; a `Stage` enum drives it; output quality is stored at start.
- Process time: `Recipe.GetProcessTime(m, q)` = `ProcessTime * m / EquipmentQualityWithUpgrades`, multiplied by `(MinimumSkill - skill + 1)` when the player's skill is below `MinimumSkill`, then `DifficultyController...GetItemCraftingTime`. `manufacturing` with `RequirePlayer` also divides by `1 + MushroomsHighOrangeNormal * 0.033` and multiplies by the ingredient group's `craftTimeFactor`.
- Result: `Recipe.GetCraftingResult` rolls `PlayerStats.SkillSuccess(RelevantSkill, CurrentFailChance(FailChance * group.failFactor, ...))`; success gives `Output`, failure `OutputFail`. `CraftingBase.CreateItems` (protected virtual) stores into `OutputStorage ?? InputOutputStorage`, liquids through `RecipeLiquidItems`, items with `ApplyQuality` meta (quality = average input quality times a factor); returned containers go back to the same storage. Skill gain: `PlayerStats.IncreaseSkill(RelevantSkill, ...)`.
- `RecipeController` maps `RecipeType` to `TaskItem.Type` for task tracking only.
- Not found in code: classes named `ChemicalLab`, `Stove`, `Oven` or `Kitchen`; those stations are presumably `manufacturing` prefabs with a recipe list in prefab data (unverified).

Mod guidance: postfix `Recipe.GetProcessTime` or `CraftingBase.EquipmentQualityWithUpgrades` for speed; `Recipe.CurrentFailChance` is the global fail hook; a postfix on `CraftingBase.CreateItems` sees `inputData` and `hasBeenDoneInSeconds`. `RemoveItems` writes skin meta into the shared `Recipe.Meta`, so that state persists across crafts.

## Changes in 0.5.13 `[code]`

- `CraftingBase.InputSlots` is cached per frame (cleared by `ClearInputSlotsCache()`, on panel close and by `UpdateAllSlots`); `CountItems`, `CompareRecipeToItems(items: null)`, `CraftingBaseHelperFunctions` and `RemoveItems` read it; `CraftingBase.InputHasItem(...)` is new and adds extra-slot amounts for the recipe UI.
- Broken state: `CraftingBase.broken`, `Break()`; `PoweredAndActive` is false while broken and `Toggle(true)` is refused (see [power-grid.md](power-grid.md)). `CraftingBase.OnDestroy` and the finalizer call `_Power.OnDestroy()` (which also runs `_Fuel.OnDestroy()` and leaves the grid).
- Recipe UI: `RecipeIngredientUI.CreateRecipeIngredientUI` has a new last parameter `object[] meta` and shows skin titles and sprites (`Item.GetSkinTitle`); `RecipeInfoUI.GetDisplayMeta(recipe, item)` passes `recipe.Meta` only for outputs.
