# Quick slots (added in build 0.5.13)
Scope: the quick-slot bar (a scroll-wheel selector shown for some interactables), its owner API, input and saving; the only implementer found is the growing bed's quick plant. Scene and prefab values are not covered.
Status: code-only (new in 0.5.13; nothing played)
Related: [growing.md](growing.md), [doors-interaction.md](doors-interaction.md), [ui-options-input.md](ui-options-input.md), [item-system.md](item-system.md)

## Where it lives [code]

- `QuickSlotBar` (singleton `instance`; `Awake` hides its panel), `QuickSlotBarSlot` (5 slots, sizes 69/60/50 by distance from the centre), `QuickSlotOption` (`itemStack`, `data`, `disabled`, `disabledText`), `IQuickSlotTarget` (`UpdateQuickSlot(RaycastHit)`). Related new types: `ModelPreview` (`CreateCopy`), `FaceBlendShapeFollower` (not read).

## How it works [code]

- Owners call `Show(owner, provider, onSelect, actionName, repeatInterval)` and `Hide(owner)`; the provider returns a `List<QuickSlotOption>`. `ShowPreview(owner, prefab, parent, pos, rot, initialize, previewKey)`/`HidePreview(owner)` show a ghost, `Refresh()` re-reads the options. In `LateUpdate` the bar hides itself if the owner did not call `Show` in this or the previous frame (or was destroyed), so an owner must call `Show` every frame while active.
- Input: the axes "Mouse ScrollWheel" and "DPadHorizontal" move the selection; "Primary Use" confirms (and repeats every `repeatInterval` while held when it is above 0). It is ignored while a menu is visible, the game is paused, `keysDisabled` is set or a dialogue is active; `GameController` also ignores "Primary Use" for its own purposes while `QuickSlotBar.HasPrimaryUseActive`. A disabled option plays a warning (`UseText.PlayWarningSound`) and shakes; `onSelect` is not called.
- Binding: `InteractObjects.GetInteractable` calls `IQuickSlotTarget.UpdateQuickSlot(hit)` on the hit interactable's components, and for `grow`/`GrowItem` falls back to `Growing.UpdateQuickSlot` on the parent. `Growing` is the only implementer found (`Show(this, provider, select, "plant", 0.2f)`, options carry `data = (Recipe, ItemStack)`, skill-locked recipes are disabled with a text, `ShowPreview` shows the plant prefab).
- Nothing is saved; `QuickSlotBar.lastSelected` is static and lasts for the session. `Growing` calls `QuickSlotBar.instance.Show` without a null check.

## Unverified

`ModelPreview` and `FaceBlendShapeFollower` internals; whether any prefab other than growing beds uses the bar.
