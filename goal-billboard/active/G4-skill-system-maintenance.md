---
goal_id: G4
title: Skill system + reactionary infrastructure maintenance
status: active
track: quality
phase: "maintenance + structural-enforcement layer (A18+A19) + skill-health framework (A28 Phase 2 refresh in progress — agentic-webdesign + agentic-chrome-tools refreshed; 1 more to go: agentic-device-testing)"
priority: P0 (elevated from P1 — trust-model)
standing: true  # maintenance goal — never terminally "done"; judging it by updated: freshness is a category error (goal-grooming-proposal-2026-06-21.md §4-G). Exempt-by-nature from the stale-alarm.
created: 2026-05-24T14:30Z
updated: 2026-06-21T21:01Z  # refreshed (tracking freshness) per goal-grooming-proposal-2026-06-21.md §4-G; kept active as a STANDING maintenance goal. No phase change (1 open A28 refresh: agentic-device-testing).
linked_plans:
  - context/markdowns/plans/automation/plan-audit-artifact-discipline-plan.md (A18)
  - context/markdowns/plans/automation/master-plan-cross-system-meta-monitoring.md (A19)
  - context/markdowns/plans/automation/anti-idle-infrastructure-plan.md
  - context/markdowns/plans/automation/graceful-shutdown-infrastructure-plan.md
linked_refs:
  - .claude/skills/  (all 7 skills)
  - .claude/memory-mirror/MEMORY.md
linked_audits:
  - A18 — Plan/audit creation gap (in_progress)
  - A19 — Cross-system meta-monitoring (in_progress, master)
  - A16 — Agent-idle root cause (in_progress)
  - A17 — 5hr-reset operator alerting (in_progress)
linked_bugs: []
linked_changes: []
serves_northern_star: G2  # migrated 2026-05-26 - goal_id=G4 (not NS) → GL; NS defaults to G2
serves_guiding_light: G4
project: OS-core  # stamped by migrate-billboard-to-shared (shared store)
---
# G4 — Skill system + reactionary infrastructure

## Why it exists

The skill system is the long-term memory of how to do work in this project. The reactionary layer is the post-run learning loop. Together they make every session resume with full context — even after compaction.

This goal exists to make skill maintenance an explicit ongoing concern rather than something that drifts in the background.

## Current state

- **7 skills active:** agentic-webdesign (meta), agentic-page-scrutiny, agentic-device-testing, agentic-chrome-tools, agentic-quality-discipline, agentic-script-design, reactionary
- **27 memory entries** in `~/.claude/projects/.../memory/` + mirror at `.claude/memory-mirror/`
- **Reactionary layer:** 10 references documented
- **Hook infrastructure:** 13 hook scripts in `scripts/hooks/`
- **Cross-link auto-rebuild** runs on every Write/Edit via PostToolUse hook
- **Agent-recommendations** scratchpads at `context/markdowns/agent-recommendations/` accumulate observations between consolidations

## Blockers

- None — runs continuously as background activity

## Next action

Continuous. Specific known follow-ups:
1. Post-deadline rename `agentic-script-design` → `agentic-architecture` (deferred per panic-mode)
2. Build `reactionary-automation-audit` skill (gray-area decisions ledger)
3. Periodic consolidation when any `agent-recommendations/*.md` crosses 30 entries
4. Beef up any ref that gets thin (currently: dual-worktree-safety.md is TBD)

## Exit criteria

G4 has **no terminal `achieved` state.** It is a standing/maintenance goal: it is `in-spec`
while all health invariants hold, and `needs-attention` when any is breached. Discrete
initiatives that live under it close individually (and increasingly should spin off as their
own scoped goal — see Notes).

The health invariants are *metrics*, not exit criteria — they belong on the dashboard health
panel (in-spec / out-of-spec), not as a finish line:
- Memory entries stay ≤ 30 (consolidate when over)
- Skill discovery descriptions accurately match content
- Cross-links stay valid (auto-rebuild hook ensures this)
- No ref >2000 lines (split when over)

## History

- 2026-05-24 — `agentic-webdesign` monolith split into 4 skills
- 2026-05-25 — reactionary skill layer created (6 refs)
- 2026-05-25 PM — S1/S2/S3 refs added (test-triage, snapshot-baseline-discipline, reactionary-test-baseline-update)
- 2026-05-25 PM — timestamp/logging/stat-aggregation refs added under agentic-quality-discipline
- 2026-05-26 AM — git-hygiene ref added under agentic-quality-discipline
- 2026-05-26 PM — A16 (agent-idle), A17 (5hr-reset), A18 (artifact-gap), A19 (master cross-system meta-monitoring) audits all catalogued + structural fixes designed. Goal scope expanded from "skill maintenance" to include the structural-enforcement layer that prevents recurring failure-class blindness. P0 elevation because trust-model rests on it.
- 2026-05-26 evening — A28 Phase 2 first per-skill refresh landed: `agentic-webdesign` upgraded with 3 new workflow refs (parallel-bg-for-web-design, cluster-distributed-visual-work, agent-orchestration-for-design-iterations) reflecting today's project growth (cluster, BG dispatch, hook chains, state substrate, time precision, anti-bottleneck testing, NS-context propagation). SKILL.md description updated to position skill as BREADTH with sibling skills as DEPTH. Scratchpad seeded at `agent-recommendations/agentic-webdesign.md`. Refresh-history JSONL created.
- 2026-05-27 — A28 Phase 2 second per-skill refresh landed: `agentic-chrome-tools` upgraded from 2 refs to 15 refs (10 per-mode + 3 cross-cutting growth refs + 2 legacy/overview). Per-mode refs (`mode-01-default-stateful` through `mode-10-focus-order-tab-loop`, each 6-9KB) extracted from the inline-SKILL.md monolith. Three new growth refs: `chrome-plugin-for-bg-dispatch.md` (cluster Phase 2 prep), `chrome-plugin-mode-decision-matrix.md` (what mode for what question/bug-class/lifecycle-phase), `chrome-plugin-cost-discipline.md` (G9 anti-bottleneck applied per mode). Mode-04 + Mode-09 explicitly cross-link to BG #74 dashboard-scrutiny usage (A19 Layer 3 workflow-bypass). SKILL.md description updated to position skill as "live-browser cross-cutting skill"; body restructured into mode-table pointing at per-mode refs. Scratchpad seeded at `agent-recommendations/agentic-chrome-tools.md`. Refresh-history JSONL entry written. Lock acquired + released cleanly.
- 2026-05-26 late — SET-005 monkey-chamber decision queued: unified "apply all 9 pending settings.json patches" batch (A25 critical findings #1+#2+#3). Includes the META-fix wiring `settings-edit-guard.sh` (Phase 1.2 of cleanup plan), the port-4399->4321 bug fix (Phase 1.1), and the 9 on-disk patches under `scripts/hooks/settings-patches/`. 4 options: yes-lift-and-apply-all / yes-select-subset / no-manual-merge / defer. User-gated; pending click in `reports/screenshot-monkey/data.json`. Once applied, UserPromptSubmit chain reaches 7+ entries, Stop chain 9+ entries, PreToolUse Write|Edit gains structural enforcement.
- 2026-05-31 — Goal-definition rewrite applied per user's 2026-05-31 chamber decision (goal-rewrite-drafts.md G4): Exit-criteria reframed from "health metrics = exit criteria" to "no terminal `achieved`; in-spec/needs-attention; discrete initiatives close individually"; scope line added to Notes (G4 owns substrate + health invariants, discrete build-outs spin off, NOT the by-goal home for one-shot research). Links + history untouched.
- 2026-05-27 PM — Event-bus pattern landed (A31 Front 2). New skill ref `.claude/skills/agentic-script-design/references/event-bus-pattern.md` (12K). Substrate: `.claude/cache/event-bus.jsonl` (append-only) + `event-bus-subscribers.json` (registry) + `scripts/event-bus/dispatch.{sh,py}` (dispatcher with mkdir lock + cursor tracking + cascade_depth ≤ 3 cap) + 3 initial consumers (feedback-loop filtered on outcome=verified; dashboard-refresh wildcard+30s-debounce; heartbeat-tier-high wildcard+per-event-type dedup). 3 producers wired additively (scheduler.py via opt-in `event_bus_path` dataclass field; agent-inbox-server.py via `_emit_event_bus_inbox_received`; audit-feedback-loop-compare.py via `_emit_event_bus_findings_landed` with EVENT_BUS_DISABLE env-var opt-out for tests). Dispatcher perf: <100ms when idle, ~123ms with 5 pending. All 83 scheduler + audit unit tests still pass. Settings patch documented at `scripts/hooks/settings-patches/event-bus-dispatcher.json` (user-gated; not auto-applied).

## Notes

- **Scope (2026-05-31):** G4 owns the skill / reactionary / hook / organic-OS substrate and its
  health invariants; discrete build-outs spin off as their own scoped goal or live under G17. G4
  is NOT the by-goal home for one-shot research — research attaches to the goal it serves.
- Skill maintenance cadence: write observations to `agent-recommendations/{skill}.md` after work blocks (NOT directly to refs); consolidate periodically
- Memory-mirror sync is canonical (subagents read mirror, not the project memory)
- "Skill" vs "ref" vs "plan" vs "note" — keep the distinction: skill = stable distillation; ref = sub-pattern of a skill; plan = evolving design; note = transient analysis
