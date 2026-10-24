# Lua functions registered by game code
Scope: the functions game classes expose to dialogue scripts (`Lua.RegisterFunction`, 128 calls in 30 files): Lua name, owner, arguments, effect. Effects are inferred from names and signatures, not from method bodies; dialogue content and the dialogue database are not covered. How tasks and relays use the bridge is in tasks-relays.md.
Status: code-only
Related: [tasks-relays.md](tasks-relays.md), [economy.md](economy.md), [police-crime.md](police-crime.md), [tenement-rent.md](tenement-rent.md), [mail-invoices-phone.md](mail-invoices-phone.md)

Owners register in `Awake`/`Start` (or a guarded register method) and, where checked, unregister in `OnDestroy`. Lua names are global; numeric arguments arrive as `double` (ids too) and are cast on the C# side. Dynamic names (`<prefix>_...`, `Condition_<name>`, `Relay_<relayName>`) come from assets and can collide if two instances share a prefix or name. [code]

## Items, money, bank

- `Inventory`: `CheckForItemDialogue(id)`, `...Amount(id, amount)`, `...LiquidAmount`, `...Owner(id, owner)`, `...Quality(id, amount, minQuality)`, `...LiquidQuality`, `...ReceiptID(id, receiptID)`, `CheckForCategoryDialogue(cat)`, `CheckIfPlayerWieldsItemDialogue(cat)`, `CountItemsDialogue(id, checkAmmo, ignoreOwner, owner, liquid)`, `InventoryIsNotFull(id, owner = -1, amount = 1)` (all return bool except the count); `AddItemDialogue(id)`, `AddItemDialogueAmount(id, amount)` (bool), the `...Drop` variants (void, drop the item), `RemoveItemDialogue(id)`, `...Owner`, `...Amount`, `...Quality`, `...LiquidQuality`, `SellItemDialogue(id, rm)`, `SellItemDialogueCategory(cat, rm)`.
- `Money`: `AddMoneyDialogue(amount, RM)`, `RemoveMoneyDialogue(amount, RM)`, `CheckMoneyDialogue(amount, RM)` (bool). `Bank`: `CreateAccount`, `EnterBank`, `CheckIfPlayerHasAccount`, `GetLoanDetailsDialogue` and `GetOwedAmountDialogue` (strings), `CheckIfLoanAmountIsZero`, `CheckIfPlayerOwesNothing`. `Trade`: `StartTrade`. `Market`: `MarketBuy`, `MarketReturnStuff` (stub methods).
- `CharacterItemManager` (and nested `GiveItemsToCharacter`): `<prefix>_PlayerHasItems` (bool), `<prefix>_GiveItems` (bool), `<prefix>_TakeItemsFromPlayerInventory`.

## Crime, prison

- `Crime`: `GetSentenceLengthPreArrest(severity)` and `GetSentenceLength` (strings), `GetCrimeAmount`, `GetCrimeFineAmount`, `CountStolenGoodsValue` (doubles), `GoToPrison(severityFactor = 1)`, `TeleportToPrison`, `PayFine`, `CanPayFine`, `CanBribe`, `TryToBribe`, `CalculateBribe` (double), `HasStolenGoods`. `Prison`: `InPrison`, `AskForSentenceTime` (registered conditionally). `Act_Police`: `QuestioningPlayer`, `ArrestingPlayer` (bool), `StartChase`, `ForceNewState`. `Act_Robber`: `Aggro`, `RobPlayer`, `TryNegotiate` (bool).

## Tasks, recipes, state checks

- `TaskController`: `TaskActive`, `TaskAnyStatus`, `TaskFinished`, `TaskFailed`, `TaskAbandoned` (task id), `ObjectiveAnyStatus`, `ObjectiveActive`, `ObjectiveAbandoned`, `ObjectiveFailed`, `ObjectiveCompleted` (task id, objective id), `SetObjectiveFinished(taskId, objectiveId)`. `TaskItemsManager`: `PlayerHasTaskItemId(id, amount = 0)`. `RecipeController`: `GiveRecipe(recipeId)`. `PlayerStats`: `GetPlayerStatDialogue(name)` (float). `PlayerSubscriptionsController`: `SubscriptionIdActive(id)`. `StartManager`: `HasSelectedStart(id)`. `DialogueCondition`: `Condition_<name>` (one per condition asset). `RelayDialogue`: `Relay_<relayName>` fires that relay's outputs.
- `DialogueCommonMethods`: `GenerateRandom(r)`, `GetGeneratedRandom`, `GetRandom(r)`, `SkillCheck(skill, failChance)` (bool), `GetSkillLevel(skill)`, `CreateTimer(name, minutes)`, `RemoveTimer(name)`, `CheckTimer(name)`, `SetWaveEffect(bool)`, `SetWaveEffectIntensity(double)`.

## World, NPCs, services

- `NPCManager`: `ChangeInteractableTalk(characterId)`, `LookAtNamedTarget(characterId, targetName)`, `LookAtNPC(characterId, targetNpcId)`, `LookAtPlayer(characterId)`. `SetAnimatorValue`: `AnimatorTrigger(param)`, `AnimatorTriggerDelay(param, delay = 0)`, `AnimatorSetBoolToTrue(param)`, `AnimatorSetBoolToFalse(param)`.
- `TenementController`: `CanRentApartment(residentId)`, `ResidentRentedAmount`, `UpgradesStarted`, `GiveTenementTask5(ignoreFirstTask)`, `ResidentRented(residentId)` (three overloads, so registered with an explicit `Type[]`), `ResidentApartmentLevel(residentId)`, `ResidentApartmentUpgradeLevel(residentId, category)`, `TenementGeneralUpgradeLevel(category)`, `ApartmentUpgradeLevel(apartmentNumber, category)`, `StartApartmentCleaningTimer(apartmentNumber, days)`. `JanitorController`: `TriggerJanitorAction(tag)`, `HasJanitorAction(tag)`. `LegalServicesController`: `CanTriggerLegalAction`, `HasLegalDebt`, `SendLegalDebtInvoice`, `QueueLegalAction`, `GetLegalActionPrice` (name argument). `Hotel`: `CheckAvailability`, `EnterReservation`, `CheckPlayerRoomStatus` (double), `GetPlayerRoomName`. `TellTime`: `TellTimeResponse(y = -1)`, `GetHour`, `GetDay`. `PlayerPhonePlanSubscription`: `GetPriceForOSPhone`, `HasSuitableDevicesForOSPhone`. `RatFightArena`: `StartBetting`, `CollectWinnings`, `CanPlaceBet`, `CanCollectWinnings`. `ShroomPeople`: `startDream`.

## Pitfalls

- `RelationshipController` registers `RelationshipIdActive` through `typeof(PlayerSubscriptionsController).GetMethod(...)`, but the method lives on `RelationshipController`, so the lookup probably returns null (runtime effect unverified).
- Owner methods that take `int` ids need a cast from the Lua `double`. No static name collides in this list.

## Unverified

All effects (inferred from names); the signature of `Bank.CheckIfPlayerHasAccount`; unregistration in owners other than `Bank`; the runtime values of the dynamic prefixes and names.
