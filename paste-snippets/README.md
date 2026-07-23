# Paste-ready snippets for Power Apps Studio

- `<Screen>.children.yaml` — ALL controls, dedented so every `- ControlName:` starts at col 0.
  Copy the whole file, Ctrl+V on the empty screen canvas.
- `<Screen>.properties.md` — screen-level formulas; `App.OnStart.fx.txt` for the App object.

IMPORTANT: FLM Page and Exec Page now read two new Power BI fields
('FP Detail Pack', 'Manager Detail Pack'). Define them as MEASURES (not
calculated columns — a column can't respond to a report filter) and add them
to the dataset first (see ../powerbi/detail-packs.tmdl) and refresh, or those
two screens error.

Order: 1) add the PBI measures + refresh, 2) App OnStart, 3) Loading screen (move first),
4) each screen: delete old controls, set screen properties, paste children.

- Loading
- Homepage
- Sales Nav List
- Sales Nav op window
- FLM Op Window
- Exec Op Window [ Untracked ]
- FLM Page
- Exec Page
- Summer Score
