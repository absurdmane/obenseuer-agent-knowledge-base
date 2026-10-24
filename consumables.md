# Consumables, smoking, alcohol, doctor
Scope: how consuming an item reaches the player stats, per-field mapping, auto-consume, drunk effect, doctor treatments. The stat side is in player-stats.md, mushroom extracts in items-recipes.md. Per-item values in `Items.json` were not checked.
Status: code-only
Related: [player-stats.md](player-stats.md), [item-system.md](item-system.md), [items-recipes.md](items-recipes.md), [economy.md](economy.md), [water.md](water.md)

## Where it lives [code]

- `OS.Items.ItemConsumable` (an `Item.ItemAction`) is the consumable. There is no class `ConsumableData`: the name is only the field `LiquidStorage.consumableData`, the `ItemConsumable` of a liquid container. `Item.ItemAction.Invoke` plays `UseSound`, then calls `Inventory.OnItemConsumed`, `Inventory.ItemSpecialTriggers` and `Inventory.CheckForAddedStolenItems`.
- `PlayerStats.UpdateValues(ItemData, ItemConsumable)` fans out to `health`, `alcohol`, `hungerAndThirst`, `mushrooms`, `tiredness`, `mentalHealth`, `addictions`, `hygiene` in that order. There is no smoking stat class: smoking lives in `PlayerStatsAddictions` (`SmokingAddiction`, `SmokingNeed`, base rate 40, change -1.25).
- Alcohol: `PlayerStatsAlcohol`; visuals `DrunkEffect`, `PlayerDrunkControls` (shared with mushroom effects). Doctor: `DoctorController`, `DoctorPanel`, `Doctor` (`treatments` list, `fsIgnore`, from scene or prefab), `DoctorPanelSelection` and `TreatmentPhysicalHealth`, `TreatmentMentalHealth`, `TreatmentAddictions`, `TreatmentFungalDisease`, `TreatmentRadiation`. Animations: `PlayerCameraAnimations.Eat/Drink`.

## How it works [code]

- `ItemConsumable.Invoke`: base invoke; takes the liquid's meta when the item holds liquid (quality comes from the liquid, not the container); `PlayerStats.UpdateValues`; removes the item when `DestroyAfterUse`; ammo handling; consumes one of `RequiredItems` (`CanUse` fails without them); `ItemsToSpawn` only when ammo is empty or absent.
- `Amount` is type-dependent: it feeds `Food` for Food, `Drink` for Drink, `Nicotine` for Smoke, `Hygiene` for Hygiene types; other types ignore it.
- Field to stat: `Food`/`Drink` to hunger and thirst (quality flag BaseStats); `Nicotine` to addictions (`SmokingNeed` falls by nicotine x 5 / (0.01 x addiction + 0.01), `SmokingAddiction` rises by nicotine x 0.125, cap 100); `Intoxication` to `alcohol` (a Drinking skill check only for the Drink type; scaled by `1 - Drinking x 0.08 x addiction%`, clamped 0.2 to 1; adds to `AlcoholDrunk`, lowers `AlcoholNeed`, adds alcohol x 0.125 to `AlcoholAddiction`); `Health` into the buffer `health.HealthToRegenerate` (food and drink get an SMV bonus or penalty above SMV progression 40); `Bleeding` into `health.Bleeding` (small negatives are ignored while bleeding is above 40); `Depression` to mental health (negative values scale with current status, perishable integrity below 0.7 weakens them, repeated use of the same item id is diminished through `usedItems`, stolen items add +0.25 x |v| + 0.85); `Hygiene`; `AlcoholAddiction`, `MushroomAddiction`, `SmokingAddiction` added directly; `SpecialEffectName`/`Amount` queued (player-stats.md). `Mode` is never read.
- `PlayerAutoConsume` handles Hunger and Thirst only, with a constant threshold 50: it takes Food/Drink items, prefers `IsSafe` then `IsHarmless`, sorts by `Item.Value` then integrity and calls `ItemInfo.UseItemDirectly`.
- `DrunkEffect.Update` starts when `AlcoholInBlood > 15` (strength `(blood - 15)/85`), stops while sleeping or in the wasteland effect, and calls `WaitingController.PassOut` above 100. Depression can trigger `DrinkRandomAlcoholFromInventory` (inventory category `"Alcohol"`) and `UseRandomMushroomsFromInventory` (`"Mushrooms"`).
- Doctor: `DoctorController.ShowMenu` opens `MenuPanels.Doctor`; `DoctorPanel` builds buttons from `Doctor.treatments`; each `DoctorPanelSelection` gives `GetStatus`, `GetAvailableTreatments` (title, price, currency) and `BuyTreatment`, which calls `Money.instance.Buy` before the subclass effect. Physical: painkillers and bandages as items added by hard-coded ids; "Heal wounds" is priced by health tier (Severe 80, Bad 50, Okay 20, AlmostHealthy 5) plus bleeding tiers, and sets `health.currentStatus = 100` and `bleeding = 0` (examination-bed teleport if health below 80 or bleeding above 20). Mental: antidepressants (tiers at 10/33/66), rubber duck for duck addiction. Addictions: only a prescription mushroom substitute when mushroom addiction is not 0. Hard-coded item ids used: 140010, 12851, 12670, 10940, 140020, 12852, 11, 15810.

## Mod guidance

Patch `PlayerStats.UpdateValues` (single entry for all consumption; a postfix still sees the item present) or the public getters `ItemConsumable.GetFood/GetDrink/GetAlcohol/GetNicotine/GetHealth/GetMentalHealt` instead of the fields (shared across instances). `PlayerAutoConsume.needThreshold` and the `DrunkEffect` thresholds are hard-coded. The `usedItems` diminishing returns are keyed by item id. `Item`'s copy of an `ItemConsumable` sets `hygiene` to 0.

## Unverified

Which items use the Drink/Eat/Smoke types and animations (prefab or item data); `Doctor.treatments` assignment; names of the hard-coded item ids.
