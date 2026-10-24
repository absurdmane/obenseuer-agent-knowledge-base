# Item system: definitions, stacks, meta, spoilage
Scope: `Item`, `ItemStack`, `ItemReference`, `ItemDatabase`, item meta classes, perishing, stolen/owner marking. Not covered: category list, item action classes, inventory slot operations, price formulas.
Status: code-only (Items.json facts are `[data]`)
Related: [inventory-storage.md](inventory-storage.md), [items-recipes.md](items-recipes.md), [economy.md](economy.md), [police-crime.md](police-crime.md), [growing.md](growing.md)

## Where it lives

- `Item`: static definition from `Items.json`: `ID`, `Title`, `Categories` (the first entry is the category), `BaseValue`, `BaseStackable` (JSON key `Stackable`), `Appearance`, `Actions`, `Attachments`, `Meta` (template array). [code]
- `ItemStack`: one stack in a slot: `itemId` (-1 = empty), `itemAmount`, `ownerId` (default -1), `refigerated`, `spoilDisabled`, `_meta` (saved). `ItemStack.CreateStackFromItem(item, amount, ownerId, meta)` is the factory. `ItemReference` is a serializable `{name, itemnamespace, ID}` pointer resolving through `ItemDatabase`. [code]
- `ItemDatabase` (`instance`): `Awake`/`OnEnable`/`Reload` read `StreamingAssets/Items.json` with FullSerializer into a static `database`, then load referenced sprites and assets. `FetchItemByID(id)` is a linear scan; `FetchItemByID(id, ns)` uses `extendedDatabase[ns]` with fallback to the base list; `FindByString` is a case-insensitive title search. [code]
- Meta classes live in the `OS.Items` namespace: `ItemPerishabledata`, `ItemQualityData`, `ItemStolenData`, `ItemOwnerPermission`, `ItemPackageData`, `ItemLiquidData`, `ItemEmptyLiquidContainerData`, `ItemAmmodata`, `ItemSkinData`, `ItemAnimalData`, `ItemFurnitureData`, `ItemGrowdata`, `ItemVariantData`, `ItemCertificateData`, `ItemGroupData`, `ItemEmptyPackagerData`, `ItemFuelData`. Helpers: `ItemMetaUtilities` (`GetMetaOfType`, `AddMeta`, `RemoveMeta`, `UpdateItemMetas`, `MetaMergeAction`, `PerishableMix`, `QualityMix`), `Item.CloneMeta`. [code]
- `Items.json` `[data]`: 1230 entries, each with `ID`, `Title`, `Categories`, `BaseValue`, `Stackable`, `Appearance`, `Actions`, `Meta`; meta entries carry a `$type` such as `OS.Items.ItemPerishabledata`. `ItemOwnerPermission` does not appear in the file: it is added at runtime.

## How it works [code]

- `Item.Value` and `Item.Stackable` are not raw: they go through the current `DifficultyInfo` (`GetItemPrices`, `GetStackSize`); raw values are `BaseValue`/`BaseStackable`.
- `ItemStack.Meta` is lazy: when `itemId != -1`, the meta is empty and the amount is above 0, the getter clones the item's template meta (`CloneMeta(fullCopy: false, ...)`). Writing into the array returned by `Item.Meta` (the template) changes every future stack; clone first.
- Perishing: `ItemPerishabledata` has `integrity` (1.0 start), `integrityLossPerHour` (0.1), `refigerationFactor`, `spoiledItem`, `dontDestroyAtZero`, `perishingDisabled`, `lastChecked`. Loss is counted per game minute, reduced by the storage's refrigeration when `refigerationFactor != 0` (`1 - storageRefigeration / refigerationFactor`, clamped 0..1). At 0 the item becomes `spoiledItem` in the same slot unless `dontDestroyAtZero`; containers recurse into children. The tick is run from `Slot.CheckItemMetas`, `CharacterSlotController`, `CollectibleItem` (dropped items), `LiquidStorage`, `Storage` and `ItemMetaUtilities.UpdateItemMetas`.
- `CloneMeta` for perishables copies integrity, loss rate, refrigeration factor, spoiled item and flags but not `lastChecked` or `refigerated`, so a clone starts a fresh clock; a hand copy of meta loses them too.
- Merging stacks takes weighted averages (`PerishableMix`, `QualityMix`).
- `ItemQualityData`: `quality` (1.0 default), `modifies` flags (Value, BaseStats, Strength, Depression, SMV, Health), `fixedQuality`. Buckets: below 0.25 Crap, below 0.5 Poor, below 0.75 Bad, below 1.25 Okay, below 1.5 Good, below 1.75 Fine, else Excellent.
- Stolen and owner marking: `Inventory.CheckForAddedStolenItems` returns early if `owner < 2` or the meta has an `ItemOwnerPermission` with `ownerPermission == true`; otherwise it sets `ItemStolenData.playerStolen`. `ItemAction.Invoke` calls it on every consumption. `ItemOwnerPermission` is attached by `CharacterItemManager`, `CollectiblePackage`, `CollectibleItem`, `Growing` and `grow`.
- `ItemPackageData` is a container with child items (`AddItemToPackage`, `Unpack`, `CountPerishingTimes`). `ItemLiquidData` holds `liquidItem`, `emptyContainerItem`. `ItemAmmodata` is ammo count.
- There is no durability field; the closest is `ItemPerishabledata.integrity` (`MeleeWeapon` reads it).

## Pitfalls

- `ItemReference(int id, string imnamespace)` passes the still-null field, not the parameter, to the first fetch, so the namespace is ignored; `ItemReference(int id)` does not null-check `ItemDatabase.instance`.
- `Item.Appearance` is used without a null check in `Item.Prefab`/`GetItemTitle`.
- `ItemDatabase.Reload`/`OnEnable` re-read `Items.json` and drop runtime changes made to shared `Item` templates.
- Patch getters rather than saved fields (`ItemStack.itemId`, `ownerId`, `refigerated`, `spoilDisabled`, `_meta`).

## Unverified

Meaning of `ownerId` values 0 and 1 (probably player and none); semantics of a negative `refigerationFactor`; whether `dontDestroyAtZero` items show as spoiled in the UI.
