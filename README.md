# C2B Hub Incentive — Yard Notice

Single-page printable notice, one sheet per hub. Static site, no build step.

## Files

| File | What it is |
|---|---|
| `index.html` | The notice. Page header, filter bar, collapsible sections, English/Hindi toggle, print button. |
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
  "n": 4,
  "recv": 79.4,
  "disp": 93.4,
  "dg": 3.8,
  "dgt": 17,
  "yms": 79.1,
  "ymsx": false,
  "base": null,
  "bar": { "recv": 85.0, "disp": 100.0, "dg": 5, "yms": 70 },
  "ch": [false, false, true, true],
  "passed": 2,
  "q": 0.5
}
```

- `n` — how many checks apply at this hub, `4` or `3`. Hubs without a yard app are
  measured on 3 and use a different ladder (all 3 / 2 of 3 / 1 or 0, keeping
  everything / **two-thirds** / nothing). Absent is treated as `4`.
- `units9` — cars stocked in during the measurement window
- `qtr` — cars projected across the quarter (drives the pool)
- `recv` / `disp` / `yms` — percentages. `dg` — dealer complaints per 100 cars
- `ymsx` — `true` where YMS is not available at the hub. Vestigial: `n` now drives
  this, and a 3-check hub simply has no fourth card. Kept because the upstream
  export still emits it.
- `base` — currently `null` for every hub and unused by the page
- `bar` — that hub's thresholds, per hub (they are no longer flat per league).
  `bar.yms` is `null` on 3-check hubs
- `ch` — pass/fail per check, in order: documents in, documents out, complaints, yard
  app. Length matches `n`
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

Motion uses its own small token set (`--ease-*`, `--dur-*`, `--stagger`). The Lego
system does not define motion, so these are local, but they are tokens so timing
stays uniform.

### Interaction

- **Page header** — sticky, carries the logo, the language toggle and Print.
- **Filter bar** — four controls: hub, league, what the team keeps, and sort order.
  League/keeps narrow the hub list; the count under the bar shows how many of the
  total survive the filters. If a filter combination matches nothing, the sheet shows
  an empty state rather than a stale hub.
- **Dropdowns** are a custom listbox, not a native `<select>`, so the hub picker can be
  searched. Keyboard: `↑`/`↓`/`Home`/`End` to move, `Enter` to pick, `Esc` to close,
  type to filter. The list is `role="listbox"` with `aria-activedescendant` on whichever
  element holds focus — the search input when the dropdown has one, otherwise the trigger.
- **Sections collapse.** Open state lives outside the render, so it survives changing
  hub, filter or language. A collapsed section shows its headline number in the header.

Hub names come from `hubs.json` and are escaped before they reach `innerHTML`.

### Motion caveat worth knowing

The money figures count up on each render. `settle()` jumps every counter and meter to
its final value, and it is wired to `beforeprint` so a print started mid-animation still
prints final numbers. It also bumps a generation counter, which is what stops a queued
animation frame from overwriting the settled values — without that, printing within
~540ms of changing hub printed a wrong payout. If you add another animated value, route
it through `settle()` too.

Everything animated is disabled under `prefers-reduced-motion: reduce` and in print.

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

Print also ignores the screen state: the page header and filter bar come off, the
sheet carries the logo itself, and **collapsed sections print open**. Someone printing
with a section collapsed still gets the whole notice.

## Rate

The per-car rate is set in `index.html` (`let RATE=15;`). Change it there if the
rate is revised, and re-announce before the quarter starts, not during.

There is also a **₹5,000 per-person cap** in the page
(`Math.min(5000, pool / h.heads)`). On the current data it never binds — the highest
per-person figure is ₹4,175 — so nothing on the notice contradicts itself today. It
would bind again if payouts rose, and when it does the "Each person gets" cell shows
₹5,000 while the four cells to its left still multiply out to more. If that happens,
either state the cap on the notice or drop it from the calculation.

## Before you make this public

The page contains hub headcount, performance numbers and per-person payout figures.
A Vercel deployment on a free plan is a public URL — unlisted, but not protected,
and indexable. Client-side passwords are not protection here: the data sits in the
page source. If this needs to stay internal, use Vercel's Deployment Protection
(password or SSO, paid plans) or host it behind the internal network instead.
