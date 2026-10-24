# Tenement (building) and apartment upgrades

Scope: Where water/electricity/heating levels are stored, `UpgradeType` enums, how an upgrade works, shop opening, the player apartment number.
Status: code-only; items marked [verified] were seen in game
Related: [power.md](power.md), [saves.md](saves.md), [furniture-areas.md](furniture-areas.md), [levels.md](levels.md)

## Where the state lives

- `TenementController.instance` — singleton, saved (`SavableScript`).
  - `.general : TenementGeneral` — building-wide upgrades.
  - `.apartments : List<TenementApartment>`; `GetApartmentFromNumber(int)`.
- Floor: `TenementApartment.apartmentFloor` (saved). Floor 1 holds the player apartment (Large) and the shop; the game's floor
  screen (`FloorUIManager`) shows one Small/Medium/Large apartment per other floor.
- **The player's apartment is `apartmentNumber == 1`**, the shop is `2` (hardcoded in `ApartmentRentUI`, `TenementUpgrade.GetName`).
  Status `UpgradeType.ApartmentStatus.Player` exists but is unused in code.

## Levels (enums in `UpgradeType`)

| What | Building (`TenementGeneral`) | Apartment (`TenementApartment`) |
|---|---|---|
| Electricity | `currentElectricity : TenementElectricity {None, Shit, Good, Best}` | `currentElectricity : ApartmentElectricity {Off, On}` |
| Water | `currentWater : TenementWater {None, Dirty, Good, Best}` | `currentWater : ApartmentWater {Off, Dirty, Good, Best}` |
| Heating | `currentHeating : TenementHeating {None, Bare, Central, District}` | `currentHeating : ApartmentHeatingMethod {None, Makeshift..Luxurious}` |

Other building categories: Stairs, Elevator, Facade, Roof, Basement, Sauna, Shrooms, Workshop, Mine, Courtyard, Fence (`currententFence` — typo in the game), SaunaAndPier, Greenhouse (`currententGreenhouse`).
Apartment: Bathroom, Wall, Balcony, Balcony2, Kitchen.

Enums `PlayerApartmentElectricity/Water/...` and `UpgradeTarget.PlayerApartment` are declared but unused — the player has a regular `TenementApartment`.

Handy getters: `TenementController.GetGeneralUpgradeLevel(GeneralUpgradeCategory)`, `GetApartmentUpgradeLevelByNumber(int, ApartmentUpgradeCategory)` → int.

## How an upgrade works

- `TenementUpgrade` (ScriptableObject): a set of `upgradeX` flags + target level `xType`, price, time, `requiredUpgrades`.
  `IsUpgraded(apartment)`, `CanUpgrade(...)`, `GetTier()`.
- Apartment upgrade lines: `TenementApartmentType.{electricity, water, heating, ...} : TenementUpgradeGroup.updateGroup[]`.
- Flow: `TenementApartment.StartUpdate` → `TenementUpgradeProgress` → on completion `TenementApartment.Update(upgrade)` / `TenementGeneral.Update(...)` sets `currentX`.
  "Upgrade finished" hook: `TenementController.OnUpgradeDone(TenementUpgradeProgress)` (private).
  It only marks the progress ready and adds the "Complete the final payment" objective. The level is applied after that payment.
- **Applying an upgrade directly** (cheats): `TenementGeneral.Update(upgrade)` / `TenementApartment.Update(upgrade)` sets `currentX`,
  `currentXUpgrade`, clears `xProgress` and calls `prefabs?.UpdateX(...)` (rebuilds the visuals). Building lines:
  `TenementGeneral.<x>Upgrades : TenementUpgradeGroup` (`updateGroup[]`). Prerequisites: `TenementUpgrade.requiredUpgrades`,
  checked by `CanUpgrade` via `IsUpgraded(apartment)`.
- `requiredUpgrades` is the **only** link between upgrades: `TenementGeneral.Update` / `TenementApartment.Update` don't touch other
  lines, and the shop UI (`UpgradePanel.IsLocked`) adds only contractor skill, "inferior upgrade" (no downgrades) and progress checks.
  Higher tiers can be bought directly (no "previous tier first"). The game never downgrades, so it has no removal cascade.
- Dev flags: `TenementController.ignoreUpgradePricesAndItems`, `completeUpgradesInstantly` (console commands of the same names).
- Visuals per level: `TenementApartmentPrefabs` / `TenementGeneralPrefabs` destroy and instantiate a prefab for the current level
  (`TenementApartmentPrefabsContent.GetWater` → `waterShit/waterGood/waterBest`, `null` for `Off`).
  What these prefabs contain (taps? pipes?) — mostly **unknown**, needs a runtime dump.
- Building electricity Shit [verified, runtime inspector]: the spawned prefab is
  `General/UPGRADES_General/Upgrade_Electricity_(lvl 1) - Shit(Clone)` (= `content.currentElectricity`). The basement panel is
  `.../Upgrade_Basement_Electricity_(lvl 1) - Shit/Main/Main/smallelectricalbox_012`, layer Default, hit by the crosshair raycast
  (`InteractObjects.currentGameObject`). It is not furniture (no `FurnitureManager`). Good/Best: not checked.

## The shop (#2) opening — the cleaning timer

- [verified] The entrance is blocked by `Apartments/Apartment 2/Detail bare walls Store/Start Blocker` (plywood planks,
  `Upgrade_NPC_MEDIUM_FrontDoor_(lvl 0) - QuarantineSeal`). It is the shop's `TenementApartmentPrefabsContent.cleaningTimerBlocker`
  (found with a reference scan). Walls (any level) and the `upgradeShopToMakeshift` objective don't remove it.
- [code] `TenementApartment.cleaningTimerCompleteDay` (saved): `-1` = not started, `> 0` = cleaning until that day (blocker hidden,
  `cleaningTimerIndicator` shown), `0` = done (blocker destroyed). Applied by `TenementApartmentPrefabs.UpdateCleaningTimerState`.
- [code] Started only from a dialogue: `TenementController.StartApartmentCleaningTimer(apartmentNumber, days)` is registered as a Lua
  function for the Dialogue System (only when the value is -1). `OnCleaningTimerDayChanged` sets 0 when the day comes.
  Which conversation starts it for the shop: not in the code (dialogue database).
- The blocker is part of the scene: after it's destroyed, setting the value back to -1 and reloading brings it back.

## Stairwell upgrades and electricity

- [verified, in game] Every stairwell upgrade after the first needs building electricity **Shit**. Yet the saved copies of the stairs
  upgrades in `Globals.tnmt` show `requiredUpgrades: []` — the requirement must be checked elsewhere (not found in code yet).
- So the stairwell's built-in lamps (Decent: `lamp_001 (*)`, `lamp_006_ON (*)`) only exist with the grid on. They still matter for
  outages (Shit). The `lamp_001` lights are marked `electricLight = false` in the prefab (see power.md).

## Where the game already uses the levels

- `InteractableWaitResident` [code, looks unused]: a "Wait" interaction that would move the **player's** hunger/thirst/hygiene/bladder
  toward targets from a tenant apartment's kitchen/water/bathroom levels. Nothing in code creates or calls it, and the author hasn't
  seen such an action in game (2026-10-16): likely a dormant feature. Residents have no needs logic of their own in code.
- Minor game bug: `TenementUpgrade.IsUpgraded` for apartment heating compares against `heatingType` (building) instead of `heatingMethod`. [verified] Shows apartment heating as already done; a mod should compare `heatingMethod` itself.

## Electricity drives a power grid (0.5.13) `[code]`

Building electricity now sets the capacity of the tenement's main power grid and every resident apartment is a consumer on it (`TenementController.CheckElectricity`, `TenementResidentPower`, `UpdateUtilities`); the level still also swaps visual prefabs. Watts per level, the subscription, and the per-apartment draw are in [power-grid.md](power-grid.md). `DevTestStatus` now also starts the last electricity and basement upgrade groups.
