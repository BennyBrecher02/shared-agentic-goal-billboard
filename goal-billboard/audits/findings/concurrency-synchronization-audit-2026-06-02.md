---
title: "OS concurrency & synchronization audit"
date: 2026-06-02
type: findings
audit_kind: system-infrastructure (concurrency / races / deadlock)
method: .claude/skills/agentic-quality-discipline/references/concurrency-synchronization-audit.md
agents: 6 parallel read-only auditors (scheduler · memory-sync · cache-ledgers · daemons · hooks · state-files+event-bus)
read_only: true
trigger: "user 2026-06-02 — an Edit 'modified since read' reject during a memory-mirror sync raised: is the OS full of races? can it deadlock?"
---

# OS concurrency & synchronization audit (2026-06-02)

> **STATUS UPDATE 2026-06-02 (later same day) — CRITICAL RESOLVED.** The lone CRITICAL (dashboard
> `reports/timelapse/<run>/data.json`, 7-merger race) is **FIXED + tested + committed**: the mergers run
> SEQUENTIALLY under a `mkdir`-lock via `scripts/dashboard-merge-run.sh` (both `dashboard-data-prime.sh` +
> `billboard-data-refresh.sh` call it), and `scripts/heal-timelapse-data.py` recovers an already-corrupt base
> (the live file WAS corrupt — one object + a trailing fragment). Gated by `scripts/tests/test-dashboard-merge-run.sh`
> (6/6). **Newly nailed root sub-cause:** the renderers shared ONE fixed tmp name (`data.json.tmp`), so
> `tmp+rename` was *not* atomic across writers — they interleaved into the shared tmp. Sub-lesson folded into the
> audit ref's POST-AUDIT LEARNINGS. Defense-in-depth follow-up (per-process tmp name) noted, not yet done. The
> 5 HIGH + MED/LOW remain open (memory-mirror HIGH was fixed earlier the same day).

## VERDICT — the two questions, answered directly

**1. Can the system DEADLOCK? — NO. Proven, not asserted.** The OS is lock-FREE or single-lock
everywhere except the scheduler, and the scheduler is **provably acyclic**: its four monitor
resources never co-hold their RLocks (`Condition.wait_for` releases the lock while blocked, and no
resource method calls another under its own lock), so the lock-order graph has **no edges → no
cycle**; every multi-resource site routes through `acquire_all` which sorts alphabetically and in
practice collapses to a singleton. There is **no `flock` anywhere**, the cross-process PID lockfile
is a single lock with automatic stale-PID reclaim, and releases are exception-safe (try/finally).
Deadlock is not reachable.

**2. Is the code "full of race conditions"? — NO, but there is a real, fixable CLUSTER.** The system
is *mostly* well-synchronized on purpose: append-only single-line < PIPE_BUF (A71), `tmp+rename`,
`mkdir`-locks, and per-session keying are used correctly in many places (the event bus, the inbox,
the JSONL ledgers, the daemons, the scheduler). The bugs are concentrated where one pattern was
applied **wrong**, and they all share a single root cause (below). **1 CRITICAL · 5 HIGH · 4 MEDIUM ·
5 LOW.** None is a deadlock; all are lost-update / corruption / TOCTOU under concurrent writers.

## THE ROOT CAUSE (one sentence, the through-line of almost every finding)
> **A read-modify-write on a shared file made to *look* safe by `tmp+rename` — but `tmp+rename`
> makes the WRITE atomic, not the read-modify-write SEQUENCE.** Two writers both read base D0, each
> writes D0+their-change, last-writer-wins → the other's change is silently lost. The atomic write
> is a **decoy**. The fix is always the same: serialize the RMW with a lock (the **event-bus
> dispatcher `dispatch.py:53-92` is the in-repo reference implementation** — mkdir-lock + stale
> recovery + atomic cursor), or redesign to append-only / per-writer-sidecar (which the inbox and
> the JSONL ledgers already do right).

## Per-subsystem verdict
| Subsystem | Verdict |
|---|---|
| **Scheduler** | ✅ CLEAN — no deadlock reachable; locking sound; 2 LOW defensive nits only. |
| **Daemons (8 autonomic)** | ✅ CLEAN — append-only per-daemon files, launchd single-instance, DRY_RUN-gated. "5/8 stuck" is **NOT a hang** — launchd between-tick idle + a cache-location split makes `daemon-health.sh` read stale timestamps. |
| **Event bus** | ✅ SOUND — mkdir-lock dispatcher + atomic O_APPEND producers + tmp+rename cursor. The pattern to copy. |
| **Bug-billboard INBOX** | ✅ A71-safe — unique-per-writer filenames, never a shared mutable file. |
| **Cache ledgers** | ◑ mostly safe (append-only / tmp+rename); **2 HIGH** non-atomic JSON rewrites. |
| **Hooks** | ◑ mostly safe; **2 HIGH + several MED/LOW** in the drain/rewrite hooks. |
| **Memory dual-mirror sync** | ⚠ **HIGH** — the trigger; a mirror edit gets silently back-overwritten. |
| **Dashboard `data.json`** | 🚨 **CRITICAL** — 7 concurrent unlocked RMW mergers drop blocks every session. |

## Findings (prioritized)

### 🚨 CRITICAL
- **C1 — dashboard `reports/timelapse/<run>/data.json`: 7 concurrent unlocked read-modify-write mergers.**
  `render-{stats,asks,meta-monitoring,cluster,meta-loop-eval,skill-metrics,bg-pattern}-data.py` each
  `json.loads(data.json) → set its block → tmp.replace`. Launched as parallel `&` subshells by
  `dashboard-data-prime.sh:36-106` (SessionStart) AND re-launched by `billboard-data-refresh.sh`
  (PostToolUse Write|Edit) → up to 14 unlocked writers racing one file. **6-of-7 blocks silently
  dropped per run**; the tmp+rename hides it. Evidence: `render-stats-data.py:534-547`. **VERIFIED.**
  Fix: run the 7 timelapse-mergers **sequentially in one backgrounded subshell** under a shared
  **mkdir-lock** (mirror `dispatch.py`); the 3 own-file renderers (billboard/subagent/scheduler) stay
  parallel (each owns its file — safe). No session-latency cost (backgrounded). **TESTABLE: run the
  hook, assert all 7 blocks present in data.json.**

### ⚠ HIGH
- **H1 — memory dual-mirror back-overwrite (THE TRIGGER). [FIXED 2026-06-02]** `memory-sync-post.sh:18`
  fires the one-way `auto-load → mirror` sync on edits to the **mirror** too, so a mirror edit is
  silently overwritten with older auto-load content; and a sync firing mid-edit caused the Edit
  "modified since read" reject. Fix applied: regex now fires the sync **only on auto-load edits**;
  `sync-memory.sh` cp is now **atomic (tmp+rename)**; discipline updated to **edit the canonical
  auto-load copy ONLY — the sync propagates to the mirror; never hand-edit the mirror.**
- **H2 — `token-metrics.json` + `daily-token-snapshot.json`: non-atomic full rewrite.**
  `run-metrics-aggregator.py:217,298` `Path.write_text` (no tmp+rename), fired EVERY SessionStart by
  `token-budget-check.sh` → two near-simultaneous sessions = lost-update + torn read for any reader.
  Fix: tmp+`os.replace` (the repo already does this at `pending-parallel-work-scan.sh:325-327`).
- **H3 — drain hooks `lost-work-drain.sh` + `user-ask-artifact-check.sh`: RMW with a FIXED tmp name.**
  Full read-rewrite of a queue that a *different* hook appends to concurrently → silent lost-update
  (H3 on `user-asks-pending.jsonl` **defeats the A40 never-miss-an-idea guarantee**) + tmp-collision
  corruption when two sessions drain at once. Fix: unique `mkstemp` + mkdir-lock, OR move the
  "processed" flag to an append-only side-file (one-direction-of-flow).
- **H4 — `bug-billboard-consolidate.sh`: master.md non-atomic rewrite + double `shutil.move`.**
  Two overlapping SessionStart consolidations → lost-update on master.md (`:268` `write_text`) and a
  `FileNotFoundError` double-move under `set -e` that aborts mid-run, leaving master.md + inbox
  inconsistent. Fix: mkdir-lock at entry (early-exit if held) + atomic master.md write + `missing_ok` move.
  *(NOTE: the inbox APPEND side is A71-safe — only the consolidate side is at risk.)*

### ◑ MEDIUM
- **M1 — `skill-cross-link-rebuild.sh` `touch`-based lock is a TOCTOU** (`[ -f LOCK ]` then `touch` —
  two fires in the gap both proceed) + non-atomic `write_text` of SKILL.md. Fix: `mkdir`-lock + tmp+rename.
- **M2 — `hedge-detect-drain.sh` lost-update** (re-derived next turn, and uses a unique mkstemp so no
  corruption — self-healing). Fix: lock the read→replace, or append-only side-file.
- **M3 — dual-orchestrator memory edits** — serialized today by the Edit optimistic-reject on the
  canonical copy; **standing rule: never replace a memory `Edit` with a blind `Write`/`cp`/`>`** (that
  bypasses the reject → silent loss). For append-style memory (MEMORY.md index), prefer per-session
  staging + a single consolidator (A71) over N sessions editing one global file.

### · LOW
- **L1 — scheduler PID-lock**: `mkdir` race throws an ungraceful traceback instead of the clean
  "already running" exit; no signal-trap release (self-heals via next-run PID reclaim). `__main__.py:81-97,118-153`.
- **L2 — `dispatch-log.csv`** header-write TOCTOU (benign; no data, first-run only).
- **L3 — global ack cursors** (`lost-work-ack.txt`, `heartbeat/last-drain-ack.txt`) are not
  per-session like the inbox (A71) — cross-session stale-watermark, self-correcting. Fix: key per-session.
- **L4 — `chamber-empty-detect.sh`** count edge-detector check-then-write (duplicate/missed transition; self-heals).
- **L5 — event-bus at-least-once**: a dispatcher SIGKILL'd between fire and cursor-write re-fires
  consumers next run (consumers are detached + mostly idempotent). Defensive only.

## Fix plan (priority order)
1. **C1 (dashboard data.json)** — serialize the 7 mergers + mkdir-lock. Highest impact; needs a test
   (assert all blocks land). The biggest change — do it carefully, don't break the dashboard.
2. **H1 (memory sync)** — ✅ DONE 2026-06-02.
3. **H2/H3/H4** — same fix family: atomic write + a lock (or append-only side-file). Build ONE reusable
   `mkdir`-lock helper (port `dispatch.py`'s) and apply at each site. Verify each.
4. **M1–M3, L1–L5** — opportunistic; M3 is a standing-rule (memory), not code.

## What's already SOUND (the reassuring half — do not "fix")
Scheduler (no deadlock), daemons, event bus, bug-billboard inbox, all PreToolUse guard-logs, the
JSONL ledgers (cot/ack/hedge/critical-finding/recurrences/…), inbox per-session keying, and every
tmp+rename full-file generator that owns its own file. The architecture is right; the cluster above is
where it wasn't followed.
