---
audit_id: A78
title: Dashboard scrutiny + plan-consolidation + creative redesign direction — the keystone control surface
status: verified_complete
catalogued: 2026-05-29T01:15Z
completed: 2026-05-29T01:30Z
verdict: "Not as broken as it feels. The #1 'unusable' cause was a serving-root trap (blank unless served from public/, no launcher) — FIXED + verified this turn (scripts/serve-dashboard.sh, npm run dashboard). The rest is a clean fix-queue + ONE taste-gated decision (creative direction). Structural keystone = the operational dashboard is a 9,017-line untracked hand-blob with no generator (S4-F1); everything else is easy after it's regenerable, risky before. Backed up this turn."
priority_when_run: P1
trigger: 2026-05-29 — user: *"scrutinize our dashboard, its ui/ux literally makes it all unusable and its such a keystone to upping our development throughput... make it professional but that doesnt mean generic, get creative and dont limit yourself to evium or reference-design design... the dashboard is not tied to one exclusive project... crosscheck for any plans related to our dashboard that can safely be plowed at for big wins"*
deferral_reason: NONE — genuinely high-value (unusable keystone, never properly scrutinized) + non-redundant (unlike the just-killed site re-audit)
serves_northern_star: G2 (the dashboard is the throughput force-multiplier for G2 + all future projects)
belongs_to_goal: G18
method: 5 parallel streams, only 1 capture-bound (lesson from the killed site-audit: don't run multiple agents on one shared browser). The dashboard is project-AGNOSTIC — design direction must not be tied to Evium/the reference design.
the_stockpile_problem: 6+ overlapping dashboard plans exist (dashboard-overhaul [active] / dashboard-scrutiny / dashboard-content-quality-uxr / dashboard-layout-neaten / monkey-chamber-ux-de-retardize / operational-dashboard-expansion). This audit applies the A77 conflict-check to consolidate them — they're a textbook stockpile needing dedup.
---

# A78 — Dashboard scrutiny + plan-consolidation + creative direction

## What the dashboard IS (scouted)

- **`reports/billboard.html`** — rendered by `scripts/render-billboard-page.py`. THE main control surface (goal billboard + bug billboard + monkey chamber + asks panel + stats). The one the user calls "unusable."
- **`reports/timelapse/index.html`** + `scripts/timelapse/template.html` — the audit time-lapse dashboard (pick-a-run + per-run section view).
- **Monkey chamber** — the decision surface (rendered within billboard; the user previously called it "incoherent horrendous ui/ux").

## The 5 streams (parallel; only #3 captures)

- **S1 — Plan cross-check + consolidation** (non-capture): read all 6+ dashboard plans, dedupe/reconcile into ONE coherent dashboard master plan (applies A77). What overlaps, what's stale, what's the single phased path. The conflict-check lesson applied to the dashboard stockpile.
- **S2 — Creative redesign direction** (non-capture): professional-but-NOT-generic, project-AGNOSTIC design directions (2-3 distinct), NOT tied to Evium/the reference design. The "get creative" ask. Reference excellent dev-tool/control-surface design.
- **S3 — Capture + UX scrutiny** (THE capture-bound one, isolated browser): render the current dashboard + capture billboard + timelapse + chamber across viewports, diagnose concretely WHY it's unusable.
- **S4 — Rendering-pipeline code audit** (non-capture): read render-billboard-page.py + the timelapse template; where is the UX rot rooted in the code (templating, hierarchy, data flow)?
- **S5 — Information-architecture redesign** (non-capture): what SHOULD the dashboard's IA be — panels, hierarchy, nav, the keystone-for-throughput framing; the project-agnostic structure.

## Findings + synthesis (all 5 streams in — 2026-05-29)

### The verdict in one breath
The dashboard **feels** unusable mostly because of **one thing**: it's blank unless served from exactly `public/`-as-web-root, and nothing did that for you (S3-#1). To anyone opening it normally it reads as broken — empty "run this script first" stubs everywhere. **Fixed + verified this turn** (`scripts/serve-dashboard.sh` / `npm run dashboard`; smoke-test confirmed the three root-absolute fetches now return 200). S3's honest kicker: served correctly, the desktop billboard + scheduler are *dense-but-coherent* — much closer to "works" than the experience suggested. The remaining problems are a **clean, ranked fix-queue** plus **one taste-gated decision** (the creative direction, yours to pick).

### The streams agree (unified diagnosis)
- **S3 (visual, the smoking gun):** 5 ranked unusability causes — (1) serving-root trap [FIXED], (2) chamber decided-cards = 10,053px prose wall, (3) ~600px persistent header chrome buries every tab (catastrophic on mobile) + a dead "server is down" agent-inbox console, (4) two permanently-dead tabs (Improvement Tracker stuck "Loading…", Subagents frozen despite fresh 59KB data — render-wiring break) + a 404 storm (12 mis-pathed thumbnails, 16-20 console errors/load), (5) fragmented nav (two disconnected front doors) + a fake "Live" badge on a stale 4-day-old run.
- **S4 (code root):** the operational dashboard the user sees = `reports/timelapse/2026-05-25-overnight/index.html`, a **9,017-line / 769KB untracked hand-blob with NO generator** (F1, the keystone). `render-billboard-page.py` no longer renders — it writes a 516-byte redirect into one tab. F2 = findings dumped as raw `<pre>`. F4 = self-inflicted `location.reload()` (~line 6503) + symlink/HMR yanking the page mid-read (the "anti-jump" bug). **Re-running the documented `generate.py` would DESTROY the 6 hand-authored operational tabs.**
- **S5 (information architecture):** the missing spine — a 4-tier IA **NEEDS-YOU > WHERE-WE-ARE > WHATS-BROKEN > LOOK-BACK**, a default **Overview** home, a left-rail shell, decision-surface at 3 altitudes, a 30-second-scan test. Today everything is a flat 7-tab row with no "what needs me now."
- **S2 (creative direction):** 3 project-agnostic directions; REC **"Terminal Garden"**. Found **brand-bleed**: the dashboard borrows Evium-green `#2DC856` (a *client* color) for tooling chrome — wrong; a tooling-grade green `#3fb950` ≠ Evium. The real styling surface is the timelapse template (13 panels).
- **S1 (plan consolidation):** 8 overlapping dashboard plans were *moderately incoherent*; consolidated into one 6-phase path (`dashboard-master-plan-consolidated.md`). The #1 documented pain — content-quality Track-B markdown-render — **was specced but NEVER shipped**. Latent bug logged: `reports/timelapse` and `public/audit-timelapse` were byte-identical/md5-match but edited separately.

### Prioritized fix-queue (classified by what's safe to plow vs. what's yours)

| # | Fix | Source | Class | Status |
|---|-----|--------|-------|--------|
| 1 | **Launcher + `npm run dashboard`** (serve from public/) | S3-#1 | ✅ SAFE — new file | **DONE + verified this turn** |
| 2 | **Back up the un-regenerable blob** (oh-shit insurance) | S4-F1 | ✅ SAFE — new file | **DONE this turn** (`dashboard-source-backup/`) |
| 3 | Kill the 404 thumbnail storm (12 mis-pathed) | S3-#4 | ⚠ BLOB-SURGERY | staged → folds into F1 generator |
| 4 | Re-wire the two dead tabs (data's fresh, render's broken) | S3-#4 | ⚠ BLOB-SURGERY | staged → folds into F1 generator |
| 5 | Remove the dead "server is down" inbox console | S3-#3 | ⚠ BLOB-SURGERY | staged → folds into F1 generator |
| 6 | Fix the fake "Live" badge (stale run mislabeled) | S3-#5 | ⚠ BLOB-SURGERY (picker) | staged |
| 7 | `<pre>` findings → markdown render (A57 Track-B, never shipped) | S4-F2 | ⚠ BLOB-SURGERY | staged → folds into F1 generator |
| 8 | Anti-jump: kill self-inflicted `location.reload()` + symlink/HMR | S4-F4 | ⚠ BLOB-SURGERY | staged |
| 9 | Collapse the 10,053px chamber prose-wall | S3-#2 | 🎨 TASTE + surgery | staged (pairs w/ direction) |
| 10 | Trim the ~600px header chrome; mobile-first | S3-#3, S5 | 🎨 TASTE (IA) | staged (pairs w/ direction) |
| 11 | 4-tier IA + Overview home + left-rail shell | S5 | 🎨 TASTE (IA) | staged (pairs w/ direction) |
| 12 | Re-theme: Evium-green `#2DC856` → tooling `#3fb950` | S2 | 🎨 TASTE | **YOUR PICK** (gates the re-skin) |

### The structural keystone (F1) — why most fixes are "staged" not "done"
Fixes 3-9 all live **inside the 9,017-line untracked hand-blob.** Editing it blind while you drive = risk on an irreplaceable, fragile file (now backed up, but still). The right move is **not** surgery — it's the keystone S4 named: **give the operational view a real generator + template** that rebuilds the 7 tabs from `data.json`, so fixes 3-9 become clean regen output instead of hand-surgery. That generator naturally pairs with the creative direction (build it generating the *new* design, not faithfully reproduce the old one then rewrite). **So the generator is gated on your direction pick** — which is why the smart drive-time work was the launcher (the one clean, safe, taste-free win) + this synthesis, not a throwaway reproduce-the-ugly-blob build.

### The ONE decision that gates wave 2 (yours)
**Creative direction** (S2): **Terminal Garden** (rec) vs the 2 alternatives. Picking it unlocks: the F1 generator (built generating that design), the re-theme, the IA restructure, and the chamber/chrome redesign — all at once, cleanly, instead of piecemeal blob-surgery. Staged in `dashboard-creative-direction.md` for your pick; I'll put it in the chamber properly when you're back unless you call it here.

## Cross-references
- Wave-1 artifacts (this audit): `dashboard-master-plan-consolidated.md` (S1) · `dashboard-creative-direction.md` (S2) · `dashboard-information-architecture.md` (S5) · `findings/A78-pipeline-code-audit.md` (S4) · `findings/A78-ux-scrutiny.md` (S3) · `reports/dashboard-scrutiny-2026-05-29/` (34 PNGs) · `dashboard-source-backup/` (blob insurance)
- A57 (dashboard content quality — the never-shipped Track-B markdown render is fix #7)
- A77 idea-conflict-check (the dedup discipline S1 applied to the 8-plan stockpile)
- G3 (audit-dashboard-evolution, paused) + G5 (billboard-dashboard-page, achieved)
- LESSON applied: 1 capture agent only (no shared-browser collision, unlike the killed site-audit); plow the safe taste-free win (launcher), STAGE the taste-gated build (don't burn budget reproducing a blob that'll be redesigned).

## Cross-references

- The 6+ existing dashboard plans (S1 consolidates them)
- A57 (dashboard content quality) + the chamber-ux plan
- G3 (audit-dashboard-evolution, paused) + G5 (billboard-dashboard-page, achieved)
- A77 idea-conflict-check (the dedup discipline S1 applies)
- LESSON from the killed site-audit: 1 capture agent only (no shared-browser collision); scrutinize what's genuinely valuable (the dashboard IS — unusable keystone)
