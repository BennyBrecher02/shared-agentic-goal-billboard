---
audit_id: A37
belongs_to_goal: G6   # by-goal parent (goal-wiring §0 tier-A — filename + related_audits:[…,G6] name the goal explicitly; the finding IS a G6 routing gap)
finding_id: A37-g6-phase2-priority-routing-gap
filed_at: 2026-05-27T03:23:00Z
phase: 2
severity: medium
category: scheduler-correctness
related_audits: [A37, G6]
status: open
proposed_owner: scheduler maintainer (deliberation required)
---

# A37 follow-up — G6 Phase 2 priority-routing gap

## Summary

`Change.priority` is declared, validated, and round-tripped through scheduler state — but **the scheduler never reads it at routing time**. The "priority routing" feature claimed by task #82 (marked completed) did not land. All scheduled changes still run mtime-FIFO regardless of priority.

This is **not** patched in A37 Phase 2 because the fix changes scheduling semantics (which tiebreaker wins, what happens when a critical change arrives behind a low-priority in-flight item) and deserves its own deliberate task.

## Specific file:line evidence

### Where priority is declared + validated

`scripts/scheduler/change.py:25`
```python
VALID_PRIORITIES = ("critical", "high", "medium", "low")
```

`scripts/scheduler/change.py:41-47`
```python
# NOTE (A37 Phase 1, 2026-05-27): `priority` is declared + validated against
# VALID_PRIORITIES, but NOT yet consumed by scheduler routing. The scheduler
# picks the oldest scheduled/*.yml by mtime (state.list()) regardless of
# priority value. G6 Phase 2 will wire priority into list() ordering — until
# then, all changes run mtime-FIFO. Do not assume `priority: critical`
# jumps the queue. See: A37 follow-up `audits/findings/A37-g6-phase2-priority-routing-gap.md`.
priority: str = "medium"
```

`scripts/scheduler/change.py:125`
```python
priority=str(data.get("priority", "medium")),
```

`scripts/scheduler/change.py:150`
```python
if self.priority not in VALID_PRIORITIES:
    # raises
```

### Where the scheduler picks the next change (priority-blind)

`scripts/scheduler/scheduler.py:444-454`
```python
# 1. Read scheduled/ FIFO
scheduled = self.state.list("scheduled")
if not scheduled:
    ...
change_path = scheduled[0]
```

`scripts/scheduler/state.py:40-49`
```python
def list(self, state: str) -> list[Path]:
    """List change.md files in the given state dir, FIFO by mtime (oldest first).
    ...
    """
    d = self.root / state
    ...
    files = [p for p in d.iterdir() if p.is_file() and p.suffix == ".md" and not p.name.startswith(".")]
    files.sort(key=lambda p: p.stat().st_mtime)
```

The sort is **purely mtime ascending**. No call site reads `priority`. A `grep -n priority scripts/scheduler/scheduler.py` returns nothing.

## Why this didn't show up earlier

Task #82 was marked completed but the only code that landed was:

1. Validation of the priority field at parse time (change.py:150).
2. Round-tripping through YAML (change.py:125).

Neither validation nor round-tripping affects routing. The integration into `state.list()` (or a sibling `state.list_prioritized()`) never happened. Tests at `tests/scheduler/test_priority.py` (if they exist) would have only tested the parse/validate path, not the integration. (Worth a follow-up: verify whether a priority test exists.)

## Proposed fix (for the future task that owns this)

### Option A — minimal: priority-aware sort in `state.list()`

Sort scheduled/ first by priority rank (critical < high < medium < low), then by mtime ascending as the tiebreaker:

```python
PRIORITY_RANK = {"critical": 0, "high": 1, "medium": 2, "low": 3}

def list(self, state: str, *, priority_aware: bool = False) -> list[Path]:
    d = self.root / state
    if not d.is_dir():
        return []
    files = [p for p in d.iterdir() if p.is_file() and p.suffix == ".md" and not p.name.startswith(".")]
    if priority_aware and state == "scheduled":
        # Read priority from each file's frontmatter; default to "medium" if missing/unparseable.
        def _rank(p: Path) -> tuple[int, float]:
            try:
                pri = _extract_priority(p.read_text(encoding="utf-8"))
            except Exception:
                pri = "medium"
            return (PRIORITY_RANK.get(pri, 2), p.stat().st_mtime)
        files.sort(key=_rank)
    else:
        files.sort(key=lambda p: p.stat().st_mtime)
    return files
```

Then `scheduler.py:445`:
```python
scheduled = self.state.list("scheduled", priority_aware=True)
```

### Option B — preserve fairness: priority + FIFO with starvation guard

Pure priority sort starves low-priority work indefinitely if critical keeps arriving. Add an aging-bonus: once a low-priority change has been in `scheduled/` longer than (say) 4× the median tick interval, promote its effective priority by one tier.

This is more complex (needs scheduler-events for tick history) but matches the FIFO+aging pattern documented in `agentic-script-design/references/opsys-synchronization.md`.

### Option C — read priority from cache, not file

Reading priority requires parsing each scheduled change's frontmatter on every tick — O(n) IO per pick. For small queues (<50 items) this is fine. For larger queues, cache priority into a sidecar `.priority` file at scheduling time and read that.

## Why this should be its own task (not auto-applied here)

1. **Scheduling-semantics change.** The current behavior — mtime-FIFO — is well-tested and predictable. Adding priority changes which-runs-when, which is a load-bearing scheduler property. Needs deliberation.
2. **Starvation risk.** Naive priority sort starves low-priority changes when critical keeps arriving. Mitigation requires aging logic, which is itself a design decision (linear vs exponential aging, age-tier promotion threshold).
3. **Test coverage required.** Need at least 5 tests: (a) priority dominates mtime, (b) same-priority falls back to mtime, (c) starvation prevention if aging is chosen, (d) backward compat when no `priority` field is present, (e) parse failure defaults to medium.
4. **Backward compat.** Existing scheduled/*.md files without `priority:` need to default to medium without surprises.
5. **Coordination with task #82.** Task #82 should either be reopened or a new task referencing this finding should be created.

## Recommended next step

Reopen task #82 with corrected description ("validation landed; routing integration still pending") OR file a new dedicated task. Block on: (a) deliberation about Option A/B/C, (b) starvation policy decision, (c) test plan.

## Cross-references

- A37 Phase 1 §B3 (the original finding)
- A37 Phase 2 application report
- `scripts/scheduler/change.py:41-47` (the in-code disclaimer added by A37 P6)
- G6 (scheduler goal)
- `agentic-script-design/references/opsys-synchronization.md` (FIFO+aging discipline)
