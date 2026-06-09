---
audit_id: A14
phase: initial
date: 2026-05-26
ran_by: agent (Opus 4.7 1M)
input: user pushback 2026-05-26T15:55 local — "are you relating time passed to our token usage warnings"
---

# A14 initial findings — what's missing from the stats system today

## TL;DR

The token tracker measures **how much** without **how fast** and **how long**. Three orthogonal axes — tokens, wall-clock, rate — only one is currently tracked. This finding inventories the gaps and the build plan.

## Gap inventory

### Gap 1 — session age / duration is computed but never surfaced

`scripts/run-metrics-aggregator.py` parses `start_ts` and `end_ts` from each JSONL but the summary output only shows the `end` timestamp, not the wall-clock duration. The user has to mentally subtract start from end every time they want to know "how long have we been at this."

**Fix:** add a `duration_seconds` + human-formatted `duration` field. Display "Session age: 3h 12min" in the summary block.

### Gap 2 — no tokens-per-minute (burn rate)

The aggregator has total tokens; it has start/end timestamps. It doesn't compute the ratio.

**Fix:** add `tokens_per_minute = effective_tokens / duration_minutes` (skip if duration < 60s to avoid divide-by-zero on fresh sessions). Display "Burn rate: 248k tok/min" in the summary.

### Gap 3 — no activity heatmap (messages per 30-min bucket)

Knowing total messages doesn't tell us whether they were front-loaded (early planning), tail-loaded (panic mode pre-reset), or steady. A simple bucket histogram fixes this.

**Fix:** bucket the message timestamps into 30-min slots. Surface as ASCII bar chart (in summary) + raw array (in cache JSON for dashboard consumption later).

### Gap 4 — Claude's 5hr rolling window is invisible to the agent

The 5-hour rolling window is enforced by Anthropic but the agent cannot query it. Today the user said "reset at 5pm, currently 3:55pm" → I had to manually compute 65min remaining.

**Fix:** `.claude/cache/5hr-window-reset.txt` stores the user-provided next-reset time (ISO 8601 UTC). `scripts/claude-5hr-window.py` CLI:
- `set <iso-time>` — record the reset
- `status` — time-to-reset countdown + projected at-reset usage
- `--watch` — refreshing one-line display (printed once; not a daemon)
- `clear` — wipe (for testing)

### Gap 5 — token-budget hook fires only on SessionStart

The hook surfaces a snapshot once, at session boot, then never updates. Long sessions go silent.

**Fix:** extend the hook output to include the time dimension (gaps 1-4 surfaced together). The hook still fires at SessionStart, but the data being surfaced is richer. Also a manual-call path: `python3 scripts/claude-5hr-window.py status` runnable anytime.

### Gap 6 — no daily global reset / cumulative tracking

If we use 45M tokens at 10am and another 25M at 4pm, today's total is 70M but no script aggregates this. A daily reset (midnight local) enables "today's total usage" + historical comparison.

**Fix:** `.claude/cache/daily-token-snapshot.json` — schema: `{ date: "2026-05-26", sessions: [{session_id, tokens, start, end}], total_effective, total_messages, by_hour: {"00": N, "01": N, ...} }`. `scripts/daily-stats.sh` prints today's snapshot. Aggregator updates it on each run.

### Gap 7 — no standing protocol for "use the system you built"

When the user asks "how much budget left," I sometimes answer from gestalt instead of running the tracker. The fix is memory: when user asks about budget OR time-in-session, **always** run BOTH the aggregator AND the 5hr-window status. Surface both numbers together.

## Build sequence

1. Catalog A14 (this file + parent catalog entry) — DONE
2. Augment `run-metrics-aggregator.py` (gaps 1-3, 6)
3. New `scripts/claude-5hr-window.py` (gap 4)
4. Extend `scripts/hooks/token-budget-check.sh` (gap 5)
5. New `scripts/daily-stats.sh` (gap 6)
6. Memory: `feedback_standing-protocols.md` + mirror (gap 7)
7. Live-data verification on this session

## Risks

- **The 5hr-reset file goes stale.** If user forgets to update it after a reset, the math projects on yesterday's window. Mitigation: `status` output prints a warning if the recorded reset is more than 5h 30min in the past.
- **Cache file races.** Aggregator + 5hr-window CLI both write to `.claude/cache/`. Mitigation: aggregator writes only `token-metrics.json` + `daily-token-snapshot.json`; CLI writes only `5hr-window-reset.txt`. No overlap.
- **Dashboard tab not built.** Per scope, dashboard tab is deferred — the data files exist; rendering lands later. No data loss.

## Acceptance for "audit complete"

- Token-budget hook output, after this build, shows: session age, tokens used, burn rate, time-to-reset countdown, projected at-reset usage, daily cumulative — in one block, in one SessionStart fire
- Manual call `python3 scripts/claude-5hr-window.py status` succeeds and prints the same time-to-reset info on demand
- Memory protocol added; mirror synced
- One live snapshot of THIS session's stats included in the audit findings (final section)

## Live snapshot — pre-build

Session info as of catalog-time:
- Aggregator data is current as of 2026-05-26T19:50:58Z (the last cache refresh)
- 5hr-window file does not exist (`.claude/cache/5hr-window-reset.txt` — to be created)
- User-provided reset: 17:00 local (which is 2026-05-26T22:00:00Z assuming -05:00 local TZ for the timestamp; pacific summer would be -07:00 = 2026-05-27T00:00:00Z; **set per user-provided 5pm local** assumption recorded; user should correct if wrong)

## Live snapshot — post-build (2026-05-26T20:08Z)

Build complete. Verified output:

### `python3 scripts/claude-5hr-window.py status`

```
── Claude 5hr rolling window ──
  Next reset (UTC):   2026-05-26T21:00:00Z
  Next reset (local): 2026-05-26 17:00 EDT
  Time until reset:   54min
  Current session:    699bce48 · age 7h 36min
  Tokens (effective): 34.78M  (70% of 50.00M ceiling, src: default)
  Burn rate:          76.2k tok/min
  Projected at reset: +4.15M → 38.93M
  Headroom at reset:  11.07M below ceiling
```

### `bash scripts/hooks/token-budget-check.sh` (excerpt — the time block)

```
── Tokens (current session, effective = in+out+cache_create) ──
███████████████████░░░░░░░░░ 34.78M / 50.00M (70%)
in=12.3k out=3.60M cache_create=31.17M (+cache_read=765.94M free-read)
msgs=1756 · ceiling source: default
── Time ──
Session age (start→now): 7h 36min
JSONL span (start→last-msg): 7h 36min
Burn rate: 76.2k tok/min · projected ceiling hit in 3h 19min
Activity (last 3h, 30-min buckets):
  13:00  ██████████████████ 89
  13:30  ████████           43
  14:00  ███                15
  15:00                     1
  15:30  █████              26
  16:00  ██                 14

── Claude 5hr rolling window ──
Next reset: 2026-05-26T21:00:00Z (in 54min)
At current burn (76.2k/min): +4.16M → 38.94M by reset

── Daily cumulative (2026-05-26 local) ──
Today's effective tokens: 34.78M
Today's message count: 1756
By hour (msgs):
  03:00  ███              60
  …
  10:00  ████████████████ 246
  11:00  █████████████    206
  …
```

### `python3 scripts/claude-5hr-window.py status --watch` (one-liner)

```
5hr-window: reset in 54min · now 34.78M @ 76.2k/min → ~38.93M by reset
```

### `bash scripts/daily-stats.sh`

```
─── Daily stats (2026-05-26) ───
  Total effective tokens today: 34.78M
  Total messages today:         1756
  Sessions active today: 1
    699bce48 · 34.78M · 1756/1756 msgs (100%)
  Hourly activity (msgs, local TZ):
    03:00  █████                    60
    …
    16:00  █                        14
```

## Cross-check of acceptance criteria

- [x] Token-budget hook output shows time + tokens + rate together in one block
- [x] `python3 scripts/claude-5hr-window.py status` returns time-to-reset + projected usage
- [x] Standing-protocols memory has the always-run-both rule (+ mirror synced)
- [x] One real-data check confirms the math works on the live session (above)
- [ ] 7th dashboard tab — DEFERRED per scope; the data files are in place (`token-metrics.json`, `daily-token-snapshot.json`, `5hr-window-reset.txt`) so a tab can be added by extending `render-billboard-page.py` later without changes to the underlying data layer

## Status

A14 catalog entry should be moved to `archived/audits/` and status flipped to `completed` (per standing-protocols audit lifecycle). NOT done in this turn — leaving `status: in_progress` so user can verify the build before archival. After confirmation, `mv` it and add a final Lessons section to the catalog entry.
