# Time, calendar and weather
Scope: the game clock, its events and timers, day counter, sleeping/waiting state, weather. Not covered: Azure sky internals, schedules of individual doors and NPCs, scripted weather logic.
Status: code-only
Related: [player-movement-tiredness-sleep.md](player-movement-tiredness-sleep.md), [growing.md](growing.md), [periodic-rendering.md](periodic-rendering.md), [saves.md](saves.md)

## Where it lives

- `TimeOfDayAzure` (SavableScript, `instance`): clock, day counter, date, events, game-time timers. `TimeAndDay`: value type (time, day, month, year, weekDay). `WaitingController`: sleep, pass-out, waiting, alarm. `PlayerStatsTiredness`: tiredness, `CheckTiredness`, `CanSleep`, `SleepTimeSpeed`. `WeatherManager` (SavableScript): weather timer, profile choice, temperature. The sun and timeline controller is the Azure package (`AzureTimeController`), not read.

## Clock and events `[code]`

- `TimeOfDayAzure.Update`: when `actualSpeed != 0` and updates are enabled, it sets the Azure day length, computes the delta and runs the timers. `actualSpeed = dayLenghtInMinutes / timeSpeedOverrideFactor` (field default 24). The real length of a day in the shipped scene is unverified.
- Static events: `SecondsPassed(int)` (argument is the game seconds since the last fire, may be more than 1 at high speed; about 70 subscribers), `MinutePassed(int)` (argument is always 60; about 20 subscribers: weather, shop hours, trade), `CurrentTimeAndDay(TimeAndDay)` (on minute change; clock, schedules, doors, bank, mail, relationships), `DayChanged(int dayCounter)` (only `RelayPlayerDistance` and `TenementController`).
- Order inside one minute change (`UpdateCurrentTimeAndDay`): SecondsPassed, MinutePassed, day check, `CurrentTimeAndDay`.
- Day rollover is detected in the minute branch: `IncreaseDayCounter` increments `dayCounter`, fires `DayChanged`, sets the dialogue variable `Day` and triggers the SURVIVE_7/30/365 achievements. `DayChanged` fires before `currentTimeAndDay` is refreshed, so date getters in a handler can be stale. `SetTimeline`/`SetDate` do not fire it; the next minute change does the check.
- `IsItDay()` is true for hours 6 to 20; plants use it. Weekday: Azure weekday minus 1, wrapped (0 = Monday).
- Timers: `TimeOfDayAzure.Timer` (callback, `remainingInSeconds`, `stopped`) lives in `timers`, saved through `Timer.OnSaving/OnLoading`. Used by dialogue timers, `MailController` and `manufacturingProcess`.
- Speed: `SetSpeedFactor` sets `timeSpeedOverrideFactor`; `SetSpeedFactorByProgressTime(processTime, 10 s)` picks a factor so a process takes about 10 real seconds. `WaitingController.Wait` uses `SleepTimeSpeed` (default 150) or a given value; `Wakeup` sets 1. `manufacturing`, `ShopBaseUI`, `Crime` and `Prison` also reset it to 1, so a value set during a wait is lost at wakeup.
- No updates run while `updateTimeDisabled` (loading): timers and events freeze. `OnSavingGame` stores `savedTimeAndDay`; on load the clock is set and updates stay disabled until it matches.

## Waiting and sleeping `[code]`

- `WaitingController.Type`: None, Sleeping, Passout, Waiting, Working, HavingFun, Reading, Playing, StaringWall, Arrest, WatchingTV, Talking, Studying.
- Sleep needs `PlayerStatsTiredness.CheckTiredness()`: `currentStatus > 10` or alcohol above 75.
- `WaitingController.CheckEvent(sec)` runs every tick from `PlayerStatsTiredness.UpdateStats` while waiting: it counts sleep time, counts down the alarm and wakes at zero, runs the queued sleep event, and wakes the player if `CanSleep` returns a reason.
- `WaitingController.BlockWaiting(obj)` / `UnblockWaiting(obj)` keep a set of blockers; `Sleep` and `PassOut` refuse while it is non-empty.
- `PlayerStatsTiredness.CanSleep` returns `None` when `currentStatus > 90` before reaching the `Reason.Tiredness` branch, so that branch looks unreachable.

## Weather `[code]`

- `WeatherManager.DeltaSeconds(60)` runs on `MinutePassed`. Weather length is 6 to 24 game hours; at expiry a random profile is chosen with a transition of 50 to 100. Static events `WeatherUpdated`, `WeatherTransitioning`. `Temperature()` = active profile curve x 40 - 20. Lightning in `WeatherManager.Update` runs only when `thunder` is on and no override sky is set.
- No season system found outside the Azure package (weather profiles may hold seasonal curves; unchecked).

## Mod guidance

Subscribe to `MinutePassed`/`DayChanged` when enabling and unsubscribe on destroy; do not read the `MinutePassed` argument; use `SecondsPassed` only for per-second work and treat its argument as game seconds.

## Unverified

Real day length; whether `dayCounter` is a saved field; units of the weather transition value.
