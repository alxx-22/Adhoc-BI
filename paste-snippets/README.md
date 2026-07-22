# Paste-ready snippets for Power Apps Studio

Per screen, two files:

- `<Screen>.children.yaml` — ALL controls, dedented so every `- ControlName:` starts at
  column 0. Select-all + copy the whole file, then Ctrl+V on the empty screen canvas.
- `<Screen>.properties.md` — screen-level formulas for the formula bar, plus
  `App.OnStart.fx.txt` for the App object.

Order: 1) App OnStart, 2) add Loading screen and move it first, 3) each screen: delete old
controls, set screen properties, paste children.

- Loading
- Homepage
- Sales Nav List
- Sales Nav op window
- FLM Op Window
- Exec Op Window [ Untracked ]
- FLM Page
- Exec Page
- Summer Score
