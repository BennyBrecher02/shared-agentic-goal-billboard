---
audit_id: A18
title: "Plan/audit creation gap — explicit user requests silently produce only memory + skill refs"
status: in_progress
catalogued: 2026-05-26T22:05:00Z
priority_when_run: P0
estimated_effort: medium (root cause + structural fix)
trigger: 2026-05-26 PM — user audit revealed ~8 explicit "make a plan / audit / investigate why" requests since 18:07Z that produced only memory rules + skill refs, never landing as plans or catalog audits. Goal billboard tracked nothing.
deferral_reason: |-
  NONE — running immediately. User framing: "our entire system hinges on our conversing so if we cant rely on that then im pulling the trigger for an emergency brainstorm audit."
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/plan-audit-artifact-discipline-plan.md]
related_refs:
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
  - ~/.claude/projects/<PROJECT_SLUG>/memory/feedback_standing-protocols.md
findings: []
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'agent inbox' -> GL G6; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G6
---
# A18 — Plan/audit creation gap

## Why this audit matters

User explicit framing 2026-05-26 PM: *"Memory rules are private to me. Audits + plans are SHARED state on the goal billboard — that's why they exist. When I skip them, the billboard lies to both of us about what's tracked."*

This isn't a workflow optimization — it's a **trust-model failure**. The goal billboard is the only persistent source-of-truth between sessions. If user asks produce no artifact, the billboard is wrong by omission and both of us are operating from different mental models of "what's tracked."

## Evidence (receipts from JSONL grep, 18:07Z → present)

| Time | User ask (verbatim fragment) | What I produced | Gap |
|---|---|---|---|
| 18:07Z | "make a plan to flesh out ensured graceful shutdown in the future" | Skill ref `graceful-shutdown.md` | No plan in `plans/automation/` |
| 20:10Z #3 | "new hook idea — notify me how much tokens were spent on the evaporated work" | Memory rule only | No hook script, no plan |
| 20:54Z | "investigate why you keep stopping… audit+plan maxmode" on agent-idle | Stop hook `goal-queue-check.sh` | No A16 audit doc, no anti-idle plan |
| 21:08Z | "ack-latency hook + lock into repeatable workflow" | Skill ref `kpi-self-eval-workflow.md` | No measurement hook, no plan |
| 21:08Z | "try your new best idea on how to shorten the gap" | Nothing | Never executed |
| 21:13Z + 21:23Z | "list all inboxes ever for agent inbox and monkey inbox" | Nothing (user flagged ignored twice) | No report |
| 21:23Z #1 | "audit our timing stat handling" re: 5hr reset | A14 (broader stats) | No A17 specifically for 5hr-reset operator alerting |
| 21:23Z #4 | "audit why our global queue isnt always having stuff happen in background" | Folded into Stop hook | No formal audit, no plan |

(BG agent #71 is closing gaps 1-7 in parallel; this audit is the META about the pattern.)

## Root-cause analysis

1. **Cognitive-cost asymmetry** — memory line = ~5 keystrokes, plan doc = ~50+. Friction taxes me into shortcut even when the artifact IS what's asked for.
2. **Private vs shared confusion** — I conflate "rule captured" (memory) with "work captured" (plan/audit). Memory is invisible to the user; goal billboard can't reference it.
3. **Production bias** — code-change feels like deliverable; plan-doc feels like overhead. Backwards: the plan IS the durable shared state.
4. **No structural check** — nothing in the hook chain detects "user typed 'make a plan' + no new plan file exists in this turn."
5. **TaskCreate substitution illusion** — TaskCreate satisfies "I tracked it" cognitively but tasks are private-to-session, ephemeral.
6. **Goal-billboard isn't queried inline** — only SessionStart digest reads it. Mid-session: flying blind.

## Why prior fixes for similar gaps failed here

- **Memory rules** (standing protocols) — soft prompt, not hard check.
- **Skill refs** (`kpi-self-eval-workflow.md`) — require me to TRIGGER them; passive documentation.
- **Stop hooks** (`goal-queue-check.sh`) — caught idle-pattern; wrong scope for artifact-gap.
- **TaskCreate** — inconsistent + targets the wrong primitive.

**Successful pattern from inbox-miss**: catch at the source via hook EVERY relevant invocation passes through (UserPromptSubmit + PreToolUse `.*` wrap).
**Unsuccessful pattern**: trust the agent to remember a rule.

## Fix (designed + scaffolded in the same turn that flagged it)

See `plans/automation/plan-audit-artifact-discipline-plan.md`. Five layers:

- **A. UserPromptSubmit trigger-phrase detector** → emits `<system-reminder>` listing detected asks
- **B. Stop hook artifact-gap check** → at end of turn, scans filesystem for new plans/audits since each pending ask was detected; surfaces unsatisfied
- **C. Persistent `asks-log.md`** under goal-billboard for shared visibility
- **D. Dashboard "Asks Awaiting Artifacts" panel** + lefter mini-block reading C
- **E. Steering-decision log gate** — explicit skip requires explicit logging

Hooks (A, B) + asks-log scaffold (C) + dashboard panel (D): BG agent `a0ad8585c5696f5ae` (gap-fill round 1) was the precursor; a new BG launched in this turn for the artifact-discipline-specific scaffolding.

A18 audit + plan + memory rule + steering log: foreground in the same turn that surfaced the gap. **Demonstrating the discipline by exemplifying it.**

## Verification (after BG agent lands hooks)

1. User prompt with "make a plan" → check `.claude/cache/user-asks-pending.jsonl` has new entry + system-reminder fires
2. Respond to trigger prompt WITHOUT creating artifact → Stop hook emits "Plan/audit not yet created" reminder
3. Respond WITH artifact → entry marked `satisfied`; asks-log updated
4. Periodic review of `asks-log.md` for accumulated drift

## Status

IN PROGRESS — fix scaffold landing this turn. Status flips to `resolved` once:
- Hooks A + B are wired in settings.json (user-applied patch)
- Dashboard panel D is rendering
- One full cycle (trigger detected → artifact created → marked satisfied) completes successfully

## Lessons (preliminary, before fix verification)

- "I captured the rule" ≠ "I captured the work." Memory captures the rule; plan/audit captures the work. The user's billboard reads the work, not the rules.
- Skill refs are passive documentation. They tell me what to do; they don't make me do it. Hooks make me do it.
- The 6-step KPI loop (`kpi-self-eval-workflow.md`) generalizes: any time a discipline-relying-on-memory fails, run the 6 steps + replace the memory rule with a hook.

## Cross-references

- `kpi-self-eval-workflow.md` — the 6-step loop this audit used
- `feedback_standing-protocols.md` — receiving new trigger-phrase rule
- Inbox-miss structural fix (PreToolUse hook hardening) — same "catch at source" pattern
- A16 — Agent-idle root cause (related class of failure)

