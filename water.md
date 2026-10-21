# Water and liquids

Scope: `LiquidStorage`, the global water source, taps as `WaterSource` recipes, filling flow, the water item family. Not covered: JSON format ([items-recipes.md](items-recipes.md)).
Status: code-only; taps seen in game are [verified]
Related: [items-recipes.md](items-recipes.md), [crafting.md](crafting.md), [power.md](power.md)

## LiquidStorage

`LiquidStorage : Interactable` — any liquid container/source (barrel, tank, sink).
- `Liquid : ItemReference` — which liquid, `LiquidAmount`, `Capacity`, `Meta`, `qualityData`, `perishableData`.
- **`Capacity < 0` = infinite source** (`HasInfiniteOutputCapasity`).
- `ExternalOutputStorage`: if `Capacity < 0`, no own `externalOutputStorage` and `!NotPartOfMainSource` —
  pours from `WaterSourceController.instance.globalStorage`. So sinks are most likely infinite "windows" into the scene's shared source.
- Filling an item: `FillItem(ref ItemStack)`, check `ItemCanBeUnfilled(ref ItemStack) : Result`
  (`Result.Empty`, `Disabled`, ... — can be returned to make "no water come out").
- Storage being interacted with: statics `LiquidStorage.currentStorage`, `CurrentOutputStorage`.
- `StorageRefigeration` — cooling of the liquid (for milk etc.).

## Scene-wide global source

- `WaterSourceController.instance.globalStorage` = `info_water_source.instance.globalLiquidStorage` (if the scene has the marker),
  otherwise the source from `globalWaterSources` matched by scene name.
- One source for the whole scene — **water quality currently does not depend on the apartment**.

## Water taps are recipes `[data]`

`Recipes.json` has a recipe type `WaterSource` with no inputs and 15 s process time; the output depends on the device's conditions:

| Recipe | Required | Excluding | Output |
|---|---|---|---|
| Water | Clean | | Clean Water |
| Dirty water | Dirty | | Dirty Water |
| Shroom Water | Infested | | Shroom Water |
| Beer | Special | | Beer |
| Cup of Water | Clean, Variant1 | | Clean Water 0.2 |
| Empty | — | Variant1 | nothing |

**No code uses `WaterSource` recipes** (only `RecipeController` maps the type to a task category) — they look like unused data.
Taps are most likely infinite `LiquidStorage`s reading the global source (see above). Old guess: a tap is a `CraftingBase` with `Recipes = [WaterSource]`, and its water quality comes from `Modifiers`
(see [crafting.md](crafting.md)). Removing Clean/Dirty/Infested from `ModifiersWithUpgrades` leaves only "Empty" → no water.
**Verify at runtime** which component a sink actually is, and how it relates to `LiquidStorage`/global source above.

Water item IDs and stats: [items-recipes.md](items-recipes.md).

## Taps seen in game [verified, runtime inspector]

- **Commercial Kitchen Table With Sink** (furniture, `sink_003b_usable`, layer Usable): a `LiquidStorage` with its own liquid
  `Clean Water [10840]`, `Capacity = -1`, and **no** external output source (`ExternalOutputStorage` is null, so it pours itself).
  So not every tap reads the global source: some are infinite sources with a fixed liquid. Clean regardless of the building's water.

## Open questions (need runtime)

- Which component the apartment sink is (CraftingBase with WaterSource? LiquidStorage?) and its `Modifiers`.
- Which liquid is in the Tenement global source, and does it change with `TenementGeneral.currentWater`.
- Where the apartment sink comes from — player furniture or part of the upgrade prefab (`TenementApartmentPrefabsContent.waterShit/Good/Best`).

## Filling flow `[code]`

`ItemCanBeFilled` / `FillItem` delegate to `ExternalOutputStorage` (the global source for infinite taps); filling progress is driven by
`LiquidStoragePanelUI` through `LiquidStorage.CurrentOutputStorage` (= `currentStorage.ExternalOutputStorage`), not by the source object.
So a postfix on the `ExternalOutputStorage` getter can swap the source per tap (a mod can do this with stand-in sources
on inactive GameObjects — inactive objects are skipped by `SaveController.ExecuteSaveLoadFunctions`).

## Timed filling loop and the liquid-loss bug [code, inferred; not reproduced in game]

The pour is not an animation but a per-frame loop in `LiquidStoragePanelUI.Update`, active while `LiquidStorage.CurrentOutputStorage.fillingItem` (fill item from source) or
`CurrentInputStorage.unfillingItem` (empty item into storage) is true. These flags are set by `LiquidStorage.FillItem` / `AddLiquidFromItem` (Fill/Empty buttons, `QuickMove`) after
`ItemCanBeFilled` / `ItemCanBeUnfilled` returned OK; the compatibility checks (same liquid, meta, free space) happen only there, once, at the start.
- Tick: every ~0.01 s of accumulated frame time `num = 0.1 * CurrentSpeed` (`FillingSpeed` / `UnfillingSpeed`, ints, default 1). Fill: `num` clamped to the item's free space and to the source's `LiquidAmount`;
  then `source.LiquidAmount -= num` (skipped for infinite sources, `HasInfiniteOutputCapasity`), quality/perishable mixing, theft counter, and
  `itemLiquidData.SetLiquidAmount(old + num, null, itemStackSize)` (floors to 0.1 L per item, see `ItemLiquidData.SetAndGetLiquidAmount`).
- The target is NOT stored at pour start. The loop uses the panel's cached `itemLiquidData` / `itemStackSize`, which `OnItemDataChanged` re-reads from whatever `ItemInfo.instance.ItemData` is now.
  `ItemInfo.ItemDataChanged` fires on every `ItemInfo.UpdateInfo` (including the one at the end of every tick) and on every selection change (`ItemInfo.Activate`).
- What stops it (`StopFilling`): `ItemInfo.instance.ItemData == null` (deselect, `ItemInfo.Deactivate`), no `ItemLiquidData` on the selected item, source empty (`OnEmpty`), item at max
  (`IsMaxAmount`), target full, panel `OnDisable`, and `UpdateButtons` when `ItemCanBeFilled` returns anything but OK/Filling/Unfilling.
- The only other code that drains a storage like this is `InteractableShower`.

**Most likely cause of "source keeps draining, target is not filling":** selecting another item while a pour runs does not stop it. `OnItemDataChanged` calls `UpdateButtons` first, and
`ItemCanBeFilled` returns `Result.Filling` as long as the flag is set (before any item checks), so the flag stays; the next tick then pours into the newly selected item B without the start-time checks:
- B has `ItemLiquidData` but `liquidItem == null` (a container that stays the same item when empty; `FillItem` normally assigns the liquid to it): `ItemLiquidData.GetLiquidAmount` returns 0 whenever
  `liquidItem == null`, so `IsMaxAmount` is never true and the displayed amount never rises, while `source.LiquidAmount -= num` runs every tick until the source is empty. Liquid is lost.
- B holds a different liquid / meta: the amount rises but the liquid kind of B is kept (the source liquid is silently converted).
- B is a plain empty container (`ItemEmptyLiquidContainerData` only), a full one, or has no liquid data: the loop stops by itself (no loss).
Moving the mouse away only deselects if the game calls `ItemInfo.Deactivate` (then `Update` stops the pour); deselect alone is safe. Whether the reported case was this needs an in-game check (log `fillingItem`, target
`liquidItem` and source amount per tick, or a runtime inspector on the barrel).

**Patch points (fix):**
- Prefix `LiquidStoragePanelUI.OnItemDataChanged(ItemData)`: if `LiquidStorage.currentStorage != null` and (`CurrentOutputStorage.fillingItem` or `CurrentInputStorage.unfillingItem`) and the panel's cached
  `itemLiquidData` is non-null and differs (by reference) from `itemData?.slotController?.itemStack.GetMetaOfType<ItemLiquidData>()` → call `__instance.StopFilling()` before the original runs.
  The meta instance stays the same during a pour (the per-tick `UpdateInfo` refires the event with the same item), so it only triggers on a real target change.
- Safety net in the same spirit: postfix `LiquidStoragePanelUI.Update` is awkward; better a prefix on `Update` that stops the pour when `itemLiquidData != null && itemLiquidData.liquidItem == null` during a fill.
- Refund/clamp is possible but weaker than prevention: remember source `LiquidAmount` and the target's `GetLiquidAmount` at start (postfix `FillItem` returning true), and in a postfix on `StopFilling` add
  `drained - gained` back to the source (clamp to `Capacity`; skip infinite sources and when the source was replaced). Cannot repair liquid already lost in saves.

## Water items `[code+data]`

| ID | Title | Drink effects | Water-group quality factor | How you get it |
|---|---|---|---|---|
| 13270 | Sewage | Health −33, Depression +3, High +3 | — (Waste) | sewer; input to Dry Sewage, Biogas |
| 12980 | Dirty Water | Amount 8, Health −1, Depression +0.66, Laxative | 0.65 | `WaterSource` recipe (Dirty) |
| 14950 | Shroom Water | Amount 8, Health −2, Depression −0.66, High +1 | 0.5 | `WaterSource` recipe (Infested), ChemicalLab |
| 10840 | Clean Water | Amount 10 | 1.0 | `WaterSource` recipe (Clean), purify Dirty/Shroom water (Distillery, ChemicalLab) |
| 15970 | Pure Water | Amount 14, Depression −0.5 | 2.0 | Distillery from Clean Water; input to Saline |
| 16200 | Spring Water | Amount 10, Depression −1 | 2.75 | shop only; no recipe uses or makes it |
| 15980 | Heavy Water | — | 0.9 | special (SNW) |

`14880 Water` is an **item group** (`Meta[0].itemGroup`): recipes ask for "Water" and accept any member, with per-member `qualityFactor` / `addSMV`.
