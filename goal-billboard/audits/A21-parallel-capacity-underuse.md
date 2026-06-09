---
audit_id: A21
title: "Parallel capacity underuse — agent launches 1 BG at a time when substrate has N decoupled candidates ready"
status: in_progress
catalogued: 2026-05-26T22:45:00Z
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-26 22:45Z — user audit: *"why are you only running one north star workflow at a time, this system is built for full parallelization and subagentry, i see us launching one subtask in background and stopping to wait, this is inneficient."* The substrate (goal billboard, asks-log, audits in_progress) routinely holds 4+ decoupled BG-eligible candidates. The agent launches 1 per response. Wall-clock cost: N× longer to clear backlog.
deferral_reason: |-
  NONE — running immediately. Sibling to A16 (idle) / A18 (artifact gap) / A19 (cross-system) / A20 (hedge-holdoff). A21 is the response-shape failure: even when not idle, even when responding to the prompt, the agent under-uses parallel capacity.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/parallel-launch-discipline-plan.md]
related_refs:
  - .claude/skills/agentic-quality-discipline/references/parallelization-checks.md
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
  - context/markdowns/goal-billboard/audits/A19-cross-system-meta-monitoring.md
  - context/markdowns/goal-billboard/audits/A20-hedge-holdoffs-and-unrouted-blockers.md
findings: []
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'cluster' -> GL G7; NS defaults to G2
belongs_to_goal: G1
serves_guiding_light: G7
---
# A21 — Parallel capacity underuse

## Why this audit matters

The Anthropic system supports launching N parallel BG agents per turn. The Evium project has comprehensive coordination infrastructure (scheduler, worktrees, scoped edit zones, file-collision detection patterns). Despite this, the agent's reflex pattern is **one user ask → one BG launched → wait for completion → next ask → one more BG**.

User framing: *"this system is built for full parallelization and subagentry, i see us launching one subtask in background and stopping to wait, this is inneficient."*

Wall-clock cost of the failure: every cluster of N decoupled candidates takes N turns instead of 1. With ~12 BGs/day average, that's 12 turns instead of 3-4. User-perceived velocity halved.

## Receipts (this session)

Decoupled candidates pending RIGHT NOW that could have been launched in parallel earlier:

| Item | Files touched | Collision with #75? |
|---|---|---|
| G2 client signoff prep — 19 fixes → delivery package | `reports/g2-signoff/` (new dir) | No |
| A14 stats correlation ANALYSIS writeup | `context/markdowns/research/` (new file) | No |
| A15 research-folder Phases 2-4 implementation | `context/markdowns/research/` + new hooks | No |
| A21 implementation (this audit's hooks) | `scripts/hooks/` + scoped dashboard | Low (different zones) |
| Per-tab visual sweep continuation (Time-lapse/Scheduler/Subagents/Improvement/Stats tabs) | `index.html` | YES — defer until #75 done |
| P0 chip WCAG fix | `index.html` | YES — defer until #75 done |
| BG-fabrication audit (Text v2 claimed changes that never landed — new failure class) | `audits/A22-…` (new) | No |
| G6 Phase 2 (priority routing) | `scripts/agent-inbox-server.py` | No |

**4-5 of these are launchable in parallel RIGHT NOW.** Pattern across the session: I launched them sequentially across multiple turns.

## Root causes

1. **Reflex "one ask, one BG"** — mental binding between user-prompt and BG-dispatch is 1:1 by default.

2. **No pre-response parallel-candidate scan** — A19 Layer 1 state-batch surfaces pending work but doesn't say "these N are launchable in parallel this turn."

3. **Coordination caution overcorrection** — fear of file collisions causes me to serialize when scoped zones would prevent collisions.

4. **Token-cost rationalization** — "5 BGs at once is expensive" is wrong reasoning. Each BG bills its own context; 5 parallel = same total cost as 5 serial; just better wall-clock + same cache utilization.

5. **No structural floor** — nothing in the hook chain says "if response has Agent dispatch AND N≥2 decoupled candidates exist AND only 1 dispatched, surface warning."

6. **`parallelization-checks.md` skill ref is passive** — gives me a 4-question checklist but only when I'm ALREADY thinking about parallelism. Doesn't pull me TOWARD it.

## Why prior fixes can't reach this

- **`parallelization-checks.md`** — passive ref; requires me to invoke.
- **Memory rule "2+ subagents in ONE tool-call message"** — only fires if I'm ALREADY launching 2+. Doesn't force me to look for 2+.
- **A19 state-batch reader** — surfaces work but doesn't say "launch all of these in parallel."
- **`goal-queue-check.sh` Stop directive** — says "launch one BG before status report." Should be "launch ALL decoupled pending in this response."

## Fix (designed + scaffolded in same turn — A21 implementation BG launching in parallel with G2/A14/A15 work)

See `plans/automation/parallel-launch-discipline-plan.md`. Four layers:

### Layer 1 — Pre-response parallel-candidate scan (`pending-parallel-work-scan.sh`, UserPromptSubmit, chains after state-batch)

Reads goal-billboard + asks-log + audits in_progress + monkey-pending. For each pending item, classify by file zones it would touch. Group into collision-free sets. Emit `<system-reminder>` "🔀 PARALLEL CANDIDATES: N decoupled BGs ready to launch this turn — [list with file zones]. Default: launch ≥3 in one tool message if pending."

### Layer 2 — Stop hook capacity check (`parallel-capacity-check.sh`)

Count Agent dispatches in this response. Compare to candidate count surfaced at UserPromptSubmit. If candidates>dispatches AND total in-flight BGs <5: emit "⚠ PARALLEL CAPACITY UNDERUSED: launched K of N candidates this turn. Re-evaluate before next response."

### Layer 3 — Dashboard "Parallel Capacity" KPI

Extend `scripts/render-stats-data.py` with `parallel_capacity` block: rolling-10 ratio of "launched ÷ candidates per response." Sparkline in Stats tab. Below 0.6 fires 6-step loop trigger.

### Layer 4 — Memory rule update + skill ref upgrade

Standing-protocols entry: "default ≥3 parallel BGs/response when pending decoupled work exists; serialize ONLY when file-zone collisions cannot be scoped around." Skill ref `parallelization-checks.md` gains "pre-flight: scan for candidates BEFORE writing response" section.

## Verification

After hooks land:
1. Synthetic UserPromptSubmit with 5 known pending items → confirm Layer 1 surfaces "5 PARALLEL CANDIDATES"
2. Synthetic response launching 1 BG → confirm Layer 2 surfaces "underused" warning
3. Same with 3 BGs → no warning
4. Stats tab "Parallel Capacity" sparkline renders + shows latest ratio
5. Live session: launch ≥3 BGs per response with pending; verify hook silent

## Status

IN PROGRESS — A21-implementation BG launched in PARALLEL with 3 other content BGs in same turn. Will resolve once layers live + 5 consecutive responses meet ≥3-BGs/turn when pending.

## Cross-references

- A16 (idle) / A18 (artifact gap) / A19 (cross-system) / A20 (hedge) — sibling response-side disciplines
- `parallelization-checks.md` — being upgraded by A21 work
- `kpi-self-eval-workflow.md` — 6-step loop applied to A21 itself in same turn

