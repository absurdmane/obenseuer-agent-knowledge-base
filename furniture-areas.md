# Furniture and building areas

Scope: `Furniture` definitions, `BuildingArea`, how to tell which room an object is in; addressable furniture/upgrade asset names and the generator.
Status: code-only
Related: [tenement-upgrades.md](tenement-upgrades.md), [power.md](power.md), [items-recipes.md](items-recipes.md), [levels.md](levels.md)

- `Furniture` (ScriptableObject) — furniture definition: `category` (Electronics, Lights, Machines, Kitchen, Bathroom, ...),
  `restrictedAreas : BuildingArea[]` — areas where it **can't** be placed (`BuildingSystemSlot` greys the slot out when
  `BuildingSystem.activeManager.area` is in the list; empty = allowed everywhere). Per-furniture values are asset data
  (addressables in `StreamingAssets/aa`), not in code or JSON — e.g. whether machines are allowed in the stairwell is unknown.
- `Furniture.BuildingArea {None, Stairs, PlayerApartment, Workshop, Outside, Greenhouse, Store, Communal, Public}`.
- `FurnitureManager` (one per building area): `area : Furniture.BuildingArea`, `placedFurniture`,
  `preplacedFurniturePrimary/Secondary`, `tenementApartmentPrefabs` (link to the apartment → `.apartmentNumber`).
- `TenementApartmentPrefabs.furnitureManager` — the apartment's area; `TenementGeneralPrefabs` holds the building's area managers
  (`waterManager`, `stairwellManager`, `basementManager`, `workshopManager`, ...).
- Player's current area: `BuildingArea.currentManager`.

**How to find a device's room**: `device.GetComponentInParent<FurnitureManager>()` (not verified that furniture is parented to its manager)
→ if `tenementApartmentPrefabs != null` — apartment `apartmentNumber`, otherwise by `area` (Stairs/Workshop → building).
Needs a runtime check, especially for objects pre-placed in the scene (not player furniture).

## Built-in objects of upgrade prefabs [verified, runtime inspector]

- Things built into an upgrade prefab have no `FurnitureManager` among their parents. Example, a switch lamp of the player's walls:
  `Apartments/Apartment 1 - Player/UPGRADES_Apartment_PLAYER/Upgrade_PLAYER_Walls_(lvl 2) - Shabby(Clone)/Clutter/Lights/Living_room/lamp_008_toggle`
  (`LightTurnOffOn` on the mesh itself). The apartment's manager is a sibling branch (`.../BuildingArea Player Apartment`).
- `UPGRADES_Apartment_*` carries `TenementApartmentPrefabs` (`.furnitureManager` = that apartment's manager); building upgrades sit
  under `TenementGeneralPrefabs` (`General/UPGRADES_General/...`). A mod can map a built-in object to its room through these.

## Furniture and other addressable assets

Placeable furniture (`Furniture` ScriptableObjects) and upgrade assets are Addressables in `StreamingAssets/aa`.
The bundles are compressed, but `aa/catalog.bin` lists asset names in plain text (~1960 `*.asset`):
`grep -aoE '[A-Za-z0-9_ /().,&-]{2,100}\.asset' catalog.bin | sort -u`. Names only — field values need a runtime dump.

- The only generator furniture is **`Mini Generator`** (`Generator : CraftingBase`, fuel-powered). No code links a generator's
  output to other devices; `Generator.nominalOutput = 500` is only shown in the panel (`GeneratorPanel`, `showOutput`).
- Upgrade assets: `Electricity-0-None` .. `Electricity-3-Best` (building), `Small-Electricity-0-Off/1-On` (apartment).
