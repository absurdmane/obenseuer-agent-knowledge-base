# Fuel, burning, candles, sauna heat, heating levels
Scope: the `Fuel` class (burn time, ash), `ItemFueldata`, the candle lantern, sauna temperature, what the heating upgrade level does, weather temperature. Generator and `PowerSource` basics are in power.md. Which devices burn fuel is prefab data (not in code); train fuel tanks are not covered.
Status: code-only (fuel item values are `[data]`)
Related: [power.md](power.md), [crafting.md](crafting.md), [tenement-upgrades.md](tenement-upgrades.md), [smv-sauna-radiation.md](smv-sauna-radiation.md), [time-and-weather.md](time-and-weather.md)

## Where it lives [code]

- `Fuel` (burn counter, fuel slot, refuse slot), used through `PowerSource._Fuel` (`RequiresFuel`) by `CraftingBase` and `Generator`. `ItemFueldata` (item meta): `burntime` (default 3600), `burntItem` (refuse item), `burntItemAmount` (float, default 1). `Lantern` (candle timer; no `Fuel`). `CandleLight`, `ExtinguishableFire`: cosmetic only. Heating levels: `TenementGeneral.currentHeating`, `TenementApartment.currentHeating`; prefab visuals in `TenementGeneralPrefabs.UpdateHeating` and `TenementApartmentPrefabs.UpdateHeating`. `Sauna` (`On`, `TurnOn/TurnOff`, `temperature`).

## How it works [code]

- Burn time is in game seconds: `Fuel.DeltaSeconds` (on `TimeOfDayAzure.SecondsPassed` while burning) subtracts elapsed seconds from `BurnTimeRemaining` (floor 0). `Fuel.Consume` adds `burntime x amount x BurnTimeFactor` per `ItemFueldata` in the stack (60 s per item without fuel meta); the whole stack is removed at once. Liquid fuel (`fuelIsLiquid`, "Liquid Container" category, liquid in `FuelCategories`) uses the liquid's `ItemFueldata` x liquid amount, else 60 s x amount. `Consume` does nothing while `BurnTimeRemaining > MaxBurnTime` (43200 s). `CraftingBase` sets `BurnTimeFactor = EquipmentQualityWithUpgrades` (the "Fuel efficiency" modifier).
- Ash is carried as a fraction per refuse item id (`refuseRemainders`, saved), so fractions add up across burns: `Fuel.AddRefuse` queues `floor(amount x burntItemAmount + remainder)` items in `RefuseToSpawn` (0.5.13; earlier builds used `Mathf.RoundToInt` per stack, which gave no ash for small stacks). `DeltaSeconds` releases them into the refuse slot in proportion to the burn (`FloorToInt((1 - BurnTimeRemaining/peakBurnTime) x peakRefuseAmount + 0.5)`), `SpawnRefuseAll` dumps the rest when the fire goes out. `[data]` At least one fuel item has `burntItem` Ash (ID 10010), `burntItemAmount` 0.1, `burntime` 1000, so it now yields one ash per ten items burned; a toy item has `burntime` 1200.
- Start and stop: `Fuel.Toggle(on)` starts the counter when there is burn time or a fuel item; `AddFuel` consumes only while active (items dropped into an idle device are consumed at `StartFuelCounter`); `OnFuelConsumeStopped` fires `onFuelConsumeStopped`, which `CraftingBase` maps to `OnSetPowerOff`; `PowerSource.FuelStateChange` maps on/off to power on/off. `CraftingBase` sets `RequiresFuel = false` for `AlwaysOn` devices. `Fuel.OnSavingGame` stores the time and `LoadedGame` adds the offline delta (negative sets `fuelOutTime`); `RefuseToSpawn` is serialized.
- Since 0.5.13 the stop path fires only when the device is switched off while the counter runs, and after a load `CraftingBase.TryStartFuelIfOn`/`StartFuelIfOn` restart the counter for a device that is on, not broken (`CraftingBase.Break()`, `broken`) and not saved as running; `CraftingBase.Toggle` refuses `on` while broken.
- `Lantern.Interact`: extinguishes a lit candle, otherwise takes `candleItem` from the inventory (`CandleInserted`). Burn time = `overrideCandleTime` if >= 0, else the candle's `burntime`, else 1000 s; a saved `TimeOfDayAzure.Timer`; on burn-out `CandleExtinguished` and `onCandleBurntOut`. No ash, no refuel.
- Sauna: `TurnOn/TurnOff` set `On` and subscribe to the clock; temperature moves toward 100 or 16 (ambient) at fixed rates (warm-up 3.33 per tick); working above 60; overheat is in smv-sauna-radiation.md. Nothing in this path reads fuel or power; no caller of `TurnOn` was found in code.
- Heating level has no warmth effect in code: it is a resident requirement (`TenementResident.heatingMin` against `TenementApartment.currentHeating`), counts in `TenementController.ResidentApartmentLevel` and the building level, and is capped by `TenementContractor.maxTenementHeatingUpgrade`. Nothing reads it for the player's body or room temperature, and no body-temperature or fire-damage code was found. Weather temperature (`WeatherManager.Temperature()`, -20..+20) reaches scenes only through `logic_weather_listener.onTemperatureUpdated`.

## Mod guidance

Burn rate: `Fuel.DeltaSeconds`; fuel value per item: `Fuel.Consume` (`BurnTimeFactor`, `MaxBurnTime` are public fields). Candle duration: `Lantern.CandleInserted`. A cold or heat mechanic needs a new hook.

## Unverified

Which devices are fuel-fired and their `FuelCategories` (prefab data); what the `BuildingSystem` check on fuel and refuse slots (around its furniture-take code) blocks; whether `SecondsPassed` seconds match the 3600 s per hour tooltip.
