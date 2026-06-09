# /contact § 01 DESKTOP form-card column misalignment (~30-35px, CROSS-ENGINE confirmed)

- **id:** 20260527T1204-main-claude-main-p2-14-contact-desktop-form-misalign
- **discovered:** 2026-05-27T12:04:00Z
- **agent:** claude-main (P2-14 calibration audit BG)
- **worktree:** main
- **route:** /contact
- **section:** § 01 form-card (desktop viewport — P0-03 closure was PARTIAL)
- **viewport-bucket:** desktop (1440×900)
- **severity:** high
- **status:** open
- **defect-type:** layout-overflow
- **dedupe-key:** /contact|§01|desktop-form-col-misalign|webkit-firefox

## Description

P2-14 chromium-desktop calibration audit cross-checked /contact § 01 at 1440×900 in webkit + firefox. Both engines independently MEASURED column misalignment of ~30-35px + flagged input affordance weakness. chromium-desktop scored "clean."

**P0-03 closure was PARTIAL** — B-K1's fix addressed mobile/iPad scope only (margin-inline + clamp + 1.5px borders + asterisk + focus-visible). Desktop column alignment + input affordance remain broken at ≥desktop viewports.

## Repro

1. Open /contact at 1440×900 in WebKit or Firefox
2. Observe § 01 form-card — left/right column misalignment ~30-35px
3. Inputs feel under-affordance (low contrast / thin / unclear focus state)

## Source

B-K5 P2-14 calibration spot-check BG report at `reports/audit-findings/B-K5-P2-14-calibration-spot-check.md`.

## Proposed fix scope

Per B-K5 recommendation: extend P0-03 fix to include webkit-desktop and firefox-desktop viewports in CSS diff scope. The mobile/iPad path is correct; need a `@media (min-width: 901px)` block that aligns the form-card columns + tightens input affordance.

## Cross-references

- P0-03 closed-but-partial in B-K1 (`reports/audit-findings/B-K1-plow-apply.md`)
- B-K5 calibration audit (origin) 
- A44 critical-finding-reflex (cross-engine consensus rule discipline gap)
