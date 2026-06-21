---
goal_id: G2
title: Evium Phase 5 site overhaul (defect remediation)
status: shelved
shelved: "2026-06-08 — SHELVED per user (not resuming near-future). Project pivoted to Back Lawrence Shul 2026-06-02. Moved active/ → paused/ (the canonical non-destructive lane; 'shelved' = paused-indefinitely, fully resumable). ⚠ DO-NOT-PUSH-EVIUM gate — see shelve_note below + context/markdowns/notes/reminders.md."
track: product
northern_star: false
ns_demoted: "2026-06-03 — demoted from Northern Star (user #63: BLS pivot). Evium parked HIGH-PRIORITY — the client deliverable awaiting client review + push; restore to NS only on explicit user say-so. Active NS is now BLS (plans/bls-mobile-build-plan.md)."
northern_star_tagline: "(HISTORICAL — pre-2026-06-03 BLS pivot) The client deliverable. 696 audit cells, twelve devices, one sprint to defect-free."
phase: sleep-plow 2026-05-28 advanced 9-device real-hardware sweep (5 iOS + 4 Android profiles) → consolidated-findings G2-CLOSE-READY YES verdict ON DISK; awaiting client review + push of 10 commits ahead origin + 3 monkey decisions + Mobile Phase 6 real-iPhone gesture
priority: P0
status_tag: awaiting-user
gated_on: "USER-owned — (1) client review of the shipped overhaul, (2) the Evium PUSH (10 commits ahead of origin), which is BLOCKED behind the ⚠ DO-NOT-PUSH-EVIUM gate: the contact/subscribe/quote forms on the frozen `main` submit nowhere (bug #20260529T0612-...; a push-time lead-loss gun, NOT a current live leak — the live tree has no forms), so the forms must be wired BEFORE any Evium push, and (3) 3 pending monkey-chamber decisions. No agent move here; correctly shelved (BLS pivot 2026-06-02), fully resumable on explicit user say-so. (Surfaced — NOT moved — per goal-grooming-proposal-2026-06-21.md §4-D.)"
created: 2026-05-24T01:57Z
updated: 2026-06-21T21:01Z
sleep_plow_artifacts:
  - reports/audit-findings/g2-second-pass/consolidated-findings.md (G2-CLOSE-READY YES verdict; 81 real-device captures)
  - 5 atomic remediation patches landed (M-NEW-01 products §01 tablet viewport scope)
  - scripts/ios-simulator-capture.sh (booted-state handling fix)
  - scripts/mobile-sim-sweep-orchestrator.sh (auto-shrink baked in)
  - 4 narrowed-but-not-shipped pins (P-001 wave3 / P-002 consolidator / P-010 local-AI / P-017 Pixel_Fold)
  - A71 mutation-OS inbox-never-lost VERIFY: 5/5 PASS (first 4/4 full-tether)
signoff_package_path: reports/g2-signoff/
linked_plans:
  - context/markdowns/audits/full-coverage-2026-05-25/
linked_refs:
  - .claude/skills/agentic-page-scrutiny/SKILL.md
  - .claude/skills/agentic-device-testing/SKILL.md
linked_bugs:
  - "#20260529T0612-main-claude-main-88d2a292 — /contact + /blog forms submit nowhere (CRITICAL; demoted to PIVOT-FROZEN 2026-06-08 via bug-billboard inbox-append — broken forms exist only on frozen `main`, the live tree has no forms; reopen when Evium un-shelves). THIS is the DO-NOT-PUSH-EVIUM trigger."
linked_changes:
  - context/markdowns/multi-change-queue/verified/20260526T0924-blog-subscribe-fix-p0-02.md
  - context/markdowns/multi-change-queue/verified/20260526T1230-contact-tablet-gutter-p1.md
  - context/markdowns/multi-change-queue/verified/20260526T1245-multifamily-quote-stats-stack.md
  - context/markdowns/multi-change-queue/verified/20260526T1246-highway-econ-narrow.md
  - context/markdowns/multi-change-queue/verified/20260526T1247-services-terminal-narrow.md
  - context/markdowns/multi-change-queue/verified/20260526T1300-phaseB-multishard-multifamily.md
  - context/markdowns/multi-change-queue/verified/20260526T1406-products-crumb-restore.md
  - context/markdowns/multi-change-queue/verified/20260526T1410-phaseD-integration-retry.md
  - context/markdowns/multi-change-queue/verified/20260526T1411-contact-form-card-p0-03.md
  - context/markdowns/multi-change-queue/verified/20260526T1416-blog-hero-equalize-p1-06.md
  - context/markdowns/multi-change-queue/verified/20260526T1544-eyebrow-token-p1-08.md
  - context/markdowns/multi-change-queue/verified/20260526T1623-blog-subscribe-p0-02-requeue.md
  - context/markdowns/multi-change-queue/verified/20260526T1755-blog-section-split-p2-10.md
  - context/markdowns/multi-change-queue/verified/20260526T1755-fleets-hero-text-shadow-firefox.md
  - context/markdowns/multi-change-queue/verified/20260526T1755-highway-hero-object-position-p1-05.md
  - context/markdowns/multi-change-queue/verified/20260526T1755-on-surface-variant-lift-p2-09.md
  - context/markdowns/multi-change-queue/verified/20260526T1755-services-logo-grid-p2-11.md
  - context/markdowns/multi-change-queue/verified/20260526T2057-hero-isolate-webkit-crumb-g2.md
  - context/markdowns/multi-change-queue/verified/20260526T2057-highway-hero-iphone-small-recenter-g2.md
serves_northern_star: G2  # migrated 2026-05-26 - goal_id=G2 matches current NS
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G2 — Evium Phase 5 site overhaul

> ## ⛔ SHELVED 2026-06-08 — DO NOT PUSH EVIUM
>
> **SHELVED 2026-06-08 per user** — not resuming near-future; project pivoted to **Back Lawrence
> Shul** on 2026-06-02 (the active Northern Star). This goal moved `active/ → paused/` (canonical
> non-destructive lane; nothing deleted, fully resumable on explicit user say-so).
>
> **⚠ DO-NOT-PUSH-EVIUM** until the contact / subscribe / quote forms are wired. Broken forms live
> on the frozen `main` branch — pushing `main` would put a lead-loss gun live (forms that *present
> as success* but send nowhere; bug `#20260529T0612-main-claude-main-88d2a292`). This is a
> **push-time gun, NOT a current live leak** (the live tree has no forms). Resume + wire the forms
> *before* any Evium push. Gate also recorded in `context/markdowns/notes/reminders.md`.

## Why it exists

The Astro rebuild has shipped the structural foundation. The full-coverage audit on 2026-05-25 identified specific defects across 8 pages + 12 device projects. Phase 5 = methodical defect remediation, verified per-device via the scheduler.

This is the actual client deliverable.

## Current state

- 8 pages built (services, multifamily, highway, fleets fullPage + products, contact, blog normal-scroll)
- 1434 audit artifacts captured 2026-05-25 (full-coverage run + overnight archive)
- Consolidated findings in `context/markdowns/audits/full-coverage-2026-05-25/consolidated-findings.md`
- Bug billboard inbox currently empty (most recent consolidation merged everything)
- **P0 batch all verified via scheduler 2026-05-26 (3/3)**:
  - P0-01 /products §01 black-rectangle → crumb restore (17s wall-clock, 3-shard)
  - P0-02 /blog §06 subscribe form overflow (verified + re-queued for regression)
  - P0-03 /contact §01 form-card edge bleed + input affordance + asterisk (23s wall-clock, 3-shard, 2-file diff)
- **P1 batch verified via scheduler**:
  - P1-04 /blog §03+§04 light-surface — **DEFERRED, awaiting user clarification** (chamber action 20:53Z requested re-pose)
  - P1-05 /highway §01 hero object-position
  - P1-06 /blog §01 NOW-READING + uneven teaser cards (equalize)
  - P1-07a + P1-07b /highway §04+§05 polish (chamber redesign action 17:34Z)
  - P1-08 eyebrow sitewide left-padding token
  - Plus: contact-tablet-gutter, multifamily-quote-stats-stack, highway-econ-narrow, services-terminal-narrow
- **P2 batch verified**:
  - P2-09 on-surface variant lift
  - P2-10 blog section split
  - P2-11 services logo grid
- **Context-drift fixes verified** (regressions caught during the P1/P2 sweep):
  - hero-isolate-webkit-crumb (cross-engine)
  - highway-hero-iphone-small-recenter
  - fleets-hero-text-shadow-firefox
- **Scheduler infrastructure changes verified**: phaseB-multishard-multifamily + phaseD-integration-retry

## Blockers

- None as of 2026-05-26T14:12Z. G1 unblocked us; scheduler is production-stable.

## Next action

With 19 fixes shipped + verified, the queue moves to:
1. **P1-04 light-surface clarification** — re-pose the question per chamber action 20:53Z (white-drift vs intentional-light disambiguation).
2. **P3 sweep** — re-audit content rendering across all 12 device projects against the post-fix baseline to surface any remaining or regression-induced defects.
3. **Snapshot baseline regeneration** — verify against clean state after the P1 + P2 batch landed.
4. **Final consolidated audit** — check the ≤3 P1-or-higher findings exit criterion before client signoff.

## Exit criteria

- [x] **3/3 P0 defects verified via scheduler (2026-05-26)**
- [ ] All P1 defects triaged
- [ ] Visual snapshots regenerated for all 12 projects from clean state
- [ ] Final consolidated audit shows ≤3 P1-or-higher findings
- [ ] User signoff for client handoff

## History

- 2026-05-24 — project bootstrap (Astro fork from the reference template)
- 2026-05-25 — full-coverage audit captured; consolidated findings produced
- 2026-05-26 AM — paused while scheduler MVP completes (G1 prerequisite)
- 2026-05-26 PM — G1 unblocked; P0 batch all 3 verified via scheduler (products / blog-subscribe / contact-form). 3-shard matrix per change.
- 2026-05-26 PM (later) — P1 + P2 + context-drift batch all verified; 19 total fixes shipped + verified via scheduler. P1-04 deferred per chamber-action user request for clarification.
- 2026-05-26T22:49Z — client signoff package assembled at `reports/g2-signoff/` (README + 00-summary + 01-fixes-manifest + 02-verification-evidence + 03-known-remaining). 5 files, all 19 fixes documented. Awaiting client review + 3 monkey decisions (P1-04 light-surface / P1-07a highway corridor / P1-07b services rewrite) before final per-device audit pass and goal closure.
- 2026-06-03 — demoted from Northern Star (user #63: BLS pivot); Evium parked HIGH-PRIORITY awaiting client review + push.
- 2026-06-08 — **SHELVED per user** ("not continuing Evium in the near future; I asked you to pin it"). Status `active → shelved`; file moved `active/ → paused/` (canonical non-destructive lane). Recorded the **DO-NOT-PUSH-EVIUM** gate (broken contact/subscribe/quote forms on frozen `main` = a push-time lead-loss gun, not a live leak). Fixed stale `linked_bugs` annotation → named the real critical bug. Contact-form bug `#…88d2a292` reframed PIVOT-FROZEN (kept OPEN, demoted from live-CRITICAL) via bug-billboard inbox-append. Gate also in `notes/reminders.md`. Backup: `context/markdowns/goal-billboard.bak-evium-20260608T162237Z`.

## Notes

- Audit type discipline: this is "content rendering verification" (per device per page per section), NOT "system infrastructure verification" — see `feedback_audit-type-disambiguation.md`
- HeaderBatteryLefter.astro is LOCKED — only its constants block can be edited
- Design is locked — no restyling / refactoring beyond defect remediation
