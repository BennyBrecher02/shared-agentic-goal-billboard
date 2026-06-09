---
audit_id: A50
title: monkey-click-fan-out.sh is stuck in LOG_ONLY/dry-run mode FOREVER — 11 real clicks fired ZERO live triggers
status: in_progress
catalogued: 2026-05-27T05:47:00Z
priority_when_run: P0
estimated_effort: small (flip env-var) + medium (verify each trigger actually works live)
trigger: |-
  2026-05-27T05:47Z — user explicit "audit and test: monkey-click-fan-out.sh fires PER-CLICK". Audit revealed claim is WRONG in practice. Fan-out log has only 4 decisions × 10 events = 40 entries, ALL with `LOG_ONLY=1` flag, ALL from 02:36-02:37Z TEST run. 11 real DECISION POSTs today (04:34Z-05:32Z) produced ZERO fan-out events. The fan-out script is invoked by the server but in dry-run mode permanently.
deferral_reason: NONE — direct cause of "clearing didn't trigger plowing" — same root as A49
related_goals: [G14, G6, G2]
related_plans: [context/markdowns/plans/automation/monkey-fan-out-dry-run-to-live-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - scripts/monkey-click-fan-out.sh (the broken-in-dry-run script)
  - .claude/cache/monkey-fanout-log.jsonl (evidence — 40 entries all LOG_ONLY=1)
  - /tmp/agent-inbox-server-4323.log (evidence — 11 real DECISION POSTs today)
  - A49 chamber-empty audit (sibling — different mechanism for state-transition triggers)
  - A36 / A40 / A44 / A48 — same substrate-built-not-activated family
findings:
  - finding_id: F50-1
    severity: P0
    title: LOG_ONLY=1 flag stuck in fan-out script forever
    evidence: .claude/cache/monkey-fanout-log.jsonl all 40 entries detail field contains "LOG_ONLY=1; would [action]"
  - finding_id: F50-2
    severity: P0
    title: 11 real clicks today produced 0 fan-out events
    evidence: server log 11 DECISION POSTs at 04:34Z-05:32Z; fan-out log latest entry 02:37Z
---

# A50 — Monkey fan-out stuck in dry-run

## The audit (what user asked for)

User: *"audit and test: monkey-click-fan-out.sh fires PER-CLICK"*

**Test result: FAIL.** It does NOT fire per-click. It fires only when invoked by the server, AND only in LOG_ONLY mode (which means: writes to log saying "would fire" but doesn't actually fire).

## Evidence chain

### Real clicks today (server log)
11 DECISION POSTs received between 04:34:35Z and 05:32:26Z:
- P1-07b-redesign / SET-001 / SET-002 / P1-07a / SET-003 / SET-004 / SET-005 / RENAME-001 / INSTALL-001 / P1-04 (plus connectivity test)

### Fan-out events (log)
40 events total, ALL from earlier test run (02:36-02:37Z):
- P1-04(a) / SET-005(yes-lift-and-apply-all) / INSTALL-001(install) / RENAME-001(yes)
- ALL entries have `LOG_ONLY=1` flag

### The gap
11 real clicks → 0 corresponding fan-out events. Even when invoked (test run), every trigger is `would-fire` not `did-fire`.

## Root causes — TWO layered bugs

### Bug 1: `LOG_ONLY=1` permanently set
The fan-out script was built with a dry-run flag, presumably for safe initial deployment. It was never flipped to live. **Build-then-forget pattern** — same as A36/A40/A48/A49.

### Bug 2: Server may not be invoking fan-out on every POST
Even after Bug 1 fixed, need to verify `agent-inbox-server.py` actually calls `subprocess.Popen(["bash", "scripts/monkey-click-fan-out.sh", ...])` on each POST. If it doesn't, no amount of dry-run flipping helps.

## Fix phases

### Phase 1 — Verification + flip (small)
1. Read `scripts/monkey-click-fan-out.sh` — find the LOG_ONLY check
2. Identify what controls it (env var? hardcoded?)
3. Test in isolation: invoke with LOG_ONLY unset, confirm each trigger ACTUALLY fires
4. Land the flip (env var default = unset OR hardcoded default = live)
5. Verify in `agent-inbox-server.py` that subprocess.Popen call exists + executes on every POST
6. Test end-to-end with synthetic POST

### Phase 2 — Per-trigger validation
Each of the 8 triggers needs verification that:
- The actual action it claims fires
- It writes the artifact it claims to write
- It doesn't break anything when wrong

### Phase 3 — Live + monitor
- Switch to live
- Watch first 5 real clicks → confirm fan-out events appear in log with `did-fire` outcomes
- If any trigger fails, log it + degrade gracefully

## Cross-references

- A49 — chamber-empty domino (sibling; different mechanism for state transitions)
- A36 — true-time discipline (similar instrumentation-without-action)
- A40 — idea-gap detector (substrate-built-not-queried cousin)
- A44 — Critical Finding Reflex (this audit IS the reflex firing)
- A48 — data.json auto-sync (chamber-side sibling)
- A47 — BG lifecycle (parallel mechanism work)

## Lessons (preliminary)

- **6th instance of same META-class today** — substrate built, action layer never activated. A36/A40/A44/A48/A49/A50.
- The pattern is so common it warrants a META-META audit: how do we know any substrate built today is ACTUALLY ACTIVE vs sitting in dry-run forever?
- A47 Phase 1 plans a dry-run-to-live verification step; A50 confirms this needs to be universal.

## Status

IN PROGRESS — audit + plan landed; Phase 1 work needed (verify + flip + test). Can be foreground work in next session OR spawned as BG with A47 discipline.

