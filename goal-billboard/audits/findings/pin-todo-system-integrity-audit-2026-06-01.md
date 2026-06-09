---
title: "Pin system health + no-duplicate-tracking-systems audit"
audit-id: pin-todo-integrity
date: 2026-06-01
trigger: "user 2026-06-01 — audit our pin system to ensure it works as intended; AND audit our system to make sure we never accidentally made two pin/todo systems (a common request from me; could one day be a conflict if we're not careful)"
scope: read-only inventory of every tracking/pin/todo substrate + pin-system health
verdict: NO catastrophic duplicate system; 2 real collision RISKS (pins↔ideas near-twins + pin-index drift)
---

# Pin/todo system integrity audit — 2026-06-01

## Part A — Pin system health: FUNCTIONAL but DRIFTING

The pin lane (`goal-billboard/pins/`) is real + load-bearing: `_index.md` defines the lifecycle (pinned → resumable → claimed → landed/abandoned); `scripts/dashboard/parse_pins.py` + `build_render_model.py` render it on the dashboard.

**A1 — index drift (MED).** `_index.md`'s "Active pins" table lists **P-001–P-016 (16 rows)**, but the directory actually holds **8 files**: P-001, P-002, P-010, P-013, P-017, P-018, P-019, P-020 (+ P-021 added today).
- **P-017–P-020 exist as files but are absent from the index** (4 unindexed pins).
- **~10 index rows (P-003–P-009, P-011, P-012, P-014–P-016) are fileless inline entries.**
- The index is a 2026-05-28 bootstrap snapshot that wasn't maintained as pins were added/converted.
→ *Fix:* reconcile `_index.md` to the actual files; decide whether fileless inline rows are legit lightweight pins or should become files.

**A2 — automation pending (LOW).** The lane's "next mechanism work" (auto-pin Stop hook on ghost/rate-limit, 30-day staleness sweep → `abandoned/`, "resume this pin" action) is **listed as TODO, not wired**. So "working as intended" = the manual-capture + dashboard-render half works; the automated-capture half is unbuilt.

## Part B — Duplicate-system check: NO second pin/todo system, but 2 collision RISKS

Every tracking substrate, by PURPOSE + GRAIN:

| Substrate | Purpose / grain | Distinct? |
|---|---|---|
| `goal-billboard/active/` (G*) | multi-session GOALS / initiatives | ✓ |
| `goal-billboard/pins/` (P*) | agent-narrowed work, paused mid-flight, resumable | ✓ (explicitly "NOT a to-do list") |
| `goal-billboard/ideas/` (I*) | captured IDEAS awaiting user greenlight (surface-only) | ✓ by intent — see Risk B1 |
| `goal-billboard/audits/` (A*) | reflective audit catalog | ✓ |
| `goal-billboard/proposals/` | net-new proposals (parked) | ✓ |
| `bug-billboard/` | BUGS (problem-grained) | ✓ |
| `multi-change-queue/` | parallel CHANGE orchestration (scheduler) | ✓ |
| `MANIFESTO-CHECKLIST.md` | user's urgent-push checklist (user owns "done") | ✓ |
| Task tool (`TaskCreate`) | ephemeral per-SESSION task list | ✓ (deliberate ephemeral-vs-persistent split) |
| `.claude/cache/*` (pending-parallel · suggested-tasks · user-asks-pending) | ephemeral runtime state | ✓ |

**Verdict: NO catastrophic duplicate.** Each substrate has a documented purpose, and BOTH the pin lane (`_index.md` "What this lane is NOT") and the ideas lane (`README.md` safety contract) ship an **explicit firewall** stating what they are NOT — the discipline that has so far prevented the dup the user fears.

**Risk B1 (the real one — MED): pins ↔ ideas are structural near-twins.** `scripts/dashboard/parse_ideas.py` is literally "a clone of `parse_pins.py`" (per `ideas/README.md`); both use `X-NNN-slug.md` + a `lifecycle:` ladder + a dashboard lane. Distinct by INTENT (pins = paused WORK; ideas = unbuilt IDEA awaiting greenlight) but the boundary is fuzzy and the machinery duplicated — exactly where a future mis-file or accidental merge could occur. → *Fix:* add a one-line **routing rule** (decision table) to both READMEs.

**Risk B2 (verify — LOW):** `_index.md` calls `plans/TODO-after-weekly-limit-resets.md` the "master TODO doc" and the pin lane its "operational expression." Confirm that's ONE system (doc = narrative, pins = operational), not a drifting second TODO list. → verify on reconcile.

## Recommendations (none auto-applied)
1. **Routing rule (B1) — the structural answer to the user's recurring worry.** Add to pins/ + ideas/ (and cross-ref goals/bugs/tasks): *"agent-converged-but-paused work → pin · user-facing idea awaiting greenlight → idea · bug → bug-billboard · multi-session initiative → goal · ephemeral this-session task → Task tool."* This is the cheap insurance against ever accidentally growing a second system.
2. Reconcile `pins/_index.md` to the actual files (A1).
3. Verify/retire the TODO-after-weekly master-doc relationship (B2).
4. (Optional) wire the auto-pin Stop hook + staleness sweep (A2) when budget allows.
