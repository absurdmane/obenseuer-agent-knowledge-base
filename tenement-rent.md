# Tenement: renting, residents, eviction, contractor
Scope: the rent loop, per-apartment income, moving residents in and out, requirement checks, contractor caps, prefab swapping. Upgrade levels are in tenement-upgrades.md, rent schedule maths and collateral in economy.md, resident NPC behaviour in npc-schedules.md. The bodies of the per-category prefab updates, resident events and upgrade-progress internals are not covered.
Status: code-only
Related: [tenement-upgrades.md](tenement-upgrades.md), [economy.md](economy.md), [npc-schedules.md](npc-schedules.md), [waste-junk.md](waste-junk.md), [power.md](power.md)

## Where it lives [code]

- `TenementController` (SavableScript, singleton): rent loop, upgrade progress, evictions and moves, residents list. `TenementApartment` (per apartment): `resident`, `deposit`, `moveStarted`/`moveSecondsLeft`, `evictionStarted`/`evictionSecondsLeft`, `onlyFurniture`, income getters. `TenementResident` (ScriptableObject): `incomeOc/incomeRm`, `otherResidents`, `disableEvictButton`, requirement fields, `CheckRequirements`. `TenementContractor` (ScriptableObject): upgrade caps, `CanUpgrade`. UI: `ApartmentRentUI`, `ApartmentInfoUI`. Prefab swapping: `TenementApartmentPrefabs`.

## How it works [code]

- Tick: `TenementController.DeltaSeconds` runs `UpdateRentPayment`, `UpdateProgesses` and `UpdateEvictions` inside one try/catch, so an exception in the rent step silently skips the other two for that tick.
- Since 0.5.13 move-in and eviction completion (and any upgrade completion) also call `UpdateResidentPower`, so the apartment's draw on the tenement power grid changes with each move ([power-grid.md](power-grid.md)).
- Rent collection: `UpdateRentPayment` returns with no residents; when `DifficultyInfo.GetRentPayment(lastPayment)` is true it sums `TenementApartment.GetOcIncome()/GetRmIncome()`, calls `Money.AddMoney` directly (no invoice) and shows "Rent paid". Per-apartment raw income is 0 unless there is a resident, `!moveStarted`, `!onlyFurniture` and `incomeOc != 0`; otherwise `incomeOc + GetValue() / ocDivider` through `DifficultyInfo.GetRentIncome`, divided by the payments per period. `TenementResident.GetMonthlyRent` = base income + `apartment.GetScaledValue()` (used for collateral).
- Renting: `ApartmentRentUI.OnRentButtonPressed` -> `TenementController.RentApartment(resident, apartment, 18f)`: evicts any current resident, sets `moveStarted` with `moveSecondsLeft = hours x 3600`, sets `deposit` from the collateral and charges it (unless silent); non-silent rents also fire achievements and `RentFade` (5 s real time). When `moveSecondsLeft <= 0`, `UpdateEvictions` calls `MoveResident`: clears `moveStarted`, runs `TenementApartmentPrefabs.UpdateApartmentPrefabs`, notifies and adds the resident's `tenementEvents`; while moving, `UpdateFurniture` destroys the flat's furniture.
- Eviction: `ApartmentRentUI.OnEvictButtonPressed` -> `StartEviction(apartment, 18f)`; `CancelEviction` aborts. At zero `EvictResident` (default flags) removes events, returns the deposit if `Money.CheckMoney` allows, otherwise creates a 14-day invoice "Rent collateral: <name>", removes the resident. Flags: `updatePrefabs` (destroys the NPC object, rebuilds visuals) and `leaveFurniture` (keeps the flat as `onlyFurniture = true`, resident kept in `evictedResidents`).
- Rent button: apartment number 1 shows "Your apartment!" (`CantRentReason.PlayerApartment`); otherwise `TenementResident.CheckRequirements` decides the label (`Size` "Wrong Size!", `Resident` "Already Occupied!", `Apartment`/`Tenement` "needs upgrades"). The evict button is hidden when `disableEvictButton` or `onlyFurniture`.
- `CheckRequirements` order: size flags (small, medium, large) against the apartment type; occupied (skipped for `onlyFurniture`); apartment minimums (electricity, water, heating, bathroom, walls, balcony, kitchen); building minimums (elevator, stairs, basement, basement sauna, outer walls, courtyard, mine). Building electricity, water, heating and roof are not checked. The first failing reason is returned.
- `TenementContractor.CanUpgrade(upgrade, apartmentType)` compares each building category with its `max*Upgrade` cap; shop apartments use `maxShopUpgrade`, other apartments `maxApartmentUpgrade`. `StartUpgrade` checks renovation objectives unless `silentUpgrade`; `UpdateProgesses` decrements `TimeSecondsLeft` and calls `OnUpgradeDone` at zero.
- Prefab swap: `UpdateApartmentPrefabs` runs furniture, then electricity, water, heating, walls, bathroom, balcony, balcony 2, kitchen, then mail location. Each takes the resident-specific prefab first (`TenementResidentPrefabs`), else the level prefab; an object is replaced only when the name differs.

## Mod guidance

Patch `TenementResident.CheckRequirements` (plain method) to change who can move in; patch `GetOcIncome/GetRmIncome` or `UpdateRentPayment` for rent changes (rent is credited at once). If you rewrite `moveSecondsLeft`/`evictionSecondsLeft`, also reset `moveStarted`/`evictionStarted`. `EvictResident` called with defaults from code differs from the UI's silent rent path (no deposit handling there).

## Unverified

Which event drives `TenementController.DeltaSeconds`; whether `deposit` and `onlyFurniture` are saved; meaning of `evictedResidents` after `leaveFurniture`; no JSON data (resident links, rent terms) was read.
