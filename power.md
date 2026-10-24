# Electricity and device power

Scope: `PowerSource`, fridges (`Storage.powerOff`), lamps, TV/radio, generator, outage behaviour. The power grid added in build 0.5.13 is in [power-grid.md](power-grid.md).
Status: partly verified (lamp and fridge facts [verified] inline); statements re-checked against build 0.5.13
Related: [power-grid.md](power-grid.md), [fuel-heat.md](fuel-heat.md), [crafting.md](crafting.md), [tenement-upgrades.md](tenement-upgrades.md), [water.md](water.md)

## PowerSource — power for machines and devices

`PowerSource` (Serializable, embedded in a device as the `_Power` field):
- `Type : PowerSourceType {Electricity, Combustion, Kinetic}`.
- `RequiresFuel` (not saved) — fuel-burning device (generator, furnace): powered = `_Fuel` is burning.
- `PowerInput` (**saved**) — "the device gets electricity from somewhere". Defaults to `true`.
- `Powered` getter = `RequiresFuel ? _Fuel.active : PowerInput`. All checks go through it and `IsPowered()`.
- `OnPowerOn/OnPowerOff/UpdateState` fire `ActOnPowered/ActonUnPowered` and the `onPowered/onUnPowered` Relays.
- `powerConsumption` (not saved, default 60 since 0.5.13) is the device's draw on its power grid: `PowerBalance => -powerConsumption` (see [power-grid.md](power-grid.md)).

Owners of `_Power`:
- `CraftingBase` (base of all workstations: `manufacturing`, `manufacturingProcess`, `Generator`, `Growing`, ...).
  `PoweredAndActive = AlwaysOn || (On && _Power.IsPowered())`. `SetPowerSourceStatus(bool)` — external supply on/off.
  Virtual `OnPowerOn()/OnPowerOut()` — state-change hooks.
- `PutItemSelection` — its own `_Power`, same scheme.

Unpowered machines in the world (e.g. the drill press in the Dekula A attic) most likely just have `PowerInput = false` in the prefab/save — **verify at runtime**.

**Patch point for "no electricity — doesn't work"**: postfix on `PowerSource.get_Powered`: if `Type == Electricity && !RequiresFuel`,
then `result &= <room has electricity>`. The saved `PowerInput` stays untouched. When the electricity level changes,
call `UpdateState`/`OnPowerOn/Out` on the room's devices, otherwise UI/sound won't refresh.
Since 0.5.13 the game gates electric non-fuel devices by a power grid ([power-grid.md](power-grid.md)); prefer a grid (or its `GridEnabled`) to a custom postfix, because the game now also writes `PowerInput`.

## Fridges — Storage

- `Storage.refigerationFactor` (taken from `StorageCategory.Refigerated` if 0) — cooling strength. `Storage.StorageSettings` = the category name.
- [data, StoragePrefabs.json] Categories with cooling: Fridge 0.5, Refigerator 0.5, Market Small Bottle Refigerator 0.5, Freezer 1,
  Chest freezer 1, Freezer Small 1, **Styrofoam Box 0.33** (the Styrofoam Cooler: no electricity involved). Nothing in `Storage`
  marks a storage as electric; tell them apart by category. The Chest Freezer prefab overrides its factor to 1.5 [data, runtime dump].
- [data, read from the game bundles with offline asset reading, see [asset-bundles.md](asset-bundles.md)] Every refrigerated `Storage` (`refigerationFactor > 0`) in the Addressables prefabs (`StorageSettings` | factor):
  Fridge | 0.5 and Freezer Small | 1 (Refrigerator, Refrigerator Alt, Refrigerator Brown, Refrigerator Brown Alt, and the kitchen upgrades `Upgrade_NPC_SMALL_Kitchen_(lvl 1..4)` Malone — two storages each, titles Fridge / Freezer);
  Refrigerator Modern (and Alt): both doors `Fridge` — upper 0.5, lower **1.0** (title Freezer, but StorageSettings is still `Fridge`, factor overridden);
  Chest Freezer: `Chest freezer` | 1.5; Commercial Refrigerator: `Market Small Bottle Refigerator` | 0.7, `maxSlotStackSize` 0;
  Styrofoam Cooler: `Styrofoam Box` | 0.33; **Kalle Ketku/Trade: `Custom` | 1.0** (a trader storage; the only refrigerated storage whose category name is not a cooling category).
  The worn Cooler Bag storage (scene object, not in the bundles) is 0.5 [verified, runtime dump]. No other prefab Storage has `refigerationFactor > 0`; 'Custom' with factor 0 is the common default for non-storage devices (stoves, machines).
  So a category-name check misses Kalle Ketku and misreads nothing else; checking `refigerationFactor > 0` (the base value) catches all, and the dump titles (Freezer, Fridge) are `Storage.title`, not the category.
- `Storage.powerOff` (public) + the `RefigerationFactor` getter returns 0 when `powerOff` or, since 0.5.13, when the fridge's own switch is off (`PowerSwitchedOn`: `showPowerButton` set and its `powerGridListener` off).
- `Storage.TogglePower(bool on)` turns cooling off/on (`ToggleRefigeration`) and calls `RefreshPowerState`; only called from `Storage` itself. The public `Storage.SetPowerSwitch(bool)` toggles the fridge's grid listener (relays `onPowerOn/onPowerOff` fire on state change, forced after load).
- Spoilage: `ItemPerishabledata.CountPerishingTime(storageRefigeration)`.

- **`powerOff` is saved** (public, no `[fsIgnore]`). Setting it via `TogglePower` would leave fridges off after the mod is removed.

**Patch point**: postfix the `RefigerationFactor` getter (→ 0 when the room has no power); spoilage reads it directly
(`RemoveMinutesFromIntegrity(..., RefigerationFactor)`). `UpdateRefigeration(RefigerationFactor)` caches it for liquids/UI,
so on a power change call it again for the room's fridges. Don't touch `powerOff`.

## Lights

- `LightController.electricLight` (default `true`) — marks an electric lamp (vs candle/fire). Used in
  `LightCullZone.ChangeIntensityMultiplier` (global dimming) and `CameraShake`.
- [verified, runtime dump] `electricLight` is unreliable both ways: the stairwell's `lamp_001 (*)/Point Light` are `false`; glowing mushrooms
  (`Green Mushrooms ...`, `Orange_mushrooms_grow`), `Barrelstove_standard` (fire) and `lantern_001_red_lit` are `true`.
  [data, runtime dump 16.10] Also flagged `true`: fire in furniture (Brick/Gas Furnace, Stone Fireplace, Red Barrel with Fire, Metal Stove,
  Ball Grill, stills, Mashing Barrel), candles, Jack O Turnip, Plutonium Core, window lights (`Tenement_window_makeshift_001`,
  `shack_window_002`), NPC body glow (`wasteland_red_body_glow_001`), lights of devices (TVs, stoves, grow beds). Not usable to tell
  electric lamps apart; a mod can use a whitelist of furniture/object names instead.
- [data, read from sharedassets4.assets (tenement upgrade prefabs) with offline asset reading, see [asset-bundles.md](asset-bundles.md)] Bare `Point Light (n)` objects under upgrade prefabs (parent is not a lamp object).
  Positions are composed up to the `Upgrade_*` root; "fixture" = a lamp-like mesh (name matches lamp/light/ceiling/lantern/...) within 0.5 m, an object that is a *sibling* of the light, so the light belongs to that fixture.
  The full per-light table (upgrade, parent, class, nearest lamp mesh and distance, range, colour, type, `electricLight`, position) is not published here;
  regenerate it with `lights.py` then `bare_lights_report.py` (see [asset-bundles.md](asset-bundles.md)). 521 rows, many duplicated by `_EXTENSIONS FOR TEST/...` copies of the player extensions.
  - Totals: fill (no lamp-like mesh within 1.5 m) 433 (of which `electricLight` true 304, **false 129**), near (0.5-1.5 m) 44, fixture 35.
  - Typical fill light: point, range 5, colour (0.49, 0.56, 0.65), y about -0.4 (cold tint, low); in the player extensions lvl 2-4 `(root)` / `Main`, NPC `Walls`/`BalconyBack`/`BalconyFront`, Greenhouse. These are room fill or fake daylight, not lamps.
    The balcony `Default/Point Light (n)` fills (range 5, same colour) have `electricLight = false`: the discriminator for daylight-style fill is the flag, not the position.
  - `Upgrade_STORE_Walls_(lvl 4) - Fine` (12 lights): 8 are fixtures (`Fluorescent_light_001 ...` 0.10-0.12 m, white, range 6, y 0.69), 1 is near `Ceiling_Shop` (1.2 m, range 14, tint 0.68/0.76/0.76, y -1), 3 are fill (range 14, same tint, y -1, only a `Quad`/`Cube` within 1 m).
    All 12 have `electricLight = true`.
  - Player extensions: lvl 1 root lights are fill 1.05-1.14 m from the wall lantern; lvl 2-4 `(root)`/`Main` lights are fill (1.65-5.3 m from the nearest lamp); the `lamp_009`, `lamp_008_toggle`, `ceiling_light_001_toggle*` children are fixtures (already under lamp parents).
  - `Upgrade_Basement_Walls_*` `Main` has one fill light 8.8 m from any lamp; the lanterns/lamps there are fixtures. `Upgrade_Basement_Heating_(lvl 0..2)` / `ShroomRoom` `Main`: range 0.4 green 0,1,0 lights (equipment indicator, `electricLight` true) and a few fill.
- `LightTurnOffOn` — switchable light with on/off materials (`turnLightOn/turnLightOff`, `on` is saved). `turnLightOff` swaps
  `lightMeshList[*].sharedMaterials[meshIndex]` to `lightOffMaterial` and deactivates `lights`; `turnLightOn` puts `LocalOnMaterial`
  (a copy of `lightOnMaterial`) back. Lamps without it have no "off" material: dim their emission.
  Since 0.5.13 it has a `noPower` gate: when set, `Awake`/`turnLightOn` call `DisableLightAndChangeMaterial` instead and `onTurnOn` fires only when powered; new members `TogglePower(bool)`, `SetEmissionMultiplier`, `RefreshMaterials`, `GetControlledLights`, `LightMeshes`; `PowerLightFlicker` ([power-grid.md](power-grid.md)) drives flicker through `SetEmissionMultiplier`.
- [verified] `LightTurnOffOn` swaps materials only on its `lightMeshList`. The Wall Light (`wall_lamp_001_toggle`) has two bulb meshes with
  `wall_lamp_001_on` (`Sphere.008`, `Sphere.011`); one stays lit when the lamp is off or unpowered. A mod can dim every glowing mesh under the lamp.
- [verified] Decoration light garlands (`Decoration Lights Medium(Clone)` / `decoration_lights_001`, also 002/003 variants) have no
  `LightController`, only an emissive mesh; animated ones carry `DecorationLights` (swaps `onMaterial`/`offMaterial` on `lightMeshes`
  via `.material` in `Update`; `StartPattern`/`StopPattern`, both end with all on). a light whitelist covers them.
- [verified] Turning only the `LightController` off leaves the shade glowing. Ceiling Light (furniture) has **no** switch component:
  `Ceiling Light(Clone)` [FurniturePlaceable] / `rotate` / `Lamp_005_ON` / `lamp_005 (n)` (MeshRenderer, MeshCollider).
- `LightController.Override(bool state)` and `disabled` — candidates for forcing a lamp off without touching saved state (check at runtime).
- Others: `CandleLight`, `LavaLamp`, `DecorationLights`, `GradientLamp`, `TimedLights`, `LightsController.controllersAll`.

## TV and radio

- `TelevisionNoise.PowerOff()/PowerOn()` — already has a "No Power" state with a UI message. Not a `CraftingBase`, no `PowerSource`:
  its own `powerOn` is **saved** (`[fsProperty]`), `isOn` is the switch. `TurnOff(playAudio, powerOff: true, force: true)` darkens it
  and keeps `isOn`; `TurnOn(bool, bool)` is the one path to "on" (`Start`, `Toggle`, `PowerOn`); `Refresh()` re-applies the look after
  loading. Mod guidance: block `TurnOn(bool,bool)` / `Interact`, show "No Power" in `UpdateUseMessage`, darken via `TurnOff(..., powerOff: true)`.
- `Radio` — no notion of power, only `RadioEnabled/CurrentState`.

## Generator

- [verified, runtime inspector] Not every `Generator` is the Mini Generator: the basement biogas has one too
  (`General/BuildingArea Basement/Biogas/compressor`, scene object, not furniture). The Mini Generator is furniture
  (`Mini Generator(Clone)` with `FurniturePlaceable`; the `Generator` sits on `mini_generator_001_LOD1`), `_Power.Type = Combustion`,
  needs fuel. Tell them apart by the `FurniturePlaceable` parent.
- [code] After loading or re-placing, a fuel device restores its burning state late: `CraftingBase.LoadDelay` (coroutine, a few frames,
  waits while `TimeOfDayAzure.updateTimeDisabled`) → `Fuel.LoadedGame()` → `_Power.UpdateState(true)`. No `OnPowerOn` fires.
  To react to "the generator runs again", postfix `Fuel.LoadedGame`.

`Generator : CraftingBase`. `nominalOutput` is gone; `Generator.powergeneration` (500, not saved) with `linkToGrid`/`showGridStatus` makes it a producer on a power grid ([power-grid.md](power-grid.md)); without `linkToGrid` nothing connects it to other devices (the scene may do it with relays).

## Devices with and without a power notion `[code]`

- No power notion at all (no field, no check): `Radio` (only `RadioEnabled/CurrentState`), `InteractableComputer`, `AlarmClock`, `Clock`, `Phone`, `DecorationLights`, `LavaLamp`, `GradientLamp` (an `on` switch and `Toggle(bool)` since 0.5.13, still off the grid), `TimedLights`, `CandleLight`. Anything a mod wants to do for them must be done from outside.
- `TelevisionNoise` — has the `No Power` state (see below).
- `Keypad.noPower` (protected bool, default false) — set only by `Keypad.ChangePowerState(bool state)` (`noPower = !state`); while true the use text is "No Power" ("Blocked" for `RotaryLock`) and `Use()` only plays the warning effect.
  Nothing in code calls it: it is driven from scene data (`Relay`/`output` events call methods by name, `Outputs.output.OnTrigger` / `outputCustomEvent`), i.e. by the game's own breaker/relay wiring.
- `Elevator.powered` (public, default true) — `Elevator.TogglePower(bool)` stops a moving cab, clears the queue, and on restore reopens the doors; no code caller, same relay wiring.
- `FloodGateControl.powerOff` (protected) — set by `TurnOn()` / `TurnOff()` (relay-called); off blocks the open/close buttons and makes the lights red/dark.
- `BatteryPowerSource : SavableScript` (needs a `Storage` on the same object) — drains `fuelDrainPerHour` from the storage's items (liquid / ammo / perishable meta per `fuelType`) over game time (`TimeOfDayAzure.SecondsPassed`), fires `RelaysON` / `RelaysOFF`; `powering` and `drainEnabled` are saved; `ToggleDrain(bool)` is relay-called. A separate mechanism from `PowerSource`.
- `NaturalLightSourceChecker.UpdateModifiers(bool _powered)` — the growing-bed "has sun" test is `powered || window natural light || no obstruction above (35 m ray)`; `_powered` is passed in by the bed when its lamp is on (see growing.md).
- `FurniturePower` was removed in 0.5.13; `PowerSource.furniturePower` is replaced by `powerGrid`.

- [data] Inventory of which furniture / built-in devices carry which of these components: not published here; regenerate with `electronics.py` + `electronics_report.py` (see [asset-bundles.md](asset-bundles.md)).
  Findings: of 654 Furniture assets, 94 carry a `_Power`-owning CraftingBase (manufacturing / manufacturingProcess / Growing / Generator / Dishwasher), 5 TelevisionNoise (Big Television, Flat Screen Television, Old Computer, Old Television, Table Fan), 2 Radio (Cheap Radio, Small Radio), 4 Clock/AlarmClock, 1 Phone, 1 Keypad (Safe), 4 BatteryPowerSource (Biogas Reactor, Brick Furnace, Charger, Gas Furnace), 6 refrigerated;
  no furniture has `InteractableComputer`, `FurnitureStats`, `DecorationLights`, `Elevator` or `FloodGateControl` (those, if present, are scene objects in other levels, not scanned beyond `level4`). Some `Combustion` devices have `PowerInput=True` and some `Electricity` devices `PowerInput=False` (Kitchen Tools, Meat Grinder): the flag, not the type, decides.
  Since 0.5.13 `FurnitureManager.NetPower`, `StatsObjects` and `TurnDeviceOn/Off` are gone (`FurnitureManager.powerGrid` replaces them); old saves carried the serialized `NetPower`, whether they still load cleanly is unverified.

## Losing power mid-work `[code]`

- `manufacturing.Craft()` refuses to craft and flashes the power icon (`FlashPowerIcon`) when `!PoweredAndActive`.
- `manufacturingProcess` (long processes) creates its `TimeOfDayAzure.Timer` with `stopped = true` when unpowered and refuses to
  start (`ProcessCanStart`) — a process **pauses**, it isn't lost. Whether a running timer is stopped on power loss goes through
  `OnSetPowerOff`/`PowerOutAndManuActive` — verify in game.
- Fuel running out: `Fuel.OnFuelConsumeStopped` → `active = false` → `OnStateChange(false)` → `PowerSource.FuelStateChange` →
  `OnPowerOff` → the owner's `OnPowerOut()` (e.g. `Generator.OnPowerOut`). Hook there to react to a generator stopping.
- Since 0.5.13 `Fuel` fires this stop path only when switched off while the counter runs; after a load `CraftingBase.TryStartFuelIfOn`/`StartFuelIfOn` restart the counter for a device that is on, not broken and not saved as running.

## Trait icons in station panels `[code]`

- Station panels show a row of trait icons: `ModifiersUI.UpdateModifiers(modifiers, quality, capacity, upgrades, manual, powerSource, generator)`,
  called from `ManufacturingPanel`, `ManufacturingProcessPanel`, `GrowingPanel`, `GeneratorPanel` with `manu.ModifiersWithUpgrades` and `manu._Power`.
  The `powerSource` argument is passed but **unused** (only the generator's fuel efficiency) — power display looks planned but unfinished.
- Each icon = `modifierPrefab` instantiated into `modifierList`, filled by `ModifierUI`: `UpdateModifierInfo(RecipeCondition)` (tooltip "Modifier: X"),
  or `UpdateModifierInfo(string title, Sprite)` for a custom one. Tooltip = `ToolTipInfo.UpdateToolTipInfo(title, details)`.
- Sprites: `ModifierSprites.instance` has one per condition, **including `Electricity`** (no recipe uses that condition, see items-recipes.md).
  Whether the sprite is actually assigned in the asset — check at runtime.
- Storage panel (fridges): `StoragePanelUI.UpdateStoragePanelUI` has a single `modifier` slot — snowflake "Refrigerator"/"Freezer"
  by `ToolTip.RefigerationFactorToString(RefigerationFactor)`; with `RefigerationFactor == 0` the icon is hidden.

## FurnitureStats (obsolete) `[code]`

Since 0.5.13 `FurnitureStats` is an empty `[Obsolete]` class (no members, no relays); references in old prefabs and saves load but do nothing. The budget it fed was replaced by the power grid ([power-grid.md](power-grid.md)).
Room membership without parenting: `FurnitureManager.ContainsFurnitureGameObject(go)` (searches `AllFurnitures`).

## How a device learns about power `[code]`

- `PowerSource.Powered` is `_Fuel.active` when `RequiresFuel`, else the saved flag `PowerInput` (default true). `CraftingBase.PoweredAndActive` = `AlwaysOn` or (`On` and `_Power.IsPowered()`). `AlwaysOn` devices ignore every power event and `Toggle`.
- Nothing polls power per frame. Events fire only when code calls `PowerSource.ToggleInput`, `FuelStateChange`, `TogglePower`/`Toggle`, `UpdateState` (after `LoadDelay`), `SetPowerSourceStatus` or `Init`. A changed `Powered` result does not notify the device by itself, except through the grid since 0.5.13: `PowerSource` implements `IPowerGridListener`, and `OnGridPowerOff/On` call `ToggleInput`, which fires the device hooks.
- `CraftingBase.Start` sets `_Power.ActOnPowered = PowerOn` and `ActonUnPowered = PowerOut`, so `PowerSource.OnPowerOn/OnPowerOff` reach the virtual `CraftingBase.OnPowerOn/OnPowerOut` (overridden by `manufacturing`, `manufacturingProcess`, `Generator`, `Growing`). `Start` also calls `OnPowerOut` for an unpowered device, so hooks there must be idempotent.
- `PowerSource.ToggleInput(on)` acts only for non-fuel sources: it fires `OnPowerOff` if it was powered, `OnPowerOn` if it was not, then sets `PowerInput`. `CraftingBase.SetPowerSourceStatus(on)` calls `ToggleInput(false)` when powered and `on == false`, otherwise writes `PowerInput` with no event. Nothing in code calls it; scene relays do (also `Keypad.ChangePowerState`, `Elevator.TogglePower`, `FloodGateControl.TurnOn/TurnOff`, `BatteryPowerSource.ToggleDrain`).
- `OnPowerOn`/`UpdateState` do not re-check the source; callers must.
- Mid-craft: `manufacturing` pauses its timer when `!_Power.IsPowered()` at load; `manufacturingProcess` refuses to start when unpowered.
- Tenement electricity: `TenementController.CheckElectricity` and `TenementResidentPower` now feed a power grid ([power-grid.md](power-grid.md)); the level still swaps visual prefabs (`TenementApartmentPrefabs.UpdateElectricity`) and `CharacterRent.Electricity` is rent data, not a device gate.
- Since 0.5.13 a grid exists in code (`PowerGrid`, `PowerMeter`, `PowerGridsController`); which scene objects carry them is asset data. `BlackoutTrigger` remains a player sleep/blackout event.

Mod guidance: to gate a device by room power, postfix `PowerSource.get_Powered` (electric, non-fuel only) or `CraftingBase.PoweredAndActive`, then call the public `CraftingBase.PowerOut()`/`PowerOn()` on each device after a change; `PowerSource.UpdateState` only runs relays and sounds, not the device hooks. Do not write `PowerInput` for a room effect: it is saved, and the flag (not `Type`) decides (some combustion devices have `true`, some electric ones `false`). Since 0.5.13 the game writes it too (`PowerSource.JoinGrid`/`SyncWithGrid` set it from the grid), so a flag you write is overwritten.
