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
