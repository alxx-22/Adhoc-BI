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

## Revision 2 — Wiz removal, visual debugging, performance

This revision was driven by **rendering every screen** (each screen's controls laid out at
their real coordinates on a 1136×640 canvas, each data-driven SVG rasterised with headless
Chromium) and inspecting the images for overlap/alignment. Changes:

**Wiz assistant removed (feature retired).** Deleted the `WIZBOT` mascot, `wizbot box`,
`Container2` and all its children (chat gallery, HTML bubbles, input, send button, typing
timer) and the `tmrPoll` polling timer from Sales Nav List; stripped the chat priming
(`colChat`, `varSessionID`, `varWaiting`, `varScrollTick`) from `App.OnStart` and the
`colChat` build from Sales Nav List `OnVisible`. The **`ChatRequests` data source is now
unused** and can be removed from the app's connections.

**Visual fixes (from the renders):**
- *Op windows* (all three): the Services-Zone right column had the "Services Pen Rate"
  tiles overlapping the "Launch Service Tracker" button — rebalanced the card/tile/button
  heights so they stack cleanly.
- *FLM Page*: the "Filter By Rep" / "Forecast Call" chips overlapped the "Tracked
  Opportunities" card header — moved the card and its gallery down so the header clears
  the chips.
- Homepage, Exec Page, Summer Score and Loading rendered clean and were left unchanged.

**Performance (presentation-safe, results identical):**
- Galleries no longer hit SharePoint per row. Per-row `LookUp('Attach Attack', …)` calls
  were repointed to the in-memory copies already built in `OnVisible` — Sales Nav row card
  → `colHasRecord`; FLM `Gallery3` filter → `colAAOppIds`; Exec `Gallery3_1` filter →
  `colHasRecord`. These collections are full copies of the table, so every row resolves in
  memory instead of a delegated server call, and it also sidesteps the delegation cap.
- `App.OnStart` no longer initialises the chat collection or session/polling state.
- Removing Wiz also removed a 1-second repeating timer (`tmrPoll`) and ~16 KB of controls
  from Sales Nav List.

### Optional follow-up: factor the XML-escape with a UDF

Every SVG escapes interpolated text with the same `Substitute(...)` chain (~50 sites). It
is safe as-is, but if you want to consolidate it, add these user-defined functions to
**App → `Formulas`** (requires the *User-defined functions* setting; test in Studio before
converting call sites, since a bad `Formulas` block prevents the app from opening):

```powerapps
EscapeXml(s:Text): Text =
    Substitute(Substitute(Substitute(Substitute(Substitute(
        s, "&", "&amp;"), "'", " "), """", " "), "<", "&lt;"), ">", "&gt;");
Clip(s:Text, n:Number): Text =
    If(Len(s) > n, Left(s, n) & "..", s);
```

Then a site like `With({c: Substitute(...large chain...)}, If(Len(c)>40, Left(c,40)&"..", c))`
becomes `Clip(EscapeXml(ThisItem.'Account Name'), 40)`. Left as a documented opt-in rather
than applied blind, because it can't be validated outside Power Apps Studio.

## Revision 3 — line-items Products fix + view-toggle layout

- **Products now show in the Line Items panel.** The line-item classification keyed
  `isProduct` off an exact `PLCODELOOK = "Product"` match, so if the FY26_BA_Hierarchy tags
  products with any other value the Products tab came up empty while OS / Non OS worked.
  `isProduct` is now defined as *"any line that is not a service"*
  (`pl <> "Service OS" && pl <> "Service Non OS"`) in all three op windows — OS / Non OS
  behaviour is unchanged, and products always appear regardless of the hierarchy's exact
  product label. Product rows also render with a blue category bar so they read distinctly.
- **Line Items / GBU Chart toggle relocated.** The two buttons were cramped into the
  headline card's bottom-right corner, overlapping the upsell chip. They're now a single
  labelled `VIEW  [ Line Items ] [ GBU Chart ]` segmented control in the top bar between the
  wordmark and the Back button, each with an icon, and each still shows its active state
  (accent fill when its panel is open).

## Revision 4 — blank gallery rows fixed (truncate-then-escape)

Some line-item / gallery rows rendered blank while the gallery still reserved their height —
the signature of an invalid per-row SVG. Root cause: the text cleaners **escaped first, then
truncated** (`With({c: Substitute(…escape…)}, If(Len(c) > N, Left(c, N) & "..", c))`). Escaping
turns `&` into `&amp;`; when `Left(c, N)` cut through an entity the row's SVG contained a broken
`…&am`, which is invalid XML, so Power Apps rendered that Image blank.

Fixed by reordering every cleaner to **truncate the raw value first, then escape**
(`Substitute(…escape( If(Len(raw) > N, Left(raw, N) & "..", raw) )…)`), so the cut can never
land inside an entity. Applied to all 43 cleaner sites across the op windows, both dashboards,
Summer Score and the Sales Nav list, plus the ribbon user-name. Verified: names with `&`/`<`
at the truncation boundary that previously went blank now all produce valid XML.

## Revision 5 — Exec Summary "Coverage by GBU" card

The Exec Summary had an empty top-right quadrant (right of the Manager/BU pivot, above the
KPI cards). Added a **Services OS Coverage by GBU** card there: the headline *% of OS value
tracked*, plus a per-GBU stacked bar (tracked green vs untracked magenta) for the top 5 GBUs
with the untracked value called out. It answers the one exec question the page didn't — how
much of the pipeline has feedback and which business units are lagging.

It uses **no new data source**: `OnVisible` aggregates a `colCoverageGBU` collection from the
`colTrackedGBU` / `colUntrackedGBU` collections already built on the page (combined, grouped by
GBU), plus `varCovPct` / `varCovBars`. If those collections are empty the card shows a
graceful "No GBU coverage data yet" placeholder.

## Revision 6 — Rep & Manager "OS Coverage & Gaps" visuals

Turned the feedback-progress breakdown into two **coverage/gaps** visuals:

- **FLM page** — the "Rep Feedback Progress" card became **REP OS COVERAGE & GAPS**: per rep,
  a tracked-vs-untracked OS-value split bar, the untracked $ gap, and the count of *untracked*
  opportunities by target type (CC / Day 1 Upsell · Low Pen Rate · No Services Op).
- **Exec page** — the "Team Feedback Progress" card became **MANAGER OS COVERAGE**, the same
  breakdown at the Manager entitlement grain.

**No new Power BI columns or measures are required** (see Revision 8 for the history). Both
collections are built in `OnVisible` the same way the app's original "feedback progress" visual
was — by grouping `PowerBIIntegration.Data` directly (`colFPDetail` in FLM, `colMgrDetail` in Exec) —
using only fields the app's Power Apps visual already exposes (`Feedback Progress`,
`Manager entitlement`, `HPE Opportunity Id`, `Services OS`, `Target Opp?`). "Tracked" is the app's
own definition (`'HPE Opportunity Id' in colAAOppIds.'Opp ID'` — the opp has a row in
`Attach Attack`), identical to the existing `colRepCompletion`, so the numbers reconcile with the
other cards and the breakdown responds live to a Power BI report filter (see Revision 8).

## Revision 7 — three post-import bug fixes

Fixes for issues found after importing Revision 6.

**1. Line-item gallery rows with `&` in the GBU rendered blank.** The line-item row SVG
uppercased the GBU with `Upper(cleanGBU)` *after* `cleanGBU` had already been XML-escaped, so
a GBU like **"HPC & AI"** became `HPC &AMP; AI` — and XML entities are case-sensitive, so
`&AMP;` is invalid and Power Apps rendered that Image blank. Fixed by uppercasing the **raw**
GBU before escaping (`Upper(If(Len(ThisItem.GBU) > 24, …))`) and interpolating the already-cased
`cleanGBU` directly. The same escape-then-`Upper` pattern in the FLM/Exec/Summer "WELCOME BACK,
`<name>`" headers was fixed the same way (uppercase the raw first name, then escape).

**2. "Index function cannot be called with an empty table" on the dashboard/op-window pages.**
Every packed column is parsed as `ForAll(Split(pack, " || "), { f: Index(Split(Value, " // "), N).Value … })`.
When a packed segment is blank or has fewer fields than expected, `Index(Split(…), N)` throws at
runtime and errors the whole screen. Hardened **every** such parse (the original packs *and* the
new FP/Manager packs) so a malformed segment degrades to a blank/zero instead of crashing:
- each inner `Index(Split(Value, …), N)` extraction is wrapped in `IfError` (`0` for numeric
  `Value(...)` sites, `""` for text sites);
- the new FP/Manager parses also drop blank segments up front
  (`Filter(Split(pack, " || "), Len(Trim(Value)) > 0)`);
- the Summer Score podium's unguarded `Index(colEngagementRanked, N)` (which crashed when the
  leaderboard had fewer than 3 people) became `Last(FirstN(colEngagementRanked, N))`, which returns
  a blank row out of range instead of erroring.
This is a pure robustness change — for well-formed data every `IfError` returns the original value.

**3. The two new Exec Summary visuals weren't loading.** The `colCoverageGBU` aggregation used
`GroupBy(colCovRaw, "GBU", "grp")` with **string** column-name arguments, but this app's Power Fx
uses the identifier form (its own working code is `GroupBy(colWonGBU, 'GBU', 'GBURows')`), so the
formula didn't compile and `OnVisible` left the collection empty — the Coverage-by-GBU card fell
back to its "No GBU coverage data yet" placeholder. Fixed to `GroupBy(colCovRaw, 'GBU', 'grp')`.
The Manager OS Coverage card is fed by the `'Manager Detail Pack'` column (Revision 6) and its
parse is now hardened per fix #2 — if that card is still empty, confirm the column exists in the
dataset (`powerbi/detail-packs.tmdl`) and has been refreshed.

## Revision 8 — coverage visuals derive from PowerBIIntegration.Data (restores dynamic filtering)

The FLM "REP OS COVERAGE & GAPS" breakdown stopped responding to a Power BI **report filter on
`[Entitled Manager Name]`** — applying that filter no longer narrowed the feedback progress to that
manager's reps.

Root cause: Revision 6 fed these visuals from a new pre-aggregated DAX **pack** (a calculated
column, `'FP Detail Pack'`, wrapped in `ALL('App Summarised table')`). A calculated column is
computed once at refresh and is static at query time, and `ALL(...)` strips every filter anyway —
so it could never respond to a report filter.

The app already had a working pattern for exactly this — the original "feedback progress" visual
(`colRepCompletion`) never used a pack; it groups `PowerBIIntegration.Data` **directly** in
`OnVisible`. Because that data is filtered by the report/page, grouping it live means a report filter
on `[Entitled Manager Name]` narrows it automatically. Fixed by rebuilding both new collections the
same way:

- `colFPDetail` = `GroupBy(PowerBIIntegration.Data, 'Feedback Progress', 'OppRows')`, then per rep:
  tracked/untracked OS = `Sum('Services OS')` split on whether the opp is in `Attach Attack`
  (`'HPE Opportunity Id' in colAAOppIds.'Opp ID'`), and CC/LP/NS = counts of untracked opps by
  `'Target Opp?'`.
- `colMgrDetail` = the same, grouped by `'Manager entitlement'` (Exec).

**No Power BI change and no new fields** — it uses only columns the app's visual already exposes
(`Feedback Progress`, `Manager entitlement`, `HPE Opportunity Id`, `Services OS`, `Target Opp?`), and
"tracked" is the app's own `Attach Attack` membership, so the numbers reconcile with the other cards.
The Revision 6 `powerbi/detail-packs.tmdl` is removed — it is no longer needed.

## What was NOT changed

- `OnVisible` / `OnHidden` blocks (spliced verbatim, including `timerotoole`/`timerdog`
  choreography and all collection builds).
- All `Items` filters, `Patch` calls ('Attach Attack', noti choices, ChatRequests), the
  Wiz chat request/poll loop, tooltips' semantic text, timer `Start`/`Reset`/durations,
  and the `borderColor` discriminator values (`#00E0AF` / `#62E5F6`) inside
  `productCollection` — they are filter keys, so they stay; rows map them to display
  colours at render time.
- Navigation targets and screen transitions.
