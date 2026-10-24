# Furniture placement, doors, locks, interaction
Scope: placing and picking up furniture, doors and locks, the interaction picker. Not covered: areas and rooms (furniture-areas.md), lockpicking minigame and crime (police-crime.md), furniture shop, door subclasses, lock UI.
Status: code-only
Related: [furniture-areas.md](furniture-areas.md), [police-crime.md](police-crime.md), [power.md](power.md), [containers-theft.md](containers-theft.md)

## Where it lives

- `Furniture` (ScriptableObject): `id`, `title`, `category`, `priceOC/priceRM`, `trash`, `restrictedAreas`, `dismantleItems`, `prefab`, `previewPrefab`, `alternativePrefabs`, `skins`, custom image fields. `FurniturePlaceable` (SavableScript): the placed instance, linked by GUIDs (`GUID`, `managerGUID`, `parentGUID`, `attachedtoGUID`, child lists); flags `global`, `disableSaving`, `decal`, `cantTake`, `isPreplacedFurniture`. `FurnitureManager`: area container with events `FurniturePlaced`/`FurnitureTaken`. `BuildingSystem` (nested `FurnitureInfo`) does placing and taking; menu in `BuildingSystemMenu`. [code]
- Doors: `Door : Interactable`, `DoorLockBypass : Interactable`. Lock logic is in `Lock`, a plain serializable class (not a MonoBehaviour) held in `Door._Lock`. A key is a `TaskItem` (`Lock.keyItem`) checked with `TaskItemsManager.CheckIfPlayerHasTaskItem`; the key is not consumed. [code]
- Interaction: `Interactable : SavableScript, IInteractable`; the player-side picker is `InteractObjects`. [code]

## How it works [code]

- Place (`BuildingSystem.Place`): refused unless the preview's `isPlaceable` is true. A wallpaper target (`FurnitureSurfaceArea`) takes the item as wallpaper. Otherwise the prefab (`Furniture.GetPrefab(alternative)`) is instantiated under the active manager; lights get a `LightController`; child furniture is registered with `AddChildFurniture`/`AddAttachedFurniture`; item-based furniture is a collectible converted by `ItemOperations.SetCollectibleItemValues`. One copy is removed from the inventory (`RemoveFurniture`).
- Take: raycast, `GetComponentInParent<FurniturePlaceable>`, `FurniturePlaceable.CanTake` (checks `cantTake`, `trash` capacity, `CanTakeItem`, required items), `Take` returns children and saved content, `BuildingSystem.AddFurniture` stores it.
- Since 0.5.13: `FurniturePlaceable.CanTake(showMessage, moveItem)` and `CanTakeItem(showMessage, skipOwnItem)` have new signatures; `Take` has a `keepItem` parameter (calls `OnTaken` and returns without making the item), stands a seated player up (`InteractableChair.StandUpFrom`) and calls `BuildingSystem.DisconnectPower`. Picking up a placed item that has a `CollectibleItem` child enters move mode (`movingPlacedItem`; `BuildingSystemMenu.Press` places it, `PlaceMovingItem` re-parents it and calls `ReconnectPower`); `BuildingSystem.OnSavingGameSecondary`, `OnLoadingGameSecondary` and `OnMapChanging` drop a moving item first (`DestroyCurrentObject`).
- Furniture in the inventory is a `BuildingSystem.availableFurnitures` entry (`FurnitureInfo`: amount, `TaskItem`, optional saved object), created through `Furniture.CreateTaskItem` and `TaskItemsManager.AddTaskItem`; equal meta merges into one entry.
- `Door.Locked` proxies `_Lock.Locked`. `Lock.Use(callback, parent)` returns `true` (open), `null` (asynchronous: key delay or lockpick minigame) or `false` (locked, `onLockedUse` relay). `Lock.LockAfterTime` re-locks through a `TimeOfDayAzure.Timer` calling `Door.Close`; `increaseLockDifficulty` adds to `Difficulty` on each re-lock, capped at 2. `Door.UseDoor(bypassLock)` picks the swing direction from the player's side; `DoorLockBypass` calls it with `bypassLock: true`. `Door.Awake` adds a navmesh `GraphUpdateScene` (penalty 100, tag 3) and an occlusion portal update.
- Interaction prompts: `InteractObjects.RaycastInteract` (first person) or `ThirdPersonInteract` (sphere overlap) finds a target on a usable layer within `useDistance`; `GetInteractable` takes the first `IInteractable` on that object whose `IsInteractDisabled()` is false; `UpdateUseMessage` fills the prompt (`Interactable.useMessage` adds the parentheses); `UseInteractable` calls `Interact`.
- Since 0.5.13 the picking distance is `GetUseDistance(target)` = `useDistance` + `Interactable.extraInteractDistance` (per object, capped by `MaxExtraInteractDistance` 5); the ray length is max(`maxDistance`, `useDistance` + 5) and the sphere overlap radius `useDistance` + 0.3 + 5, filtered per collider; `IsWithinUseDistance(Interactable)` is public. `GetInteractable` also calls `IQuickSlotTarget.UpdateQuickSlot(hit)` on the hit components ([quick-slots.md](quick-slots.md)). `GameController` ignores "Primary Use" while `QuickSlotBar.HasPrimaryUseActive`.

## Pitfalls

- `GameObjectExtensions.GetInterfaces` looks only at `MonoBehaviour`s on the same GameObject: an `IInteractable` on a parent or child is invisible to `GetInteractable`, so the collider and the script must share a GameObject.
- Lock state logic shared by doors and containers lives in `Lock.Use`, not in `Door`.
- Saved: `Lock.Locked`, `Difficulty`, `Lockpickable`; not saved (`fsIgnore`): `keyItem`, `UnlockDelay`, relays, the timer object. Patch getters rather than changing saved fields.
- Hook points: `BuildingSystem.Place/Take/AddFurniture/RemoveFurniture`, `FurniturePlaceable.CanTake`, `Lock.Use`, `Door.UseDoor`, `InteractObjects.GetInteractable`.

## Unverified

Whether a placed `FurniturePlaceable` is parented to its `FurnitureManager`; where the lock check sits in `Door.Open`; who sets `keyItem` (prefab or scene data); whether a key is consumed on any path.
