# Clean Enterprise Card — as applied to Attach Wizard

Canvas: locked **1136 × 640** (unchanged from the original app).
Brand accent: the app's existing HPE green already matches the system's default ramp, so
the accent family is used as-is.

## Colour tokens

| Role | Value |
|---|---|
| Page canvas | `#f7f7f7` |
| Card / surface | `#ffffff` |
| Contrast fill (chips, empty avatars) | `#f2f3f4` |
| Sub-card tint (nested tiles) | `#fbfcfc` |
| Hairline border | `#d4d8db` |
| Primary text | `#292d3a` |
| Secondary text | `#3e4550` |
| Muted / captions | `#606a70` |
| Brand accent | `#01a982` |
| Accent dark (headline numbers) | `#006750` |
| Accent deep | `#018f6e` |
| Accent darkest | `#06372e` |
| Accent bright | `#7cf5cf` |
| Accent pale tint (mine/selected) | `#d1ffee` |
| Accent soft tint | `#bdf5e4` |
| Warning text / tint | `#d36d00` / `#FBEEDF` |

### Categorical palette (in order)

| # | Colour | Meaning in this app |
|---|---|---|
| 1 | `#0070f8` | Products (PL class) · "CC / Day 1" filter · EXPAND points |
| 2 | `#04909d` | Services OS · "No Services Op" filter · ATTACH points |
| 3 | `#009a71` | Services Non OS · "Low Pen Rate" filter · BACKLOG points · tracked lists |
| 4 | `#7764fc` | Won value · ENGAGE points |
| 5 | `#cc54a4` | Untracked opportunities |
| 6 | `#d25f4b` | At risk / low completion (<50 %) |

Progress-bar health mapping (rep/team feedback): ≥80 % → accent `#01a982`,
≥50 % → `#04909d`, else `#d25f4b`. Pivot "untracked" heat: ≥0.75 of max → `#d25f4b`,
≥0.5 → `#d36d00`, ≥0.25 → `#E0A300`, else muted.

Rank / medal (Summer Score podium): gold `#E0A300`/`#FBF1D6`, silver `#8C99A4`/`#EDEFF1`,
bronze `#B5742E`/`#F4E9DD`.

## Motion

One shared stylesheet string, `varSvgCss`, is set in `App.OnStart` and injected into
every SVG (`<defs><style>" & varSvgCss & "</style>…`). Classes:

| Class | Effect | Spec |
|---|---|---|
| `.fu` | fade-up (cards) | opacity 0→1, translateY(12px)→0, .55s cubic-bezier(.2,.7,.2,1) both |
| `.fo` | fade-in (nested) | opacity 0→1, .5s ease-out both |
| `.pp` | pop (avatars/chips) | opacity 0→1, scale(.5)→1, .5s cubic-bezier(.2,.7,.2,1) |
| `.gx` | grow-x (bars) | scaleX(0)→1, .65s cubic-bezier(.2,.7,.2,1), origin left |
| `.rw` | row-wipe (list rows) | opacity 0→1, translateX(-18px)→0, .5s |
| `.ring` | idle halo pulse | scale .75→2, opacity .45→0, 2.8s infinite |
| `.brz` | breathe | opacity .9↔.45, 2.6s infinite |
| `.bob` | bob + tilt | translateY(0↔-2px) rotate(-4↔4deg), 2.4s infinite |
| `.spin` | loading ring | rotate 360°, 1s linear infinite |

Staggering is per-sibling `style='animation-delay:0.10s'` (+~0.06–0.08 s each); podium
winner reveals last. The stylesheet ends with the reduced-motion guard
(`@media(prefers-reduced-motion:reduce){…{animation:none}}`). SVG-local keyframes
(`wave`, `blink`, `bub1/2` on Wiz; `wp`, `tmb` on the wipe; `barGrow`, `fadeUp` in the
GBU chart) each carry their own guard.

## Shape & elevation

- Cards: rx 13–16, `#ffffff`, 1 px hairline; soft shadow = black rect, opacity .06–.10,
  offset +4 px, `feGaussianBlur stdDeviation='5'` (`varShadowFilter`).
- Nested tiles: rx 10, `#fbfcfc`, hairline, 5 px category left-bar (rx 2.5) + 3.5 r dot.
- Chips/pills: fully rounded, `#f2f3f4` or status tint, 9–11 px semibold.
- Avatars: accent circle + white initials for "you"; `#f2f3f4` + accent-dark otherwise.
- Mine/selected: `#d1ffee` fill + 2 px `#01a982` border + `YOURS`/`YOU` accent chip.

## Rendering conventions

- Every visual is a data-driven SVG on an Image control:
  `"data:image/svg+xml;utf8," & EncodeUrl("<svg …>…</svg>")`, single-quoted attributes,
  `ImagePosition.Fit` (wipe overlay uses `Fill`), viewBox matched to control aspect.
- Animation replay: each SVG embeds `<desc>k=" & varAnimKey… & "</desc>`; bumping the key
  in `OnVisible`/`OnSelect` changes the image string and replays the entrance.
- Human text is XML-escaped via `Substitute` chains and truncated to fixed budgets before
  interpolation; numerics coerced with `IfError(…, 0)`; all geometry via `Text(Round(…))`
  with `[$-en-US]` formats where decimals are required.
- Wiz chat bubbles: HTML text control — right-aligned accent bubble (user) vs left white
  hairline bubble (Wiz), `white-space:pre-wrap`, radius 14/4.
- Galleries: `TemplatePadding = 0`, `ShowScrollbar = false`, one SVG image per row.
- Landing: `Loading` screen (spinner ring = `#d4d8db` track + accent arc, `.spin`) with a
  repeating timer that navigates when `PowerBIIntegration.Data` is ready + manual Enter.
- Wipe: `WipeOverlay_*` image (accent-dark→accent gradient, translateX −112 %→0→112 %,
  1.1 s) shown while `varNavBusy` is true; `tmrWipe_*` clears the flag. Real navigation
  still uses the platform Cover/UnCover transitions.
