---
belongs_to_goal: G2   # by-goal parent (goal-wiring §0 tier-A — filename/topic names the goal)
pin_id: P-002
title: Consolidator BG → formal G2-CLOSE verdict
created: 2026-05-28
narrowed_in: marathon session 2026-05-27 → 2026-05-28
why_paused: ghosted with the wave-3 batch (rate-limit-hit)
resume_criteria: budget for ~500k token BG dispatch. Can run on existing 7 evaluators (wave 1 + wave 2 — partial close); ideal after P-001 lands for full 12-device coverage
estimated_cost: ~500k tokens
leverage_g2: 14.0
ns_relation: DIRECT G2 close — produces the OFFICIAL G2-CLOSE-READY verdict that I previously declared prematurely
last_touched: 2026-05-28
state: pinned
---

# P-002 — Consolidator BG → formal G2-CLOSE verdict

## Why this matters

Throughout the marathon session I (the agent) declared "G2-CLOSE-READY" multiple times based on partial-device coverage. The user correctly called this out as premature each time. The CONSOLIDATOR BG produces the formal cross-engine merge + verdict per the operational plan § 5 — not an ad-hoc claim.

## Inputs

- ALL evaluator markdown files under `reports/audit-findings/g2-second-pass/`
- Currently 7 files (wave 1: 5 devices + wave 2: 2 devices)
- If P-001 lands first, 12 files total

## Consolidation rules (per ops plan § 5)

1. **Secondary-engine-anchored merge** — webkit + firefox primary; chromium-desktop confirmatory only (per P2-14 calibration)
2. **Fix-verification scoreboard** — per Phase 5 fix × all evaluated devices
3. **Regression vs closure vs did-not-land** classification per close-gate criteria:
   - ≤3 P1+ findings
   - 0 regressions
   - 0 did-not-land
   - ≥95% YES rate
4. **Patches already shipped** (since wave-1 dispatch) ACK'd as RESOLVED:
   - `382886e` fleets-04-caption-zorder (F-NEW-03 + F-NEW-04)
   - `1389aad` highway-01-mobile-hero (crumb-clip + CB-1 narrow miss)
   - `c19645e` services-05-logo-narrow (D-NEW-narrow-01)
   - `153b4fc` eyebrow-cluster-fixes (F-NEW-01 reinterpreted + F-NEW-02)
   - `1158d35` products-01-tablet-viewport-scope (M-NEW-01 + D-NEW-tablet-01)
5. **Explicit verdict line** — YES (close-ready) OR NO + reason

## Output

`reports/audit-findings/g2-second-pass/consolidated-findings.md`

## Dispatch prompt — verbatim when resuming

```
BG agent: read ALL `reports/audit-findings/g2-second-pass/*.md` files,
apply the secondary-engine-anchored merge per ops plan § 5,
produce `reports/audit-findings/g2-second-pass/consolidated-findings.md`
with cross-engine merge + fix scoreboard + regression classification
+ explicit G2-CLOSE verdict line. ACK already-shipped patches
(commits 382886e/1389aad/c19645e/153b4fc/1158d35). Cost <500k tokens.
```

## Cross-references

- P-001 — full wave 3 makes this consolidator's scope complete
- `context/markdowns/plans/g2-second-pass-audit-operational-plan.md` § 5 — consolidation rules
- `audits/full-coverage-2026-05-25/calibration-spot-check.md` — P2-14 engine-role assignments
