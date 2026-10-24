# Equipment: worn items, hand items, flashlight
Scope: character equip slots, how equipping routes by category, hand items and their input routing, the flashlight, what worn items change. Slot indexes and quick move are in inventory-storage.md, the backpack in backpack.md, the melee damage path in combat-npcs.md. `Dosimeter`, `Osrat`, `Detonator`, `WieldableGuitar`, the appearance system and `FirstPersonHands` are not covered.
Status: code-only
Related: [inventory-storage.md](inventory-storage.md), [backpack.md](backpack.md), [combat-npcs.md](combat-npcs.md), [smv-sauna-radiation.md](smv-sauna-radiation.md), [item-system.md](item-system.md), [skills.md](skills.md)

## Slots and classes [code]

- Character slot indexes (`Inventory.StartInit`, `GetEquipSlotIndex`, `Equip`): 0 hat and 1 accessory (no controller), 2 Head (`HeadSlotController`), 3 Face (`FaceSlotController`), 4 Watch (`WatchSlotController`), 5 Backpack, 6 Wieldable Primary (right hand), 7 Wieldable Secondary (left hand). All eight are used.
- `CharacterSlotController` (SavableScript base): per-slot state `equipped`, `currentItem`, `handItem`, perishable and ammo meta, `slotObject`; only `unequipped` is saved; subclasses set `SlotIndex` and `OnDropped`/`OnTaken`.
- `ItemEquipable` (`Item.ItemAction`): `type` (None, Tool, Weapon, Backpack, Mask, Head, Watch), `bothHands`, `useEquip`, `wieldPrefabPath` (Resources path). The `type` enum is read but routing does not use it.
- Hands: `PlayerHandItems` (singleton) holds `righthandItem` (slot 6) and `lefthandItem` (slot 7) behind `IPlayerHandItem`; implementations in namespace `WieldableItem`: `BasicWieldable` (no-op base), `BasicWieldableItem`, `Flashlight`, `MeleeWeapon`, `WieldableGuitar`, `Dosimeter`, `Osrat`, `Detonator`.

## How it works [code]

- Using an equipable item: `ItemEquipable.Invoke` loops over the item's categories and calls `Inventory.EquipItem("<category>", slot)` with `Wieldable Primary`, `Wieldable Secondary`, `Backpack`, `Face`, `Head` or `Watch`. If the item already sits in its equip slot it is unequipped, otherwise swapped in (`SwapItems`); the backpack has its own path `EquipBackpack`. Slots 6 and 7 call `PlayerHandItems.DropR/DropL/TakeR/TakeL`; slots 2 to 4 call their controller's `OnDropped/OnTaken/OnReturn` (also called for already equipped items).
- `PlayerHandItems.UpdateItemInHand` destroys the old hand object, instantiates the new prefab under `PlayerLocator.instance.playerHead` at local (0, -0.33, 0) and calls the equip hooks. Worn head, face and watch items instead instantiate under `ThirdPersonCameraController.instance.transform`. `bothHands` waits in a coroutine until the other hand is free. Primary use, hold, stop hold, secondary use and reload go to the right item; the `Auxiliary*` methods go to the left; with `useEquip` set the right item's `onPrimaryUse` is never called (as read).
- Flashlight: `onPrimaryHold` charges a boost (hold about 1 s, lasts 2 s, cooldown 10 s, drain 0.5 per minute against 1.25 normal); the battery is the item's `ItemAmmodata`; `DrainCharge` runs on `SecondsPassed`; intensity lerps between `intensityEmpty` and `intensityFull`; reload swaps in a battery only at 66% charge or less.
- `MeleeWeapon.Speed` = mushroom animation speed + skill/7; durability is `ItemPerishabledata.integrity`. Lockpick: a wielded item of category `Lockpick` (`PlayerLockpicking.CheckIfPlayerHasLockpick` -> `Inventory.CheckIfPlayerWieldsItem("Lockpick")`; `BreakLockpick` removes the hand item). `Extractable` checks the `Tools` category and `Act_Robber` the `Weapon` category the same way.
- Effects of worn items: Face items 15420 Facemask, 17110 Gas Mask, 17130 Filterless Gas Mask, 17380 White Mask `[data]`: `HasMushroomProtection` (facemask with integrity above 0, or gas mask with ammo above 0), `HasGasProtection` (gas mask with ammo above 0), `HasFacemaskEquipped`; used by `PlayerStatsHealth` damage handling. `CharacterSlotController.DeltaSeconds` drains worn ammo each second when `characterSlotDrain` is set, and always for gas mask 17110. A worn item with `ItemPerishabledata` drains hygiene each game second, growing as integrity drops (capped at 2 per 1000 s). Earmuffs 19480 (Head) call `UnderwaterEffect.Muffle/Unmuffle`. The watch slot (categories `Clock`, `Clock Calendar`) gates what `ShowTime` displays. Hat and accessory slots are read by no code beyond their allowed categories. Item quality flags apply only to consumables and values, not equipment.

## Mod guidance

A new wearable slot would need a free index (none is) and save changes; prefer a new category in an existing slot. Patch the protection getters (`HasMushroomProtection`, `HasGasProtection`) instead of saved fields. A visual for a worn item must parent like the game does (third-person camera for worn items, `playerHead` for hand items). `Flashlight.slotType` only changes tutorial hints.

## Unverified

Which items use which wield prefab, and whether head and face items have one; whether the hat and accessory slots belong to an appearance system; the condition in the `LockpickUI` caller of `BreakLockpick`.
