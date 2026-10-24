# Economy: money, shops, bank, rent
Scope: player cash, item price scaling, shops and traders, bank accounts and loans, rent and invoices. Not covered: bank UI flows, `InvoiceController` internals, `MarketShop` scanning, per-difficulty preset values, story economy.
Status: code-only
Related: [items-recipes.md](items-recipes.md), [item-system.md](item-system.md), [tenement-upgrades.md](tenement-upgrades.md), [police-crime.md](police-crime.md), [time-and-weather.md](time-and-weather.md)

## Where it lives

- `Money` (SavableScript, `Money.instance`): two currencies. OC is an int (`OCMoneyAmount`); RM is a float facade (`RMoneyAmount`) over an int in tenths (`OCtoRM = 0.1f`). Nested `money` struct (OC + RM). `Money.Account` (`dailyIncome`, `dailyExpendure`) has no readers and looks unused. `Money.BankAccountData` (in `Money.bankAccounts`): bank name, OC, RM, `Frozen`, `Interest` 0.015, `DebtInterest` 0.03, `CurrentDebt`, `MonthlyDownpayment`. [code]
- `BaseShop` (Interactable): `sellPercent` 90, `buyPercent` 110, trader cash ranges (OC 5000-10000, RM 1000-2000), restock settings. UI: `ShopBaseUI`, `ShopController`; variants `FurnitureShop`/`FurnitureShopUI`, `TrainShop`/`TrainShopUI`. `MarketShop` (checkout conveyor, `Pay`). `Trade` (NPC trader, `TradeSpecial` subclass, `TradePanel`). [code]
- `Bank` (+ `BankUI`, `BankPanel`): accounts, `CalculateMaximumLoan`, `TakeLoan`, `PayLoan`, `Exchange`. `InvoiceController.CreateInvoice(debtor, money, days, ..., onPaid, onExpired)` handles all debts and bills. `PlayerSubscription` is a recurring cost as an invoice. [code]
- `DifficultyInfo`/`DifficultyController`: multipliers for prices, trader wealth, rent income, renovation cost, stack size, rent payment mode. [code]
- Income seen in code: selling to shops and traders, tenement rent. No Job/Salary/Income class exists. [code]

## How it works [code]

- Money changes go through `Money.AddMoney`/`RemoveMoney` (`RemoveMoney` clamps at 0); they update the dialogue actor fields `Player.Money`/`Player.RealMoney`, the money panels and play a sound. Writing `OCMoneyAmount`/`RMoneyAmount` directly skips all that. Lua functions `AddMoneyDialogue`, `RemoveMoneyDialogue`, `CheckMoneyDialogue` are registered in `Money.Start`.
- Price scaling has one point: `Furniture.PriceOC/PriceRM` and `Item.Value` are getters passing the base through `DifficultyInfo.GetItemPrices` (multiplier `prices`); shops, trades and theft all read them. `DifficultyInfo.GetItemPrices` writes back to `prices` (Disabled becomes Default on first read).
- Shop prices: buy = base x `buyPercent`/100, sell = base x `sellPercent`/100 (both currencies). Buying needs `PlayerHasMoney` in both; selling needs `CheckIfShopHasMoney` and `CanSell`. Trader cash is rerolled by `MoneyRestock` (scaled by `traderWealth`) on day change (`restockEveryDay`) or when a random timer between `minRestockingTime` and `maxRestockingTime` runs out (unit unconfirmed).
- Since 0.5.13 trader wealth can be Infinite (`DefaultMultiplier.Infinite`, selectable only for `traderWealth`; `DifficultyInfo.GetTraderWealth` returns the input unchanged): `DifficultyController.InfiniteTraderWealth` makes `BaseShop.CheckIfShopHasOc/Rm` true, `AddBuyMoney`/`RemoveSellMoney` no-ops, the `Trade.CurrentMoney` setter a no-op and `Trade.CheckIfTraderHasMoney` true (`MoneyRestock` under the flag was not checked). `Trade.CanSell` gained an optional `checkMoney` (default true; `TradeSpecial.CanSell` too).
- `Trade.GetItemPrice` (static) = `Item.GetItemSalePrice` x base price x stolen factor (owner > 0), optionally minus container and liquid values, at least 1. Theft value of collectable furniture = `PriceRM + 0.1 x PriceOC`.
- Interest: `Money.CurrentTime` (private, runs on day change) multiplies balances by `(1+Interest)^(days/7)` and debt by `(1+DebtInterest)^(days/7)`; `days/7` is integer division, so gaps under 7 days give nothing.
- Debt repayment: each rent cycle with `CurrentDebt > 0` creates a 7-day downpayment invoice of `MonthlyDownpayment` divided by the payments per month (Daily 30, Weekly 4, Biweekly 2, Monthly 1), capped at the debt.
- Rent schedule `DifficultyInfo.GetRentPayment`: Daily at 19:00 or later on day change; Weekly and Biweekly share one check (weekday 6, 19:00+), so Biweekly does not look like a two-week gap; Monthly checks day 0; non-daily modes skip if the last payment was within 2 days. Collateral (`GetRentCollateral`): Daily 0.25x, Weekly 1x, Biweekly 2x, Monthly 3x monthly rent (`TenementController`, invoice "Rent collateral", 14 days).
- Loans: `Bank.CalculateMaximumLoan` = min(supply x 10, max loan, `TenementFinancingRatio` x raw tenement income x `PaymentTimemonths`) per currency, minus `CurrentDebt`, then converted (about 10 OC per RM).

## Mod guidance

Postfix `Furniture.PriceOC/PriceRM` and `Item.Value` to scale all prices; the public non-virtual `BaseShop.GetOcBuyPrice/GetRmBuyPrice/GetOcSellPrice/GetRmSellPrice` are shop-specific targets. Use `AddMoney`/`RemoveMoney`. Prefer patching methods over adding saved fields (`Money.bankAccounts` is saved).

## Unverified

Preset values per difficulty (asset data); unit of the restock timer; whether Biweekly rent is meant to differ from Weekly; whether `Money.Account` is used.
