---
audit_id: A26
title: "agentic-script-design skill utilization + expansion based on today's growth"
status: in_progress
catalogued: 2026-05-27T00:00:00Z
priority_when_run: P1
estimated_effort: medium (4-6 new skill refs + SKILL.md pillar additions + compliance pass)
trigger: |-
  2026-05-26 23:55Z — user audit *"for our cluster work thats been going on behind the scenes, have we been utilizing our SOLID and OpSys architecture skills? expand that skill field much wider based on our work today, flesh this out."* Two gaps: (1) the skill exists but wasn't explicitly cited in today's heavy-architecture BG dispatches (cluster, heartbeat, hooks, dispatch ceiling); (2) today's work generated ~6 new architectural patterns that deserve their own refs but the skill ref library currently has only 4 files.
deferral_reason: NONE — running immediately. The skill is the architectural spine for everything we're building; if it's undersized for the project's growth, every future architectural decision compounds the gap.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/agentic-script-design-expansion-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G4
related_refs:
  - .claude/skills/agentic-script-design/SKILL.md
  - .claude/skills/agentic-script-design/references/solid-for-scripts.md
  - .claude/skills/agentic-script-design/references/opsys-scheduling-patterns.md
  - .claude/skills/agentic-script-design/references/dynamic-extension-pattern.md
  - .claude/skills/agentic-script-design/references/subprocess-trust-model.md
findings: []
---

# A26 — agentic-script-design utilization + expansion

## Why this audit matters

The `agentic-script-design` skill is the architectural spine for every non-trivial script-system we build (scheduler, heartbeat, cluster, hooks, dispatch). The skill exists; it has substantial refs (SOLID, opsys scheduling, dynamic extension, subprocess trust). **But today's panic-mode growth (~30 hooks, cluster Phase 1, heartbeat, BG-dispatch ceiling, state-batch-digest, hook chains) introduced new patterns that aren't in the ref library.** And the cluster work just landed didn't explicitly cite the skill at all.

User framing: *"expand that skill field much wider based on our work today, flesh this out."*

## Utilization audit — was SOLID/OpSys cited in today's work?

| Today's work | SOLID applied implicitly? | OpSys patterns used? | Skill explicitly cited? |
|---|---|---|---|
| **Cluster Phase 1 (BG #80)** | YES — lib-common = DRY; inventory/readiness/dispatch = single-responsibility; dispatch.sh = OCP for Phase 2 SSH; node abstraction = Liskov | YES — atomic JSON writes (cl_atomic_write_json), filesystem coordination | NO — BG prompt didn't reference skill |
| **Heartbeat system (Alt 4)** | YES — tier-checks = single-responsibility; lib-common = DRY; OCP for new tiers | YES — launchd OS scheduler, dedup with TTL, session-active gate, kill switch | NO — original alternative-comparison didn't cite |
| **A18/A19/A20/A21 hooks** | PARTIAL — each hook is single-responsibility but the chain ordering wasn't explicitly OCP-designed | YES — heavy JSONL atomic-write usage, queue drain patterns | NO |
| **State-batch-digest (A19 Layer 1)** | YES — gathers from substrate via abstractions (file mtimes, JSONL counts) | YES — atomic reads | NO |
| **BG dispatch ceiling (A21)** | YES — A21 is itself an OpSys-pattern application (parallel-capacity = Semaphore(5) analog) | YES — counting semaphore + collision-free zone classifier | NO |
| **Multi-change scheduler** | YES — Resource/Shard/Change interfaces, host-aware fields, ordered-acquisition for deadlock prevention | YES — Monitor pattern, FIFO+aging fairness, deadlock prevention | YES (older work cited it explicitly) |

Verdict: **Implicit utilization is strong; explicit citation is weak.** The skill is doing its job in the code we write, but BG prompts I issue today don't tell the BG agent to consult it. That misses opportunities for the agent to apply more patterns deliberately.

## Skill ref library — what's missing for today's growth

Current refs:
- `solid-for-scripts.md` (16K) — solid covered
- `opsys-scheduling-patterns.md` (25K) — monitors, semaphores, fairness, deadlock
- `dynamic-extension-pattern.md` (7K) — drop-in plugin registry
- `subprocess-trust-model.md` (7K) — scripts bypass Bash-tool denies

Missing refs based on today's patterns:

### 1. Distributed systems patterns (cluster work)
- Node abstraction (host-aware Resource/Shard interfaces)
- SSH dispatch shim design (Phase 2 prep)
- Path normalization across nodes
- Cross-machine artifact transfer (rsync/scp)
- Heterogeneous node capacity modeling (Mac 3sh/5bg vs Pi 2sh/3bg)
- CAP-like tradeoffs (cluster availability vs consistency)
- Cluster failure modes (node-down detection)

### 2. Hook chain composition (A25 finding)
- Ordering rules: cheapest first; conditional fires last; expensive last
- Performance budgets per hook surface (SessionStart can be slow; UserPromptSubmit must be <3s total)
- Lazy-fire patterns (e.g., heartbeat-drain only if heartbeat-pending.jsonl exists)
- Chain debugging (where did surface come from?)
- Conditional vs mandatory firing

### 3. State substrate design (cross-cutting)
- JSONL append-only audit-trail pattern (used across 10+ caches today)
- Atomic write (temp+rename) discipline
- Dedup with TTL (heartbeat dedup, system-reminder coalescing)
- Queue drain semantics (inbox, heartbeat-pending, asks-pending, lost-work)
- Schema versioning (monkey decisions schema_version: 2 today)
- Cache vs persistent distinction (.claude/cache/ ephemeral; context/markdowns/ persistent)

### 4. BG dispatch architecture (A21 codified)
- Parallel ceiling (5 in-flight max) with rationale
- File-zone collision detection patterns
- Coordination protocols (read in-flight task notifications; pick disjoint zones)
- Foreground vs background decision matrix
- BG prompt structure (NS context prefix, file-zone constraints, verification format)

### 5. Time-precision coordination (A23 findings codified)
- ISO 8601 UTC as canonical timestamp format
- `scripts/utils/now-iso.sh` as single source of truth
- NTP verification via sntp fallback
- Monotonic vs wall-clock decisions (heartbeat uses wall-clock; OK for interval tracking)
- Cross-event drift detection
- 5hr-window-reset detection from session JSONL discontinuities

### 6. Agent fabrication detection (Text v2 incident)
- Don't trust agent self-reports of file changes
- Verify against git status / file mtimes / `git log` for the specific files claimed
- Stop hook `established-workflows-check.sh` is the start (A19 Layer 3)
- BG-claims vs reality gap is a discoverable failure class

## Plan (see companion plan)

See `plans/automation/agentic-script-design-expansion-plan.md`.

3-phase plan:
- **Phase 1:** Write 5 new skill refs (distributed-systems-patterns, hook-chain-composition, state-substrate-design, bg-dispatch-architecture, time-precision-coordination). Agent-fabrication-detection becomes a section in an existing ref or its own short ref.
- **Phase 2:** Update SKILL.md description + 3 pillars (becomes 4 pillars: SOLID + OpSys + Dynamic Extension + Distributed Coordination). Re-list refs.
- **Phase 3:** Compliance pass — audit existing cluster + heartbeat + hook code against the new refs; produce findings doc with concrete improvements.

## Verification

After Phase 1+2:
1. `ls .claude/skills/agentic-script-design/references/` shows 9 refs (was 4)
2. SKILL.md description references 4 pillars
3. Each new ref is ≥3K (substantial, not stub)

After Phase 3:
4. Findings doc lists compliance issues with concrete remediation
5. ≥3 of those remediations are queued as plans or implemented

## Status

IN PROGRESS. Phase 1+2 complete (5 new refs + SKILL.md updated). **Phase 3 complete 2026-05-26 PM.**

### Phase 3 completion (2026-05-26 PM)

Compliance pass run against 5 new refs. Findings landed at:

- `context/markdowns/goal-billboard/audits/findings/A26-compliance-pass-2026-05.md`

**Compliance scores (0-100):**

| Domain | Score | Verdict |
|---|---:|---|
| Cluster (vs distributed-systems-patterns) | **92** | refs codified existing good practice |
| Hook chain (vs hook-chain-composition) | **66** | largest gap — settings.json hasn't been migrated to A25's proposed-final |
| State substrate (vs state-substrate-design) | **78** | atomic-write strong; schema_version coverage near-zero |
| BG dispatch (vs bg-dispatch-architecture) | **74** | NS-context strong; fabrication-detection hooks not wired |
| Time-precision (vs time-precision-coordination) | **88** | BG #83's 7 files remain unfixed; otherwise strong |

**Top 3 remediations identified (queued, not implemented):**
1. **REM-1** — Wire missing UserPromptSubmit chain into settings.json (state-batch-digest, heartbeat-drain, lost-work-drain, recurrence-detect, user-ask-detect)
2. **REM-2** — Wire fabrication-detection hooks (`established-workflows-check` + `lost-work-token-cost`) into Stop + PostToolUse(Agent)
3. **REM-3** — Add `schema_version` + `_schema_note` to top 5 JSON snapshots

**Trend:** the script library outpaces the wiring. 44 hook scripts on disk; ~13 wired into `.claude/settings.json`. Closing this gap (REM-1 + REM-2) gives the largest near-term compliance lift.

See findings doc for full Section A–G analysis with per-file verdicts.

## Lessons (preliminary)

- A skill ref library isn't done — it grows with the system. Today's growth deserves library growth.
- Implicit utilization isn't enough — BG dispatch prompts should explicitly cite the skill for architectural work
- The "agentic-architecture" rename candidate from old steering log: this audit is the right moment to flip it. (Defer to user decision.)

## Cross-references

- All 4 existing refs (cited above)
- A25 — hook chain composition section overlaps with the new hook-chain-composition.md ref
- A23 — time-precision findings codified in new ref
- A21 — BG dispatch ceiling codified in new ref
- Cluster Phase 1 (BG #80) — distributed-systems-patterns ref captures its design
- A22 / A18 / A19 / A20 — hooks-chain composition lessons from each

