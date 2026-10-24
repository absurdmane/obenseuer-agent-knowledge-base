# Item and recipe data (JSON)

Scope: `Items.json` / `Recipes.json` in StreamingAssets: shape, encodings, recipe conditions in use. Water items are in [water.md](water.md); furniture/asset names in [furniture-areas.md](furniture-areas.md).
Status: code + data (JSON queried directly)
Related: [crafting.md](crafting.md), [water.md](water.md), [furniture-areas.md](furniture-areas.md)

Item and recipe definitions are plain JSON in `Obenseuer_Data\StreamingAssets\` — query them with Python instead of decompiling.

| File | Encoding | Shape |
|---|---|---|
| `Items.json` (~1230 items) | UTF-8 BOM (`utf-8-sig`) | list of `{ID, Title, Categories, BaseValue, Stackable, Appearance, Actions, Meta}` |
| `Recipes.json` (~890 recipes) | **UTF-16** | `{"Recipes": [ {ID, Name, Type, RequiredConditions, ExcludingConditions, input, output, ProcessTime, FailChance, RelevantSkill, metaSettings, ...} ]}` |
| `StorageSpawnCategory.json`, `StoragePrefabs.json`, `Characters.json`, `NPCBehavior.json` | — | not explored yet |

`input`/`output` entries: `{"Key": {"name", "ID"}, "Value": amount}`. `Actions[].Type == "Drink"` holds drink effects (`Amount`, `Health`, `Depression`, `SpecialEffectName`, ...).

## RecipeCondition usage

Recipes are filtered by the device's conditions (see [crafting.md](crafting.md)). Notable required conditions:
`WaterSource`: Clean / Dirty / Infested / Special(Beer) / Variant1(cup); `Machining`+`Press`: **Industrial** (watches, press plates);
`Oven`: Clean, Steam; `Grinder`: Clean, Crude; `Grow*`: Sun, Large, Moist; `Compost`: Advanced.
Industrial recipes, all of them: Steel / Brass / Silver / Gold mechanical watches (`Machining`, skill 7–10), Press Plates (`Press`).
Which devices carry `Industrial` is prefab data (not in JSON) — runtime. Machine furniture assets in the catalog: Metalworking Lathe,
Geared Head Drill, Hydraulic Press, Printing Press, Compressor, Rolling Machine, Textile Mill, Sewing Machine, Grain Mill, ...
`Electricity` exists in the enum (and has an icon in `ModifierUI`) but no recipe requires it.

## Mushroom extracts [data + code]

Three liquid items (`Categories` Liquid, Special, Mushrooms; stack 32), each with one `Drink` action (Amount 2, `DestroyAfterUse` false), so they can be drunk directly:

| Item (ID) | BaseValue | SMV | SMVProgressionRate | Depression | Health | High | MushroomAddiction |
|---|---|---|---|---|---|---|---|
| Green Mushroom Extract (14910, also category "Mushrooms Green") | 284 | 10 | 0.3 | -10 | -20 | 100 | 50 |
| Pure Green Mushroom Extract (18090, "Mushrooms Green") | 548 | 0 | 0 | -10 | -10 | 100 | 50 |
| Red Mushroom Extract (15790) | 941 | 101 | 101 | -100 | -100 | 100 | 50 |

No food, drink, tiredness, bleeding, hygiene or special effect values; no meta. Code (`PlayerStatsMushrooms.UpdateValues`): `MushroomsUsed += GetHigh()` (absorbed over time by `UpdateMushroomsHighValue`); `MushroomNeed -= High * 5 / (0.01 * addiction + 0.01)`; `MushroomAddiction += High * 0.15` (cap 100); `SMVProgression += SMV` (scaled by `(1 - SMVProgression/100)^2` when 0 < SMV < 100, so 101 is applied in full); `SMVProgressionRate` gets `Abs(smv * (10 - rate) / 100)` when the item's rate is 0 and SMV > 0, else the item's rate (scaled the same way when < 100, full when >= 100). Health and depression go through the usual consumable path.
Recipes (Recipes.json): `Mushroom extract` (ID 10440, `ChemicalLab`, tag "Open Sewer Magazine", no recipe book required): input Bag of Green Mushrooms (High Potency) 18002 + Empty Syringe, output Green Mushroom Extract 0.1 + Glowing Bottoms 0.4 (+ syringe back). Green extract (14910) is an input of about a dozen "poisoning" recipes (about 12 hits in Recipes.json; e.g. Socca Junior + Empty Syringe + Evil Bear Candy Bag + 0.1 extract -> Poisoned Socca Junior + syringe + Plastic Trash; skill Chemistry, `ProcessTime` 500, on failure 9% chance of `Mushrooms` damage 5 to the player). Pure Green (18090) appears as input and output in a few more recipes (lines near 77 000 and 88 000, not read). Red extract (15790) is in no recipe: it can only come from a quest/scene source or the console.
So spiking food and drink with extract is a general recipe pattern ("Poisoned <item>"), not item meta. Not read: the full list of poisoned items, the pure-extract recipes, and which item is used for the wine.

## Exact statistics `[data]`

Parsed with a JSON loader (not text search). `Recipes.json` is utf-16 with a `{"Recipes": [...]}` wrapper; `Items.json` is utf-8-sig.

Recipes: 929 recipes, IDs -1 to 10924. Count per `Type`: Basic 191, Stove 108, Oven 73, Furnace 69, Mashing 51, Distillery 46, Fermenter 42, Grow 40, ChemicalLab 32, Bottler 30, PaperWork 28, Anvil 28, Grinder 22, Machining 20, BreedAnimals 16, GrowVines 13, Kitchen 12, Compost 12, ChemicalRefinery 12, GrowMushrooms 11, Electronics 9, Woodworking 7, Printer3D 7, Crusher 7, WaterSource 6, Sewing 6, Press 6, Debug 5, CoffeeMaker 5, Loom 5, Microwave 4, Engineering 3, GrowAquatic 2, Malting 1. `RelevantSkill` counts: Cooking 404, Manufacturing 107, Farming 82, Mashing 51, Distilling 46, Chemistry 44, Brewing 42, None 34, Smithing 29, Paperwork 28, Machining 18, Sewing 10, Electronics 9, Engineering 9, Carpentry 6, Mining 6, Medical 3, Malting 1.

Median `ProcessTime` (game seconds) / `FailChance` per type: Basic 60 / 0.05, Stove 120 / 0.15, Oven 600 / 0.25, Furnace 3600 / 0.025, Mashing 7200 / 0.161, Distillery 3600 / 0.078, Fermenter 150000 / 0.104, Grow 14400 / 0.01, ChemicalLab 2000 / 0.2, Anvil 3000 / 0.15, PaperWork 700 / 0.8, Machining 4300 / 0.36, Compost 82000 / 0.0, Kitchen 1 / 0.2, WaterSource 15 / 0.0. Highest `MinimumSkill` is 10; `HardMinimumSkill` true in 4 recipes; `RecipeRequired` (must be learned first) in 113; `useOnlyStolenItems` in 0.

`RequiredConditions` values in use: Sun 55, Clean 31, Special 23, Large 21, Crude 19, Moist 11, Water 10, Steam 9, Advanced 7, Industrial 5, Variant1 5, Small 3, Normal 3, Infested 2, Dirty 1. `ExcludingConditions`: Makeshift 39, Crude 38, Water 8, Advanced 6, Infested 5, None 4, Dirty 2, Variant1 1, Clean 1. `metaSettings` keys: `ammoInputItemsToIgnore`, `certificateLinkedItem`, `fixedQuality`, `mailName`, `mailReceiver`, `packagedItems`, `putLiquidsIntoContainer`, `qualityFactor`, `qualityFactorFail`, `returnLiquidContainers`, `setAmmoOfMax`, `setAmmoOfMaxFail`, `setIntegrity`, `setIntegrityFail`, `useAmmoAsInput`, `useFixedQuality`, `useLiquidContainers`.

Items: 1240 items, IDs 0 to 300050, 0 duplicate IDs; 790 items have more than one category; 114 distinct category values. First-category counts (top): Food 371, Liquid 125, Valuable 63, Hardware 47, Growable_plant 44, Mash 42, Meat 40, Burnable 39, ItemGroup 38, Junk 37, Seeds 34, Wine bottle 25, Medical 23, Soft drink 21, Smokes 21, Alcohol 18, Animal 16, breedable_animal 15, Readable 14, Dough 12, Electronic 12. Stack sizes: 1 573, 8 393, 16 108, 4 66, 64 32, 32 30. 773 items have actions.

Meta `$type` counts: ItemPerishabledata 462, ItemQualityData 378, ItemAmmodata 156, ItemFueldata 95, ItemGrowdata 59, ItemGroupData 38, ItemLiquidData 21, ItemPackageData 13, ItemEmptyLiquidContainerData 8, ItemAnimalData 7, ItemVariantData 5, ItemWallpaperData 5, ItemConnectivityData 4, ItemMailableData 3, ItemReceiptData 3, ItemScraperData 2, ItemInvoiceData 1, ItemWritableData 1, ItemTextData 1, ItemOSCoinWalletData 1, ItemEmptyPackagerData 1, ItemCertificateData 1, ItemFurnitureData 1. Action `Type` values: Food 411, Drink 193, Container 34, None 28, Smoke 20, Medicine 12, Slaughter 8, Hygiene 4, PiggyBank 3, Bandage 3, Pet 3, ScratchCard 2, Caffeine 1.
