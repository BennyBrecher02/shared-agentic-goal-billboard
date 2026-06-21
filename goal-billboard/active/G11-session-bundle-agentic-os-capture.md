---
goal_id: G11
title: Session bundle + agentic-OS data capture (analyzer-rich retrospection)
status: active
track: quality
northern_star: false
guiding_light: true
priority: P1
phase: "Phase 2 LANDED — 5 analyzers total (themes + KPI deltas + token efficiency + recurrence detection + domino effectiveness) running in parallel on POC bundle in 0.14s wall-clock; aggregator extended with priority section ordering + phase auto-bump"
created: 2026-05-27T00:30:00Z
updated: 2026-06-21T21:01Z  # refreshed (tracking freshness) per goal-grooming-proposal-2026-06-21.md §4-G — real progress (the capture half of the capture-vs-act seam with G14); not stalled, just un-stamped. No phase change.
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/session-bundle-infrastructure-plan.md
linked_audits:
  - A31 — Pre-restart prep master (origin)
linked_skills:
  - .claude/skills/reactionary/  (the event-triggered counterpart; G11 is the session-shaped extension)
  - .claude/skills/agentic-quality-discipline/references/session-bundle-discipline.md  (Phase 1 ref)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G11 — Session bundle infrastructure

## Why it exists

User explicit 2026-05-27 00:25Z: *"can we make the act of capturing the data from the graceful startup until a graceful shutdown as another data bundle... im seeing insane potential from this data, these hours long plow runs and their context and stats etc will be useful for eval and improvement... systemwide domino ideas youll sprout."*

Analog to `reports/timelapse/` data bundles but for SESSION-shaped windows. Captures full agentic-OS activity between graceful-startup and graceful-shutdown for retrospective analysis.

## Why it's load-bearing

- A30 + G10 capture state (snapshot = where we are). G11 captures the journey (bundle = how we got here).
- A24 meta-loop-eval is monthly. G11 is per-session.
- Reactionary skills are event-triggered. G11 is session-shaped.
- Cross-session trend detection requires bundles to compare against.
- Insights from analyzers feed back into the system (recommendations, recurrence signals, KPI tracking).

## Current state

- **Phase 1 LANDED** (2026-05-27 02:30Z) — POC bundle for the 2026-05-26 plow run created. Real numbers: 13.23 hour window, 182 BGs (180 dispatched in-session), 18 audits opened (A9/A10/A12/A13/A14/A15/A16/A17/A18/A19/A20/A21/A22/A23/A24/A25/A26/A30), 2 closed (A13, A15), 10 findings docs landed, 10.96M tokens consumed, 2 memory rule files added, 5 skill refresh entries. Top theme: `audit-catalog-findings` (55 hits, 33%); runners-up: scheduler-multi-change (29), monkey-chamber-domino (23), graceful-shutdown-resume (14), g2-phase5-evium (10). Bundle creator runs in 1s, analyzer in <2s, aggregator <1s — well under <5s budget.
- **Phase 1 deliverables** (7-of-7 landed):
  1. `scripts/session-bundle-create.sh` (bundle creator, ~620 lines)
  2. `scripts/session-bundle/analyzer-themes.py` (first sub-analyzer)
  3. `scripts/session-bundle/aggregator.sh` (insights.md merger)
  4. `scripts/graceful-shutdown-snapshot.sh` (TODO comment added for Phase 2 wire-in)
  5. POC bundle at `reports/session-bundles/2026-05-26T11-00-00Z/` (20 files, ~188KB)
  6. `.claude/skills/agentic-quality-discipline/references/session-bundle-discipline.md` (10.3K)
  7. G11 + A31 frontmatter updates
- **Phase 2 LANDED** (2026-05-27 02:55Z) — 4 additional sub-analyzers + aggregator extended; 5 analyzers running in parallel on POC bundle. Wall-clock 0.14s (parallelism wins; sequential would be ~3-5s). New analyzer outputs on POC bundle: KPI deltas baseline (2.5KB; no prior bundle to compare yet — populates from session #2), token efficiency (8.5KB; 304K tokens-per-deliverable, audit category dominates at 59/180 BGs), recurrence detection (5.4KB; 3 A19 events + 31 frustration-flagged prompts + topic continuation infrastructure), domino effectiveness (3.9KB; 6 event types tracked, all have wildcard coverage so 0 missing dominoes). Aggregator now uses priority section ordering (themes → kpi-deltas → token-efficiency → recurrence-detection → domino-effectiveness) and auto-bumps `manifest.phase` to 2 when ≥5 analyzers present. Total `insights.md` now 24.7KB (was 4.3KB Phase 1).
- **Phase 2 deliverables** (7-of-7 landed):
  1. `scripts/session-bundle/analyzer-kpi-deltas.py` (ack-latency + parallel-capacity + pushback + skill-quadrant)
  2. `scripts/session-bundle/analyzer-token-efficiency.py` (per-category histograms + outliers)
  3. `scripts/session-bundle/analyzer-recurrence-detection.py` (A19 events + frustration prompts + cross-session continuation flag)
  4. `scripts/session-bundle/analyzer-domino-effectiveness.py` (producer→consumer matrix + missing dominoes detection)
  5. `scripts/session-bundle/aggregator.sh` extension (priority ordering + phase auto-bump)
  6. POC bundle re-run with all 5 analyzers (5 insights-by-domain files; insights.md regenerated)
  7. G11 + A34 + plan frontmatter updates
- Phases 3-8 (per A34): queued. Phase 3 adds 8 more analyzers (13 total per A34 vision).

## Bundle structure summary

`reports/session-bundles/<session-start-iso>/`:
- bundle-manifest.json (schema_version=1)
- snapshot-entry.json + snapshot-exit.json
- jsonl-diffs/ (8+ delta files)
- audit-changes/ (opened/closed/findings)
- monkey-clicks.jsonl
- token-curve.csv
- bg-dispatch-log.jsonl
- memory-rules-added.md
- skill-refresh-history-during-session.jsonl
- insights.md (parallelized-analyzer output)
- insights-by-domain/ (per-domain sub-insights)

## Blockers

- Phase 1: none — BG launching this turn
- Phase 5 (auto-archive): needs storage management decisions later

## Exit criteria

Phase 1: ✅ bundle created for TODAY's session as POC; manifest schema validated; 1 analyzer runs
Full G11: 5+ analyzers in parallel; insights surface in resume hook; cross-session trend detection working; auto-archive at 30 bundles
**G11 exits as `achieved` after 5 consecutive sessions produce bundles + at least one cross-session trend insight surfaces actionably.**

## History

- 2026-05-27 00:30Z: G11 created; A31 origin + session-bundle-infrastructure-plan landed; BG D launching for Phase 1 POC
- 2026-05-27 02:30Z: Phase 1 LANDED. Bundle creator + analyzer-themes + aggregator stub + 1 POC bundle for the 2026-05-26 session + skill ref. 7-of-7 deliverables shipped. The agentic-OS now produces session-shaped retrospective bundles; the analyzer pattern is parallelizable and tier-aware via G9. Bundle creation 1s, analyzer <2s, aggregator <1s — under <5s shutdown budget. Phase 2 next: 4 more analyzers in parallel.
- 2026-05-27 02:55Z: Phase 2 LANDED. 4 new analyzers (kpi-deltas, token-efficiency, recurrence-detection, domino-effectiveness) + aggregator extension shipped. 5 analyzers run in parallel in 0.14s wall-clock on the POC bundle (massive headroom under the <30s budget A34 sets for the 13-analyzer Phase 3 target). All 4 new insights-by-domain files between 2.5KB-8.5KB; combined `insights.md` 24.7KB. Per-BG token isolation is the remaining gap for the token-efficiency analyzer; cross-session deltas/continuation populate from session #2 onward. Phase 3 next per A34: 8 more analyzers + master system audit + post-analysis hooks ecosystem.

## Notes

- G11 SERVES G2 (the Northern Star) indirectly — better session retrospection = faster compounding of learning = faster G2 delivery and post-G2 work.
- Reactionary layer (event-triggered post-run learning) is the per-tool analog. G11 is the per-session umbrella.
- Token-budget continuity is captured in token-curve.csv; cross-session burn-rate trend visible.
