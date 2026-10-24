# Mail, parcels, invoices, phone, legal services
Scope: how mail and parcels are delivered, random mail and fake invoices, invoices and their expiry, the phone, legal actions. Mail and call text content is not covered. Money maths and rent are in economy.md, crime records in police-crime.md, tasks and notes in tasks-relays.md.
Status: code-only
Related: [economy.md](economy.md), [police-crime.md](police-crime.md), [tasks-relays.md](tasks-relays.md), [time-and-weather.md](time-and-weather.md), [achievements-boot.md](achievements-boot.md)

## Where it lives [code]

- `MailController` (SavableScript, `instance`): `Mail` subclasses (`ItemMail`, `TaskItemMail`, `NoteMail`, `RecipeMail`, `MoneyMail`, `TaskMail`, `FurnitureMail`, `BundleMail`, `ParcelMail`), `Target` enum (Player, World, PlayerParcelLocker); lists `receivedMail`, `receivedPackages`, `sentMail`, `unprocessedMail` (mail to timer). Delivery objects: `Mailbox`, `ParcelLocker`. Auto-mail: `RandommailSender`, `PlayerMailSubscription`. Item meta: `ItemMailableData`, `ItemInvoiceData`, `ItemConnectivityData`.
- Phone: `Phone` (Interactable), `PhoneUI`, `PhoneNumber` (ScriptableObject), `PhoneNumberDatabase`, `PhonecallRelay`, `PlayerPhonePlanSubscription`. Invoices: `InvoiceController` (`invoices`), `InvoicePaymentMachine`, `logic_invoice_sender`, `FakeInvoiceReceiver`. Legal: `LegalServicesController`.

## Mail and parcels [code]

- `MailController.SendMail(Mail, Target, _delay)` decides immediate or delayed from the controller's own field `delay` (never assigned in code, so it comes from scene or prefab data; a null field throws), not from the argument. Delayed mail waits in `unprocessedMail` on a `TimeOfDayAzure.Timer` (minutes x 60). The `SendMail(string, ...)` overloads (item, note, recipe, money, furniture) take the controller delay for Player and parcel-locker targets when none is given, so changing the field changes all such mail. `Mailbox.InsertMail` uses the delay only when `ItemMailableData.useDelay` is set.
- Routing: Player to `receivedMail` (+ `MailReceived`); PlayerParcelLocker to `receivedPackages` (+ `PackageReceived` and a notification item); World to `sentMail` (+ `MailSent`). Pending timers are saved; on load `Mail.OnLoading()` re-links them and the events fire after two frames.
- `Mailbox.TakeMail` takes everything from `FetchMail()` and calls `TakeMailitem` per entry: items to the inventory (remainder dropped), task items and objectives to `TaskItemsManager`/`TaskController`, notes to `NoteController`, recipes to `RecipeController`, money to `Money.AddMoney`, furniture to `BuildingSystem.AddFurniture`, parcels everything inside. `ParcelLocker` removes a parcel only when the player takes it.
- `RandommailSender.CurrentTime` (on the minute/day event) sends at most once per check; skipped on weekday 6 and outside hours 2 to 12; a gap of two or more days is caught up with one send per missed day. Each send picks a random count of `randomMail` entries (item, note, recipe or parcel). `SendFakeInvoice` runs per catch-up iteration with chance `randomInvoiceDailyChance` (0.15), a random debtor and price range; with `randomInvoiceRMChance` it is in RM; created with `isfake: true` (7 days).

## Invoices and legal [code]

- `InvoiceController.CreateInvoice(debtor, money, days, delay, isfake, onPaid, onExpired)` splits OC and RM into two `ItemInvoiceData` entries; `dueDate = dayCounter + days`. `CheckDueDates` runs once per day: an expired real invoice commits an Administrative crime (10% of the OC amount, or RM as is, times 1.1, no criminal), fires `onExpired` and is removed; with fewer than 2 days left a reminder copy ("REMINDER:" in the debtor) is mailed every daily check in that window (duplicates). Expired fake invoices are never processed and stay until paid. `PayInvoice` needs `Money.CheckMoney`, fires `onPaid`, removes the invoice (fake ones give achievement `PAY_FAKE_INVOICE`); `CanPayInvoice` refuses expired ones but `PayInvoice` itself does not. `InvoicePaymentMachine` lists invoice items in its storage and pays through `PayInvoice`.
- `PlayerSubscription` (ScriptableObject): payment by invoice (`CreateInvoice(subscriptionName, GetCost(), paymentTime)`) or by bank account. `ItemConnectivityData.CheckPlan` reads the active phone-plan subscription to decide whether a device is connected.
- Legal Lua functions (registered in `Start`): `CanTriggerLegalAction`, `HasLegalDebt`, `SendLegalDebtInvoice`, `QueueLegalAction`, `GetLegalActionPrice`. `QueueAction(name)` (once per name) queues a timed action and, if priced, creates an invoice with `onExpired = AddDebt`. Price of `ClearCrime` scales with `Crime.CountCrimes` (per target NPC when `actionData` is an id); `UnfreezeAccount` is a flat price. On the timer, `ClearCrime` calls `Crime.ClearCrime` and `UnfreezeAccount` sets `BankAccountData.Frozen = false`. `SendDebtInvoice` bills debt x 2 with `onPaid = ClearDebt`; its expiry commits a crime again.

## Phone [code]

- `Phone.Interact`: a phone owned by an NPC (`owner.ID > 0`) with no incoming call shows "Owned" and a warning; otherwise it answers an incoming call or opens the dial pad. `PhoneUI.PressCall` -> `PhoneNumberDatabase.CallNumber`: own list first, then the global database from `Resources` (`Dialogue/Phone`); unknown gives `notReachable`, a network mismatch `outsideNetwork`, a failed roll against `answerChance` `notAnswering`. After dialling there is a random 1 to 12 s delay before `Phone.StartCall`. `Phone.ReceiveCall` ignores calls while a menu is open, a call is active or one is incoming. `PhoneNumber.Network` is a flags enum (Obenseuer, Outside, Wasteland).

## Pitfalls

Patch `Target` routing or getters rather than the saved fields (`receivedMail`, `invoices`, `debt`). A mod reading mail sees duplicate reminders.

## Unverified

Value and location of `MailController.delay`; whether `InvoiceController`, `LegalServicesController` and `RandommailSender` save their timers and queues; the meaning of weekday 6 (assumed Sunday); `FakeInvoiceReceiver` (bank deposits, account freezing chance) was only skimmed.
