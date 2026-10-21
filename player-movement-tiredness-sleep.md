# Player movement, tiredness and sleep

Scope: Walk/run/crouch speeds (no stamina), tiredness, sleep debt, coffee, beds, pass-out, sleep events, alarm clock. Numbers are C# field defaults; the scene/prefab may override them (not checked at runtime).
Status: code-only
Related: [levels.md](levels.md), [tenement-upgrades.md](tenement-upgrades.md)

## Movement / sprint

- `OpenSewerCharacterController` (ECM `BaseCharacterController`), `.instance`. Speeds: `_walkSpeed` 2.5, `_runSpeed` 5,
  `_swimmingSpeed` 5, `_crouchSpeed` 1, first person `_backwardSpeed` 1 / `_strafeSpeed` 2.
- `GetTargetSpeed()`: swimming → swim speed; else run speed when `!walking` (or gamepad: always run) and not crouching;
  crouch overrides. In first person speed drops up to ×0.5 when moving >15° off the look direction.
- Sprint = input binding `"Walk"` toggling the private `walking` flag. `ControlsSave.instance.controlsSave.toggleSprint`:
  toggle vs hold.
- **No stamina.** Running costs nothing: no stat reads speed/`walking`. Only modifier: `OrangeMushroomEffect.GetRunSpeedModifier`
  (adds up to +50% run/swim speed).

## Tiredness — `PlayerStats.instance.tiredness` (`PlayerStatsTiredness`, saved)

- Ticked from `PlayerStats.DeltaSeconds` (on `TimeOfDayAzure.SecondsPassed`) once per accumulated game minute
  (`LocalSeconds > 59`); per-minute amounts below are rate/60.
- `currentStatus` 0..100 (can go down to -100). Awake: `+Exhaustionrate`(6)/h, scaled by `1 - MushroomsHigh/100`.
  Asleep: `-RestRate`(10)/h. So ~16.7 h awake fill 0→100, ~10 h sleep empty it.
- At 100 awake → `WaitingController.PassOut()`.
- `tirednessToAdd`: delayed buffer, drained by `tirednessToAddRate` (5) per minute into `currentStatus`.
  Consumables (`UpdateValues`): 30% of `ItemConsumable.Tiredness` immediately, 70% into the buffer;
  negative Tiredness also adds `|T|*0.2` to `sleepdebt`.
- `SpecialEffect.Coffee` / `SleepingPills` (`PlayerStats.ProcessItemSpecialEffects`) feed `tirednessToAdd` gradually, every
  `SecondsPassed` tick (0.25 or 0.25% of the remainder per tick).
- **Sleep debt** (`sleepdebt`): grows while the buffer is negative (5/min, i.e. every point of coffee alertness is borrowed) and
  while `currentStatus < 0` (exhaustion rate). Repaid into `currentStatus`: awake `min(1, 5*(debt/100)^2)`/min (slow when small),
  asleep 5/min (fast: debt turns into tiredness, so sleep lasts longer).
- Below -100 with a negative buffer: the rest of the buffer becomes health damage (`HealthToRegenerate -= |buffer|*0.5`).
- Effects of high tiredness: `SleepyEffect` post-process from 50% (full at 100); contributes to mental health decay
  (`PlayerStatsMentalHealth`, `generalStatusEffectCurve`); bar shakes > 90.
- Item data (Items.json): coffees -15..-20 (+Coffee effect 10–15), OS Energy Drink -22, colas -3..-10, tea -0.5;
  Sleeping Pills +50, Prescription +100, Bleach +33. Sleeping pills use plain `Tiredness`, not the `SleepingPills` effect.

## Sleep — `WaitingController` (+ `Sleep` beds, `WaitingUI`)

- `WaitingController.Type`: Sleeping and Passout count as `IsSleeping`; Waiting/Working/Reading/WatchingTV… are `IsWaiting`.
- Bed `Sleep.Use()`: refused if owned by an NPC/occupied or `!tiredness.CheckTiredness()` (needs tiredness > 10 or blood
  alcohol > 75). Opens the time-set clock (`ShowTimeSetUI`); `WaitingController.Sleep(setTime)` → `StartSleep`.
- Setting a wake time needs an alarm: an `AlarmClock` furniture within 1 m of the bed (`Sleep.connectedAlarmClocks`) or the
  Alarm Clock item (ID 10670) in inventory (`WaitingUI`). Without it you sleep until rested.
- While sleeping time runs at `SleepTimeSpeed` (150×) via `TimeOfDayAzure.SetSpeedFactor`.
- Wakes up when: tiredness reaches 0 (only if blood alcohol < 1), the alarm fires, or after 1 h of sleep a need blocks it
  (`CanSleep`: bleeding > 33, near-death health, hunger > 90, thirst > 90; mental health/tiredness > 90 only for waiting).
  `CantSleepReasonDialogue` is shown as the wake-up reason.
- During sleep: hunger slowed by `SleepHungerThristRateFactor`, hygiene and addictions don't progress, mental health
  (`currentStatus`, higher = worse) drops by `mentalHealthChangeRate`/h.
- Bed quality: `Sleep.quality - PlayerStatsMentalHealth.expectationOffset*10` → `ChangeMentalHealthTarget(-q)` and a new
  expectation level. Label in `Sleep.GetTitle()`: Torturous < -20 < Aching < -10 < Uncomfortable < -3.33 < Okay < 3.33 < Pleasant
  < 10 < Relaxing < 20 ≤ Comfy, plus Unsafe/Safeish when not safe.
- **Pass out** (tiredness 100): `StartSleep(quality 15, ...)` with `passoutEvents` (+ crazy neighbour events for that start).
  `Crime` arrest also forces a pass-out of type Arrest (tiredness raised to ≥ 10).
- **Sleep events** (`SleepEventController.CalculateEvent`): none if `playerInSafeArea` or bed `safe`. Otherwise random events
  from the list, each with chance `eventRisk * (1 - playerSafetyFactor)` %; fires 1–7.8 h into sleep. Kinds: theft (items,
  money, most valuable), beating, arrest (if crimes), quarantine, covered with junk, nightmare, invoice, SMV infection…
  Each event's `SleepEvent.dialogue` plays on wake-up. Which events sit in `passoutEvents` / beds' `sleepingEvents` and their
  risks is asset data — not dumped yet.
- Handlers (`SleepEventController.Event_*`): `Quarantine` is an empty stub (only logs). `Nightmare` plays scary sounds,
  each with a 10% chance to wake the player, no stat change. `Invoice` adds an invoice item from "Jorma's protection services":
  25% RM 10–149, else OC 100–1049 (`InvoiceController.CreateInvoiceItem`). `InfectSMV`: `SMVProgressionRate += 0.5`,
  hygiene -10. `CoveredWithJunk`: adds random junk items. `CrazyNeighbor`: rolls 20% items / 10% money / 5% best item stolen,
  50% junk, 5% SMV +0.2, 10% hygiene -30.
