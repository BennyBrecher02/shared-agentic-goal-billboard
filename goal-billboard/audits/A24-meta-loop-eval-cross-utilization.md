---
audit_id: A24
title: "Cross-utilization hidden power points + missing periodic meta-eval skill"
status: in_progress
catalogued: 2026-05-26T23:05:00Z
priority_when_run: P0
estimated_effort: medium (skill design + script + dashboard surface)
trigger: |-
  2026-05-26 22:59Z — user audit *"ive been asking you deep thinking system design and overhaul questions that have been improving our progress throughput, can we make a looped eval skill for occasionally poking at our software-cross-utilization-hidden-powerpoints?"* The user has been driving deep meta-improvements (A18 → A19 → A20 → A21 → A22 → A23) — those came from their queries, not from any internal periodic process. The skill they want: an OFFLINE periodic ritual that ASKS those questions WITHOUT user prompting, surfacing untapped cross-tool integrations.
deferral_reason: NONE — running immediately. Without this skill, the project's improvement velocity DEPENDS on the user asking the right question. With it, the system self-surveys cross-utilization opportunities.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/meta-loop-eval-skill-build-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G4
related_refs:
  - .claude/skills/agentic-quality-discipline/SKILL.md
  - .claude/skills/reactionary/  (the post-run learning layer)
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
findings: []
---

# A24 — Meta-loop-eval cross-utilization

## Why this audit matters

The project has accumulated rich infrastructure:
- 7 capability skills
- 18 audit catalog entries
- 30+ plans
- 6 active goals
- ~15 hooks
- Multi-change scheduler
- Heartbeat polling
- Dashboard with 7+ tabs
- Monkey chamber with 4 categories
- Asks-log
- Goal billboard
- Reactionary layer

**Each piece is valuable. But the COMBINATIONS aren't all wired.** Some examples found just by glancing:

- Heartbeat detects monkey-stale → could auto-promote to bug billboard (not wired)
- Scheduler tick_end → could auto-trigger feedback-loop comparison (partially wired)
- Lost-work detector + recurrence-detect → could auto-spawn 6-step-loop investigator BG (not wired)
- State-batch-digest + parallel-capacity-check → if N pending + low ratio, could auto-launch K of N at user's next prompt (not wired)
- Asks-log + failure-class-registry → if a class recurs 3+ times, auto-create A* audit (not wired)
- Cluster (G7) + parallel-capacity-check → distribute the 5-BG ceiling across nodes (not wired; design exists)
- NS deep-integration + token attribution + heartbeat → "G2 used 80% of today's tokens"-style insight (Phase 4 design exists; not wired)

**These are "hidden power points"** — combinations where two existing primitives compound but haven't been integrated. User framing: "software-cross-utilization-hidden-powerpoints."

User pattern observation: every meta-audit today (A18-A23) emerged from a USER PROMPT identifying a gap. The user is doing the work this skill should be doing in their absence.

## Root cause

No periodic ritual that asks "what combinations of existing tools aren't yet wired?" The reactionary layer (`.claude/skills/reactionary/`) runs on EVENT triggers (audit completed, matrix run, scheduler tick). There's no equivalent for "weekly self-survey of cross-utilization opportunities."

## Fix — looped-eval skill

See `plans/automation/meta-loop-eval-skill-build-plan.md`. Three components:

### Component 1 — CLI tool `scripts/run-meta-loop-eval.sh`
Periodic scanner. Identifies cross-utilization opportunities by:
- Listing all hook scripts + the data they write
- Listing all hook scripts + the data they read
- Finding pairs where Hook-A writes X and Hook-B could read X but doesn't
- Listing all goals + the audits they cite + identifying audits-without-goal-link or goals-without-audit-link
- Surfacing "primitive combinations not wired":
  - Heartbeat events × Bug billboard
  - Scheduler events × Heartbeat tier triggers
  - Asks-log × Failure-class-registry (auto-audit-creation)
  - Improvement-tags × Conversation-pattern KPI
  - NS context × BG dispatch prompts (Phase 4 of NS deep-integration)
  - Cluster nodes × Parallel-capacity ceiling

Output: `reports/meta-loop-eval-YYYY-MM.md` with proposed cross-wirings + priority scores.

### Component 2 — Dashboard panel `Untapped Integrations`
- Renders from `reports/meta-loop-eval-*.md`
- Lefter mini-block: count of untapped integrations
- Each row: pair-of-tools · proposed wiring · estimated effort · priority score · "queue as plan?" link

### Component 3 — Skill ref `meta-loop-eval.md`
Stable distillation under `.claude/skills/agentic-quality-discipline/references/` once Component 1 + 2 are proven. Contains:
- When to run (cadence: monthly default; manual any time; auto-cron opt-in)
- What to look for (the cross-utilization heuristics)
- How to act on findings (queue as plan, link to A* audit, monkey-chamber escalation)
- Connection to reactionary layer (this is `proactive` counterpart)

## Verification

After implementation:
1. `bash scripts/run-meta-loop-eval.sh` produces a report with ≥5 cross-utilization findings
2. Dashboard "Untapped Integrations" panel renders
3. Manually queue one finding as a plan → confirms the workflow works end-to-end
4. Skill ref drafts after first stable run

## Status

IN PROGRESS. Implementation BG launching this turn alongside A23 BG.

## Cross-references

- `kpi-self-eval-workflow.md` — 6-step REACTIVE loop; A24 is the PROACTIVE counterpart
- Reactionary layer (`.claude/skills/reactionary/`) — event-triggered; A24 is cadence-triggered
- A19 master meta-monitoring — A24 is the offline scheduled version
- G4 skill system + reactionary infrastructure — A24 extends this goal's scope

## Lessons (preliminary)

- The reactionary layer is event-triggered (post-run learning). A24 fills the OPPOSITE corner: scheduled-time-triggered (proactive survey).
- The user's been doing this manually all day; encoding the ritual transfers the cognitive load to the system.
- Each meta-audit today (A18-A23) is a "hidden power point" that was DISCOVERED by the user. A24's job is to discover the NEXT ones before the user has to.

