---
belongs_to_goal: G2   # by-goal parent (goal-wiring §0 tier-A — filename/topic names the goal)
pin_id: P-001
title: Wave 3 audit completion (5 missing devices)
created: 2026-05-28
narrowed_in: marathon session 2026-05-27 → 2026-05-28 (multiple turns; second-pass audit operational plan)
why_paused: rate-limit-hit ghosted 6 wave-3 BG dispatches mid-flight; auto-pinned per A47 + new dual-signal ghost detection
resume_criteria: budget refreshed (next 5hr window or weekly limit reset) AND PIL pre-shrink reliable per device (chromium-android-narrow's BG had to self-shrink — script needs to bake into capture pipeline)
estimated_cost: ~2.5M tokens (5 evaluator BGs × ~500k each)
leverage_g2: 16.7
ns_relation: DIRECT G2 close — completes 5 of 12 device coverage gap; without this, "true 100% e2e" remains incomplete
last_touched: 2026-05-28
state: pinned
---

# P-001 — Wave 3 audit completion (5 missing devices)

## Scope

Re-dispatch the 5 audit evaluator BGs that rate-limit-ghosted:

1. webkit-iphone-large
2. webkit-iphone-small
3. chromium-android (full Pixel)
4. chromium-android-tablet
5. firefox-laptop + chromium-laptop (combined; bundled in original dispatch)

Plus the consolidator (tracked separately as P-002).

## Sibling reference files (read first by each evaluator)

- `reports/audit-findings/g2-second-pass/wave1-webkit-desktop-evaluator.md` (51KB — highest-coverage anchor)
- `reports/audit-findings/g2-second-pass/wave2-webkit-ipad-mini-evaluator.md` (49KB — 100% coverage exemplar)
- The operational plan: `context/markdowns/plans/g2-second-pass-audit-operational-plan.md`

## Dispatch prompts (copy verbatim when resuming)

Each evaluator's prompt is in chat history of the 2026-05-28 session under "wave 3 dispatch" block. They mirror the wave-1 pattern with these additions:
- Use `audit-ready/` not raw vq-captures (PIL pre-shrink dependency)
- Reference wave-1 + wave-2 siblings as format/scope anchors
- Verify ALL 5 second-pass remediation patches (commits `382886e` / `1389aad` / `c19645e` / `153b4fc` / `1158d35`)

## What lands on completion

7 of 7 device evaluator artifacts under `reports/audit-findings/g2-second-pass/` covering:
- Wave 1: firefox-desktop / chromium-desktop / webkit-desktop / webkit-iphone / chromium-android-narrow ✅ already landed
- Wave 2: webkit-ipad / webkit-ipad-mini ✅ already landed
- Wave 3: webkit-iphone-large / webkit-iphone-small / chromium-android / chromium-android-tablet / firefox-laptop / chromium-laptop ❌ pending this pin

## Cost-aware resume notes

- Each BG ~500k tokens budget; running 5 in parallel concentrates ~2.5M consumption in one cycle
- Recommend launching post-mutation-OS B-gate (already done) AND post-PIL-bake-into-capture-pipeline
- DO NOT re-dispatch into the same rate-limit-wall pattern — verify clearance via test BG first
- Consolidator (P-002) can run on existing 7 evaluators OR wait for wave-3 completion to maximize coverage

## Cross-references

- P-002 (consolidator) — depends on this (or runs on 7-of-12 coverage)
- A47 BG lifecycle discipline — ghost handling
- A22 cluster-and-ns-integration-gap — if M2 cluster ready, sharding cuts cost
