# Player stats and needs
Scope: `PlayerStats` and its parts: hunger, thirst, bowel, bladder, health, mental health, hygiene, alcohol, addictions, mushrooms, conditions, skills. Tiredness and sleep are in the sleep note; sauna, duck, radiation and gambling parts are only named.
Status: code-only
Related: [player-movement-tiredness-sleep.md](player-movement-tiredness-sleep.md), [combat-npcs.md](combat-npcs.md), [items-recipes.md](items-recipes.md), [water.md](water.md), [time-and-weather.md](time-and-weather.md)

## Where it lives

- `PlayerStats` (`PlayerStats.instance`) holds the parts as public fields: `health`, `mushrooms`, `mentalHealth`, `alcohol`, `sauna`, `addictions`, `hungerAndThirst`, `tiredness`, `duck`, `hygiene`, `radiation`, `gambling`, `conditions`. Skills: `Skills` array of `PlayerSkill`, enum `PlayerStats.PlayerSkillType` (about 25 entries). One class per part (`PlayerStatsHealth`, `PlayerStatsHungerAndThirst`, ...). Conditions: `PlayerStatsDiseasesAndConditions` (`activeConditions`) and `BaseHealthCondition` ScriptableObjects. Difficulty: `DifficultyInfo.GetGeneralNeedsSpeed`, `GetSmvProgressionRateSpeed`, `mentalHealthDisabled`. [code]

## How it works [code]

- Tick: `PlayerStats.DeltaSeconds` (private, driven by the seconds event) runs only when `Active && !StatsProgressDisabled`. Once `LocalSeconds` passes 59 it calls `UpdateStats` on health, mushrooms, mental health, hunger/thirst, tiredness, addictions, hygiene, radiation and conditions (so these change once per game minute). Alcohol, duck, sauna and special-effect processing run on every call.
- Consumption: `PlayerStats.UpdateValues(ItemConsumable)` fans out to each part (skipped when `StatsProgressDisabled`). Special effects queue in `unprocessedSpecialEffects` and drain in `ProcessItemSpecialEffects` (at most 0.25 per tick or 0.25% of the remainder): Laxative to Bowel, Radiation to `radiation.Dose`, Coffee and SleepingPills to `tirednessToAdd`.
- Hunger and thirst: base 10/h and 15/h scaled by `GetGeneralNeedsSpeed`; hunger rate is reduced by mushroom high and alcohol addiction. Bowel fills at 0.66 x hunger rate, bladder at 0.75 x thirst rate. Sleeping multiplies rates by 0.33. At 100 hunger or thirst health takes 25/h. Bowel above 100 calls `Defecation`; bladder above 100 puts urine in a liquid storage when possible (mental target +10, hygiene -20), otherwise mental target +33, hygiene -50.
- Health: `currentStatus` 0..100 with a buffer `HealthToRegenerate` (1 per minute tick). `bleeding` 0..100 decays and drains health at 0.1 x bleeding per tick. `Damage` routes Bleed into `bleeding` and everything else into `currentStatus`. Mushroom damage has a 30% chance to add SMV progression and is ignored with mushroom protection, the wasteland effect or SMV above 50; gas is ignored with gas protection. Death goes through `PlayerStats.death` (checks `Godmode`, then `Death.PlayerDeath`).
- Mental health: higher is worse, 0..100, moves toward `mentalHealthTarget` at 0.5/h. `effectiveMentalHealth` = status / (1 + 0.1 x alcohol in blood) + hygiene effect; at 100 or more a random depression event fires and status and target drop by 20. Asleep status drops 0.5/h; awake, pressure comes from health, tiredness, needs, addiction needs and mushroom high. `mentalHealthDisabled` forces 0.
- Hygiene: higher is better (field default 75, `ResetStats` 100). Awake -2/h, asleep -1/h. Under 50 an SMV chance grows; at 10 or less flies appear and `lowHygieneConditions` apply. Mental effect = -(hygiene x 0.02 - 1) x 10 - `expectationOffset`.
- Alcohol: `AlcoholDrunk` moves into `AlcoholInBlood` at 2.5/min; blood alcohol burns 0.5/min.
- Addictions (alcohol, mushroom, smoking, gambling): addiction falls 1.25/h (gambling 0.5/h); need grows by addiction x base rate (40/h, gambling 6/h); withdrawal adds 50 x addiction% per hour to mental status; consuming lowers need and raises addiction.
- Mushrooms: `MushroomsHigh` (and orange variants) absorbs `MushroomsUsed` at 5/min, decays 1/h. `SMVProgression` 0..100 grows with `SMVProgressionRate`; at 100 it kills the player; health is capped at 100 - SMV.
- Conditions: `BaseHealthCondition.ApplyCondition` rolls probability x factor, sets `strength` in `strengthRange`; non-stacking conditions add to the existing entry, stacking ones add the asset itself. `UpdateStats` changes strength hourly and removes it at 0 if `autoremove`. It writes `strength` on the shared asset before the duplicate check.
- Skills: `GetSkill` = floor of the skill (1 if missing); `IncreaseSkill` caps at `PlayerStats.maxLevel` (10); `GetSkillWithEffects` = skill + high/10, x (1 - SMV/200), clamped 0..15.
- Saved by FullSerializer (`OnSavingGame`/`OnLoadingGame`); `ResetStats` clears all parts, `ClearAllNeeds` sets health and hygiene to 100 and zeroes needs, tiredness and sleep debt.

## Mod guidance

Patch the public rate getters `PlayerStatsHungerAndThirst.GetHungerRate/GetThirstRate` and `PlayerStatsHygiene.GetHygieneRate` instead of fields (saved). `PlayerStats.UpdateValues` is the consumption entry point. `DeltaSeconds`, `ProcessItemSpecialEffects` and `UpdateStats` are private. Check `PlayerStats.death` before adding death triggers.

## Unverified

Whether `SkillQuality` multiplies by blood alcohol as read; whether addiction needs pause while asleep; default values of the difficulty multipliers; which conditions exist (asset data).
