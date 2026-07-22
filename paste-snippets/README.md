# Paste-ready snippets for Power Apps Studio

- `<Screen>.children.yaml` — ALL controls, dedented so every `- ControlName:` starts at
  column 0. Copy the whole file, Ctrl+V on the empty screen canvas.
- `<Screen>.properties.md` — screen-level formulas for the formula bar; `App.OnStart.fx.txt`
  for the App object.

Order: 1) App OnStart, 2) add Loading screen (move first), 3) each screen: delete old
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
