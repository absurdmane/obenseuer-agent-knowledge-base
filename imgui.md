# IMGUI (mod menus) — pitfalls

Scope: Unity immediate-mode GUI lessons for in-game mod menus and small mod windows: window title placement, popups, rects across scroll views, layout. Tooling, not game knowledge.
Status: partly verified (items marked [verified] were seen in game)
Related: [dev-console.md](dev-console.md)

- **Window title**: the window style draws it at `padding.top + contentOffset.y`, and clips text to the area *inside* the
  padding. So with a tall title bar (big `padding.top`) the title either lands under the content or, pulled up with a negative
  `contentOffset`, is clipped away. Pass no title to `GUI.Window` and draw it in the window function (`Theme.DrawTitle`). [verified]
- **`GUI.Window` doesn't size itself**: a fixed rect clips content when padding or fonts grow. Small windows: `GUILayout.Window`
  (sizes to the content).
- **`GUI.ModalWindow` / a second `GUI.Window`** can end up behind the main window. For popups inside a window, draw them in the
  window itself: take input at the start of the window callback (before the page's controls see the click, `Event.Use()` on hits),
  paint at the end. Same as the hint (`Menu.Hint`). [verified: the ModalWindow dropdown appeared under the menu]
- **Rects across scroll views**: store them in screen GUI coordinates during `Repaint` (`GUILayoutUtility.GetLastRect` +
  `GUIUtility.GUIToScreenPoint`), convert back with `ScreenToGUIPoint` where they're drawn. Don't rely on `GetLastRect` in a click event.
- **Layout and events must build the same controls** in the same order every event of a frame (Layout, Repaint, mouse); decide
  wrapping and visibility from values that don't change mid-frame (e.g. the window rect is applied after `GUI.Window` returns).
- **Layout**: `BeginHorizontal`/`BeginVertical` (or `HorizontalScope`/`VerticalScope`) are Row/Column; `FlexibleSpace` centres.
  No gap, no wrap, no layers: wrap is computed by hand (`CalcSize`), overlays are `GUI.*` with absolute rects.
- **Horizontal scrollbar in a scroll view = some row is wider than the view.** A row's width is the sum of its children's
  minimum widths; text that doesn't word-wrap (labels without `wordWrap`, buttons, text fields sized to their text) can't shrink.
  Keep labels and list buttons word-wrapping, put long option rows in a dropdown, wrap button rows by hand (`Gui.WrapToggles`),
  cap text fields at the room left. A menu can log "... tab is N px wide" once per tab when it happens.
- **`GUI.Button` fires on MouseUp.** A "click outside closes it" handler on MouseDown runs first; remember what it closed so the
  same click on that popup's button doesn't reopen it.
- **Styles**: copies made with `new GUIStyle(GUI.skin.button)` before the theme is applied keep the old look; rebuild cached styles
  when the skin changes (`Gui.ResetStyles`). Rounded corners = a small rounded texture as a 9-slice (`border` = radius).
- **Glyphs**: the default font has `×` and `✓`, but may lack `✕`, `▲▼` and other symbols.
