# Shop internals: furniture shop, checkout, vending, tickets
Scope: how shop screens and stock work, the market checkout, vending machines, restock and saving. Price maths and trader cash are in economy.md. `TrainShop`, `ShopController`, bottle-recycling payout and the checkout placement code are not covered.
Status: code-only
Related: [economy.md](economy.md), [doors-interaction.md](doors-interaction.md), [interactable-catalog.md](interactable-catalog.md), [police-crime.md](police-crime.md), [map-travel-trains.md](map-travel-trains.md)

## Where it lives [code]

- `BaseShop` (shared cash, restock timer, percentages, price getters); `FurnitureShop` (stock `availableFurnitures` built from `itemsLists`; sell rules `canSellItems`, `sellItemsSameAsBuy`, `buysAnything`, `sellLists`); `FurnitureShopUI` (20 lines: player side is `BuildingSystem.availableFurnitures`, blueprints from `BuildingSystem.blueprints`); `ShopBaseUI` (about 1100 lines: Buy, Sell, shop-side `AddItem`, and also crafting, dismantling and auto-consume).
- `MarketShop` (checkout conveyor, about 900 lines; states Standby, Scanning, Pay). `VendingMachine` (nested `VendingMachineItem`; stock lives in the machine's 8-slot `Storage`). `Market` is a stub (several methods throw `NotImplementedException`). `BottleRecycling` opens `BottleRecyclingPanel` (payout in the panel, not read). `InteractableTicketMachine` (ticket sale plus a passport validity timer).

## How it works [code]

- Buy (`ShopBaseUI.Buy`): `PlayerHasMoney` gates it; `Money.RemoveMoney` for OC and RM at the shop's buy prices; the item leaves the shop; `BuildingSystem.AddFurniture` gives it to the player. Sell (`ShopBaseUI.Sell`): the item must not be placed in the world (except trains), `CheckIfShopHasMoney`, `CanSell`; `Money.AddMoney` at sell prices; the item goes to the shop and leaves the player (`BuildingSystem.RemoveFurniture`, or `TrainShop.DestroyTrainFromTrack`). Prices are `(int)(Price x percent/100)`: truncated, not rounded.
- Restock: `BaseShop.Restock` = `MoneyRestock` (cash, scaled by trader wealth) + `UpdateShopItems`; triggered by a new day (`restockEveryDay`, `restockedDay`) or by `timeToRestock` (random between 80000 and 90000, probably seconds, about 22 to 25 hours; the handler is named `DeltaSeconds` but hooked to `MinutePassed`). `FurnitureShop.UpdateShopItems` clears and rebuilds `availableFurnitures`, so anything the player sold to the shop is lost at restock.
- Saving: `BaseShop` saves `currentOcMoney`, `currentRmMoney`, `timeToRestock`, `initialized`, `loadedCustomImages`; config (`shopMoney*`, percents, times) is `[fsIgnore]` and comes from the prefab; `FurnitureShop` saves each stock entry through `SavePath()`/`LoadFromPath()`; `LoadDelay` adds elapsed time to the timer.
- `MarketShop`: each scanned `ConveyorItem.price` is summed; `GetItemSalePrice` = `Item.GetItemSalePriceUnrounded(meta)` x `priceFactor` (default 1.1) converted with `Money.OCToOtherCurrency`; `Pay` charges `GetCurrentPrice()` (minimum 1) with `Money.Buy`, gives the items owner NPC 0 (unless `retainOwner`, which changes theft logic for them), converts the bag and fills an `ItemReceiptData` (RM sum = price x 10).
- Since 0.5.13 the checkout persists items: `conveyorItems` is a saved list filled in `OnSavingGame` (`GetConveyorItemStacks`) and restored on `SaveController.LoadingDone` (`RestoreConveyorItems`); `OnMapChanging` returns conveyor items to the owner's shelves (`ReturnItemsToStorages`), destroys the conveyor objects and calls `ResetState`. `HandleConveyorItemPlacement` now takes an `ItemStack` (the caller removes the slot item right after), scanning starts in `StartScanning`, and the cashier looks at the scanned item (`CharacterHeadController.LookAtTarget`, `StopLookAtTarget`) with a new comment key.
- `VendingMachine.Buy`: stock check; price = base sale price x `basePurchasePrice` (1.2) or `absolutePrice`, minimum 1; buying a stolen item with a witness calls `Crime.instance.CommitCrime`; then `Money.Buy` and the item is created. `RefillStock` resets each `currentStock` to `stock` (initial roll on `spawnChance`, then `MinutePassed` via spawn settings when `respawnEnabled`).
- `InteractableTicketMachine.BuyTicket` uses `Money.Buy(ticketPrice, currency, true)` (`ticketPrice` 25, `fsIgnore`); `waiting`, `hasTicket`, `passportValidTime`, `currentPassport` are saved. `Money.Buy(int, Currency, showNoMoney)` shows a "not enough" notification; the struct overload is silent.

## Mod guidance

Public non-virtual postfix targets: `BaseShop.GetOcBuyPrice/GetRmBuyPrice/GetOcSellPrice/GetRmSellPrice`, `Restock`, `MoneyRestock`, `MarketShop.GetItemSalePrice/GetCurrentPrice`, `VendingMachine.Buy/RefillStock`. `ShopBaseUI.Buy/Sell` are private. Patch the protected `FurnitureShop.UpdateShopItems` to keep player-sold stock. `BaseShop` subscribes to `MinutePassed` in `Start` and unsubscribes in `OnDestroy`; do the same. Do not call `Market` methods.

## Unverified

Unit of the restock timer; whether `FurnitureShop.availableFurnitures` is serialized directly; the meaning of `ItemOperations.AddItem(receiptItem.ID, -1, ...)` in `Pay`; whether a `Market` exists in any scene.
