# SMV, sauna, radiation, duck, face protection
Scope: the stat parts not in player-stats.md: sauna overheating, radiation dose, the duck addiction, SMV progression and its wasteland effect, face-slot protection. The wasteland scene content and creatures (`Osrat`) are not covered; story-linked details are [REDACTED].
Status: code-only
Related: [player-stats.md](player-stats.md), [consumables.md](consumables.md), [skills.md](skills.md), [water.md](water.md), [items-recipes.md](items-recipes.md)

## Where it lives [code]

- Sauna: `PlayerStatsSauna` (`PlayerStats.sauna`: `Overheat`, `ApplySauna`, `ApplyCooling`, `UpdateStats`); the scene `Sauna` object calls `ApplySauna(this, delta)`; `SaunaCooling` (SavableScript trigger) calls `ApplyCooling` once per second while `On` and the player is inside.
- Duck: `PlayerStatsDuck` (`PlayerStats.duck`, updates every second). Radiation: `PlayerStatsRadiation` (field `Dose`), `RadiationController` (`instance`, `MeasureRadiation`, `GetBackgroundRadiation`), sources `point_radiation` and `TriggerRadiation`; readers `Dosimeter`, `TreatmentRadiation`.
- SMV: there is no SMV class; the fields are on `PlayerStatsMushrooms` (`SMVProgression` 0..100, `SMVProgressionRate` per in-game hour, may be negative). The wasteland effect is `SMVEffects` (`instance`) with `WastelandTrigger`, `WastelandGlow`, `LeakSphere`. Protection: `FaceSlotController` (slot index 3).

## Sauna [code]

- `Overheat` rises when `Sauna.temperature` is above `OverheatThresholdTemp` (default 50): temperature/threshold plus `löyly` x `Overheatrate`. Only non-passive saunas give Sauna skill. Above `OverheatThreshold` + 5 per skill level above 1 the player takes Fire damage per tick; above 100, `UpdateStats` deals Fire damage each minute (reduced about 5% per level, minimum 0.1). `Overheat` cools 0.5 per minute; `ClearAllNeeds` zeroes it.
- Per tick in a sauna (scaled by `löyly` and a skill factor of 0.1 to 1): lowers `AlcoholInBlood`, `AlcoholAddiction`, `SmokingAddiction`, adds thirst, lowers mental status and target, improves hygiene, lowers `SMVProgressionRate` by `ReduceSMV`. `ApplySauna` sets the rate to -0.25 whenever it ends below 0.25, so a sauna turns any positive rate into -0.25 in one tick.
- `SaunaCooling` (`coolingRatePerMinute` 33, `ImproveHygienePerMinute` 25) cools `Overheat`, improves hygiene and can add bleeding, damage, SMV, rate, highs and alcohol per minute; `InteractableShower` sets these fields.

## Radiation [code]

- Each game minute `UpdateStats` adds `MeasureRadiation(playerHead)/60` to `Dose` and fades 10 per hour (`DoseFadeout`). Above `DoseDamageThreshold` (1000), severity = (Dose - 1000)/(100000 - 1000); each minute it deals silent Radiation damage of severity x 100/60, lowers `SMVProgressionRate` by severity x 4.25/60 and applies `highDoseConditions`.
- `MeasureRadiation` = background (0.0004, or 15 while the wasteland is active) + each active `point_radiation` (1/d^2, zero beyond `maxRange`, reduced by `wallBlockFactor`) + `TriggerRadiation` values for non-player objects. `TriggerRadiation` (per hour) adds radiation/3600 to `Dose` per second while the player is inside; `dontDamagePlayer` caps `Dose` at the threshold. The item special effect `Radiation` adds to `Dose` (at most 0.25 per tick); `ClearAllNeeds` zeroes it. In the wasteland the background (15/h) exceeds the fade (10/h): `Dose` climbs about 5 per hour (derived).

## Duck [code]

Active only while the player holds a task item (story-linked, [REDACTED]); losing it clears the status. Need comes from the count of `requiredInventoryItem` (floor 0.5, cap 20, grows 1e-5 per second). With enough items `currentStatus` follows count/need; without, it grows by need/count x 0.005 (0..100) and above `negativeEffects` (60) mental status rises by status/10000 per tick. Achievements: `DUCK_ADDICTION`, `DUCK_ADDICTION_CURED`, `ALL_ADDICTIONS_CURED`.

## SMV [code]

- Growth: `ItemConsumable.GetSMV` times (1 - SMV/100)^2 when the item value is under 100, `GetSMVRate` likewise (an item with SMV but no rate pulls the rate toward 10); mushroom damage in `PlayerStatsHealth.Damage` (+0.01 x damage to SMV and +0.001 x damage to the rate on a roll of at most 30 out of 0..100, so about 31%); hygiene (`PlayerStatsHygiene`), negative effects while waiting (`InteractableWait`), shower cooling, item special effects.
- Reduction: sauna, radiation, negative item values, a negative rate, and `ChangeSMVProgression` (a negative result resets both to 0 and triggers `SMV_CURED`). Use `ChangeSMVProgression`/`ChangeSMVProgressionRate`; direct writes skip the cure achievement and the diminishing factor.
- Each game minute `PlayerStatsMushrooms.UpdateStats` adds `GetSmvProgressionRateSpeed(rate/60)` when the rate is not 0; above 100 it caps at 100 and calls `PlayerStats.death(DamageTypeNames.SMV)` (skipped by `Godmode`); health is capped at 100 - SMV; particles start above 60.
- Effects: mental status worsens above SMV 50 (x (SMV-50)/50); `GetSkillWithEffects` x (1 - SMV/200); consumable mushroom highs change above 40; bowel and bladder output becomes the SMV variant with probability SMV% from SMV 5.
- Wasteland: `SMVEffects.Update` starts it when `PlayerHasSMV` is true, no dialogue is active and no timer runs (`probability` below 100 enables a timer); `StopWasteland` when it is false. `IsWastelandIsActive` is read by mushroom damage (ignored entirely), mushroom effects, background radiation, `InteractableTalk` talk restrictions, `TriggerMultiple.dontTriggerInWasteland` and sound. `WastelandTrigger`, `LeakSphere`, `WastelandGlow` call `EnableWasteland`/`StopWasteland`. Nothing assigns `PlayerHasSMV` in code (scene glue).

## Face protection [code]

`FaceSlotController.HasMushroomProtection`: item 15420 with `integrity` above 0, or item 17110 with `ammo` above 0; it blocks mushroom damage. `HasGasProtection`: item 17110 with `ammo` above 0; it blocks gas damage. `HasFacemaskEquipped` (15420) only silences the mushroom-damage message. `OnDropped`/`OnTaken` show or hide the mask visuals (17110, 17130, 17380); `OnTaken` does not hide `gasMaskAudio`.

## Unverified

Names of items 15420, 17110, 17130, 17380 (check `Items.json`); whether gas-mask ammo is consumed by blocked damage; usage of `SMVEffects.MinSMVProgression` (50) and `MinSMVDeathAlert` (90); duck tuning values in the asset.
