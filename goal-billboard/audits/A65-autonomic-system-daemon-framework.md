---
audit_id: A65
title: Autonomic system + daemon/worker/pool framework — structural fix for recurring idle-laziness
status: in_progress
catalogued: 2026-05-27T17:30:00Z
phase_1_landed_at: 2026-05-27T17:30:00Z
priority_when_run: P1
estimated_effort: medium (audit + skill ref + memory rule + 8-daemon catalog this turn; per-daemon implementation deferred)
trigger: 2026-05-27 user explicit (3rd time the class fired this session) — *"why were we just sitting idle with no research? should we maybe also seperately without ereasing my last research plan, maybe consider on long running research daemon, or just flesh out any useful daemons/workers/pools for our system?"*
deferral_reason: NONE — framework design + 8-daemon catalog land this turn; per-daemon implementation deferred per workflow-fidelity constraint.
related_goals: [G2]
related_plans: []
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/autonomic-system-discipline.md (NEW; daemon framework)
  - .claude/memory-mirror/feedback_autonomic-system.md (NEW; standing protocol; both auto-load + mirror)
  - context/markdowns/organic-os/autonomic/README.md (NEW; sub-folder index)
  - A58 audit (Organic OS parent — autonomic is new top-level subsystem)
  - A63 audit (idle-capacity research-furthering — autonomic structurally implements its intent)
  - A47 audit (BG lifecycle — daemons follow the same wrapper discipline)
  - A62 audit (adaptive immunity — A65 IS the response to A63 recurring as a meta-pattern)
findings:
  - idle-laziness-class: even after A63 codified idle-capacity research-furthering, I stayed idle during A64 BG waits. The 7 safety boundaries were correct; my application was the failure. Per A62, "agent will be more careful" is NOT a structural mutation. The structural fix is a daemon that consumes idle slots WITHOUT requiring agent discipline.
  - daemon-vs-discipline: A63 = discipline (works only if agent applies it); A65 daemon = structural (works regardless of agent state).
  - new-subsystem-not-extension: autonomic deserves its own organic-os/ sub-folder alongside Brain/Heart/Immune/Organs because it's a distinct mode (autonomous-background) not a subset of any existing.
---

# A65 — Autonomic system + daemon/worker/pool framework

## What this audit names

The user has named (3rd time this session, in escalating frustration) the same pattern: I dispatch parallel BGs, idle slots accumulate, I don't use them. A63 codified the discipline. I still failed it. The user's response is correct: **the structural fix is not more discipline; it's a daemon.**

## The principle

> **Discipline is fragile; structure is permanent. When a discipline is the right rule but its application keeps failing, the next mutation is to make the rule structural — a daemon, a worker, a pool — that runs without agent participation.**

This is A62 adaptive-immunity applied to the discipline-application failure mode. The hardware tier is "the daemon exists and runs"; software tier is "the daemon's wrapper / safety contract"; biology tier is "the memory rule that says how to use the daemon."

## Naming + organic-OS placement

**Autonomic system** — companion to Brain (voluntary/conscious), Heart (rhythmic), Immune (defensive). The autonomic nervous system in biology runs heart rate, digestion, breathing — autonomously, without conscious thought. Our system needs an equivalent layer for background research, memory consolidation, audit grooming, and other "always-on" maintenance.

New organic-os/ folder: `organic-os/autonomic/README.md` (this audit's companion index).

## The 8 useful daemons/workers/pools

Ranked by leverage:

### 1. Research-furthering daemon (the A63 structural fix)

**Purpose:** maintains a research backlog; consumes idle BG slots when:
- N BGs are in flight + cap allows more (A21 ≤5)
- Token budget < 80% session ceiling
- No user pivot signal
- Parent task has natural research candidates

**Mechanism:** lives in `.claude/cache/research-backlog.jsonl`; reads/writes via a wrapper `scripts/research-daemon.py`. Heart's tier-medium consumer triggers it on heartbeat ticks; agent can also push items.

**Integration:** A63 discipline → A65 daemon — A63 is what to do; A65 ensures it happens.

### 2. Memory-consolidator daemon

**Purpose:** scans `feedback_*.md` rules + skill refs periodically; detects duplicates / outdated / orphan rules / broken cross-references; proposes consolidations to user.

**Mechanism:** runs on Heart tier-low (hourly or session-boundary). Output: `reports/memory-consolidation-proposals.md` with rank-ordered suggestions.

**Closes:** A64 BG-B's 11 decoupling defects fired because nothing scans memory for drift. A daemon would catch these in normal turn 1 not turn 16.

### 3. Audit-catalog maintainer daemon

**Purpose:** ensures audit IDs sequential (no gaps, no recycling), all referenced files exist, status transitions logged, retroactive instance maps updated.

**Mechanism:** runs on Heart tier-low. Output: `reports/audit-catalog-health.md`.

**Closes:** orphan audits (A59 + A60 per BG-B); missing back-references.

### 4. Steering-log compactor daemon

**Purpose:** the steering log grows unbounded. A daemon would summarize entries older than N days into a digest section while preserving raw lines.

**Mechanism:** runs on Heart tier-low (weekly cadence). Writes `notes/steering-decisions-log-digest.md` alongside the canonical log.

### 5. Bug billboard groomer daemon

**Purpose:** scan archived bugs for emerging patterns; surface "this defect class has fired N times in last week" as steering-log entry + immune-system signal #5.

**Mechanism:** runs on Heart tier-medium. Output: `reports/bug-class-patterns.jsonl` + dashboard surfacing.

### 6. CoT ledger consolidator daemon

**Purpose:** when CoT chains get long (50+ entries on one goal), summarize older entries into a digest; preserve raw chain.

**Mechanism:** runs on Heart tier-low. Per-chain digest at `.claude/cache/cot-ledger-digests/<chain_id>.md`.

### 7. Snapshot drift scanner daemon

**Purpose:** scan `tests/visual.spec.ts-snapshots/` periodically; flag if N% of snapshots are stale (file mtime > T) or if recent commits would invalidate; surface for user-authorized regen.

**Mechanism:** runs on Heart tier-medium.

### 8. Statistical correlator daemon

**Purpose:** mines `reports/bg-dispatch-log.jsonl` + `reports/chamber-post-log.jsonl` + steering log over time; surfaces correlations (e.g. "BG ghost rate up 40% this week" / "audit creation rate up 3x"). Per A41's "powerful uses of session JSONLs."

**Mechanism:** runs on Heart tier-low. Output: `reports/correlation-trends.md`.

## Daemon safety contract (universal)

Every daemon ships with:

1. **`--dry-run` default** — runs in dry-run for 24h before live; output goes to `.claude/cache/<daemon>-dry-run.jsonl`
2. **Wrapper-only invocation** — like `dispatch-bg.sh` / `post-to-chamber.py` / `inbox-write.py`; never invoked directly by agent
3. **Cost gate** — won't run if token budget > 80% OR no idle slots in A21 cap
4. **Pivot-signal respect** — if user signals plow/pivot, daemons pause
5. **Idempotent** — re-running produces no extra side effects
6. **CoT entry per fire** — writes to Brain's ledger when it acts
7. **Steering log entry per significant finding** — visibility

## Subsystem boundaries (vs Brain/Heart/Immune/Organs)

- **Brain** = voluntary reasoning (CoT chains). Agent-driven.
- **Heart** = rhythmic substrate (heartbeat). Periodic, not goal-directed.
- **Immune** = defensive response (chant + adaptive). Reactive to threats.
- **Organs** = compute substrate (resources/shards/clusters). Worked on.
- **Autonomic** (NEW) = background workers that DO things autonomously without needing the agent's direct action this turn.

The autonomic layer can be FIRED by Heart (a heartbeat tick) but the daemon does the actual work. Heart is metronome; autonomic is the body's autonomous functions.

## Phase 1 — this turn (LANDED)

- ✅ A65 audit (this file)
- ✅ Skill ref `autonomic-system-discipline.md`
- ✅ Memory rule `feedback_autonomic-system.md` (both auto-load + mirror)
- ✅ `organic-os/autonomic/README.md` sub-folder index
- ✅ `organic-os/README.md` updated with autonomic row
- ✅ Steering log entry
- ✅ Audit catalog update (A65 added; next ID → A66)

## Phase 2-N — deferred (one daemon at a time, dry-run first)

Per A47 24h dry-run discipline + workflow-fidelity:
1. Daemon #1 (research-furthering) first — directly closes A63 recurring failure
2. Then #2 (memory consolidator) — fires when MEMORY.md / skill ref drift accumulates
3. Then #3 (audit catalog maintainer)
4. Then remaining 5 as needed

Each daemon: 24h dry-run → review log → flip to live → VERIFY closes intended class.

## Cross-references

- A58 audit (Organic OS parent)
- A63 audit (idle-capacity research-furthering — A65 IS the structural-mutation companion that closes A63's discipline-application failure)
- A47 audit (BG lifecycle — daemon wrapper precedent)
- A62 audit (adaptive immunity — A65 is hardware-tier mutation against the discipline-failure class)
- A64 BG-B (decoupling defects fired because no daemon scans for drift — A65 daemon #2 closes that class)
- A41 audit (powerful uses of session JSONLs — A65 daemon #8 implements)

## Status

PHASE 1 LANDED 2026-05-27T17:30Z. Phase 2-N (per-daemon implementation in dry-run sequence) deferred to subsequent turns / next session.

## Lessons

- **3rd-time class trigger justifies structural mutation.** User has now flagged "you sat idle" three times. Discipline didn't stick. Daemon will.
- **A62 acid test applied to A63:** A63 had biology (skill ref + memory rule) but no hardware (the agent could always choose not to apply) → recurrence inevitable. A65 adds the hardware tier (daemons run regardless of agent state).
- **The autonomic layer is the missing organic-OS subsystem.** Brain (voluntary) + Heart (rhythmic) + Immune (defensive) + Organs (substrate) + Autonomic (background workers) = a complete organism. Future work: Sensory (perception layer for dashboard / Chrome / Playwright); Reproductive (creation layer for new agents / changes).
