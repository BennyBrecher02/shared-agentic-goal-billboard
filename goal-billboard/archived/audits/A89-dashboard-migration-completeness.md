---
id: A89
title: Dashboard migration completeness — what did the new DEV CONTROL dashboard drop from the v3 / audit-timelapse generation?
type: audit
status: completed
created: 2026-05-31
trigger: User worry — "the dashboard evolved through multiple generations and the newer one dropped features the earlier ones had." Specifically the v3 left rail (Time-lapse · Billboard · Scheduler · Subagents · Monkey Chamber · Improvement Tracker · Stats Correlation + rail vital-signs) vs the new DEV CONTROL Blueprint rail.
scope: COMPLETENESS / MIGRATION audit across all dashboard surfaces. Read-only. Recommend only.
cluster: dashboard
siblings: [A57-dashboard-content-quality-and-uxr, A78 (IA/creative-direction streams), A48-data-json-auto-sync-from-inbox-history, A51-dashboard-textarea-survives-hmr-reload]
---

# A89 — Dashboard Migration Completeness Audit

> **Bottom line up front.** The user's worry is **substantially correct on two features and a class of rail
> widgets**, and **incorrect on one** they specifically named. The new "DEV CONTROL" dashboard
> (`generate_dashboard.py`) **kept Stats Correlation** (the user feared it lost it — it didn't, though at
> shallower depth). It **dropped the embedded Time-lapse / scrub-bar reviewer** (now an external doc, and the
> rail doesn't even link to it), **dropped the Improvement Tracker** (the per-(route,section,device)
> improve/stay/regress verdict matrix — genuinely gone, its data source `improvement-tags.json` is orphaned
> from the new generator), and **dropped ~6 persistent rail vital-signs** (Cluster, Untapped Integrations,
> Skill Health, Recurrences, live tokens/min burn in the rail, 5hr-reset in the rail) — collapsing them into
> nav count-badges + a single Overview "System Pulse" row.
>
> Of these: **Improvement Tracker = the real loss** (recommend PORT-FORWARD — it's the only surface that
> answered "did the fix actually work per-cell," and the IA plan's own rail mockup listed it). **Time-lapse
> embed = SUPERSEDED-but-orphaned** (the reviewer still exists standalone; the bug is the missing jump-off
> link). **Rail vital-signs = MIXED** (some superseded by the Pulse row / Stats view, two — Skill Health,
> Untapped — genuinely forgotten).

---

## 1. Inventory — every dashboard surface that exists

The dashboard topology is **four generations deep**. The `public/dashboards.html` hub is the Rosetta stone —
it explicitly pill-labels each surface `canonical` / `legacy · frozen` / `archive`.

| # | Surface | Title / header | Generator | Output | Status (per hub pills + wiring) |
|---|---------|----------------|-----------|--------|----------------------------------|
| 1 | **DEV CONTROL dashboard** (the "new" one) | `◈ DEV CONTROL` / Blueprint skin | `scripts/dashboard/generate_dashboard.py` (279 KB) + `build_render_model.py` + ~14 `parse_*.py` | `reports/dashboard-build/dashboard.html` → symlinked `public/dashboard-build/` | **CANONICAL** (hub: `canonical · blueprint`). 13 views. Actively built (mtime today). |
| 2 | **v3 "operational dashboard"** (the one in the user's screenshot) | `Audit time-lapse` page-title, but a 7-tab operational dash: Billboard · Scheduler peek · Subagent transparency · Monkey Communication Chamber · Improvement Tracker · Stats Correlation · Time-lapse | **was** `scripts/render-billboard-page.py` — **now a frozen static artifact** | `reports/timelapse/2026-05-25-overnight/index.html` (769 KB) — **byte-identical** (same md5 `adf4e2d…`) to the backup `context/markdowns/dashboard-source-backup/operational-dashboard-2026-05-29.html` | **LEGACY · FROZEN** (hub pill). No live generator anymore (see note A). This IS the "v3." |
| 3 | **Audit time-lapse per-run reviewer** (the ORIGINAL one) | `Audit time-lapse — <run-id>` | `scripts/timelapse/generate.py` (26 KB) + `scripts/timelapse/template.html` (46 KB), driven by `scripts/post-audit-timelapse.sh` | `reports/timelapse/<run-id>/index.html` → symlinked `public/audit-timelapse/` | **LIVE but narrow.** This is the pure capture-reviewer: before/after/diff rail tabs + Walkthroughs (per-route / per-section reels) + scrub-bar player. Still wired via `post-audit-timelapse.sh`. (Note B.) |
| 4 | **Dashboards hub** (front door) | `◈ Dashboard Hub — Blueprint` | hand-authored (no generator) | `public/dashboards.html` | **CURRENT.** Links: Manifesto Checklist · Operational Dashboard (`canonical·blueprint` → `/dashboard-build/dashboard.html`) · Bug Billboard · Creative-direction demos (`archive`) · Audit Time-lapse (`legacy·frozen` → `/audit-timelapse/index.html`). |
| 5 | **Creative-direction demos** (A78) | `Dashboard — creative direction demos (A78)` | hand-authored | `reports/dashboard-demos/{index,A-flight-deck,B-terminal-garden,C-blueprint}.html` | **ARCHIVE** (hub pill + `_decision.json status:pending`). The 3 skins the user chose from; Blueprint (C) won → became #1's skin. |
| 6 | **billboard.html redirect stub** | — | `scripts/render-billboard-page.py` (current behavior) | `reports/billboard.html` → `public/billboard.html` | **REDIRECT-ONLY.** Now just `<meta refresh>` to the frozen v3 `?tab=billboard`. (Note A.) |
| 7 | **Run-picker index** | `Audit time-lapse` (run list) | `scripts/timelapse/` family | `reports/timelapse/index.html` + `public/audit-timelapse/index.html` | **LIVE.** Lists timelapse runs (`2026-05-25-overnight`, `20260525-merged`, etc.). |

> **Note A — the v3 generator is retired in place.** `scripts/render-billboard-page.py` (606 lines) still
> exists and is referenced by hooks (`billboard-data-refresh.sh`, `dashboard-data-prime.sh`), but its HTML
> output is now a **redirect stub** (lines 417-432: `OUTPUT.write_text(redirect_html…)`; the full-HTML write
> at line 437 is **commented out**). It still emits `reports/billboard/data.json` (line 579). So the v3 7-tab
> HTML is a **frozen snapshot** at `reports/timelapse/2026-05-25-overnight/index.html` with **no live
> regenerator** — what you see there is data baked on 2026-05-27.

> **Note B — two different "time-lapses" share a folder.** `reports/timelapse/<run>/` holds BOTH (3) the pure
> per-run reviewer (most runs) AND (2) the one frozen run that got the full 7-tab operational shell bolted on.
> The generator in `scripts/timelapse/template.html` produces ONLY the reviewer (it has the
> before/after/diff/reels/scrub markup but **zero** `page-billboard`/`page-improvement` markup — grep: 0
> matches). The 7-tab superset only ever existed in `render-billboard-page.py`'s now-disabled full-HTML path.

---

## 2. Every feature / section / view / vital-sign in the OLD v3 + reviewer

### 2a. v3 operational dashboard — the 7 main pages (`id="page-*"`)
Confirmed via `id="page-…"` anchors in the frozen `2026-05-25-overnight/index.html` / identical backup:

| v3 page | `id` | What it showed |
|---------|------|----------------|
| **Billboard** | `page-billboard` | Northern Star + Guiding Lights + goal cards + an **Asks-Awaiting-Artifacts** panel (`#ar-billboard`). |
| **Scheduler peek** | `page-scheduler` | Scheduler run state / health (`#ar-scheduler`). |
| **Subagent transparency** | `page-subagents` | Dispatched subagents + verdicts; embedded `#sa-improvement` block. |
| **Monkey Communication Chamber** | `page-monkey` | Chamber decisions / asks (`#ar-monkey`). |
| **Improvement Tracker** | `page-improvement` | **"For every shipped fix: did the targeted (route, section, device) cells actually improve, stay the same, or regress?"** Reads `reports/improvement-tags.json` (`expected_severity_delta: "critical/high -> clean"`). The before/after **quality-delta** verdict matrix. |
| **Stats Correlation** | `page-stats` | "tokens · time · burn rate" + 5hr rolling window + hourly messages + recent sessions + burn-rate sparkline (`#ar-stats`). |
| **Time-lapse** | `page-timelapse` | The embedded per-run capture reviewer (before/after/diff + reels + scrub-bar). |

### 2b. v3 persistent left rail ("lefter") — the vital-signs
The v3 rail was **stat-bearing**, not just nav. Confirmed CSS classes + rendered labels in the frozen HTML:

| Rail vital-sign (`.lefter-*`) | Origin | What it surfaced |
|-------------------------------|--------|------------------|
| `lefter-tokens` | — | Token budget % + bar. |
| `lefter-time` + `lefter-time-label "5hr reset"` | — | Session clock + **5hr-window reset countdown**. |
| `lefter-asks` | A18 | **Asks pending** count (yellow→pink escalation). |
| `lefter-waiting` | A20 | **Waiting-on-you** count. |
| `lefter-inflight` | — | **In-flight subagents** w/ elapsed + ETA. |
| `lefter-inbox` | — | Monkey inbox count. |
| `lefter-cluster` | A22 | **Cluster** node status. |
| `lefter-ns` (`lefter-ns-mini`) | — | NS-context mini. |
| `lefter-meta` / `lefter-meta-loop` | A24 | **Untapped Integrations** (meta-loop). |
| `lefter-skill-health` | A28 | **Skill Health** mini-block. |
| (rendered) **Recurrences** | — | Repeated-request recurrence count. |
| (rendered) **Burn rate · tokens/min** | A14/A43 | Live burn rate in the rail. |
| `lefter-brand` / `lefter-toggle` / `lefter-foot` | — | Brand plate, collapse toggle, footer (chrome, not data). |

### 2c. Audit time-lapse reviewer (surface #3) — its features
Confirmed via `scripts/timelapse/template.html` classes:
- **Before / After / Diff** rail tabs (`data-tab`).
- **Walkthroughs** (`data-cp="reels"`) — two reel modes: **Per route · one device at a time** + **Per section · every device** (`reel-tab`, `reel-list`, `reel-thumb`, `reel-modal`).
- **Scrub-bar player** (`player`, `player-scrub`, `scrub-fill`, `player-stage`, `player-progress`, `player-controls`) — the time-lapse scrubbing the user remembers.
- **Heatmap — all sections, all devices** (an `<h1>`).

---

## 3. Every view in the current DEV CONTROL dashboard

13 views via `data-page=…` (rail buttons → `<section id="page-…">`), grouped by the 4 IA tiers
(`rail-group-label`):

| Tier (rail divider) | Views (`data-page`) | Notes |
|---------------------|---------------------|-------|
| *(top, ungrouped)* | `overview` · `board` (unified board) | Overview = default landing (A NeedsYou / B WhereWeAre / C SystemPulse / D Jump-offs). |
| **Needs You** | `chamber` · `handoff` · `asks` | chamber = renamed Monkey Chamber; handoff = NEW (Cursor handoff queue). |
| **Where We Are** | `goals` · `pins` | |
| **Health** | `bugs` · `scheduler` | |
| **Look Back** | `stats` · `subagents` · `plans` (Plans/Audits) · `git` (Git/Rollback) | |

Rail **nav badges** (counts only, `data-key`): `asks, board, bugs, chamber, git, handoff, pins, subagents`.
Plus a persistent **"needs-you" header chip** (`◈ N need you`, 145 `chip` refs). Overview **System Pulse**
row cells: Open bugs · Scheduler · Last activity · Services · Activity(3h).

---

## 4. Coverage matrix — every OLD feature mapped to the new dashboard

Legend: **✅ PRESENT** (where) · **🔁 SUPERSEDED** (covered elsewhere) · **❌ MISSING** · **🪦 INTENTIONALLY-CUT** (with evidence).

### 4a. v3 main pages

| OLD v3 page | New status | Where / evidence |
|-------------|-----------|------------------|
| Billboard | ✅ PRESENT (split) | → `goals` view (NS + Guiding Lights + cards) + `board` (unified board) + Overview B. |
| Scheduler peek | ✅ PRESENT | → `scheduler` view (`page-scheduler`, `parse_scheduler_state.py`). |
| Subagent transparency | ✅ PRESENT | → `subagents` view (`parse` via `render-subagent-data.py`/model). |
| Monkey Communication Chamber | ✅ PRESENT (renamed) | → `chamber` view (`page-chamber`, badge `chamber`). "Monkey" branding dropped per A46/A57 de-retardize, tone retained in copy. |
| Stats Correlation | ✅ PRESENT (shallower) | → `stats` view: **burn/min + band** (gen line 2713), **5hr window minutes_remaining** (line 2716), hourly, sessions, sparkline. `correlate_stats.py` exists. **BUT** the v3 standalone "tokens × time × activity" correlation panel was richer; the new stats view is condensed. See §5-G3. **User's specific fear (lost Stats Correlation) is unfounded — it's there.** |
| **Improvement Tracker** | ❌ **MISSING** | **No `page-improvement`. No `scripts/dashboard/*.py` reads `improvement-tags.json`** (grep across all parsers = 0). The new `git` view's "verdict" is a *rollback-safety* verdict (`.cc-verdict "can this be safely undone?"`) + before/after **screenshots** — **NOT** the per-cell improve/stay/regress **quality delta**. Different concept. See §5-G1. |
| **Time-lapse (embedded)** | 🔁 SUPERSEDED but ❌ **un-linked** | The reviewer survives standalone (surface #3). But the new dashboard **embeds nothing** (`scrub`/`reel`/`walkthrough`/`player-stage` = **0** in `dashboard.html`) **and the Overview's D·Jump-offs list does NOT include time-lapse** (jump-offs are: git, stats, subagents, plans, audits, pins, chamber — gen lines 2173-2181). The IA plan §3b rail mockup explicitly listed `▷ Time-lapse` under Look Back — **the built rail omits it.** See §5-G2. |

### 4b. v3 rail vital-signs

| OLD rail vital-sign | New status | Where / evidence |
|---------------------|-----------|------------------|
| Tokens (budget %) | 🔁 SUPERSEDED | Folded into `stats` view + header chip context. No persistent rail token bar. |
| 5hr reset (in rail) | 🔁 SUPERSEDED | Now in `stats` view (`five_hr_window`, line 2698/2716), not persistent in rail. |
| Asks pending (A18) | ✅ PRESENT | Rail badge `asks` + `asks` view. |
| Waiting-on-you (A20) | 🔁 SUPERSEDED | Folded into the **needs-you header chip** (`N need you`) + Overview A. No distinct rail block. |
| In-flight subagents (elapsed/ETA) | 🔁 SUPERSEDED (degraded) | `subagents` view + Overview "Last activity" pulse cell. The **live elapsed+ETA rail ticker is gone** (it was a glanceable rail widget; now you must open the view). |
| Monkey inbox count | ✅ PRESENT | Rail badge `chamber`. |
| **Cluster (A22)** | ❌ **MISSING** | `cluster` = **0** refs in `generate_dashboard.py`. No rail block, no pulse cell, no view. (`reports/timelapse/.../data-cluster.json` still produced but unconsumed by new dash.) See §5-G4. |
| NS-context mini | 🪦 INTENTIONALLY-CUT (reasonable) | NS-context is an internal dispatch detail; no operator action. Drop is defensible. |
| **Untapped Integrations (A24)** | ❌ **MISSING** | No counterpart anywhere in the new dashboard. See §5-G5. |
| **Skill Health (A28)** | ❌ **MISSING** | `skill-health`/`skill health` = **0** refs. No counterpart. See §5-G5. |
| **Recurrences** | ❌ **MISSING** | No recurrence count surfaced (the A40 recurring-request signal). See §5-G5. |
| Burn rate (tokens/min, in rail) | 🔁 SUPERSEDED | Moved into `stats` view (`burn_rate_per_min`, line 2713). Not a persistent rail widget. |
| Brand / toggle / foot | ✅ PRESENT | New rail has brand plate (`rail::before`), collapse (responsive §1158), SIG-ramp foot (`rail::after`). |

### 4c. Audit time-lapse reviewer features (surface #3)

| Reviewer feature | New dashboard status | Evidence |
|------------------|----------------------|----------|
| Before/After/Diff capture tabs | 🔁 SUPERSEDED (different form) | New `git` view has a before→after **commit** viewer (`ba-img-before/after`, lines 3083-3124); the per-capture before/after/diff *audit* tabs live only in the standalone reviewer. |
| Walkthroughs / reels (per-route, per-section) | ❌ NOT in dashboard | Lives only in surface #3. Not embedded, not linked from the new rail. |
| Scrub-bar player | ❌ NOT in dashboard | Same — standalone reviewer only. |
| Heatmap (sections × devices) | ❌ NOT in dashboard | Standalone reviewer only. |

---

## 5. The gaps — flagged, with a recommendation each

### G1 · Improvement Tracker — **PORT-FORWARD** (the real loss) 🔴
**What's gone:** the per-(route, section, device) **improve / stay / regress** verdict for every shipped fix,
driven by `reports/improvement-tags.json`. **Evidence it's orphaned:** no `scripts/dashboard/*.py` references
`improvement-tags.json` (grep = 0); the data is still *written* (by `backfill-improvement-tags.py`,
`audit-feedback-loop-compare.py`, `render-subagent-data.py`) but **read by nothing in the new dashboard.**
**Why it matters:** this was the ONLY surface answering *"did the fix actually work?"* — the closed loop on
quality. The new `git` view answers a different question (*"can I undo this commit?"* + shows screenshots),
**not** whether the targeted cells improved. **This looks forgotten, not cut** — the IA plan (§2 surface
table line 115 + §3b rail mockup line 163 `↗ Improve`) **explicitly kept it as a Look-Back view.** The build
simply never wired `page-improvement`.
**Recommendation:** add an `improvement` view (or fold a verdict-delta panel into the `git` view) that reads
`improvement-tags.json`. Low effort — the data and the v3 render logic both exist; it's a re-wire, not a
rebuild. *(NB: per "User owns done" — flag as a gap; do not build without greenlight.)*

### G2 · Time-lapse — **SUPERSEDED but RE-LINK** (cheap fix) 🟡
**What's gone from the dashboard:** the embedded scrub-bar / reels reviewer — AND, more importantly, **any
pointer to it.** The reviewer itself is alive at `public/audit-timelapse/<run>/index.html` (surface #3,
`post-audit-timelapse.sh` still wired). So this is **superseded** (kept as a deliberately-separate per-run
doc, consistent with the IA's "never autoplays, Tier-3 jump-off" rule, line 95/160) — **except the jump-off
link was dropped.** The Overview D·Jump-offs (gen lines 2173-2181) lists git/stats/subagents/plans/audits/
pins/chamber but **not time-lapse**, and the rail has no `▷ Time-lapse` despite the IA mockup (line 160).
**Recommendation:** add one **jump-off tile + rail link** "▷ Time-lapse → newest run" pointing at
`/audit-timelapse/index.html` (the run-picker) or the latest run. Do **not** re-embed the player (correctly
kept separate). Tiny change; restores discoverability. This is the closest thing to "simply forgotten."

### G3 · Stats Correlation depth — **MOSTLY SUPERSEDED** (verify, low priority) 🟢
**User's named fear is unfounded** — burn-rate, 5hr window, hourly, sessions, sparkline all migrated to the
`stats` view. The only shortfall: the v3 standalone panel framed it as an explicit **"tokens × time ×
activity" correlation** (the cross-dimension view A14 added); the new `stats` view shows the same *inputs*
but is more of a KV dashboard than a correlation plot. `correlate_stats.py` (36 KB) still exists.
**Recommendation:** SUPERSEDED — accept as-is unless the user specifically misses the correlation framing,
in which case surface `correlate_stats.py` output as a panel in the `stats` view. No urgency.

### G4 · Cluster status — **MISSING; PORT-FORWARD-LITE or CUT** (decide) 🟡
The A22 cluster node indicator has no counterpart (`cluster` = 0 in the new generator), though
`data-cluster.json` is still produced. **Recommendation:** if multi-node cluster work is live, add a
one-cell cluster indicator to the System Pulse row (cheap). If cluster is dormant, mark **INTENTIONALLY-CUT**
and stop emitting `data-cluster.json`. Needs a user signal on whether cluster is active — don't assume.

### G5 · Skill Health (A28), Untapped Integrations (A24), Recurrences (A40) — **MISSING; likely FORGOTTEN** 🟡
These three rail mini-blocks vanished with no replacement and no jump-off. They're system-meta signals
("is a skill rotting," "what integration are we under-using," "what is the user asking repeatedly"). None
appear in the IA plan's rail mockup (§3b) — so unlike G1, there's **no documented intent to keep them**,
which makes "intentionally simplified" *plausible* but undocumented.
**Recommendation:** these are the honest **"forgotten in the rail-flattening"** cases. The IA tiered-rail
redesign collapsed the stat-bearing lefter into nav-badges + one Pulse row, and these three had no obvious
tier home. Suggest: a small **"System meta" group** on the Overview (or a `health`-tier strip) carrying
Skill Health + Recurrences (both still have data producers); treat Untapped Integrations as lowest priority
(its A24 meta-loop value was always speculative). Confirm with the user before building — could equally be a
deliberate de-clutter.

---

## 6. Honest summary for the user

- **You were right** that the new dashboard dropped real things: the **Improvement Tracker** (G1 — the
  per-cell "did the fix work" matrix; its data is orphaned, looks forgotten), the **embedded Time-lapse +
  its link** (G2 — reviewer still exists standalone but nothing points to it), and **~5 rail vital-signs**
  (G4/G5 — Cluster, Skill Health, Untapped, Recurrences, plus the live in-rail burn/inflight tickers).
- **You were wrong about one you named:** **Stats Correlation survived** (G3 — burn-rate + 5hr + sessions
  all in the `stats` view, just less explicitly "correlation"-framed).
- **The redesign was deliberate, not lossy-by-accident, for most of it:** the v3's **stat-bearing left rail**
  was intentionally flattened into nav-badges + a header "needs-you" chip + one Overview "System Pulse" row
  (documented in `dashboard-information-architecture.md` §3b, the A78 IA stream). That trade bought a clean
  tiered rail and a real "what needs me now" landing surface — at the cost of the always-visible rail
  telemetry. Whether that trade went too far on G4/G5 is a **judgment call for you.**
- **The two cheapest, highest-value restorations:** G2 (one jump-off link — pure oversight) and G1 (re-wire
  the existing `improvement-tags.json` into a view — data + old render logic both still exist).

> Nothing in this audit was applied. All recommendations await your greenlight (per "User owns done" +
> decision-handling discipline — dashboard rebuild is exactly the kind of structural change that's gated).
