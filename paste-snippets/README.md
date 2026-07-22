# Paste-ready snippets for Power Apps Studio

Per screen, two files:

- `<Screen>.children.yaml` — ALL controls for that screen, dedented so every
  `- ControlName:` starts at column 0. Select-all + copy the whole file, then in
  Studio (with the screen empty) press Ctrl+V on the canvas. Copy the entire file,
  never a partial selection - a paste fails if the first line loses its indentation.
- `<Screen>.properties.md` — the screen-level formulas (Fill, OnVisible, OnHidden...)
  to paste into the formula bar, plus `App.OnStart.fx.txt` for the App object.

Order: 1) App OnStart, 2) add the Loading screen and move it first, 3) each screen:
delete old controls, set screen properties, paste children.

- Loading
- Homepage
- Sales Nav List
- Sales Nav op window
- FLM Op Window
- Exec Op Window [ Untracked ]
- FLM Page
- Exec Page
- Summer Score
