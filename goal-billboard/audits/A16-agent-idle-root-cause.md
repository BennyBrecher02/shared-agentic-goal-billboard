---
audit_id: A16
title: "Agent-idle root cause — why responses end without progressing the queue"
status: in_progress
catalogued: 2026-05-26T22:00:00Z
priority_when_run: P0
estimated_effort: medium (root-cause investigation + structural fixes + verification)
trigger: User pushback 2026-05-26T20:54Z — "investigate why you keep stopping the conversation fully without progressing our queue. audit+plan maxmode, go for it"
deferral_reason: NONE — running now. Stop-hook band-aid (goal-queue-check.sh) already shipped 2026-05-26 PM but the root-cause analysis was never written up as a formal audit. Without the audit, we'll keep re-discovering the same failure modes from different angles.
related_goals: [G1, G2]
related_plans:
  - context/markdowns/plans/automation/anti-idle-infrastructure-plan.md
related_refs:
  - scripts/hooks/goal-queue-check.sh
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
  - .claude/memory-mirror/feedback_standing-protocols.md
findings: []
owner: agent
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G1
---
# A16 — Agent-idle root cause audit

## Why this audit matters

Repeated user pushback: the agent ends responses **without launching the next BG agent**, even when:
- the goal billboard has active goals in non-terminal phases
- the audit catalog has audits with `status: in_progress`
- the inbox is empty (no waiting on user)
- token budget + 5hr-window have headroom

The Stop hook (`scripts/hooks/goal-queue-check.sh`) was added 2026-05-26 PM as a band-aid — it surfaces a system-reminder if pending work exists when the agent stops. That mitigates the symptom but **does not address the root causes**. Without root-cause coverage we'll see the same idle behavior recur in slightly different shapes.

## Hypothesized root causes (≥5)

### RC-1: "Queue empty" semantic collision

The agent's TaskCreate list (in-session) can be empty while the goal billboard has long-running work. The agent equated "TaskCreate done" with "queue done." Fixed in part by the Stop hook reminder, but the agent still mentally exits when the small list is clear.

**Evidence**: multiple session transcripts where the agent declared "queue empty" with G2's P1 list waiting + audits A13/A14/A15 in_progress at the same moment.

### RC-2: No structural BG-launch budget per user response

There is no rule that says **"every user response must launch ≥1 BG agent unless [exception]"**. The agent decides per-turn whether to launch; without a floor, it sometimes opts not to. The right shape is: minimum 1 launch per response, with documented exceptions.

**Evidence**: the user's framing was exactly this — "I want at minimum one BG agent kicked off per response unless [exception]."

### RC-3: Misclassifying "wait for verification" as "wait for user"

When a recently-launched BG agent is still running, the agent thinks "I'm waiting for that to finish — I'll stop." But:
- Multiple BG agents in parallel is the design (parallel-coverage discipline)
- Verification of a prior change does NOT block launching the next change (decoupled changes)

**Evidence**: the multi-change scheduler explicitly supports parallel changes; the agent's mental model didn't.

### RC-4: Confusion between "report" mode and "execute" mode

When the user asked a question that LOOKS like reporting ("what's the status of..."), the agent slipped into a pure-status-report turn and ended without execution. But almost every report should ALSO advance the queue — status report + launch is the right shape.

**Evidence**: user said "you keep stopping… without progressing our queue" — implying the agent was answering rather than acting.

### RC-5: No "minimum exit checklist" before the agent can stop

Other systems with auto-shutdown semantics have an exit checklist (Mac launchd: prevent_idle_sleep flag; ssh sessions: TTL settings). The agent has no such checklist. Stop happens whenever the agent decides "nothing left." A pre-stop checklist would prompt: have you (a) launched ≥1 BG agent if eligible, (b) consumed inbox, (c) updated the goal billboard, (d) recorded steering decisions, (e) heartbeat-drained?

### RC-6: Hook output not surfaced to the agent's "decide to stop" moment

The Stop hook surfaces a reminder for the NEXT response, but the **same-turn** decision to stop is made WITHOUT seeing the goal billboard state. The reminder shapes the next turn, not the current one. So the agent stops first, then the reminder appears. By that time the user has seen "you stopped" and the trust hit is taken.

### RC-7: KPI loop not auto-firing on the "you stopped without progressing" condition

The kpi-self-eval-workflow ref documents the 6-step loop for KPI regressions. "Agent stopped without progressing" is a regression itself but no measured KPI fires the loop. We need a KPI for it.

## What the Stop hook fixes vs what it doesn't

### What it fixes (the band-aid)
- Surfaces a goal-billboard reminder on the NEXT response, so eventually the agent re-engages.
- Catches "queue empty" when audits or active goals exist.

### What it doesn't fix
- The agent still stops first, surface second. The user sees the stop.
- No floor on per-response BG launches.
- No exit-checklist at the moment of stopping.
- No KPI tracking the regression to drive auto-fixes.
- No semantic test of "does the user want a report or action right now?"

## Recommended additional structural fixes

The full plan lives in `context/markdowns/plans/automation/anti-idle-infrastructure-plan.md`. Headline items:

1. **BG-launch budget protocol**: minimum 1 BG-agent launch per user response unless ≥1 of these exceptions hold: (a) destructive action awaiting user nod; (b) ambiguity requires user input; (c) token budget >85% AND <30min to reset; (d) user explicitly told us to wait. Each exception logged inline in the response (so the user can disagree).
2. **"Queue empty" semantic test**: before the agent decides to stop, it MUST run `bash scripts/hooks/goal-queue-check.sh` from inside the response (not just rely on the Stop hook to catch it after-the-fact). If non-empty → MUST launch.
3. **Heartbeat-tied surfacing**: when a BG agent completes during dormancy, the heartbeat queue surfaces the completion on the next prompt. Agent reacts to it within the same response.
4. **Escalation ladder**: if the Stop hook fires 3 times in a row in the same session (= structural fix isn't working), surface a stronger system-reminder telling the agent to immediately surface the failure and switch to acting.
5. **KPI for idle**: track `idle_responses_per_session` (responses where Stop hook fired despite pending work). The 6-step loop runs whenever this regresses.

## How to trigger

User trigger 2026-05-26T20:54Z met. Re-trigger naturally if: another idle-stop occurs after the anti-idle plan ships AND a Stop-hook reminder was already in the prior turn.

## Resolution criteria

- A16 catalogued (this file) + linked plan landed
- Stop hook + at least 2 of the 5 structural fixes implemented + verified by 1 session
- KPI for `idle_responses_per_session` recorded for 3 sessions
- Memory in `feedback_standing-protocols.md` updated with the BG-launch budget protocol

## Cross-references

- `scripts/hooks/goal-queue-check.sh` — the Stop hook band-aid
- `context/markdowns/plans/automation/anti-idle-infrastructure-plan.md` — full plan
- `agentic-quality-discipline/references/kpi-self-eval-workflow.md` — KPI auto-loop pattern
- A14 (stats correlation) — provides the data substrate for the idle-responses KPI
