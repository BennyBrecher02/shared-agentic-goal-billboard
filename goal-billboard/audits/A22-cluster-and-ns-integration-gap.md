---
audit_id: A22
title: "Cluster capacity unreached + Northern Star/Guiding Light shallow integration"
status: in_progress
catalogued: 2026-05-26T23:00:00Z
priority_when_run: P0
estimated_effort: large (multi-month — Phase 1 is days; Phases 2-3 require hardware + multi-week implementation)
trigger: |-
  2026-05-26 22:55Z — user request *"start working towards the cluster north star/guiding light now too... like if i have a second mac slave then we could be running many tasks and going for many guiding lights to work in truer parallelization, so lets kick this off and also make a note for better integration of northern star/guiding lights into our projects nitty gritty logic and thats not a small note thats a major major ask."* Two gaps surface together: (1) cluster work has been parked despite plans existing (`cluster-readiness.md`, `m2-ssh-scheduler-integration.md`, `m2-mac-test-farm.md`); (2) Northern Star + Guiding Lights exist as DASHBOARD features but don't propagate through plans/audits/BG-prompts/state-batch/hooks. A21 just locked in single-machine parallel discipline; A22 catches the cluster + NS-deep gap.
deferral_reason: NONE — running immediately. Cluster Phase 1 is hardware-independent; can build now. NS deep-integration is plan-side work; can start now. Both decoupled.
related_goals: [G7, G8]  # being proposed in this turn
related_plans:
  - context/markdowns/plans/automation/cluster-readiness.md
  - context/markdowns/plans/automation/m2-ssh-scheduler-integration.md
  - context/markdowns/plans/automation/m2-mac-test-farm.md
  - context/markdowns/plans/automation/cluster-kickoff-plan.md  # NEW this turn
  - context/markdowns/plans/automation/northern-star-deep-integration-plan.md  # NEW this turn
related_refs:
  - context/markdowns/goal-billboard/audits/A11-concurrency-ceiling.md
  - context/markdowns/goal-billboard/audits/A21-parallel-capacity-underuse.md
findings: []
serves_northern_star: G2  # migrated 2026-05-26 - path 'cluster' -> GL G7; NS defaults to G2
belongs_to_goal: G17
serves_guiding_light: G7
---
# A22 — Cluster capacity unreached + NS/GL shallow integration

## Why this audit matters

Two structural gaps revealed by the same user prompt:

### Gap 1: Cluster work parked
- `cluster-readiness.md` exists as architectural design, `kind: design-parked`, blocked on "M2 ready" / "Pi ready" signals.
- `m2-ssh-scheduler-integration.md` + `m2-mac-test-farm.md` exist as parked plans.
- **NO hardware-independent foundation work has happened.** The moment user has a second Mac SSH-accessible, there's no shim/CLI/dashboard panel ready to plug into.
- A11 (concurrency-ceiling) demonstrated single-machine ceiling at 3 shards. A21 (parallel-capacity) lowered the practical BG-agent ceiling to 5. **Cluster nodes are the only way past these ceilings.**

### Gap 2: Northern Star / Guiding Lights shallow
- NS is a DASHBOARD FEATURE: `northern_star: true` field on G2; renders as a marquee.
- Guiding Lights: multi-track strip shows G1/G2/G6 tracks; renders as a sub-strip.
- **But neither flows through artifacts, BG prompts, hooks, or scheduler logic.** A BG dispatch doesn't see "this clears the path for the Northern Star." A plan doesn't carry which NS/GL it serves. Token spend isn't attributed per NS/GL. State-batch-digest doesn't surface "NS at risk because pending work piling up." Stop hook doesn't check "did this response advance an NS?"
- **The NS/GL is a label, not a load-bearing structure.** Decisions made anywhere in the project can't be evaluated against "does this serve the NS?" because the NS context isn't in the artifact's neighborhood.

## Evidence

### Cluster underuse receipts
- 9h of session today; many parallel slots wasted; single-machine ceiling hit repeatedly
- Wall-clock cost: ~3-4× longer to clear backlog vs cluster of 3 Mac nodes (per `cluster-readiness.md` cluster-math table)
- Plans dated 2026-05-26 morning; status PARKED for ~17h despite continued bottleneck

### NS/GL shallow integration receipts
- `goal-billboard/active/G7-*.md` (cluster, being created in this turn) — would benefit from `serves_northern_star: G2` if NS context was a standard field across artifacts. Currently no such convention.
- BG dispatch prompts I write don't include "this clears the path for: G2." So BG agents lose the strategic frame.
- State-batch-digest shows "2 stale goals" but doesn't say "the Northern Star is one of them" or "G7 has been stale 17h while G2 is the only thing actively advancing."
- The 22:08Z dashboard complaint ("text is the same") — would have been caught earlier if "did this response advance G2?" check existed.

## Root causes

1. **Parking is sticky.** A plan marked `design-parked` requires explicit re-activation. The user has to remember to ask. The agent doesn't proactively check parked plans against current capacity.

2. **NS/GL was scoped to dashboard.** Original design was visualization. Never extended to artifact frontmatter / hook surfacing / BG prompt context / scheduler integration.

3. **No "what serves the NS?" pre-flight.** Each response decides its own priorities locally; the NS isn't a constraint.

4. **No cross-machine fan-out primitive.** Even with a second Mac, the scheduler currently has `host:` fields but no actual SSH dispatch implementation (per `cluster-readiness.md` Step 3-5 SOLID requirements).

## Fix (designed + scaffolded in same turn — A22 implementation BG queued)

### Cluster: hardware-independent foundation NOW (Phase 1)
See `plans/automation/cluster-kickoff-plan.md`. 5 components:
- `scripts/cluster/inventory.sh` — lists local + SSH-discoverable nodes
- `scripts/cluster/readiness-check.sh` — verifies remote SSH + git + python + node before dispatch
- `scripts/cluster/dispatch.sh` — placeholder shim; runs locally today; pluggable for SSH tomorrow
- `scripts/cluster/lib-common.sh` — shared functions
- Dashboard "Cluster" panel + lefter mini-block showing online nodes + capacity

Phase 2 (hardware-gated): real SSH dispatch + scheduler integration + artifact transfer.
Phase 3 (post-Phase-2): distributed test matrix + cross-node coordination.

### NS/GL deep integration (4 layers)
See `plans/automation/northern-star-deep-integration-plan.md`. Major plan:
- **Layer 1: Frontmatter propagation** — every plan/audit/BG-dispatch-prompt/scheduler-change carries `serves_northern_star:` field (default: current NS; explicit if different).
- **Layer 2: Hook surfacing** — state-batch-digest + recurrence-detect + workflow-check all include "NS at risk?" check.
- **Layer 3: Token-spend attribution** — per-response token cost tagged to which NS/GL it advanced.
- **Layer 4: Dashboard NS-everywhere indicator** — every panel shows "clearing path for: G2" badge.

## Verification

After implementation:
1. Cluster inventory script lists this Mac
2. Readiness check passes for localhost
3. Dispatch shim accepts task specs + executes
4. Dashboard Cluster panel renders with 1 node online
5. Lefter mini-block shows "1 node · 3 shards" or similar
6. NS-context field appears in newly-created artifacts (audit/plan templates updated)
7. State-batch-digest mentions Northern Star explicitly when stale

## Status

IN PROGRESS. Foundation landed. Cluster Phase 1 impl BG in flight. NS deep-integration Phase 1 complete (88+ artifacts retrofitted; linter clean). **NS deep-integration Phase 2 in flight (2026-05-26 23:18Z)** — hook surfacing layer.

**Phase 1 link (complete):**
- Migration script: `scripts/migrate-add-ns-field.py`
- Linter: `scripts/lint-ns-frontmatter.py`
- Skill ref: `.claude/skills/agentic-quality-discipline/references/northern-star-context-propagation.md`
- Plans README created: `context/markdowns/plans/README.md`
- Audits README schema extended with the new fields

**Phase 2 link (in_progress, this turn):**
- `scripts/hooks/state-batch-digest.sh` — section 8 NS-at-risk: stale `updated:` > 6h | pending NS-attributed monkey decisions | linked-audit staleness > 6h
- `scripts/hooks/recurrence-detect.sh` — NS keyword harvest (title + phase + tagline + linked-plan filenames) → recurrence header gains `🌟 NS-BLOCKING (G<N>) —` prefix when topics overlap; per-topic `[NS-blocking]` flags; escalated closing line; JSONL extension with `ns_blocking`, `ns_blocking_topics`, `serves_northern_star` fields
- `scripts/hooks/parallel-capacity-check.sh` — per-candidate `serves_northern_star:` lookup via best-effort frontmatter read; header gains `(↗ NS-blocking: K of N candidates serve <NS>)` when applicable; NS-blocking unlaunched candidates rendered with `**bold + 🌟`; JSONL adds `current_ns` + `ns_blocking_candidates`
- NEW `scripts/hooks/ns-advance-check.sh` — Stop hook walks session JSONL backward to the last *real* user-prompt boundary (distinguishing `tool_result` user-events from human prompts), extracts Edit/Write/MultiEdit/NotebookEdit `file_path` inputs, checks each file's `serves_northern_star:` against the current NS goal, emits one `<system-reminder>` when a response did substantive edits but advanced zero NS-relevant files. Silent on read-only or NS-advancing responses. Always logs to `.claude/cache/ns-advance-check.jsonl`.
- Settings patch: `scripts/hooks/settings-patches/ns-advance-check.json` (Stop entry only; the three in-place hook extensions don't need new settings rows)

Will flip to `resolved` when:
- Cluster Phase 1 scripts live + dashboard panel rendering
- NS-context frontmatter field present in ≥10 artifacts (Phase 1 landed 88+ in this turn)
- State-batch-digest surfaces "NS at risk" check working (Phase 2 — verified 2026-05-26 23:18Z: "🌟 NS AT RISK: G2 — 8 monkey pending" emitted live; synthetic-substrate test confirmed stale-`updated:` + linked-audit branches)
- 1 BG dispatch demonstrates auto-loaded NS context (Phase 3)

## Cross-references

- A11 — concurrency-ceiling (single-machine bottleneck) → cluster is the answer
- A21 — parallel capacity (single-machine reflex pattern) → cluster removes the ceiling entirely
- `cluster-readiness.md` — architectural design this kickoff executes
- `northern-star-feature-plan.md` — Phase 1 NS dashboard feature this deep-integration extends
- G7 + G8 — new goals proposed in this turn (asking user to confirm during their next response)

## Lessons (preliminary)

- Parked plans need a "review parked plans" Stop hook check — if a parked plan's unblock-condition might now be met, surface it.
- Dashboard features that aren't ALSO load-bearing in the project's nitty-gritty are just decoration.
- The Northern Star is a constraint, not a label. Constraint = "every decision evaluable against it." Label = "card on dashboard."
- Cluster + NS-deep-integration are independent (different file zones) — parallel BGs can build them simultaneously.

