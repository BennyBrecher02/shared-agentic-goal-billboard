---
audit_id: A49
title: Chamber-empty domino + auto-plow pivot — when banana backlog drains, system auto-pivots to G2 plowing (closes the trigger gap A44 reflex caught)
status: in_progress
catalogued: 2026-05-27T05:36:05Z
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-27T05:36Z — user explicit: *"why didnt clearing the monkey backlog trigger our planning+plowing toward the full coverage e2e we planned with the hook plans and domin plans??? also that plan is wayyyy out of date by now probably and should be not scrapped but rather updated/upgraded."* A44 critical-finding-reflex firing — same class as A36 (instrumentation without action), A40 (substrate without query), A48 (status without auto-sync). HERE: triggers without backlog-state-aware pivot.
deferral_reason: NONE — substrate exists (G6 Phase 3 fan-out + backlog status in data.json); just missing the empty-state pivot mechanism
related_goals: [G14, G6, G2]
related_plans: [context/markdowns/plans/automation/chamber-empty-domino-and-plow-pivot-plan.md, context/markdowns/plans/g2-full-coverage-plow-plan.md (TO BE WRITTEN OR REFRESHED)]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - scripts/monkey-click-fan-out.sh (G6 Phase 3 — fires per-click; A49 extends with backlog-state awareness)
  - context/markdowns/plans/automation/monkey-domino-trigger-upgrade-plan.md (predecessor; A49 = next-layer)
  - A36 (instrumentation without action), A40 (substrate without query), A44 (reflex), A48 (status sync) — A49 is the action-pivot sibling
findings: []
---

# A49 — Chamber-empty domino + auto-plow pivot

## The user's frustration (verbatim)

*"why didnt clearing the monkey backlog trigger our planning+plowing toward the full coverage e2e we planned with the hook plans and domin plans??? also that plan is wayyyy out of date by now probably and should be not scrapped but rather updated/upgraded."*

## What's broken — the TRIPLE gap

| Gap | What | Why it bit us |
|---|---|---|
| 1 | `monkey-click-fan-out.sh` fires **per-click** via inbox-server POST handler — not on chamber state changes | I flipped 9 decisions via direct data.json edit → bypassed inbox-server entirely → 0 fan-out fires |
| 2 | Even for the 3 legit chamber clicks that DID POST (SET-005, RENAME-001, INSTALL-001), the fan-out's 8 triggers (asks-log, steering, heartbeat-tier-high, NS-attribution, dashboard, scheduler-auto-queue, bug-billboard, feedback-loop) **do NOT include "pivot to G2 plowing"** | "Plowing" wasn't part of the original G6 Phase 3 design |
| 3 | **No "backlog drained to 0"** detection exists anywhere → no signal even to consider plowing | Empty-state was assumed to be the user's pivot decision, not a system trigger |

A44 reflex check: this is the SAME class as A36/A40/A48 — substrate built (G6 Phase 3, data.json status, A46 empty-troop-rest UI) but the ACTION layer (auto-pivot) missing.

## The redesign — 3-layer auto-pivot

### Layer 1 — Detection: chamber-empty hook
`scripts/hooks/chamber-empty-detect.sh` (PostToolUse on Bash matching `data.json|monkey-decisions|sync-monkey`):
- Reads `public/screenshot-monkey/data.json`
- Counts decisions with `status == 'pending'`
- If 0 AND last-non-zero state was within last 5 min (recency check; avoids firing on session-start when chamber's been empty for hours): **EMIT chamber-empty event**
- Writes `.claude/cache/chamber-empty-events.jsonl` with `(timestamp, prior_count, drained_via=click|flip|sync)`

### Layer 2 — Domino: chamber-empty fan-out
`scripts/chamber-empty-fan-out.sh` (invoked by Layer 1 hook):
- Trigger A: Steering-log entry "banana backlog drained — system auto-proposing plowing pivot"
- Trigger B: Read `context/markdowns/plans/g2-full-coverage-plow-plan.md` for next-batch proposal
- Trigger C: Surface "🐒 Chamber clear — ready to plow N items in batch B-K?" in next user-prompt response via inbox message (urgent priority)
- Trigger D: Pre-stage the G2 batch BG prompts (don't launch; just have them ready in `/tmp/g2-plow-ready/`)
- Trigger E: Update billboard with "READY-TO-PLOW" indicator chip

### Layer 3 — Plow plan currency
`context/markdowns/plans/g2-full-coverage-plow-plan.md` — REFRESHED on every drain event:
- Latest outstanding G2 audit findings (queries `context/markdowns/audits/`)
- Sorted by severity × scope
- Grouped into batches of ≤5 changes per scheduler tick (per A21 ceiling)
- Per batch: estimated tokens, estimated wall-clock, dependencies
- LIVE document — auto-refreshed; user reads then decides

### The pivot moment (UX)

After Layer 1+2 fire, user's next chat response sees:
```
🍌 Chamber clear (just drained 9 decisions).
   Ready to plow G2 batch B-K1: 4 audit findings, ~1.2M tokens, ~25min wall-clock.
   Reply "plow" to launch the batch, or "next" to see batch K2.
```

User's "plow" = launch the BG batch automatically; "skip" = stay in current focus.

## Phase plan

### Phase 1 — Foreground design + BG build
- Foreground: A49 audit + plan + plow-plan skeleton
- BG #A: build Layer 1 + Layer 2 scripts + dry-run hook + unit tests; A47 discipline
- BG #B: refresh full-coverage E2E plan (`g2-full-coverage-plow-plan.md`) with current audit findings, batches, estimates

Token budget BG #A: <300k; BG #B: <250k.

### Phase 2 — Wire + live
- Wire Layer 1 hook into `.claude/settings.json` via settings-patches (joins SET-005 batch wave or A49-specific subsequent batch)
- 7-day dry-run observation
- Switch to LIVE

### Phase 3 — Generalize the pattern
- Chamber-empty is the FIRST "X-state-becomes-N" trigger
- Same pattern applies to:
  - Inbox-drained (queue=0) → propose next BG dispatch
  - All-tests-pass (failures=0) → propose next change merge
  - Scheduler-empty → propose next batch from billboard
- Build a small generic substrate `scripts/hooks/state-becomes-N.sh` and register specific events

## Cross-references

- G6 Phase 3 monkey-domino-trigger-upgrade (predecessor; A49 extends with state-awareness)
- A36 / A40 / A44 / A48 — same "substrate exists but action missing" family
- A46 empty-troop-rest UI — co-fires on chamber-empty
- A40 idea-gap-detector (sibling: catches gaps; A49 catches READINESS-to-pivot)
- G2 — Northern Star (THE pivot target)
- G14 — master audit owner

## Risks + mitigations

- **False fire on session-start** (chamber's been empty for hours; hook fires when agent reads data.json on boot) → Mitigation: Layer 1 recency check (was non-zero in last 5 min)
- **Cascade plowing** without user pause → Mitigation: pivot SURFACES proposal; user must reply "plow" to launch
- **Plow plan rot** → Layer 3 auto-refresh keeps it current per drain event
- **Wrong batch sequencing** → Plow plan groups intelligently; user can pick alternate batch

## Lessons (preliminary)

- THIS is the 5th instance of the same class today (A36/A40/A48 prior; A49 here). The class: **"hook chains do per-event refresh but never propose ACTIONS based on state transitions."**
- The fix pattern keeps repeating: detect state transition → propose specific action → user one-word approves → execute.
- Phase 3's generalization (`state-becomes-N`) addresses the META-class — every "X just became 0" deserves the same treatment.

## Status

IN PROGRESS — design landed; Phase 1 BGs queued.

