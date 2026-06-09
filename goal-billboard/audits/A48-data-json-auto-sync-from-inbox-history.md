---
audit_id: A48
title: data.json auto-sync from inbox-history — banana backlog status must derive from delivered decisions, not be manually flipped
status: in_progress
catalogued: 2026-05-27T05:16:36Z
priority_when_run: P0
estimated_effort: small-medium
trigger: |-
  2026-05-27T05:16Z — user explicit: *"if i answered these how is it still present in banana backlog visbly when all others have been succesfully vanished? actually looking now ONLY SOME have vanished?? we need audit/testing."* A19 RECURRENCE + A44 REFLEX firing — twice in same session I had to manually flip data.json (6 decisions then 3 more); the structural answer is auto-derive.
deferral_reason: NONE — substrate exists (inbox files); just needs the sync mechanism
related_goals: [G14, G6]
related_plans: [context/markdowns/plans/automation/data-json-auto-sync-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - A40 — idea-gap detector (substrate-built-but-not-queried cousin)
  - A46 — chamber UX (consumer of data.json status)
  - bug-billboard `20260527T0430-main-claude-main-monkey-yes-click-lost-on-port-conflict.md` (predecessor incident)
findings: []
---

# A48 — data.json auto-sync from inbox-history

## The recurrence

Twice this session I manually flipped data.json status pending→decided to match delivered inbox messages:
- Pass 1 (05:02Z): 6 decisions (SET-001/002/003/004, P1-07a, P1-07b-redesign)
- Pass 2 (05:16Z): 3 more decisions (SET-005, RENAME-001, INSTALL-001) — I missed them in Pass 1

User's exact frustration: *"if i answered these how is it still present in banana backlog visbly when all others have been succesfully vanished? actually looking now ONLY SOME have vanished?? we need audit/testing."*

**That's a structural problem.** Data.json should not need manual flipping. The inbox-history (`context/markdowns/agent-inbox/processed/*.md`) has the ground truth. Status should DERIVE from it, not be a separate write.

## The 4 paths to fix

### Path A — Auto-sync script (cron-style)
`scripts/sync-monkey-decisions-from-inbox.py`:
- Reads all `processed/*DECISION*.md` files
- For each decision id mentioned, ensures data.json `decisions[].status = 'decided'` + `decided[].option_id/other_text/decided_at` set
- Idempotent: safe to run on every render-data refresh
- Wire into `scripts/hooks/dashboard-data-prime.sh` so it runs whenever data.json is regenerated
- **Recommended path** — cleanest, lowest disruption

### Path B — Dashboard-side override
`loadMonkey()` extension: after fetching data.json, ALSO fetch `/agent-inbox/recent` (already does this for `loadMonkeyHistory`); for any decision with a matching DECISION message in history, override status to decided in-memory before render
- No data.json write needed
- But: ephemeral; doesn't fix the JSON itself; future loaders also need this logic
- Less recommended (band-aid)

### Path C — Pre-render hook
Before dashboard-data-prime regenerates data.json, run the sync script. Same as Path A but explicitly tied to render flow.
- Same as A; just different invocation point

### Path D — Server-side
Modify agent-inbox-server.py to write decision-state into a sidecar file (`reports/screenshot-monkey/decision-state.json`); dashboard reads BOTH data.json and decision-state.json
- New file = new schema = more complexity
- Better if we ever want multi-writer data.json (we don't)

**Recommendation: Path A.** Single script + hook wire.

## Phase 1 — sync script (BG)

`scripts/sync-monkey-decisions-from-inbox.py`:
```python
# Parses DECISION <id>: <option> — <other_text>
# regex: r'^DECISION\s+([A-Za-z0-9_-]+):\s*(\S+)(?:\s+—\s+(.*))?$'
# For each match: update data.json decisions[id].status to 'decided'
# Preserve existing 'decided' block if present; only flip status
```

Wire into `scripts/hooks/dashboard-data-prime.sh` as the LAST step before write.

Also: backfill the localStorage-clearing instruction into the script's output ("if banana backlog UI shows decisions you've answered, run this script + clear localStorage").

## Phase 2 — Audit + test coverage

Unit tests in `tests/hooks/test_sync_monkey_decisions.py`:
- Decision with answered inbox message → status flipped
- Decision without inbox message → status unchanged
- Decision already decided → no-op
- Multiple answers (re-opens) per decision → preserve most-recent
- Malformed inbox file → skip + warn

Integration test: run the sync, verify data.json matches expected diff.

## Phase 3 — A44 reflex retroactive

Per A44 (critical-finding-reflex): this audit IS the reflex firing. Add this case to A44's worked-examples list as a confirmed firing.

## Cross-references

- A19 — recurrence-detection (twice manual = structural)
- A40 — idea-gap-detector (cousin: substrate built but not queried)
- A44 — critical-finding-reflex (THIS firing)
- A46 — chamber UX (consumer)
- bug-billboard `20260527T0430-*` (origin incident)
- Path forward: integrates with A47's BG-discipline (sync runs as a hook, not a long BG)

## Lessons (preliminary)

- **Pass 1 missed Pass 2 decisions** because the decision-list snapshot I worked from was stale (didn't include SET-005/RENAME-001/INSTALL-001 which user answered AFTER my flip).
- Manual sync is fundamentally racy: the user is answering decisions WHILE I'm fixing them.
- Auto-sync derives from ground truth at each render — no race possible.
- This is the SAME pattern as A40 (gaps) and A44 (findings) — substrate exists, derivation logic missing.

## Status

IN PROGRESS — design landed; Phase 1 sync script queued; user will run manually OR I can spawn the BG (your call).

## Shared hardware barrier (2026-05-28 — adaptive-immunity retro-sweep)

A48's missing HARDWARE tether (nothing prevented a manual status write to `data.json` that drifts from delivered-decision ground truth) is now provided by a **shared barrier** built once and claimed by THREE instances — A57 (dashboard-content) + A48 (this) + A61 (inbox-clog). Per the retro-sweep roadmap (`audits/findings/adaptive-immunity-retro-sweep-roadmap-2026-05-28.md`): "block direct writes to `data.json` / inbox except via the sanctioned wrapper" — the single highest-leverage move in the sweep.

- **HW guard:** `scripts/hooks/wrapper-only-write-guard.sh` — PreToolUse `Write|Edit` hook. A direct write to `reports/screenshot-monkey/data.json` (the surface where manual status flips caused A48's drift) is flagged and pointed at the sanctioned sync path (`sync-monkey-decisions-from-inbox.py` via `dashboard-data-prime.sh`), which re-derives status from ground truth. Same `data.json` HW zone A48 shares with A57.
- **VERIFY:** `tests/wrapper-only-write-guard-test.sh` — A71-shaped, 5/5 passing (assertion 2 covers the `data.json` direct-write path). A48 ALSO retains its own real VERIFY (`test_sync_monkey_decisions.py`) for the idempotent-re-derive property; the two are complementary (barrier blocks the raw write; the sync test proves derivation correctness).
- **Mode:** ships in **WARN/dry-run** (`exit 0`). Kill-switch `EVIUM_WRAPPER_GUARD_OFF=1`.
- **Live-flip is USER-GATED:** WARN→BLOCK (`EVIUM_WRAPPER_GUARD_MODE=BLOCK`) + `settings.json` wiring is an explicit USER action after a 24h dry-run review — NOT an agent turn. See the hook header for the flip steps.