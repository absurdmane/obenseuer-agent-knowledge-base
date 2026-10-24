# Growing: planting, harvest, spoilage

Scope: `Growing` beds/terrariums: planting flow, harvest, on-vine spoilage, bush lifetime, sun condition, produce quality and freshness.
Status: code-only; re-checked against build 0.5.13
Related: [crafting.md](crafting.md), [inventory-storage.md](inventory-storage.md), [items-recipes.md](items-recipes.md)

## Classes

- `Growing : CraftingBase` — a bed / terrarium / mushroom box. `GrowingSlots : List<GrowingSpot>` (one plant per spot),
  `ManualHarvest` — if true, `Growing.Harvest()` / `Harvest(GameObject)` (UI harvest) do nothing and plants are picked in the world.
  `PauseGrowth/UnpauseGrowth/StartRot/StopRot`, `OnPowerOut/OnPowerOn`.
- `GrowingSpot : SavableScript` — slot; `.Plant` (GameObject), `HarvestPlant()`, `AddPlant(...)`.
- `grow : Interactable` — the plant itself (hitbox at the root). Saved: `Age`, `Paused`, `Rotting`, `RottingOverride`, `FullGrownAge`,
  `ownerId`, `PermissionToTake`, `currentSkin`. `grownItems` (single-produce yield), `seedItem`, `uprootItems`,
  `visibleGrownItems : GrowItem[]` (fruits hanging on multi-harvest plants), `growSpoil : GrowSpoil` (whole-plant death),
  `dontDestroyOnPickUp` (plant survives harvest and restarts).
- `GrowItem : Interactable` — one fruit on a multi-harvest plant. `readyToPick`, `age`, `currentSpoilTime`, `itemAmount`, `seedItemAmount`.
- `GrowingPanel` (singleton `instance`, `ActiveManu`) + `GrowingPanelUI.recipeListUI : RecipeListUI` — the bed UI.

## Planting flow

`RecipeButton.OnPointerClick` → `RecipeInfoUI` shows the recipe + a "Plant" create button (`GrowingPanel.SetCreateButtonValues("Plant", ..., "Grow")`)
→ `Growing.Grow()`: amount = `min(GetRecipeMultiplier(selectedRecipe), FreeGrowingSlotCount())`
→ `AmountSliderUI.GrowItems(growing, recipe, amount)`: amount > 1 → slider popup (`AmountSliderType.Grow`), `PressedOk` →
`GrowingPanel.instance.ActiveManu.StartGrow(currentValue)`; amount == 1 → `StartGrow(1)` directly.

`Growing.StartGrow(int amount)`: `RemoveItems(selectedRecipe, ...)`, then for each empty spot `PlantExternal(output, spot, ...)`.
Per plant: skill check `PlayerStats.SkillSuccess(...)` — on failure the plant gets `RottingOverride = true` and yield ×0.33.
Input item quality is **not** passed to the plant (only `currentSkins`, animal age data) — planting the worst items is free.

- Since 0.5.13 `CraftingBase.RemoveItems` reads `items ?? InputSlots` (earlier: only `InputOutputStorage`), in **slot order** (first matching slot first). `CraftingBase.InputSlots` is the device storage plus `ExtraInputSlots`; `Growing.ExtraInputSlots` adds all player slots (`Inventory.AllSlots(ignoreBackpack: false, ignoreCharacterSlots: true)`) only while the bed's panel is open for this device or during quick-plant.
- Each seed and its produce are **separate recipes** with the same output (e.g. Turnip ← `Turnip Seeds` and Turnip ← `Turnip`),
  so the UI shows two buttons per plant. Recipe types: `Grow`, `GrowMushrooms`, `GrowVines`, `GrowAquatic` (60 recipes, 44 outputs).
- `RecipeButton` already has a hover mode: `togglePress == false` → info shown on `OnPointerEnter`, hidden on exit.
  Set via `RecipeListUI.CreateButton` → `SetTogglePress(toggleInfoPress)`.

## Harvest

- `grow.Interact()` on the plant root (0.5.13): returns early unless `IsWithinHarvestReach()`; holds to uproot only when `Uproot() && !IsDead()`, otherwise calls `Harvest()` -> `HarvestExternal(out destroy, out full, hasSpace)`: if any `visibleGrownItems` are ready it **picks all of them** (each `GrowItem.GetItem()` adds to the player's inventory) and returns; otherwise it yields `seedItem` + `grownItems` (x `Age/FullGrownAge`) and removes the plant (`RemoveAfterHarvest`; a manual harvest uses `RemoveWithAnimation`, with `PlantMoveAnimation` swaying plants in on planting and lifting them out on harvest).
  `grow.Harvest()` still passes a `hasSpace` that always returns true for single-produce plants (**no free-space check** there); fruit is checked per `GrowItem.InventoryHasSpace()` ("Inventory full" use text).
- Dead and unfinished plants: `grow.Uproot()` is true for a dead plant (`growSpoil.spoiled`) or a not-yet-grown one with no fruit. Since 0.5.13 a dead plant is no longer hold-to-uproot: `Interact` calls `Harvest()` and the use text is "Uproot dead plant" (no "Hold press"); a not-yet-grown plant with no fruit still uses `HoldToUseController.instance.StartUpdate(Harvest)`. `grow.UprootDeadPlant` was renamed `UprootItems`; dead-plant items are taken only if they fit, otherwise the plant stays.
- `GrowItem.Interact()`: pick one fruit; returns early unless within use distance (`InteractObjects.IsWithinUseDistance`); checks `GrowItem.InventoryHasSpace()` first ("Inventory full" use text). `GrowItem.PlayHarvestAnimation` spawns a copy with `ModelPreview.CreateCopy` and runs `PlantMoveAnimation.Harvest`.
- UI cells: `GrowingPanel.UpdateItemsInProgress` fills `ProgressSlot`s, and on non-`ManualHarvest` devices calls `AddActiveGrowing`.
  `ProgressSlot.OnPointerClick` → `Growing.Harvest(GameObject)` harvests one plant into the device's storage.
- `Growing.Harvest()` (the `GrowingPanelUI.harvestButton`, non-`ManualHarvest`; that button is not visible in game) — harvests every spot into the device's `OutputStorage ?? InputOutputStorage`
  (`CreateItems`) — items land in whatever slots are free, mixed with seeds.
  Since 0.5.13 `Growing.HasSpaceFor` checks `OutputStorage ?? InputOutputStorage` plus the player's slots first; without room nothing is taken and `Inventory.inventoryPanelUI.InventoryFull()` is called.

## Spoilage on the vine (multi-harvest "already half rotten")

`GrowItem.UpdateGrow`: once `age >= fullGrownAgeFinal` the fruit is `readyToPick` and `UpdateSpoilTime(sec)` starts at once:
`currentSpoilTime += sec`, and the fruit's `meta` `ItemPerishabledata.integrity = 1 - currentSpoilTime / randomSpoilTime`.
`GetItem()` hands out that meta, so a fruit picked halfway through its on-vine spoil window has ~50 % integrity, and keeps
perishing in the inventory (`integrityLossPerHour`). At 100 % the fruit just **vanishes** — `ToggleVisibility(false)` (`SetActive(false)`) + `ResetGrow`, no item spawned — and regrows after
`startGrowAgain`. With `dontRemoveWhenSpoiled` it stays hanging and picking it gives `spoiledItem`.
Default `spoilTime = 14400` s (prefab values unknown — runtime dump). So "half rotten at harvest" is this timer, by design or not.
Side bug: a spoiled fruit gives `spoiledItem` × `seedItemAmount` (not `spoiledItemAmount`) and only if `seedItemAmount > 0`.
Bug, fixed in 0.5.13 [code]: in 0.5.13 `GrowItem.UpdateSpoilTime` sets `spoiled = true` in an `else` branch after `ResetGrow()`, so a regrown fruit is no longer flagged rotten (verify in game). In earlier builds `ResetGrow()` cleared `spoiled` and it was then set to true; `spoiled` is a private unsaved field, so the regrown fruit counted as rotten (`Spoiled()` true) until the game was reloaded.

## Bush lifetime (when multi-harvest plants stop fruiting) [code]

The plant's own `GrowSpoil` timer starts once the plant is fully grown (`grow.UpdateGrow` → `growSpoil.UpdateSpoilTime`).
It runs ×10 faster while `grow._Rotting` is true (`Rotting || RottingOverride`).
- `RottingOverride` is set when planting fails the skill check (`Growing.StartGrow`). Such a plant rots from the start, with yield ×0.33.
- `Rotting` is set by `Growing.StartRot()` / `StopRot()`. **No C# caller** — called from scene/prefab events (Relays, UnityEvents).
  A player's guess: unsuitable conditions (Sun, Moist), or no fuel in the stove of the farm above the shop. Verify at runtime.
  [user report, not explained] with the farm stove unlit nothing grew. The stove is not a temperature check in code; it can only matter as the bed's `PowerSource` (fuel-burning, `Powered` = fuel active). By the rule below an unpowered normal bed keeps growing, so either that farm bed is `AlwaysOn` (then `PauseGrowth` fires) or the cause is something else. Needs the bed's prefab `AlwaysOn`/`_Power` (offline asset reading) or a runtime check. `[data]` In `electronics-inventory.md` (furniture, upgrades, tenement `level4`) every `Growing` found is `Electricity PowerInput AlwaysOn`, none is fuel-burning; the farm above the shop is not in the scanned data (other scene).
- `Growing.OnPowerOut` does **not** pause growth for a normal device. `CraftingBase.OnPowerOut` returns true
  unless `AlwaysOn`, and `Growing` then returns early. It only fires `onInActive` (prefab relays, e.g. lamp off) and updates the panel.
  `PauseGrowth` runs only for `AlwaysOn` devices, which looks inverted. [verified by user: a switched-off terrarium by a window keeps growing]
  `grow.Paused` plants don't age, but rotting ones still update (`UpdateGrow`: `Paused && !_Rotting` → return).
- `spoilTime` (default 14400 s) ± `spoilTimeVariation` → `randomSpoilTime` is the bush's lifespan after maturity.
  `currentSpoilTime` and `randomSpoilTime` are saved.
- At `stopVisibleGrowItems` (default 0.75 of the lifespan) `StopGrow()` turns true. Picked or vanished fruit no longer reappear,
  but fruit already growing keep growing.
- At 100 % `spoiled = true`. `grow.UpdateGrow` returns early, so its fruit freeze. Interact = uproot (hold), but `HarvestExternal`
  still hands out ready fruit first.
- The bush colour shifts along `gradientColorChanges`. Prefab values per plant are unknown (runtime dump).

## Sun condition: windows, open sky, grow lamps [code]

`NaturalLightSourceChecker` (on a `Growing`) adds/removes conditions in `Growing.Modifiers` (`[fsIgnore]`, so not saved).
It's set up per prefab: `onSuccessAddCondition` (usually Sun), `onFailAddCondition`, and the matching remove conditions.
`HasSun()` = `powered` **or** within a window light's radius (`WindowNaturalLight.InNaturallightRadius`, ×`lightsourceRadiusFactor`,
optional line of sight) **or** nothing blocks the sky straight above for 35 m. **No time-of-day check**: a window gives Sun at night too.
- `powered` (serialized) is the built-in grow lamp (terrarium, Flowerbox With Lamp). It's set by `UpdateModifiers(bool)`, which has
  **no C# caller** — either scene/prefab events or a fixed prefab value. Runtime check.
- Re-evaluated on `WindowNaturalLight.SourcesChanged`, `SaveController.LoadingDone`, or `RefreshModifiers()`.
  `BuildingSystem` shows the result in the placement preview.
- Power loss: see Bush lifetime above. Growth isn't paused, so the device just loses whatever its lamp gave.
  `PauseGrowth` also sets `grow.GrowOnlyDuringDay` for Sun devices, but that field is **never read**. (`grow.Day` is assigned and also unused.)

## Growth clock and game speed [code]

`grow` has no timer of its own: `grow.DeltaSeconds(int)` is subscribed to `TimeOfDayAzure.SecondsPassed`, adds up `timepassed` and calls `grow.UpdateGrow(timepassed)` once it exceeds 60 s. `Age`, `GrowItem.UpdateGrow` and `GrowSpoil.UpdateSpoilTime` all use those seconds (`_Rotting` makes spoil ×10).
`SecondsPassed` carries `TimeOfDayAzure.deltaTime` = difference of `timeController.GetTimeline() * 3600` between frames. `TimeOfDayAzure.Update` sets `timeController.SetNewDayLength(actualSpeed)` every frame, with `actualSpeed = dayLenghtInMinutes / timeSpeedOverrideFactor`. So `timeSpeedOverrideFactor` (the dev console and dev menus set it directly; `SetSpeedFactor` does the same) speeds up the game clock and with it growth, by code. Not seen in game.
Something else resets the factor to 1 via `SetSpeedFactor(1f)`: `Crime`, `manufacturing` (~line 478), `Prison`, `ShopBaseUI`, and the end of a wait in `WaitingController`; `CardGameTable` sets 0.25 while playing and restores it. A mod or menu that sets the factor once can lose it after such an event, so re-apply it every frame or watch the live value.
`grow.UpdateGrow` returns early if `grow.Paused` (set by `Growing.PauseGrowth`, which fires on power loss only for `AlwaysOn` devices, see Bush lifetime) or the plant is already spoiled.

## Produce quality and freshness [code]

- Harvested items get `ItemQualityData.quality` (0.05..2) from `CraftingBase.ApplySkillQuality` → `PlayerStats.SkillQuality`
  (Farming skill × sobriety × (1 − failChance) × equipment quality, plus Gaussian noise σ 0.2). Same for `GrowItem.GetItem`,
  `grow.SetMetaValues` and `Growing.CreateItems`. Tiers: `ItemQualityData.GetQuality` (<0.25 Crap, <0.5 Poor, <0.75 Bad, <1.25 Okay, <1.5 Good, <1.75 Fine, else Excellent).
- Freshness is `ItemPerishabledata.integrity` (0..1).
- Stacking: solid items merge only within the same quality tier (`ItemMetaUtilities.QualityMatch`) and integrity within 0.025
  (`CompareItemMetas`).
- Vanilla inventory sort (`Inventory` comparer): name → quality desc → integrity desc → sale price desc → amount.
- Owner: harvested stacks carry the plant's `grow.ownerId` (NPC farms = `Autofarm`); `PermissionToTake` adds `ItemOwnerPermission`.
- Animal cages are `Growing` too (`AnimalBreeder` takes `GetComponent<Growing>()`).

## Patch points

- One-click plant: call `Growing.StartGrow(n)` directly (set `selectedRecipe` first), skipping `Grow()` / `AmountSliderUI`.
- Harvest all on a bed: iterate `Growing.GrowingSlots`, `grow.HarvestExternal` / `GrowItem.GetItem` per plant.
- Free inventory space: `Inventory.instance.InventoryIsNotFull(itemId, owner, amount, meta)`.
- Add with overflow to the ground: `ItemOperations.AddItemsAndDropRemaining(item, owner, slot, meta, amount)`; plain drop: `ItemOperations.DropItem(...)`.

## Quick plant, repeat harvest, variation (0.5.13) `[code]`

- `Growing` implements `IQuickSlotTarget` ([quick-slots.md](quick-slots.md)): `UpdateQuickSlot(RaycastHit)` shows the quick-slot bar with the recipes that pass `CanQuickPlant` and `RecipeInfoUI.CheckSkillSoft` (skill-locked ones are disabled with a text). The planting spot (`GetQuickPlantSpot`) is an empty spot within 2.5 m of the player and 1 m of the aim point, or directly under the player when `quickPlantUnderPlayer` is set. `QuickPlant` calls `StartGrow(recipe, 1, spot, items)` (a new overload with a target spot and the chosen stacks first) and blocks primary use for 1 s; `Growing.OnManualHarvest()` stamps a time that blocks quick-plant for 1 s. Holding Interact on a `Growing`, `grow` or `GrowItem` target calls `InteractObjects.RepeatHarvest` after 0.4 s, which calls `Growing.RepeatHarvest(point)` every 0.2 s within about 0.5 m.
- The skill gain per planted spot uses a literal 5 instead of the batch amount (`plantSkillGainMultiplier` = 5 is declared but unused at that call). The plant sound falls back to `RandomAudioLibrary.instance.plant`.
- `grow.FullGrownAgeVariation` (not saved) varies the grow time in `PlantExternal` and `InitializePreview` (the random state is saved and restored); `GrowItemAgeVariation` gives each plant a `growItemAgeOffset` added to its fruit's `fullGrownAgeFinal`; `noGrowItemRandomization` skips the 50% random hiding of fruit. `grow.InitializePreview` builds the ghost copy for the quick-plant preview. `GrowItem.GivesItems` (`itemAmount > 0`) gates item grants in `GetItem`.
