---
title: "Script housekeeping audit — useless-bloat check"
date: 2026-06-02
type: findings
audit_kind: code-hygiene (conservative dead-script sweep)
agent: a5638143 (conservative, evidence-first; bias-to-keep per user instruction)
trigger: "user 2026-06-02 — 'we probably have 100+ scripts; no problem keeping as many as needed, but make sure no useless bloat — be extremely careful not to mindlessly call something bloat, be thorough.'"
---

# Script housekeeping audit (2026-06-02)

## VERDICT: the codebase is clean. ZERO dead scripts. The only bloat was 25 committed `.pyc` — fixed.

318 script files (188 `.sh` + 130 `.py`) cross-referenced against **all** wiring surfaces:
settings.json hooks, inter-script calls, tests, `.claude/workflows/`, launchd plists, plans/skills/docs,
the **event-bus subscriber registry**, the **cold-RAM work-item catalog**, the **services-registry**, and
pytest discovery. Result:

| Category | Count |
|---|---|
| LIVE (wired/referenced/transitively-invoked) | ~300 |
| STANDALONE-TOOL (on-demand: audits, migration, recovery, lint — keep) | ~18 |
| SUPERSEDED / DUPLICATE / ORPHAN | **0 / 0 / 0** |

**The only genuine bloat (not a script): 25 tracked `*.pyc` under `scripts/scheduler/**/__pycache__/`** —
committed build artifacts that slipped past the ignore rule (the sibling `scripts/__pycache__` *was*
ignored). **FIXED 2026-06-02:** `git rm --cached` the 25 + added `**/__pycache__/` to `.gitignore`.

## Why "bias to keep" was right — traps the agent avoided (NOT bloat)
- `parse_scheduler_state.py` / `cross-software-check.py` (tiny) — **deliberate test fixtures** for `convention-f-closure.sh` (docstrings confirm).
- `cold_ram_constants.py` + zero-ref test files — LIVE via Python `import` (omits `.py`) + pytest *discovery* (not path-referenced).
- ~30 scripts not named in settings.json (e.g. `event-bus/dispatch.py`, `brain-note.sh`, `render-stats-data.py`) — **transitively invoked** by a live hook, or registered in the cold-RAM catalog / event-bus / services-registry.
- Apparent name-collisions (two `dispatch.sh`, two `lib-common.sh`, two `base.py`) — distinct-purpose or wrapper+impl, not duplicates.
- "Built-but-orphaned" analytics (`analyze-token-burn.py`, session-bundle pipeline) — the repo's own `built-but-not-used-analytics-audit.md` already classifies these as **surface-more, never delete**.

## Note (adjacent, not bloat)
Several analytics producers are wired-but-under-surfaced (the "last-mile telemetry gap") — a *surfacing*
gap documented elsewhere, NOT a deletion question. Full per-category detail: agent run a5638143.
