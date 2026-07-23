# Paste-ready snippets for Power Apps Studio

- `<Screen>.children.yaml` — ALL controls, dedented so every `- ControlName:` starts at col 0.
  Copy the whole file, Ctrl+V on the empty screen canvas.
- `<Screen>.properties.md` — screen-level formulas; `App.OnStart.fx.txt` for the App object.

NOTE: FLM Page and Exec Page build their "OS Coverage & Gaps" visuals by grouping
PowerBIIntegration.Data directly in OnVisible (colFPDetail / colMgrDetail), using
only fields the app's Power Apps visual already exposes ('Feedback Progress',
'Manager entitlement', 'HPE Opportunity Id', 'Services OS', 'Target Opp?'). No new
Power BI column or measure is required, and the breakdown responds to a report
filter on [Entitled Manager Name] (see README Revision 8).

Order: 1) App OnStart, 2) Loading screen (move first),
3) each screen: delete old controls, set screen properties, paste children.

- Loading
- Homepage
- Sales Nav List
- Sales Nav op window
- FLM Op Window
- Exec Op Window [ Untracked ]
- FLM Page
- Exec Page
- Summer Score
