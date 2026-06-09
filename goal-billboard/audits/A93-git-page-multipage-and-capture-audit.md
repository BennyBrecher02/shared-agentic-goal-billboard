---
title: "Git page — multi-page change visibility + before/after capture-integrity audit"
audit_id: A93
status: in_progress
date: 2026-05-31
trigger: user opened a random git item (commit 382886e) and found two defects
verdict-class: in-progress
produced-via: parallel read-only audits (charter here; findings linked below)
related:
  - context/markdowns/goal-billboard/audits/findings/git-button-persistence-ack-audit.md
---

# Git page — multi-page visibility + capture-integrity audit

## Trigger

The user opened a random git item to check progress — **commit `382886e`** ("Un-stacked overlapping caption text on Fleets + Products", a **2-part bundled** change touching `fleets` + `products`) — and reported two defects. This charter frames the two parallel audits that answer "how wide does each spread?"

## Finding 1 — multi-page change visibility (REGRESSION of a previously-warned issue) · severity MED

> User: *"im still seeing us blatantly mention a change effects many pages with no clear way to see every effected pages changes, we see just the chosen one… how many other items are victim of this previously warned error we couldve avoided had you been listening to me all along."*
> User self-correction: *"(actually double checking i do see an option to swap above the image … fleets EXACT products EXACT — thats good but it needs to be a bit more noticeable)"*

**Read:** the per-page before/after swapper **exists** but is **under-emphasized** — the user nearly missed it. The data is present; the affordance is not prominent enough. The user states this was **warned before**, so the audit must also surface the asked-vs-built history (when was multi-page completeness promised, why did it regress).

## Finding 2 — broken before/after capture · severity HIGH (data integrity / trust)

> User: *"i just checked this items second picture and its before and after seems broken, i cant spot any diff sliding the image, its fleet page i see a diff but products slider seems no diff which is worrying."*

**Read:** for `382886e` part 2 (the **products** page before/after), the slider shows **no visible difference**, while the **fleets** page shows a real diff. If before/after pairs are identical or mis-captured, the user **cannot trust what they're keeping or undoing** — this strikes at the rollback page's entire reason to exist. Highest-priority of the two.

## Audit A — multi-page swapper: completeness + noticeability + history

- Enumerate every multi-page / multi-part git item in `reports/dashboard-build/render-model.json`; verify each HAS the swapper and that it lists ALL affected pages (no page silently dropped).
- Measure the swapper's rendered noticeability (in-browser, computed-style) and recommend a concrete, buildable way to make it prominent.
- Grep the session history for prior user warnings about multi-page before/after completeness → the asked-vs-built gap.
- → findings: `audits/findings/git-multipage-swapper-audit.md`
- Boundary: does NOT cover the collapsed-card hierarchy (owned by the separate card-scrutiny BG).

## Audit B — before/after capture-integrity sweep (how wide does the broken-diff spread?)

- Confirm/deny the reported `382886e` products case (compare the before vs after image for that page).
- Sweep EVERY item's before/after captures (esp. multi-page + multi-part), per affected page; flag any pair that is identical / near-identical (missing diff = broken capture) or where the "after" doesn't reflect the described change.
- Diagnose the likely capture-pipeline root cause + recommend a fix.
- → findings: `audits/findings/git-beforeafter-capture-integrity-audit.md`

## Explicit skip (A18 / never-miss-an-idea)

The repeated-request hook fired on keyword **'progress' (89×)** and demanded a `repeated-request-gather` workflow. **Declined as a false positive** — 'progress' is matched as a common English word across unrelated contexts (its grepped "past fixes" — Hero.astro, Android Studio install, cold-start cost — are mutually unrelated, proving keyword-noise, not a coherent repeated feature ask). The GENUINE recurrence (multi-page before/after completeness — "you were warned before") is instead folded into **Audit A's history dimension**. Logged here as the explicit-skip rationale.

## Fix sequencing

Both audits are READ-ONLY and run in parallel with the two in-flight BGs (Track A = chamber-lockin build editing the monolith; Track B = collapsed-card scrutiny). The FIXES — (1) make the swapper noticeable, (2) repair the broken before/after captures — land AFTER Track A frees `generate_dashboard.py`, batched with the card redesign so the 8,300-line monolith is edited once, coherently. Capture repair (Finding 2) is prioritized over swapper polish (Finding 1) by severity.
