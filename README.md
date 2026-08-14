# C2B Hub Incentive — Yard Notice

Single-page printable notice, one sheet per hub. Static site, no build step.

## Files

| File | What it is |
|---|---|
| `index.html` | The notice. Hub selector, English/Hindi toggle, print button. |
| `hubs.json` | The data. **This is the only file you change each month.** |
| `assets/favicon.svg` | Browser-tab icon — the Cars24 icon mark. |

`index.html` has a copy of the data baked in as a fallback, so the file still works
if someone opens it straight off their desktop. When served over http it fetches
`hubs.json` and uses that instead.

## Deploy

1. Create a **private** GitHub repo and push these files to the root.
2. On vercel.com, sign in with GitHub → **Add New → Project** → import the repo.
3. Framework Preset: **Other**. Leave Build Command and Output Directory empty.
4. Deploy.

Every push to `main` redeploys automatically.

## Monthly refresh

Replace `hubs.json`, commit, push. Vercel rebuilds in seconds. No code change.

Each hub object:

```json
{
  "hub": "Noida sec-37 Parking",
  "league": "A",
  "heads": 11,
  "units9": 451,
  "qtr": 4510,
  "recv": 79.4,
  "disp": 93.4,
  "dg": 3.8,
  "dgt": 17,
  "yms": 79.1,
  "ymsx": false,
  "bar": { "recv": 83, "disp": 90, "dg": 5, "yms": 70 },
  "ch": [false, true, true, true],
  "passed": 3,
  "q": 0.75
}
```

- `units9` — cars stocked in during the measurement window
- `qtr` — cars projected across the quarter (drives the pool)
- `recv` / `disp` / `yms` — percentages. `dg` — dealer complaints per 100 cars
- `ymsx` — `true` where YMS is not available at the hub, which auto-passes that check
- `bar` — that hub's four thresholds
- `ch` — pass/fail per check, in order: documents in, documents out, complaints, yard app
- `passed` — count of `ch` that are true
- `q` — what the team keeps: 1.0 / 0.75 / 0.5 / 0

`ch`, `passed` and `q` are computed upstream, not in the page. Keep them consistent
with `bar` or the notice will show a stamp that disagrees with the payout.

## Design

The page is built on the **Cars24 Lego design system**. The tokens are inlined in
`index.html` — there is no build step and no external stylesheet — in the same four
layers the system defines:

1. **Primitives** (`:root`) — raw hex, spacing, border and radius values. Never used
   directly by a component.
2. **Semantic** (`[data-brand="cars24"]`) — intent-named aliases onto those primitives:
   `--lego-color-surface-*`, `--lego-color-text-*`, `--lego-color-border-*`.
3. **Typography** — the Lego type scale as classes (`.h1`, `.label-2`, `.body-3`, …),
   mobile sizes with desktop overrides at `1024px`. Letter-spacing is `0` throughout,
   per the scale. Brand font is **Geist**; `IBM Plex Sans Devanagari` carries the Hindi
   copy, which Geist does not cover.
4. **Components** — everything else.

If you edit the CSS, keep to the tokens: no raw hex, no raw px for
padding/gap/radius/border-width, no bare `font-weight` numbers. The only literal
pixel values below the token blocks are media-query breakpoints, the page
`max-width`, a flex basis, and the screen-reader clip.

The logo is inlined as SVG rather than referenced from the sprite, because Chromium
does not paint a sprite `<use>` reference when printing.

### Print

The notice is designed to be **one A4 sheet per hub**. A4 is narrower than the mobile
breakpoint, so `@media print` re-forces the wide grids and tightens the section
padding; without that it spills onto a second page. The tallest hub currently renders
at ~1000px against ~1032px of usable A4 height, so there is not much headroom — if you
add a row or a section, re-check that it still prints on one page.

## Rate

The per-car rate is set in `index.html` (`let RATE=15;`). Change it there if the
rate is revised, and re-announce before the quarter starts, not during.

There is also a **₹5,000 per-person cap** in the page
(`Math.min(5000, pool / h.heads)`). Where it bites, the "Each person gets" cell shows
₹5,000 while the four cells to its left still multiply out to more than that — the
notice shows arithmetic that does not produce its own answer. On the August data this
affects one hub (Hyderabad-2: ₹6,262 uncapped). Either state the cap on the notice or
drop it from the calculation; leaving it silent invites the question at the yard.

## Before you make this public

The page contains hub headcount, performance numbers and per-person payout figures.
A Vercel deployment on a free plan is a public URL — unlisted, but not protected,
and indexable. Client-side passwords are not protection here: the data sits in the
page source. If this needs to stay internal, use Vercel's Deployment Protection
(password or SSO, paid plans) or host it behind the internal network instead.
