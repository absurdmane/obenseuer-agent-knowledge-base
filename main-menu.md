# Main menu

Scope: the title-screen main menu: its canvas, buttons and gamepad navigation. Not covered: the options, load and character sub-menus beyond their names, the in-game pause menu.
Status: partly verified
Related: [imgui.md](imgui.md)

## Layout [verified] (UI inspector dump, 2026-10-20)

- The menu is drawn on a sign in the scene, not as a screen overlay: canvas `Menu` at `Other/Sign Menu/sign_003/Menu` is
  `renderMode=WorldSpace`, `localScale=0.001`, rect 7300x550, `CanvasScaler` ConstantPixelSize, `GraphicRaycaster`, `CanvasGroup`.
- `Menu` carries `MainMenu`, `MainMenuNavigationHandler`, `Animator`, `AudioSource`. Its child `Main Menu` (rect 100x100,
  centred) carries `MainMenuNavigationHelper` and holds the 6 buttons.
- Buttons (children of `Main Menu`, in sibling order): `New Game`, `Load Game`, `Options`, `???`, `Credits`, `Quit`. A 3x2 grid:
  top row New Game / Load Game / Options, bottom row ??? / Credits / Quit. `Main Menu` has no layout group: buttons are placed
  by hand.
- Each button: `Image` (UISprite, grey 0.33), `ContentSizeFitter` PreferredSize/PreferredSize, `VerticalLayoutGroup`,
  2 children incl. a TMP text (`Text (TMP)`); height 60.5, width follows the text (83–210).

## Code

- `MainMenu` (singleton `MainMenu.Instance`): button handlers `NewGame`, `LoadGame`, `Options`, `Credits`, `Quit`, `Back`;
  `currentMenu` (`CurrentWindow`) tracks the open sub-window.
- `MainMenuNavigationHelper : UINavigationHelperBase`: gamepad/keyboard navigation with six serialized button fields
  (`newGame`, `loadGame`, `options`, `unknown`, `credits`, `quit`). Up/down/left/right are hard-coded per button
  (`GetButtonAbove` etc.); the default selection is `newGame` (`OnEnable`).

## Mod guidance

- A new button cloned into `Main Menu` gets mouse clicks through the existing raycaster, but gamepad navigation will not reach
  it unless the `GetButton*` methods of `MainMenuNavigationHelper` are patched to include it.
- Because the canvas is world-space, a cloned button must keep the sign's scale; a screen-space overlay is a separate option.
