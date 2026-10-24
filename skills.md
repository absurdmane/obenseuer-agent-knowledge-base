# Skills: gain, use, UI
Scope: the player skill system: enum, storage, the shared gain formula, where each skill is gained and used, the character panel. The stat parts are in player-stats.md, recipe skill data in items-recipes.md.
Status: code-only
Related: [player-stats.md](player-stats.md), [crafting.md](crafting.md), [police-crime.md](police-crime.md), [items-recipes.md](items-recipes.md), [consumables.md](consumables.md)

## Where it lives [code]

- `PlayerStats`: enum `PlayerSkillType` (0..24: None, Lockpicking, Drinking, Cooking, Mashing, Brewing, Distilling, Farming, Malting, Manufacturing, Paperwork, Chemistry, Carpentry, Machining, Engineering, Bribery, Cleaning, Sewing, Electronics, Mining, Music, Smithing, Medical, Computers, Sauna), class `PlayerSkill` (`Type`, `Skill` float; `Level` is its floor), saved array `Skills` (one entry per type, each starting at 1), `static int maxLevel = 10` (not saved), `GetSkill`, `GetSkillWithEffects`, `IncreaseSkill`, `SkillSuccess`, `SkillQuality`, `TriggerSkillAchievement` (no case for Medical or Computers).
- UI: `CharacterPanel` (one `CharacterPanelSkill` per entry; shows only skills above 1, else "No learned skills"), `CharacterPanelSkill` (name, "LVL n", XP = floor(Skill x 1000), a bar of `maxLevel` segments), `PlayerSkillIncreaseUI` (popup, fades after 8 s), `RecipeInfoUI` reads `GetSkill`.

## Formulas [code]

- `PlayerStats.SkillSuccess(type, failChance, dontIncrease, factor, crafttime)`: `value` = crafttime/1800 when given, else 1, clamped 0.05 to 5; success gain = value x factor x 0.1 x ff, failure gain = value x factor x 0.05 x ff, with ff = failChance (0.01 when 0). Every gain funnels into `PlayerStats.IncreaseSkill(type, amount, notification)`: false for None or at `maxLevel`, amount clamped to 0..`maxLevel`, popup when the integer level changes, achievement at max. Hook `IncreaseSkill` to catch all gains.
- `PlayerStats.SkillQuality` = `GetSkillWithEffects` (0.5.13; earlier `GetSkill`) x 0.2 x recipe skill factor x sobriety x (1 - fail) x equipment x quality factor.

## Gain and use per skill [code]

- Crafting skills (Cooking, Mashing, Brewing, Distilling, Malting, Manufacturing, Paperwork, Chemistry, Carpentry, Machining, Engineering, Cleaning, Sewing, Electronics, Mining, Smithing, Medical, Computers): gained in `CraftingBase` (+0.1 x `SkillGainFactor` x recipe `SkillGainFactor` x multiples x max(fail chance, 0.01)); used in `Recipe.GetProcessTime` (multiplied by `MinimumSkill - skill + 1` below the minimum), `Recipe.CurrentFailChance` and `SkillQuality`. The recipe's `RelevantSkill` is data (items-recipes.md).
- Lockpicking: `PlayerLockpicking` coroutine (success +v, failure +0.2v, v = 1/(c^2 + 35c) clamped 0.01..1 with c the lockpick chance); feeds `LockpickChance`. Drinking: `PlayerStatsAlcohol.UpdateValues` (Drink items only) and `SetAlcohol` (alcohol x (1 - Drinking x 0.08 x addiction%), minimum 0.2). Farming: planting in `Growing` (0.001 + 0.003 x fail chance, times factor and amount), planting fail chance, harvest quality. Bribery: `Crime` bribe check (base fail 0.7) and `Crime.CalculateBribe` (`SkillQuality(Bribery, 0.35)` clamped to at least 0.25; bribe = crimes x 5 / quality); `Act_Robber` also rolls it. Music: `WieldableGuitar` (base fail 0.5). Sauna: `PlayerStatsSauna.ApplySauna` (gain; overheat threshold +5 per level, overheat damage x (1 - (skill-1) x 0.1), minimum 0.1).
- Other sources: readables (`ItemReadable`: `IncreaseSkill(skill, SkillAmountToGive)` while `GetSkill < SkillCap`, defaults cap 5 and random skill; per-item values are data), dialogue `DialogueCommonMethods.SkillCheck` (+0.1 per check; fail chance minus `skillWithEffects` x 0.05) and the Lua `GetSkillLevel`, `WorkableResourceSource.Hit` (+0.005 per hit; damage x (1 + skill/3), extra yield up to skill/3), `InteractableWaitWork.WaitDone` (skill roll with factor = hours; `RequiredSkill` gate; salary quality), `ShopBaseUI` build (roll with fail 0.1; `TaskItemBlueprint` process time lerps to x0.5 across levels 1 to `maxLevel`), `MeleeWeapon.Speed` (+ skill/7, integer division), `PlayerStatsEdit.ChangeSkills` and the console `skill` command.

## Pitfalls

- `PlayerStats.OnLoadingGame` adds missing skill types with `Skill = 1`, but when the missing index is below the current list size it assigns `list[i] = new PlayerSkill()`, overwriting an existing entry instead of inserting.
- `CharacterPanelSkill` builds its bar from `maxLevel` once; changing `maxLevel` later needs a rebuild. Patch the getters (`GetSkill`, `GetSkillWithEffects`) rather than the saved `Skills`.

## Unverified

Per-item `SkillAmountToGive`/`SkillCap` values and per-recipe skill factors (data); how the Drinking gain and `SetAlcohol` interact in play.
