# Attach Wizard — "Clean Enterprise Card" Re-skin

A full visual re-skin of the **Attach Wizard** Power Apps canvas app (1136 × 640) to the
Clean Enterprise Card design system: soft grey canvas, white rounded cards, one brand
accent, big-number/small-label hierarchy, and a shared entrance/idle motion language.

**Every data binding, filter, collection build, Patch, timer choreography and navigation
target is preserved verbatim from the original app.** Only presentation changed.

## What's in here

```
Src/
  App.pa.yaml                          shared design-system assets (OnStart)
  Loading.pa.yaml                      NEW branded splash screen (spinner + Enter)
  Homepage.pa.yaml                     hub: welcome hero, 3 role cards, Summer Cup banner
  Sales Nav List.pa.yaml               pipeline list, campaign code, notifications, Wiz chat
  Sales Nav op window.pa.yaml          opportunity detail + service tracker (rep view)
  FLM Op Window.pa.yaml                same, FLM variant (generated clone)
  Exec Op Window [ Untracked ].pa.yaml same, Exec variant (generated clone)
  FLM Page.pa.yaml                     team dashboard: KPis, SSP chart, rep progress
  Exec Page.pa.yaml                    exec dashboard: pivot, team progress, KPIs
  Summer Score.pa.yaml                 leaderboard: podium, attainment, category tiles
DESIGN-SYSTEM.md                       tokens, motion classes, category palette mapping
```

## How to apply

The `.pa.yaml` files follow the Power Apps canvas YAML source schema (same format as the
`Src/` folder inside a `.msapp`). Apply with the Power Platform CLI / git-integration
YAML editing (preview), or copy the per-control `Image` / property formulas into Power
Apps Studio.

Two setup notes:

1. **Screen order** — make `Loading` the first screen in the app so the branded splash
   shows on launch. Its repeating timer auto-navigates to `Homepage` the moment
   `PowerBIIntegration.Data` has rows; the Enter button is the manual fallback.
2. **App.OnStart** — replace your OnStart with `Src/App.pa.yaml`'s version. It keeps all
   original state priming and adds `varSvgCss` (shared animation stylesheet),
   `varShadowFilter` (soft-shadow recipe), and the `varNavBusy` / `varWipeKey` pair that
   drives the page-transition wipe.

## What was deliberately changed (presentation only)

- **Dark theme → light card system.** Screen fills are now `#f7f7f7`; every panel is a
  white rounded card (radius 13–16, 1 px `#d4d8db` hairline, soft blurred shadow).
- **Top ribbon** on every screen: brand glyph + `Attach Wizard · <view>` wordmark and an
  identity pill with the signed-in user's initials. The old reverse-color HPE logo images
  (`Image2`, `Image2_1..3`) were folded into the ribbon and removed.
- **Page transition wipe**: an accent-gradient overlay (`WipeOverlay_*`) sweeps over each
  navigation, keyed by `varWipeKey`, on top of the platform Cover/UnCover transitions.
  A hidden 1.1 s timer clears the flag; navigation never depends on the wipe.
- **Row/state treatments** in the Sales Nav list: *tracked by me* = pale-tint fill
  `#d1ffee` + 2 px accent border + `YOURS` chip; *tracked by someone else* = neutral chip
  with the editor's name; *untracked* = `NEW` chip. Same underlying
  `LookUp('Attach Attack', …)` logic as before.
- **In-formula chart fragments** (`varChartBars` in the op windows' GBU chart,
  `varSSPBars` and `varWonCycleText` built in the FLM/Exec `OnVisible`) keep their exact
  data math; only fill/colour attributes were re-tokenised.
- **Hidden legacy controls dropped**: labels/rectangles with `Visible: =false` that
  duplicated SVG content, and the permanently hidden `Gallery5*` GBU list. The visible
  overlay labels with timer-staged reveals (`Label8*`) are kept and restyled.
- The opportunity ID in the op-window headline is now drawn inside the headline SVG
  (the old selectable `HtmlText1` was removed); the ID also appears in the Service
  Tracker header.
- Small copy fixes where text is purely presentational (e.g. "Elligible" → "ELIGIBLE").

## What was NOT changed

- `OnVisible` / `OnHidden` blocks (spliced verbatim, including `timerotoole`/`timerdog`
  choreography and all collection builds).
- All `Items` filters, `Patch` calls ('Attach Attack', noti choices, ChatRequests), the
  Wiz chat request/poll loop, tooltips' semantic text, timer `Start`/`Reset`/durations,
  and the `borderColor` discriminator values (`#00E0AF` / `#62E5F6`) inside
  `productCollection` — they are filter keys, so they stay; rows map them to display
  colours at render time.
- Navigation targets and screen transitions.
