# Junk, trash containers, janitor actions, cleaning timer
Scope: where junk items come from, the placeable trash container, the janitor action registry, the apartment cleaning timer. No dirt or room-decay simulation exists under these names (searches for junk, trash, sludge, sewage, dirt, decay found none); most "clean" hits are the building-system cleaning menu UI, which is not covered here.
Status: code-only
Related: [tenement-upgrades.md](tenement-upgrades.md), [crafting.md](crafting.md), [player-stats.md](player-stats.md), [water.md](water.md), [saves.md](saves.md)

## Where it lives [code]

- `JanitorController` (SavableScript): a registry of "janitor actions" keyed by a string tag; a scene-state refill and reset system, not dirt cleanup. Implementers: `StorageJanitorAction`, `RelayJanitor`.
- Junk: `SleepEventController.junkItems`/`junkItemAmount`/`AddRandomJunk` (sleep events); `CraftingBase.RemoveAmmo` with `Recipe.disableJunkSpawn`; `Dishwasher.GenerateRecipe`. Trash container: `FurnitureTrash` (+ `FurniturePlaceable`). Cleaning timer: `TenementController.StartApartmentCleaningTimer`, `OnCleaningTimerDayChanged`, `TenementApartment.cleaningTimerCompleteDay`, `TenementApartmentPrefabs.UpdateCleaningTimerState`.

## How it works [code]

- Janitor registry: `AddJanitorAction`/`RemoveJanitorAction` keep a non-saved list; `TriggerJanitorAction(tag)` (a Lua function) runs every registered item with that tag, then stores `savedActions[tag]` with the scene in `processedLevels`. On load, `LoadDelay` re-runs saved actions once per scene not yet processed. `HasJanitorAction` and `JanitorActionInstances` are queries. `StorageJanitorAction.TriggerAction` fills empty storage slots with `itemToFill` (owner = the storage owner, filtered by allowed category; all slots with `overrideStorage`) or sets a liquid with `LiquidStorage.SetLiquid(..., -1f, ...)`.
- Sleep-event junk: `Event_Covered` and, with 50% chance, `Event_CrazyNeighbor` call `AddRandomJunk`, which picks a count in `junkItemAmount` (exclusive upper bound) and adds random `junkItems` entries to the player's inventory, not to the room.
- Ammo junk: when a recipe consumes an `ItemAmmodata` input and ammo reaches 0 with `destroyWhenEmpty` (default true), the slot item is removed; with `spawnJunk` (`!recipe.disableJunkSpawn`) its `itemsToSpawn` go to the owner's inventory first. `Dishwasher.GenerateRecipe` sets `disableJunkSpawn`, `useOnlyStolenItems`, `keepItemOwnerOnFail`, `useAmmoAsInput`, skill Cleaning; fail chance = `Value x difficultyPerOC` (0 to 0.99 since 0.5.13), process time = `Value x processTimePerOC` clamped to `[minProcesstime, maxProcesstime]` (new field, default 86400); the recipe name is `Item.GetSkinTitle(meta)` and an `ItemSkinData` of the source item is copied into `recipe.Meta`.
- `FurnitureTrash.Take` adds each stack with `ItemOperations.AddItems(notify: true)`; `CanTake` checks `Inventory.InventoryIsNotFull`; on pickup `FurniturePlaceable` calls `Take`, `DestructibleList.DestroyMapItem(trash.GUID)` and destroys the object.
- Cleaning timer: the Lua function `StartApartmentCleaningTimer(apartmentNumber, days)` sets `cleaningTimerCompleteDay = currentDay + days` only when it is -1; `OnCleaningTimerDayChanged` (on `DayChanged`) sets it to 0 once the day is reached and refreshes visuals; while the value is above 0 the apartment shows an indicator and a blocker, at 0 the blocker is destroyed.
- `RecipeType.SludgeTank` appears in no code branch and in no `Recipes.json` recipe `Type`; one readable item in `Items.json` lists "SludgeTank" in its `RecipeTypes`. [data]

## Pitfalls

- The cleaning timer start requires -1 but completion writes 0, so a completed apartment cannot restart it (no other writer found); a timer started with 0 days on day 0 also reads as done.
- `FurnitureTrash.items` is `[fsIgnore]`; where trash contents persist is not established.
- `JanitorController.janitorables` is `[fsIgnore]` and re-registers on start; the per-(scene, tag) counts update only on add and remove, and `StorageJanitorAction.OnDestroy` removes without updating them, so counts can go stale.
- `RemoveAmmo` in the dry-run path does not reduce ammo but still runs the destroy and junk branch at ammo 0.

## Unverified

Whether dirt or room decay exists anywhere (asset data or another system); values of `junkItems`/`junkItemAmount` (scene or prefab data); what the `SludgeTank` entry does.
