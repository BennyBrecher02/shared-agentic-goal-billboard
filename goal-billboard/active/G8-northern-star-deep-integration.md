---
goal_id: G8
title: Northern Star / Guiding Light deep integration (load-bearing, not just dashboard)
status: active
track: quality
northern_star: false
guiding_light: true
priority: P1
phase: "Phase 3 in flight — BG dispatch wrapper (agent-dispatch-with-ns-context.sh + bg-dispatch-wrapper.sh; A29 Layer 5 closed simultaneously). Open: exit-criteria #4 (per-NS token attribution) + #5 (clears-path badge) — both dashboard surfaces under G18."
created: 2026-05-26T23:00:00Z
updated: 2026-06-21T21:01Z  # kept active + linked UNDER G21 (belongs_to_goal: G21) per the NS-lock decision (goal-grooming-proposal-2026-06-21.md §4-E); stamp refreshed. No phase change.
belongs_to_goal: G21  # G8 is the deep-integration ARM of the portable agentic-OS (the Northern Star); sits UNDER G21 per the 2026-06-21 NS-lock. Reciprocal of G21.linked_goals. (NOTE: serves_northern_star below still reads G2 — historical pre-pivot; the live NS is G21. Left as-is — a future batch re-points the many serves_northern_star:G2 stamps to G21.)
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/northern-star-deep-integration-plan.md
  - context/markdowns/plans/automation/northern-star-feature-plan.md (Phase 1, completed; this extends)
linked_audits:
  - A22 — cluster + NS integration gap (origin)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G8 — Northern Star / Guiding Light deep integration

## Why it exists

User framing 2026-05-26 PM: *"make a note for better integration of northern star/guiding lights into our projects nitty gritty logic and thats not a small note thats a major major ask."*

Today the NS is a DASHBOARD FEATURE. After G8 it becomes a CONSTRAINT against which every decision in the project is evaluable.

The gap:
- `northern_star: true` is a goal-file frontmatter field; no other artifact carries NS context
- Dashboard shows the NS as a marquee; nothing else surfaces it
- BG dispatch prompts I write don't include "this clears the path for: G2"
- Token spend isn't attributed per-NS
- Hooks don't surface "NS at risk"
- The scheduler doesn't tag Changes with which NS they serve
- Decisions are made locally per response; the NS isn't a project-wide constraint

After G8: NS context propagates through plans, audits, BG prompts, hooks, scheduler, token attribution, every dashboard panel.

## Current state

- Phase 1 NS dashboard feature: shipped (`northern-star-feature-plan.md`)
- Deep-integration Phase 1 (this turn): **in flight** — frontmatter convention + migration script + linter + skill ref landed; 88+ artifacts retrofitted; idempotency + linter clean
- Master plan written: `northern-star-deep-integration-plan.md` (6 layers, 5 phases, multi-week effort)

**Phase 1 deliverables (this turn):**
- Plan: `context/markdowns/plans/automation/northern-star-deep-integration-plan.md`
- Schema docs: `context/markdowns/plans/README.md` (new) + `context/markdowns/goal-billboard/audits/README.md` (extended)
- Migration script: `scripts/migrate-add-ns-field.py`
- Linter: `scripts/lint-ns-frontmatter.py`
- Skill ref: `.claude/skills/agentic-quality-discipline/references/northern-star-context-propagation.md`
- SKILL.md updated with Plan/audit conventions section

## Phases (from master plan)

### Phase 1 — Frontmatter propagation
- Schema field `serves_northern_star:` added to all artifact templates
- Migration script for existing artifacts
- Linter for missing field

### Phase 2 — Hook surfacing
- state-batch-digest gains NS-status check
- recurrence-detect escalates on NS-blocking topics
- parallel-capacity-check surfaces NS-blocking pending
- Pre-response NS-advance check (new optional hook)

### Phase 3 — BG dispatch wrapper
- `agent-dispatch-with-ns-context.sh` prepends NS context to every BG prompt
- Agent (me) uses wrapper consistently

### Phase 4 — Token-spend attribution
- JSONL schema extension for ack-latency + lost-work
- Stats tab "Token spend by NS" chart

### Phase 5 — Dashboard NS-everywhere
- Every panel gets small "clears path for: G<NS>" badge
- Consistent placement + style
- vq:capture + 8-axis rubric verification

## Blockers

- None hardware-side
- A21 parallel-launch-discipline (just landed) — Phase 2's NS-blocking detection plugs into the new state-batch-digest

## Next action

- Phase 4 — token-spend attribution per NS: extend `.claude/cache/ack-latency.jsonl` + `lost-work.jsonl` schema with `serves_northern_star` field; add "Token spend by NS" pie chart to dashboard stats tab
- Phase 5 — dashboard NS-everywhere indicator on every panel + style discipline + vq:capture verification
- Adopt the Phase 3 wrapper for next BG dispatch (eat own dog food); confirm cognitive-cost reduction empirically

## Exit criteria

"Done" is pinned to these 5 concrete deliverables:
1. Every plan/audit/change record carries `serves_northern_star:` — ✅ landed (Phase 1).
2. Every BG dispatch prepends NS context — Phase 3 wrapper landed; adopt consistently.
3. `state-batch-digest` fires "NS AT RISK" when stale — Phase 2 landed; confirm firing in production.
4. Stats tab shows per-NS token attribution — Phase 4, **open**.
5. Every dashboard panel shows the "clears path for: G<NS>" badge — Phase 5, **open**.

When 4 + 5 land and 1–3 are verified-in-production, G8 → `achieved`.

## History

- 2026-05-31: Goal-definition rewrite applied per user's 2026-05-31 chamber decision (goal-rewrite-drafts.md G8): Exit-criteria tightened to the 5 concrete deliverables with explicit landed/open status (1 landed, 2–3 landed-confirm, 4–5 open) + the "4+5 land and 1–3 verified → achieved" pin; explicit non-goal paragraph added to Notes (G8 is NOT the by-goal grouping mechanism; `serves_northern_star` is the SPINE field, `belongs_to_goal` is the parent field; no future NS migration may conflate them). Links + history untouched.
- 2026-05-26 PM: G8 created; A22 catches the shallow-integration gap; master plan landed; first artifacts (this goal + G7 + A22 + cluster-kickoff-plan + this NS-deep plan) demonstrate the new `serves_northern_star:` field
- 2026-05-26 23:45Z: Phase 1 landed — migration script + linter + skill ref + 88+ retrofitted artifacts. Linter reports 0 missing; idempotency verified. Phase 1 deliverables 7-of-7 complete.
- 2026-05-26 23:18Z (in flight): Phase 2 — hook surfacing. `state-batch-digest.sh` gained section 8 NS-at-risk check (stale `updated:` > 6h | pending monkey decisions | linked-audit staleness). `recurrence-detect.sh` harvests NS keywords (title + phase + tagline + linked-plan filenames) and prepends `🌟 NS-BLOCKING (G2) —` to the system-reminder header + per-topic `[NS-blocking]` flags + replaces the closing line with an escalated directive. `parallel-capacity-check.sh` annotates each candidate with `serves_northern_star:` attribution, emphasizes NS-blocking entries with `**bold + 🌟 NS-BLOCKING (G<N>)**` in the surfaced warning, and adds `(↗ NS-blocking: K of N candidates serve <NS>)` to the header. NEW `scripts/hooks/ns-advance-check.sh` Stop hook walks the session JSONL backward to the last real user-prompt boundary (distinguishing tool-result `user` events from human prompts), extracts Edit/Write/MultiEdit/NotebookEdit `file_path` inputs, checks each file's `serves_northern_star:` frontmatter, and emits a single `<system-reminder>` only when a response did substantive edits but advanced zero NS-relevant files. Always appends `.claude/cache/ns-advance-check.jsonl` for the audit trail. Settings patch at `scripts/hooks/settings-patches/ns-advance-check.json` — user must add the new Stop entry manually (settings-edit-guard blocks agent writes to `.claude/settings.json`).
- 2026-05-26 23:48Z (in flight): Phase 3 — BG dispatch wrapper. Two new scripts landed: `scripts/agent-dispatch-with-ns-context.sh` (the augmenter — reads BG prompt body from stdin or `--prompt-file`, identifies current Northern Star + Guiding Lights from `goal-billboard/active/G*.md` frontmatter, computes file-zone heuristic from path patterns in body, prepends a strategic context block, outputs augmented prompt to stdout; flags `--copy-to-clipboard` for pbcopy and `--write <path>` for review-file). `scripts/bg-dispatch-wrapper.sh` (higher-level convenience — wraps the augmenter, adds file-zone auto-detection, checks today's `.claude/cache/scheduler-events/*.jsonl` for collision-relevant in-flight changes, prints a copy-pasteable Agent-tool invocation skeleton; does NOT actually invoke Agent tool — chat agent still does that). Skill refs updated: `.claude/skills/agentic-script-design/references/bg-dispatch-architecture.md` gained an "NS context auto-prepending (Phase 3 of G8)" section with before/after example + A29 Layer 5 connection; `.claude/skills/agentic-quality-discipline/references/northern-star-context-propagation.md` gained a "Phase 3 — BG dispatch wrapper" section covering both scripts + the Phase 1/2/3 surface table + the retrospective-vs-prospective NS-check relation (Phase 2's ns-advance-check audits AFTER; Phase 3's augmenter frames BEFORE). A29 Layer 5 (lower cognitive cost of BG launches → prevents lazy-regression) is structurally closed by the same code. Self-demo verified: synthetic `cluster Phase 2 SSH` prompt → NS context block at top + 7 active GLs listed + G7 GL inferred (cluster keyword) + in-flight scheduler changes detected + original prompt body preserved after `YOUR TASK FOLLOWS:` separator.

## Notes

- **Explicit non-goal (anti-recurrence guard, 2026-05-31):** G8 is NOT the by-goal grouping
  mechanism. `serves_northern_star` is the SPINE / Mission field (one NS at a time); the
  parent-goal field is `belongs_to_goal` and is separate (see goal-wiring-analysis §0). No future
  NS migration may write `serves_northern_star` as a stand-in for a parent goal.
- This goal is META — it serves all NS work indirectly. By making NS a constraint, it improves prioritization across all other work.
- Phase 5 verifies via vq:capture (per A19 Layer 3 — workflow-mandatory).
- NS is ONE active label at a time; Guiding Lights are multi. G8 enforces both consistently.
