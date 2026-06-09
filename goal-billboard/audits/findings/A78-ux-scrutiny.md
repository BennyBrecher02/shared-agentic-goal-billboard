---
audit_id: A78
title: Dashboard UX scrutiny — why the control surface is "unusable"
type: content-rendering + ux-diagnosis
status: complete
created_at: 2026-05-29
captures: reports/dashboard-scrutiny-2026-05-29/
surfaces: billboard / monkey-chamber / timelapse / picker / scheduler / subagents / improvement / stats
viewports: desktop-1440, mobile-390
serving_model: public/ as web root (NOT file://, NOT Astro dev server)
---

# A78 — Dashboard UX scrutiny

## What was captured

The "dashboard" is ONE 769 KB / 9,017-line static file:
`reports/timelapse/2026-05-25-overnight/index.html`, driven by `?tab=` query params
(`timelapse` | `billboard` | `scheduler` | `subagents` | `monkey` | `improvement` | `stats`).
`scripts/render-billboard-page.py` no longer renders a page — it writes a 516-byte
redirect stub into that tab. A second, unrelated page —
`reports/timelapse/index.html` — is a run "picker."

Captured full-page at 1440 and 390, **twice**: once over `file://` (baseline) and once
over HTTP with `public/` as web root (the real serving model). 30 PNGs + 2 metrics
sidecars in `reports/dashboard-scrutiny-2026-05-29/`. The `http-*` PNGs are the
truthful ones; the bare-name PNGs show the file:// failure mode (see Defect #1).

---

## THE 5 THINGS THAT MAKE IT UNUSABLE (ranked by usability damage)

### #1 — It is silently, totally broken unless served from exactly one web root. (CRITICAL)
Open the dashboard the obvious way — double-click the HTML, or serve `reports/` — and
**every data-driven tab is empty.**

- Evidence (`billboard__desktop-1440.png`, file://): the Billboard renders the section
  headers "Active goals", "Paused", "All linked plans", "Recent verified changes" as
  **empty stubs**, with the line *"could not load /billboard/data.json — run
  `python3 scripts/render-billboard-page.py` first."* The data file is 192 KB and present;
  the page just can't reach it.
- Evidence (`chamber-pending__desktop-1440.png`, file://): the chamber is a single error
  card — *"Chamber data is unreachable. The dashboard could not fetch
  `/screenshot-monkey/data.json`."* Pending 0 / Decided 0 / All 0.
- Root cause: the loaders fetch **root-absolute** URLs (`/billboard/data.json`,
  `/screenshot-monkey/data.json`, source lines 4544–4549). Those only resolve when the
  server's web root is `public/`, which carries the symlinks
  `public/billboard → ../reports/billboard`, `public/screenshot-monkey → …`. Serve from
  the repo root and `/billboard/data.json` is **404** (verified). Open via `file://` and
  Chromium blocks the `fetch()` entirely (opaque-origin policy) — so even the relative
  fallback `../../billboard/data.json` (line 4773), whose target file genuinely exists,
  also fails.
- Why this is #1: a control surface that shows blank panels with a "run this script"
  error to anyone who opens it the natural way reads as **broken**, regardless of how good
  the populated version is. There is no in-repo server script and no documented
  "serve `public/` on :PORT then open /audit-timelapse/…" runbook. This single
  serving-model fragility is the most likely literal source of "unusable."
- Fix direction: (a) ship a one-liner launcher (`scripts/serve-dashboard.sh` → `http.server`
  rooted at `public/`, prints the URL); OR (b) make the loaders relative-first so the file
  resolves from the dashboard's own directory; OR (c) inline the JSON at render time so the
  page is self-contained. (b)+(a) is the cheap, robust combo.

### #2 — The monkey chamber's decided cards are an unreadable wall of prose. (HIGH — this is the "incoherent horrendous" surface)
With data loaded (`http-chamber-all__desktop-1440.png`), the chamber is **10,053 px tall** —
12 decision cards stacked vertically, each a dense block of small-type body copy: a long
question, then multi-paragraph "context" running to ~6–8 lines of 13 px text, then inline
screenshots, then the next card. No card is collapsed; no card is scannable; there is no
summary/expand affordance. You cannot answer "what needs my decision?" by glancing — you
have to read an essay per card.
- Evidence: `_crop-chamber-cards.png` (readable zoom) — the "Blog page has two light
  sections…" card alone is a full paragraph of prose before the screenshots, and it is one
  of twelve.
- This is exactly the A57 complaint ("absolute incoherent horrendous ui/ux") persisting on
  the *rendered* side even after the content-lint work: the per-card content may now pass
  the linter, but the **layout** still presents 12 essays in a single scroll with no
  density control.
- Fix direction: collapse cards to a one-line headline + status chip by default; expand on
  click; move context/screenshots behind the expand. Cap the visible question to ~2 lines
  with a "more" toggle. The Pending/Decided/All filter already exists — lean on it so the
  default view is short.

### #3 — A persistent ~600 px chrome stack buries every tab's actual content, worst on mobile. (HIGH)
Above *every* tab sits a fixed stack that never collapses: the **Northern Star band** +
the **Agent-inbox console** ("server is down — messages will queue locally…", with a text
input + priority dropdown + Queue button) + the page title + its description + a Refresh
control + (on chamber) the status chips + the category chips.
- Desktop: on a 900 px-tall viewport you scroll past NS-band + inbox console before the
  first goal card or decision. Tolerable but heavy.
- Mobile (`_crop-mobile-billboard-top.png`, `_crop-mobile-chamber.png`): catastrophic. On a
  390×844 screen the NS-band + inbox console + chips consume the **entire first screen and
  then some** — on the chamber the title "Monkey Communication Chamber" wraps to 3 lines and
  the filter chips sit at ~760 px down, so zero decisions are visible above the fold. The
  agent-inbox console (a niche, currently-dead feature — "server is down") gets the same
  prime real estate on mobile as on desktop.
- Note: the left nav DOES correctly collapse to a 48 px icon rail ≤768 px (source 1725–1762),
  so the sidebar is *not* the problem — the always-on header furniture is.
- Fix direction: collapse the agent-inbox console to a single icon/button on mobile (it is
  optional and currently non-functional); make the NS-band compact (one line) below 768 px;
  let the tab's own content start near the top.

### #4 — Two tabs are permanently dead, and the whole thing throws a 404 storm on every load. (HIGH)
- **Improvement Tracker** (`http-improvement__desktop-1440.png`): stuck forever on
  *"Loading the before/after comparison…"* — never resolves. scrollH = 900 (= viewport;
  i.e. empty). A nav item that leads to a perpetual spinner.
- **Subagents** (`http-subagents__desktop-1440.png`): every module is a frozen loading
  placeholder — "Reading session tags for in-flight dispatches…", "Aggregating bytes by
  class…", "Reading the last 20 completion records…", "Bucketing the last 24h…",
  "Cross-referencing G2 dispatches…". None populate. scrollH ≈ 998 (near-empty). Yet
  `reports/subagents/data.json` is 59 KB and fresh — so this is a render-side wiring break,
  not missing data.
- **404 storm**: on EVERY tab (billboard, chamber, all of them) the page eagerly requests
  **12 missing thumbnails** — `/reports/vq-captures-archive/2026-05-25-overnight/<engine>/home/section-01.jpg`
  — all 404 (verified). These are timelapse "before" thumbs, mis-pathed (the public symlink
  is `audit-timelapse → reports/timelapse`, not `…/vq-captures-archive`) AND loaded even when
  you're on a non-timelapse tab. Result: 16–20 console errors per page and broken-image
  requests on first paint.
- Fix direction: hide dead tabs (or render an honest empty-state instead of an eternal
  spinner); fix the thumbnail path to go through the real symlink; lazy-load thumbs only on
  the timelapse tab.

### #5 — Navigation is fragmented across two disconnected front doors, and the "live" run is a lie. (MEDIUM-HIGH)
- There are **two entry points with no link between them**: the run **picker**
  (`reports/timelapse/index.html` — actually the cleanest, most legible page in the system;
  see `http-picker__desktop-1440.png`) and the **dashboard** itself. The dashboard has no
  "back to runs" affordance; the picker has no path into billboard/chamber. A new user
  doesn't know which URL is "the dashboard."
- The picker offers 2 runs; one is labeled **"Current run (live, symlinked) · in progress"**
  with a green "Live" pill — but it's the stale `20260525-merged` run from four days ago with
  byte-identical tallies to the archived one (17 crit / 34 high / 39 med / 200 low / 395
  clean). A "Live / in progress" badge on dead data actively misleads.
- Within the dashboard, the only nav is the left icon rail (7 tabs). It works, but several of
  its destinations are the dead/broken tabs from #4, so the nav over-promises.
- Fix direction: pick ONE front door (make the dashboard's timelapse tab embed the run
  switcher, retire the standalone picker — or vice-versa); kill or correct the false "Live"
  badge; trim nav to tabs that actually render.

---

## Per-surface summary

**Billboard** — *Populated state is actually OK-structured* (Northern Star hero → Guiding
Lights track table → Active-goals card grid → audits → plans), see `_crop-billboard-top.png`.
Problems: (a) #1 makes it blank in the wrong serving context; (b) it is *very* long —
9,849 px desktop / 19,730 px mobile — every goal, every plan, every audit row dumped inline
with no pagination or collapse; (c) the Guiding-Lights table cramps badly on mobile
(P0·G2 / P1·G14 jammed); (d) #4's 404 thumbs fire here too. It's information-dense to the
point of being a scroll-marathon, but it is not incoherent — fixable with collapse/paginate.

**Monkey chamber** — see #2 (wall-of-prose cards) and #3 (buried below header furniture).
The empty Pending state ("THE TROOP IS AT REST · 0 pending") is fine and on-brand. The
problem is the Decided/All view's density, not the empty state.

**Timelapse** — the only fully-working data tab. But the landing
(`http-timelapse-landing__desktop-1440.png`) is a dense "worst sections" card row followed
by a **giant multi-device heatmap matrix** that scrolls for thousands of px with tiny,
near-unreadable per-cell/row labels, then a wall of "Counterfeit findings" raw text. High
cognitive load; no obvious "start here." The heatmap is the centerpiece but isn't legible at
the scale it renders.

**Picker** — cleanest, most usable page in the whole system; ironically it's the orphan
(see #5) and surfaces a false "Live" run.

**Scheduler / Stats** — partially populated. Stats (`http-stats__desktop-1440.png`) shows
several modules in "no data yet — run python3 scripts/render-X.py" empty states interleaved
with populated ones, plus its own giant heatmap; reads as half-built. Scheduler renders a
compact set of cards (scrollH ≈ 1534) and is the least offensive of the data tabs.

---

## Honest framing

The *populated, correctly-served* dashboard is dense-but-coherent on desktop billboard and
scheduler. The "unusable" verdict is driven by, in order: (1) it's blank unless served from
one specific root with no launcher, (2) the chamber's decided-card prose wall, (3) the
mobile header-furniture burial, (4) two dead tabs + a per-load 404 storm, (5) two
disconnected front doors with a fake "Live" badge. Fix #1 first — it's the difference
between "broken" and "busy."

## Captures
All in `reports/dashboard-scrutiny-2026-05-29/`. Truthful (HTTP-served) set is `http-*.png`;
`*__*.png` without the `http-` prefix demonstrate the file:// failure mode (Defect #1).
Readable crops: `_crop-billboard-top.png`, `_crop-chamber-cards.png`,
`_crop-mobile-billboard-top.png`, `_crop-mobile-chamber.png`. Metrics:
`_capture-metrics.json` (file://), `_capture-metrics-http.json` (HTTP). Reproducible
harness: `_capture.mjs`, `_capture_http.mjs` (pin Chromium 1223 headless-shell; serve
`public/` on :8800 for the HTTP run).
