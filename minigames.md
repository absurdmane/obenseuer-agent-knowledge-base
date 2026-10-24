# Mini-games, gambling, keypads, computers
Scope: card tables, slot machine, coin pusher, gambling stats, `Keypad`, computers. Lockpicking is in police-crime.md. Card rules, NPC card AI and computer file/email content are not covered; keypad code values are scene data and not recorded.
Status: code-only
Related: [police-crime.md](police-crime.md), [player-stats.md](player-stats.md), [economy.md](economy.md), [power.md](power.md)

## Where it lives [code]

- Cards: `CardGameTable` (table interactable: stakes, payout, tutorial hook), `CardGame` (abstract base: players, turn and state events), `CardGameUI`, `CardGameNPCComments`; concrete games `ChinaHand` and `Ristiseiska`.
- Coin pusher: `PajatsoGameplay` (+ coin gate, gutter). Slot machine: `SlotMachineGameplay` (states Standby, Playing, Doubling), `SlotMachineHelper` (outcome tables, win lines, doubling).
- Gambling stats: `PlayerStatsGambling` (held by `PlayerStats.gambling`); addiction fields are in `PlayerStatsAddictions` (`GamblingAddiction`, `GamblingNeed`).
- Keypad and rotary lock: one class, `Keypad` (`KeypadType.Keypad` or `RotaryLock`), UI in `KeypadUI`/`RotaryLockUI`. Computers: `InteractableComputer`, `ComputerScreen`, `ComputerProgram*`, `ComputerEmail(s)`, `ComputerFile(s)`.

## How it works [code]

- Card table: `CardGameTable.SelectCardGame` instantiates a `CardGame` prefab, clamps the player count to NPCs + 1 and opens a money slider (skipped when `maxBetAmount == 0`). `StartSelectedGame(stake)` charges the stake (`Money.Buy`, or an item when `useItemAsBet`), blocks waiting, sets the time speed factor to 0.25 and starts. On finish `CardGameTable.OnGameFinished(placement)` pays stake multipliers 2, 1.2, 0.8, 0 for placements 1 to 4 (`Money.AddMoney` or `ItemOperations.AddItems`); first place fires `onGameWin`, the others `onGameLose`, `onGameEnded` always. It is `async void`, so a postfix runs before the awaited cleanup.
- Card-game events are static on `CardGame` (cleared in `Awake`/`OnDestroy`) and invoked through `InvokeSafely`; state is static, so two tables at once would interfere.
- Slot machine: stakes come from a serialized `stakes` array, charged from `_moneyInMachine`. Each round is decided up front in `DeterminePredeterminedOutcome`: win probability 0.33 (`WinChance`) when the unlocked cylinders agree, otherwise a random non-matching result. Doubling: a correct Heads/Tails guess pays 2x, CoinEdge 1x, anything else 0.
- Coin pusher: coins go in via `AddMoneySlot`, are launched with a power-up curve and land in gates (`CoinGateHit` adds `coinsWon`) or the gutter (`CoinGutterHit` counts a loss); collecting pays `Money.AddMoney(coinsWon, RM: false)`.
- `PlayerStatsGambling.UpdateValues(wins, loses)`: adds to `OCWon`/`OCLost`, raises `GamblingAddiction` on wins (capped), lowers it on losses, adjusts mental health and `GamblingSatisfactionByOCWon`; wins reduce `GamblingNeed`. Callers: `CardGameTable.OnGameFinished` (RM amounts x 10), `PajatsoGameplay.CoinGutterHit` and `OnCollectWins`. Waiting also adds gambling addiction per hour (`InteractableWaitWork.GamblingAddictionPerHour`).
- Keypad: `Keypad.code` is a 4-int array (`fsIgnore`, set in the scene or prefab). `KeypadUI.NumberPressed` collects 4 digits and checks on the fourth; success calls `Keypad.TriggerSuccess` (sets `open` when `enableLockOnSuccess`, fires `onSuccess`), failure `TriggerFailure` (`onFailure`). `Keypad.Lock` fires `onLock`; `noPower` is set by `Keypad.ChangePowerState`. `open` is saved.

## Pitfalls

- For a rotary lock `RotaryLockUI.Failure` runs instead of `TriggerFailure`, so `onFailure` does not fire.
- `KeypadUI.ValidateCode` indexes every code element and relies on exactly 4 digits.
- A patch that changes a keypad's `open` must survive save and load.
- Stake or payout changes also change the gambling-satisfaction drift in `PlayerStatsGambling`.

## Unverified

What "OC" stands for in the gambling strings (probably the in-game coin); how often the 0.33 slot win chance plays out; whether `ChinaHand` and `Ristiseiska` are the only `CardGame` subclasses.
