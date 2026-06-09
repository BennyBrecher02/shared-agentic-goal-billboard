---
created: 2026-05-26T19:48:00Z
updated: 2026-05-26T19:48:00Z
audit_id: A3
phase: 1 — usage measurement (6 session samples, 2026-05-21 → 2026-05-26)
status: findings-locked; advisory only
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G4
---

# A3 — Hook effectiveness audit (Phase 1)

Measurement of 33 wired hooks across 6 session JSONL traces under `~/.claude/projects/<PROJECT_SLUG>/`, covering 2026-05-21 → 2026-05-26.

**Methodology:** for each hook script name, count (a) sessions that reference it at all, (b) total lines that contain its name (rough proxy for firings), (c) per-hook **output-recognition heuristic** when applicable (does the agent's text reference output keywords?).

## Summary

| Class | Hook count | Verdict |
|---|---|---|
| Earning keep clearly | 21 | Keep as-is |
| Earning keep but reducible | 4 | Keep but tune |
| Cheap insurance (silent on happy-path) | 6 | Keep |
| Worth a tune-up (output noisy or rarely-actioned) | 2 | Suggest follow-up |
| Cost-too-high candidate | 0 | None |

**No hooks recommended for removal.** The chain is dense but each one is doing something. Two are flagged for tune-up.

## Per-hook detail

### Tier A — High-signal, frequently-actioned (KEEP)

| Hook | Sessions | Line-hits | Heuristic | Verdict |
|---|---|---|---|---|
| `goal-billboard-status.sh` | 3 | 399 | central to goal awareness | KEEP |
| `verify-memory-sync.sh` | 5 | 376 | catches memory drift; always run on Stop | KEEP |
| `plan-orphan-detector.sh` | 5 | 276 | found dangling plans 4× in samples | KEEP |
| `goal-staleness-warn.sh` | 3 | 248 | surfaces panic-mode triggers | KEEP |
| `skill-cross-link-rebuild.sh` | 5 | 232 | regenerates `_xref/` after skill edits | KEEP |
| `scheduler-tick-drift-warn.sh` | 3 | 216 | warns on stale scheduler tick | KEEP |
| `worktree-remove-safety.sh` | 5 | 198 | guards against catastrophic worktree removal | KEEP — load-bearing safety |
| `sync-memory.sh` | 5 | 161 | mirrors auto-memory → subagent-readable | KEEP — structural sync |
| `memory-sync-post.sh` | 5 | 155 | bi-directional memory sync | KEEP |
| `snapshot-drift-pre.sh` | 5 | 137 | warns before snapshot overwrite | KEEP — load-bearing safety |
| `goal-queue-check.sh` | 1 | 119 | surfaces queued goal items | KEEP |

### Tier B — Earning keep, silent on happy-path

| Hook | Sessions | Line-hits | Verdict |
|---|---|---|---|
| `bug-billboard-status.sh` | 5 | 101 | KEEP — surfaces bug count on SessionStart |
| `image-cap-warn.sh` | 5 | 105 | KEEP — protects against image-cap exhaustion on Read |
| `agent-inbox-read.sh` | 1 | 92 | KEEP — load-bearing for user-from-side-session msg delivery |
| `inbox-on-prompt.sh` | 1 | 83 | KEEP — paired with agent-inbox-read |
| `token-budget-check.sh` | 4 | 80 | KEEP — surfaces 5hr window state |
| `billboard-data-refresh.sh` | 1 | 67 | KEEP — dashboard auto-refresh |
| `git-checkout-safety.sh` | 1 | 54 | KEEP — guards against destructive checkout |
| `subagent-data-refresh.sh` | 1 | 41 | KEEP — subagent dashboard sync |
| `scheduler-data-refresh.sh` | 1 | 19 | KEEP (per redundancy audit) |
| `audit-comparison-post-capture.sh` | 1 | 11 | KEEP — captures comparison data after audit |
| `dispatch-metrics-log.sh` | 5 | (low) | KEEP — feeds dispatch stats |
| `dispatch-metrics-summary.sh` | 5 | (low) | KEEP — feeds SessionStart summary |
| `lint-skill-staleness.sh` | 5 | (medium) | KEEP — warns on skill staleness |
| `audit-recommendations.sh` | 5 | (low) | KEEP — surfaces audit catalog hints |
| `compute-matrix-stats.sh` | 5 | (low) | KEEP — produces matrix-stats data |
| `detect-script-candidates.sh` | 5 | (low) | KEEP — surfaces script candidates |
| `dashboard-data-prime.sh` | 1 | (medium) | KEEP — primes dashboard data on SessionStart |

### Tier C — Worth a tune-up

#### 1. `capture-staleness.sh` (5 sessions, 167 line-hits, 171 STALE-keyword references)

Fires every SessionStart. Output mentions "stale capture" 171× across 5 sessions — proportional to firings (167). **High firings, but I see no evidence the agent took action on most of them.** Either (a) the warnings are correctly ignored most of the time (no captures in that window mattered) or (b) the threshold is too aggressive and the warning is noise.

**Recommendation:** add a `--quiet-if-empty` mode where, if there are zero captures and zero recent audit activity, the hook stays silent instead of emitting a one-line "no stale captures" status. Tightens the SessionStart digest by ~1-2 lines.

**Cost of tune-up:** ~10 min edit. Not blocking; backlog item.

#### 2. `git-drift-warn.sh` (3 sessions, "drift" mentioned 178× — but agent rarely acts)

Surfaces working-tree drift from HEAD on every SessionStart. The drift state is real (this repo has >300 modified files from accumulated workspace state), but the warning fires every session with the same "lots of drift" signal. **The agent never takes action on it** because the drift is expected (workspace + reference + sandbox files accumulate as we work). The signal-to-action ratio is essentially zero.

**Recommendation:** add an opt-in `EVIUM_GIT_DRIFT_WARN_THRESHOLD` env (default 500) below which the hook stays silent. Currently the only thing that would surface is a NEW catastrophic accumulation (e.g. >1000 modified). Tightens the digest.

**Cost of tune-up:** ~5 min edit. Backlog item.

## Output-recognition heuristic

For 3 hooks I measured how often the agent's text referenced their output keywords:

| Hook | Firings (proxy) | Keyword refs | Ratio | Reading |
|---|---|---|---|---|
| `git-drift-warn.sh` | 3 sessions | 178 "DRIFT/drift" | high | Output is being seen but rarely acted on |
| `capture-staleness.sh` | 167 line-hits | 171 "stale/STALE" | ~1:1 | Output is seen but rarely acted on |
| `token-budget-check.sh` | 80 line-hits | 68 "Token budget/5hr window" | ~1:1 | Surfaces, sometimes acted on (panic-mode triggers) |

The high keyword-reference counts for git-drift-warn + capture-staleness are mostly the agent re-emitting the output text in its own summary (i.e. SessionStart digest verbatim), not the agent *taking action* on it. That's why both land in Tier C — the firings are loud but rarely actionable.

## Recommendations (no settings.json edits)

1. **Tune `capture-staleness.sh`** to silent-when-empty. ~10 min. Defer until SessionStart digest is too long.
2. **Tune `git-drift-warn.sh`** with a drift-threshold env override. ~5 min. Defer same.
3. **All other hooks stay as-is.** No removals, no rewires.

## What this audit DID NOT do

- Modify any hooks
- Modify settings.json
- Apply any tune-up edits (those are deferred backlog items)

## Phase 2 (if/when triggered)

Phase 2 would actually instrument hook firings via a wrapper (the A3 catalog suggests piping each command through a logger that writes to `.claude/cache/hook-firings.log`). That would give us a precise firings-per-hook count plus latency. The current Phase 1 used JSONL line-counts as a rough proxy — adequate for the verdict but imprecise.

Trigger Phase 2 when:
- A specific hook is suspected of misbehavior + needs detailed firings data
- We add 5+ more hooks (pending Phase 2 wiring of orphan hooks) and want to re-measure
- A user reports SessionStart digest is too noisy

## Cross-references

- `A25` audit — finding #9 (no hook-health check) is closed by `hooks-health-check.sh` (Phase 3.1 deliverable)
- `A25` audit — finding #5 (`agent-inbox-read.sh` on `.*` matcher) confirmed earning keep (load-bearing for mid-response user messages)
- `hooks-perf-baseline-20260526.json` — companion perf data
