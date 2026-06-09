# /products § 01 DESKTOP eyebrow + pillar-readout collision (CROSS-ENGINE confirmed)

- **id:** 20260527T1204-main-claude-main-p2-14-products-desktop-eyebrow
- **discovered:** 2026-05-27T12:04:00Z
- **agent:** claude-main (P2-14 calibration audit BG)
- **worktree:** main
- **route:** /products
- **section:** § 01 (desktop viewport — distinct from P0-01 mobile black-rectangle)
- **viewport-bucket:** desktop (1440×900)
- **severity:** high
- **status:** open
- **defect-type:** layout-overflow
- **dedupe-key:** /products|§01|desktop-eyebrow-pillar-collision|webkit-firefox

## Description

P2-14 chromium-desktop calibration audit cross-checked /products § 01 against webkit-desktop + firefox-desktop at 1440×900. Both engines independently flagged eyebrow + pillar-readout layout collision at this viewport. chromium-desktop scored "clean" (missed it).

This is DISTINCT from the P0-01 mobile black-rectangle artifact (which B-K1 just closed for ≤640px). The desktop bug is the eyebrow component layering over the pillar-readout text.

## Repro

1. Open /products at 1440×900 in WebKit or Firefox
2. Observe § 01 — eyebrow + pillar-readout overlap

## Source

B-K5 P2-14 calibration spot-check BG report at `reports/audit-findings/B-K5-P2-14-calibration-spot-check.md`.

## Cross-references

- B-K5 calibration audit (origin)
- P0-01 just-closed in B-K1 (different fix; mobile only)
- A44 critical-finding-reflex (cross-engine consensus rule discipline gap)
- Recommendation from P2-14: add to next plow batch as desktop-specific eyebrow fix
