---
audit_id: A19
title: "Cross-system meta-monitoring — recurring-failure-class blindness + workflow bypass"
status: in_progress
catalogued: 2026-05-26T22:25:00Z
priority_when_run: P0
estimated_effort: large (multi-layer structural fix; ~3-5 hours of work spread across foreground artifacts + BG implementations + verification)
trigger: |-
  2026-05-26 22:08-22:25 PM — user inbox + meta-prompt revealing that despite 18 catalog audits + 30+ plans + memory rules + hooks, the agent doesn't detect WHEN user is repeating themselves, doesn't proactively query the existing logging substrate, and bypasses the established scrutiny workflows when iterating (e.g. Text v2 + Lefter v2 BG agents iterated on intuition; never ran vq:capture + 8-axis rubric on the dashboard itself). User's framing: *"how can things possibly ever slip through the cracks? youre not using our logging at all... theres an invisible master plan and master audit that must be made."*
deferral_reason: NONE — running immediately. A18 closes per-prompt artifact gaps; A19 closes cross-prompt pattern blindness + workflow bypass. Both are load-bearing.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/master-plan-cross-system-meta-monitoring.md]
related_refs:
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
  - .claude/skills/agentic-page-scrutiny/SKILL.md
  - context/markdowns/goal-billboard/audits/A18-plan-audit-artifact-creation-gap.md
  - ~/.claude/projects/<PROJECT_SLUG>/memory/feedback_standing-protocols.md
findings: []
serves_northern_star: G2  # migrated 2026-05-26 - path 'meta-monitoring' -> GL G4; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G4
---
# A19 — Cross-system meta-monitoring

## Why this audit matters

A18 is the per-prompt fix: detect a trigger phrase in this prompt, surface a reminder, check at Stop that an artifact landed. **But A18 doesn't see RECURRENCE.** If the same complaint lands 5 times in one day, A18 only catches whether THIS prompt produced an artifact — it doesn't notice the user has been asking the same thing for hours.

The user's framing is exact: *"our entire system hinges on our conversing so if we cant rely on that then im pulling the trigger for an emergency brainstorm audit."* The system has accumulated impressive substrate — 18 catalog audits, 30+ plans, 6 active goals, multiple JSONL logs, hooks chained on every event. **And things still slip.** A19 audits WHY.

## Failure-class receipts (this session, 2026-05-26)

| Class | First raised | Repetitions | Eventually fixed by |
|---|---|---|---|
| Inbox-miss (messages not delivered/acked in chat) | ~15:37Z | 5+ | PreToolUse hook `<system-reminder>` wrap (2nd hardening) |
| Idle pattern ("you keep stopping") | 20:32Z | 3+ | Stop hook `goal-queue-check.sh` structural directive |
| Skipped artifacts (plans/audits never created for asks) | 18:07Z | 8 examples (per A18 receipt grid) | A18 trigger-phrase detector |
| Stale state (5hr reset offset, G2 phase) | 21:23Z | 2 | `.claude/cache/5hr-window-reset.txt` fix + G2 phase bump |
| Dashboard quality not improving despite iterations | 21:42Z | 4 (21:42Z, 21:44Z, 22:08Z, this prompt) | **NOT YET — designed in master plan; BG launching to fix via established workflows** |
| Cringey phrasing ("monkey chamber walker") | 20:23Z | 2 | Monkey style memory rule |
| Time estimate exaggeration | 13:25Z | 1 (caught early) | Calibration table |
| Workflow bypass (LLM iterating without running scrutiny) | implicit, this prompt | many | **NOT YET — Layer 3 of master plan addresses** |

## Root causes (meta-level)

1. **No recurrence detector** — Each user pushback is treated as a fresh issue. No automated grep against prior session prompts for the same topic + frustration markers ("still", "again", "I keep asking", "haven't").

2. **No "established workflows" enforcement** — I don't structurally verify "am I using the right rubric/capture/test before declaring done?" Specifically: edited dashboard files → did I run vq:capture? Edited scheduler → did the test pass? The check is invisible until the user audits.

3. **Existing logging substrate is unread per-response** — `.claude/cache/user-asks-pending.jsonl`, `lost-work.jsonl`, `ack-latency.jsonl`, `scheduler-events/*.jsonl`, `heartbeat-pending.jsonl`, `goal-billboard/active/*.md`, `screenshot-monkey/data.json`, `improvement-tags.json` all carry state. I query them at SessionStart and rarely thereafter.

4. **Audit registry isn't cross-linked** — A1-A18 are siloed. No "audit-of-audits" panel showing patterns: how many audits are `in_progress`? Which trigger conditions are firing repeatedly? Are some audits effectively duplicates?

5. **No conversation-pattern KPI** — Ack-latency + lost-work + improvement-tags exist as POINT metrics. No aggregate "user-pushback per N prompts" trend that fires the 6-step loop when worsening.

6. **No pre-response state-batch reader** — Stop hook fires AFTER my response (too late to inform it). UserPromptSubmit only drains inbox + heartbeat (narrow scope). Nothing reads ALL the substrate state in one shot and surfaces it BEFORE I form a response.

## Why prior fixes can't reach this

- A18 = per-prompt artifact check. Doesn't see prompts N-1, N-2, N-3.
- `goal-queue-check.sh` (Stop) = AFTER response, not before; surfaces goal staleness, not user repetition.
- `inbox-on-prompt.sh` = drains inbox; doesn't cross-reference inbox content against past prompts.
- Memory rules = read at SessionStart; not enforced inline.
- Skill refs = passive documentation; require me to invoke them.

The COMMON GAP: **catch-at-source patterns exist for individual SIGNALS (inbox, goal staleness, asks)** but no pattern exists for **STATE BATCH** or **CROSS-PROMPT PATTERN**.

## Five-layer master plan (designed in same turn — see `master-plan-cross-system-meta-monitoring.md`)

- **Layer 1: Pre-response state-batch reader** (`state-batch-digest.sh`, UserPromptSubmit) — one consolidated `<system-reminder>` listing pending asks, lost work, ack-trend, stale goals, heartbeat alerts, monkey clicks pending. Catches the substrate I'm not reading.

- **Layer 2: Recurrence detector** (`recurrence-detect.sh`, UserPromptSubmit) — greps last-24h JSONL for matching topics + frustration markers; surfaces `<system-reminder>` "RECURRENCE DETECTED: topic X raised N times" with past-attempt list. Forces 6-step loop, blocks band-aids.

- **Layer 3: Established-workflows enforcement** (`established-workflows-check.sh`, Stop) — if response edited dashboard/UI/scheduler files without running the matching scrutiny pipeline (vq:capture, scheduler test), surface bypass warning + require steering-log skip.

- **Layer 4: Dashboard "Recurring-Pattern Watch" panel** + lefter mini-block — failure-class registry with recurrence counts visible to both of us. SHARED state.

- **Layer 5: Conversation-pattern KPI** — pushback-per-N-prompts rolling metric in Stats Correlation tab. Sparkline + threshold-alert when worsening.

## Specific application to the 22:08:37Z complaint

The inbox said: "the writing style of monkey notes are not user friendly also theres still major design kinks like rounded corners in squares and text not standing out well etc, why havent you been scrutinizing and iterating on the dashboard with our established workflows?"

Per Layer 3 (established-workflows enforcement), Text v2 + Lefter v2 BG agents should have been REQUIRED to run vq:capture + apply the 8-axis rubric. They didn't. They iterated on LLM intuition. The structural fix is Layer 3 catching this BEFORE the user has to.

Per Layer 4 (failure-class registry), "dashboard quality not improving despite iterations" should have been visible after the 2nd recurrence at 21:44Z, with the registry surfacing "this has happened twice already today — escalate to scrutiny pipeline." A BG agent is launching this turn to actually RUN the rubric pipeline on the dashboard.

## Verification (after BG agents land hooks + scrutiny)

1. Layer 1: synthetic UserPromptSubmit → confirm state-batch reminder fires with correct counts
2. Layer 2: synthetic prompt mentioning "inbox still not delivered" → confirm recurrence reminder
3. Layer 3: synthetic dashboard edit with no vq:capture → confirm bypass warning
4. Layer 4: dashboard panel renders failure-class registry; recurrence counts update
5. Layer 5: stats tab sparkline shows conversation-pattern metric
6. Dashboard scrutiny BG produces before/after rubric scores; defects A/B/C/D fixed

## Status

IN PROGRESS. Will flip to `resolved` once:
- Layers 1-3 hooks wired in settings.json (user-applied patch)
- Dashboard panel + KPI rendering
- One full cycle: recurrence detected → 6-step loop fired → resolution landed (verified by rubric improvement)
- Dashboard scrutiny pass closes the 22:08:37Z defects

## Lessons (preliminary)

- A18 is necessary but not sufficient. A19 catches what A18 can't (recurrence + workflow bypass).
- "Catch at source" needs to apply to STATE not just SIGNALS. Layer 1 is the state-batch generalization of `inbox-on-prompt.sh`.
- The user's pushback pattern is itself a KPI. Track it; act on it via the 6-step loop.
- LLM-driven iteration without running the actual scrutiny pipeline = workflow bypass. The pipeline IS the verifier. Bypassing it is the same failure class as agent-eyeballing browser state (which standing-protocols already bans).

## Cross-references

- A18 — companion audit (per-prompt artifact gap)
- `kpi-self-eval-workflow.md` — the 6-step loop applied to A19 itself
- `feedback_standing-protocols.md` — receiving new state-batch + recurrence rules
- `agentic-page-scrutiny/SKILL.md` — the rubric Layer 3 will enforce
- Inbox-miss structural fix (PreToolUse `<system-reminder>` wrap) — same pattern Layer 1+2 borrow

