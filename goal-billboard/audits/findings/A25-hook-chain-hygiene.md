# A25 — settings.json health + hook-chain hygiene

**Type:** READ-ONLY research audit (system infrastructure verification)
**Audited:** 2026-05-28
**Scope:** `.claude/settings.json` hook wiring vs `scripts/hooks/` files; port-bug; src-edit-guard verification; health signals.
**Verdict:** Hook chain is structurally healthy. The reported "~18 orphans" is **stale/overcounted** — the real count is **5** (all intentional-standby, none dangerous-if-deleted). The reported "port bug" is **real and live** but is NOT the dashboard port — it's an agent-inbox-server `4322` vs `4323` default contradiction, with its guard sitting unwired. The reported "guard left unwired" maps to two findings: `inbox-server-guard.sh` (truly unwired) and `src-edit-guard.sh` (correctly wired — verified good).

---

## 1. Reconciliation table — wired vs file-present

**Totals:** 71 `.sh` files present in `scripts/hooks/`. 66 of them wired in settings.json. **5 orphans, 0 dangling, 0 non-executable, 0 true duplicates.**
(Plus 6 wired scripts that live in `scripts/` root, not `scripts/hooks/` — all present + executable: `sync-memory.sh`, `audit-recommendations.sh`, `compute-matrix-stats.sh`, `detect-script-candidates.sh`, `bug-billboard-consolidate.sh`, `bug-billboard-status.sh`, `goal-billboard-status.sh`, `lint-skill-staleness.sh`, `verify-memory-sync.sh`.)

| Hook script (`scripts/hooks/`) | Wired? | Event(s) | Present | Exec | Notes |
|---|---|---|---|---|---|
| ack-latency-measure.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| agent-inbox-drain-notification.sh | ✅ | Notification | ✅ | ✅ | |
| agent-inbox-drain-stop.sh | ✅ | Stop | ✅ | ✅ | co-exists w/ agent-inbox-stop-drain (see §2 note) |
| agent-inbox-read.sh | ✅ | PreToolUse `.*` | ✅ | ✅ | |
| agent-inbox-stop-drain.sh | ✅ | Stop | ✅ | ✅ | |
| audit-close-check.sh | ✅ | Stop | ✅ | ✅ | |
| audit-comparison-post-capture.sh | ✅ | PostToolUse Bash | ✅ | ✅ | |
| bg-auto-stop.sh | ✅ | Stop | ✅ | ✅ | |
| **bg-ghost-detect-dual-signal.sh** | ❌ **ORPHAN** | — | ✅ | ✅ | newer (2026-05-28) refinement of bg-stuck-warn; CLI/standby |
| bg-stuck-warn.sh | ✅ | Stop | ✅ | ✅ | |
| billboard-data-refresh.sh | ✅ | PostToolUse Write\|Edit | ✅ | ✅ | |
| **bottleneck-detect.sh** | ❌ **ORPHAN** | — | ✅ | ✅ | Immune-system centerpiece; designed for 24h dry-run BEFORE wiring (A47 Rule 5 / A58) |
| capture-staleness.sh | ✅ | SessionStart | ✅ | ✅ | |
| chamber-empty-detect.sh | ✅ | PostToolUse Bash | ✅ | ✅ | |
| cot-checkpoint-pre.sh | ✅ | PreToolUse Edit\|Write\|Bash | ✅ | ✅ | |
| cot-on-prompt.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| cot-resume.sh | ✅ | SessionStart | ✅ | ✅ | |
| cot-seed.sh | ✅ | SessionStart | ✅ | ✅ | |
| cot-staleness-warn.sh | ✅ | Stop | ✅ | ✅ | |
| critical-finding-detect.sh | ✅ | Stop | ✅ | ✅ | |
| dashboard-data-prime.sh | ✅ | SessionStart | ✅ | ✅ | |
| dispatch-metrics-log.sh | ✅ | PostToolUse Agent | ✅ | ✅ | |
| dispatch-metrics-summary.sh | ✅ | SessionStart | ✅ | ✅ | |
| established-workflows-check.sh | ✅ | Stop | ✅ | ✅ | |
| event-bus-dispatch-trigger.sh | ✅ | PostToolUse Write\|Edit | ✅ | ✅ | |
| git-checkout-safety.sh | ✅ | PreToolUse Bash | ✅ | ✅ | |
| git-drift-warn.sh | ✅ | SessionStart | ✅ | ✅ | |
| goal-queue-check.sh | ✅ | Stop | ✅ | ✅ | |
| goal-staleness-warn.sh | ✅ | Stop | ✅ | ✅ | |
| heartbeat-drain.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| **heartbeat-independent-tick.sh** | ❌ **ORPHAN** | — (launchd-bound) | ✅ | ✅ | Heart SPOF fix; meant for launchd not settings.json. Template present, **NOT installed** (see §3 + §5) |
| hedge-detect.sh | ✅ | Stop | ✅ | ✅ | |
| hedge-detect-drain.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| hooks-health-check.sh | ✅ | SessionStart | ✅ | ✅ | |
| idea-gap-surface.sh | ✅ | SessionStart | ✅ | ✅ | |
| image-cap-warn.sh | ✅ | PreToolUse Read | ✅ | ✅ | |
| **inbox-server-guard.sh** | ❌ **ORPHAN** | — | ✅ | ✅ | A45 server-death closure; **the "guard left unwired"** (see §3 + §4-port) |
| inbox-mutation-guard.sh | ✅ | PreToolUse Write\|Edit (block #2) | ✅ | ✅ | |
| inbox-on-prompt.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| inbox-schema-validate-pre.sh | ✅ | PreToolUse Write\|Edit | ✅ | ✅ | |
| lost-work-drain.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| lost-work-token-cost.sh | ✅ | PostToolUse Agent | ✅ | ✅ | |
| memory-sync-post.sh | ✅ | PostToolUse Write\|Edit | ✅ | ✅ | |
| ns-advance-check.sh | ✅ | Stop | ✅ | ✅ | |
| organic-os-pulse.sh | ✅ | SessionStart | ✅ | ✅ | checks bottleneck-detect *existence* only — does NOT exec it |
| parallel-capacity-check.sh | ✅ | Stop | ✅ | ✅ | |
| pending-parallel-work-scan.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| plan-orphan-detector.sh | ✅ | Stop | ✅ | ✅ | |
| recurrence-detect.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| research-cross-link-harvest.sh | ✅ | PostToolUse Write\|Edit | ✅ | ✅ | |
| research-folder-advise.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| research-surface-on-resume.sh | ✅ | SessionStart | ✅ | ✅ | |
| scheduler-data-refresh.sh | ✅ | PostToolUse Write\|Edit + PostToolUse Bash | ✅ | ✅ | wired twice across two matchers — legit (see §2) |
| scheduler-heartbeat-coordinator.sh | ✅ | PostToolUse Bash | ✅ | ✅ | |
| scheduler-tick-drift-warn.sh | ✅ | PreToolUse Bash | ✅ | ✅ | |
| sessionstart-services-digest.sh | ✅ | SessionStart | ✅ | ✅ | |
| settings-edit-guard.sh | ✅ | PreToolUse Write\|Edit | ✅ | ✅ | |
| sim-emu-session-teardown.sh | ✅ | Stop | ✅ | ✅ | |
| skill-cross-link-rebuild.sh | ✅ | PostToolUse Write\|Edit | ✅ | ✅ | |
| snapshot-baseline-staleness-check.sh | ✅ | PreToolUse Bash | ✅ | ✅ | |
| snapshot-drift-pre.sh | ✅ | PreToolUse Bash | ✅ | ✅ | |
| **src-edit-guard.sh** | ✅ | PreToolUse Write\|Edit | ✅ | ✅ | **VERIFIED — see §4-src** |
| state-batch-digest.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| subagent-data-refresh.sh | ✅ | PostToolUse Agent | ✅ | ✅ | |
| test-e2e-dev-server-check.sh | ✅ | PreToolUse Bash | ✅ | ✅ | references :4321 dev port (correct — see §4-port) |
| test-pipestatus-capture.sh | ✅ | PostToolUse Bash | ✅ | ✅ | |
| token-budget-check.sh | ✅ | SessionStart | ✅ | ✅ | |
| user-ask-artifact-check.sh | ✅ | Stop | ✅ | ✅ | |
| user-ask-detect.sh | ✅ | UserPromptSubmit | ✅ | ✅ | |
| worktree-remove-safety.sh | ✅ | PreToolUse Bash | ✅ | ✅ | |
| **wrapper-only-write-guard.sh** | ❌ **ORPHAN** | — | ✅ | ✅ | A57/A48/A61 shared barrier; designed for dry-run review BEFORE wiring |

---

## 2. Orphans / dangling / duplicates

### Orphans — confirmed count: **5** (reported ~18; the ~18 is stale/overcounted)
All 5 are git-committed (clean), executable, well-documented, and **intentional standby** — none is dead code:

1. **`bottleneck-detect.sh`** (19 KB, 2026-05-28 17:19) — Immune-system innate detector (A58/A74). 8-signal bottleneck monitor. Designed to run a **24h `--dry-run` window before wiring** per A47 Rule 5. `organic-os-pulse.sh:130` only checks its *existence* as a vital sign; it never execs it. → **Safe to wire** (as a Stop or SessionStart WARN-mode hook) once dry-run validated; do NOT delete.
2. **`heartbeat-independent-tick.sh`** (18 KB, 2026-05-28 17:08) — Heart's independent pulse (A74 SPOF fix). Built to be **launchd-bound, NOT a settings.json hook**. A `heartbeat-launchd.plist.template` exists at `scripts/heartbeat/` (StartInterval 600) but is **not installed** in `~/Library/LaunchAgents/`. → **Safe to wire via launchd** (not settings.json); do NOT delete. (See §5 SPOF.)
3. **`inbox-server-guard.sh`** (16 KB, 2026-05-28 17:13) — A45 server-death closure. **This is "the guard left unwired."** Detect/restart for the agent-inbox-server. Dry-run-default + kill-switch + bounded recovery — safety contract already built in. → **Safe to wire** on SessionStart (`--sessionstart` mode) AND/OR a periodic hook; high value given the live port conflict in §4. Do NOT delete.
4. **`bg-ghost-detect-dual-signal.sh`** (4 KB, 2026-05-28 05:27) — Refined dual-signal BG ghost detector (supersedes single-signal logic in `bg-stuck-warn.sh`/`bg-auto-stop.sh`). Emits JSONL; built as a **CLI/standby tool**. → **Intentional standby**; candidate to fold into `bg-auto-stop.sh` later. Safe to keep unwired.
5. **`wrapper-only-write-guard.sh`** (10 KB, 2026-05-28 19:05) — Shared hardware barrier closing A57+A48+A61 (blocks direct Write/Edit to `data.json`/inbox outside sanctioned wrappers). Designed for **dry-run review before wiring**. Complementary to the already-wired `inbox-schema-validate-pre.sh`. → **Safe to wire** (PreToolUse Write\|Edit, WARN mode) after dry-run; do NOT delete.

**Why "~18" was wrong:** likely a pre-wiring snapshot. Four post-audit hooks were wired in the last two commits (`050f77f` "wire 4 post-audit hooks", `8b115dd` "+ A40 detector") — `audit-close-check`, `sim-emu-session-teardown`, `organic-os-pulse`, `critical-finding-detect`, `idea-gap-surface` all moved from orphan→wired recently. The count has been actively shrinking; ~18 was a high-water mark before this sprint's wiring.

### Dangling — **0**
Every settings.json command resolves to a present, executable file. No missing targets.

### Non-executable — **0**
All 66 wired hook files + 9 wired root scripts are `-rwxr-xr-x`.

### Duplicates / shadowing — **0 true; 1 soft (cosmetic)**
- **Not duplicates** (different event or matcher — both fire legitimately):
  - `verify-memory-sync.sh` → wired on **Stop** AND **SessionEnd** (intentional belt-and-suspenders).
  - `scheduler-data-refresh.sh` → wired on **PostToolUse Write\|Edit** AND **PostToolUse Bash** (different matchers; both relevant).
  - `agent-inbox-drain-stop.sh` and `agent-inbox-stop-drain.sh` are **two different files** (not a typo-dupe) — both on Stop. Worth a glance for overlap, low priority.
- **SOFT duplicate (cosmetic):** `PreToolUse` has **two separate `matcher:"Write|Edit"` blocks** (settings.json L196–211 and L213–220). The first holds inbox-schema/settings-edit/src-edit guards; the second holds `inbox-mutation-guard.sh` alone. Functionally fine (all run), but they should be **one block** for readability. Evidence: `jq '.hooks.PreToolUse[] | select(.matcher=="Write|Edit")'` returns 2 objects. → **Recommend merge** (cosmetic, low risk).

---

## 3. Port-bug assessment — **REAL & LIVE, but not the dashboard**

The reported "port bug" is **not** about the dashboard. `serve-dashboard.sh` uses `:8787` consistently (L25 `PORT=8787`, all echo/exec lines reference `$PORT`), and it is **not a hook** — nothing in settings.json references 8787. Dashboard port is clean. ✅

The real port problem is the **agent-inbox-server `4322` vs `4323` contradiction**:

| Source | Port | Evidence |
|---|---|---|
| **Canonical registry** (`required-services.json`) | **4323** | `context/markdowns/required-services.json:7` — `"port": 4323`, health URL `:4323`, start cmd `AGENT_INBOX_PORT=4323` |
| `wake-restart.sh` (recovery) | **4323** | `scripts/wake-restart.sh:37-38` starts on 4323; L27 kills stale 4322/4323; L10 calls 4322 "legacy port from pre-A45 migration" |
| `inbox-server-guard.sh` defaults | **4323** | `scripts/hooks/inbox-server-guard.sh:143-146` (matches registry) |
| **`agent-inbox-server.py` DEFAULT** | **4322** ⚠️ | `scripts/agent-inbox-server.py:46` — `PORT = int(os.environ.get("AGENT_INBOX_PORT", "4322"))`; docstring L11 `default 4322` |

**Live runtime state (probed 2026-05-28):**
- `:4323` → **Python agent-inbox-server (PID 9835) LISTEN** ✅ — the real server, correctly on canonical port (started with the explicit `AGENT_INBOX_PORT=4323` env).
- `:4322` → **node (PID 63627) LISTEN** ⚠️ — a node/Astro-class process squatting the inbox-server's *default* port. This is **exactly the A45 incident pattern** (port squatted by node → chamber clicks 404 → silently lost).

**The bug:** `agent-inbox-server.py`'s hardcoded default (`4322`) contradicts the canonical port (`4323`) that every other component agreed on after the A45 migration. Today it's masked because the server is launched with the explicit env var. But **any code path that launches the server WITHOUT `AGENT_INBOX_PORT=4323`** (a bare `python3 scripts/agent-inbox-server.py`, a forgotten env in a new wrapper, a fresh-machine setup) lands on `4322` — which is **currently occupied by a squatter** — reproducing the silent-click-loss class.

**Compounding:** the guard built specifically to detect/recover this exact failure — `inbox-server-guard.sh` — is the **unwired orphan #3**. The defense exists but isn't deployed. `sessionstart-services-digest.sh` IS wired and would *report* a down inbox-server, but it's read-only (never restarts) and only checks the registry port 4323 (it would not notice the 4322 squatter as a problem because the real server is up on 4323).

- **What:** `agent-inbox-server.py` default port (4322) contradicts canonical (4323); 4322 is live-squatted by node.
- **Evidence:** `scripts/agent-inbox-server.py:46`; `context/markdowns/required-services.json:7`; `lsof -iTCP:4322` → node PID 63627, `:4323` → Python PID 9835.
- **Risk:** MEDIUM-HIGH. Latent. Any env-less server launch → collision on 4322 → 404 → silent monkey-chamber click loss (the named A45 fear). Defense (`inbox-server-guard.sh`) is unwired.
- **Recommendation (PROPOSE — `agent-inbox-server.py` is `src/`-adjacent operational code; NOT applied here):**
  1. Change `agent-inbox-server.py:46` default from `4322` → `4323` so the default agrees with the registry (removes the contradiction entirely). One-line change; **gated** (operational script edit).
  2. Wire `inbox-server-guard.sh --sessionstart` into SessionStart (after `sessionstart-services-digest.sh`) so the guard that defends this class actually runs. **Gated** (settings.json deploy, A66).
  3. Optionally investigate/clear the node squatter on 4322 (PID 63627) — out of audit scope; surface to user.

---

## 4. `src-edit-guard.sh` verification — **PASS** ✅

- **Wired correctly:** PreToolUse, matcher `"Write|Edit"` (settings.json L196–209, 3rd command in the block). Matcher is right for catching src edits via either tool. ✅
- **Ordering:** runs *after* `settings-edit-guard.sh` in the same block (`inbox-schema-validate-pre` → `settings-edit-guard` → `src-edit-guard`). Sensible — settings.json protection first, then src. ✅
- **Executable:** `-rwxr-xr-x`, shebang `#!/usr/bin/env bash`. ✅
- **WARN-mode logic — sound:** (file `scripts/hooks/src-edit-guard.sh`)
  - Mode defaults to `WARN` (L34: `MODE="${EVIUM_SRC_EDIT_GUARD_MODE:-WARN}"`) — correct for the 24h dry-run window per A47 Rule 5. ✅
  - Whitelist override `EVIUM_SRC_EDIT_OK=1` → early `exit 0` (L37-39) — per-action override, matches decision-handling discipline. ✅
  - Match patterns `*/EviumOverhaul/src/*` AND `src/*` (L46-47) — catches both absolute and relative paths. ✅
  - Every detection logged to `.claude/cache/src-edit-guard.jsonl` (L50) in both modes — good for dry-run review. ✅
  - WARN: logs + surfaces to stderr + `exit 0` (allows write). BLOCK: `exit 2` + actionable recovery message (L52-70). Correct exit-code semantics. ✅
- **Minor gaps (non-blocking, PROPOSE):**
  - **Pattern repo-name coupling:** `*/EviumOverhaul/src/*` is hardcoded to the repo folder name. A worktree under `.claude/worktrees/<name>/src/...` would NOT match the absolute pattern (it has no `/EviumOverhaul/` segment) — though the relative `src/*` arm would still catch a relative path. Low risk for normal use; note for completeness.
  - **No JSON-parse guard if input missing `file_path`:** `FILE` becomes `""` and falls through to `exit 0` — safe default (fail-open), acceptable for WARN. ✅
  - **Recommendation:** keep in WARN for the 24h window, review the JSONL, then flip to BLOCK via `EVIUM_SRC_EDIT_GUARD_MODE=BLOCK` (matches the in-file comment L33). No code change needed.

**Verdict:** the recent wiring is correct. No action required beyond the routine WARN→BLOCK promotion after dry-run review.

---

## 5. Health signals (slow / probe-fail / not-exec)

From `hooks-health-check.sh` (SessionStart, threshold `HEALTH_SLOW_MS=2000`) + `hooks-perf-baseline-20260526.json` (3-iter timing, measured 2026-05-26T23:47Z):

- **Not-exec / probe-fail:** none. All wired scripts present + executable; no `MISSING`/`NOT_EXEC`.
- **SLOW (>2000ms) — 2 SessionStart hooks blow the budget:**
  - 🐢 **`detect-script-candidates.sh` — p95 15,618 ms / max 15,618 ms** (slowest of all 33; called as `detect-script-candidates.sh --threshold 3 --sessions 5` on SessionStart). Nearly **8× the 2000ms warn threshold.** This single hook dominates SessionStart latency. → **Bottleneck candidate** (this is exactly what the unwired `bottleneck-detect.sh` signal #6 "hook latency >5s" is built to flag).
  - 🐢 **`goal-billboard-status.sh` — p95 7,328 ms** (SessionStart). ~3.6× threshold.
  - ⚠️ `goal-queue-check.sh` — p95 1,120 ms (Stop) — under threshold but the slowest Stop hook; watch.
  - All others < 700 ms (most < 200 ms). SessionStart wall budget is being eaten almost entirely by the two above (~23s combined p95).
- **Caveat:** the baseline is from **2026-05-26**; several hooks have changed since (e.g. `critical-finding-detect.sh`, `idea-gap-surface.sh`, `state-batch-digest.sh`, `inbox-on-prompt.sh` all edited 2026-05-28 and are NOT in the baseline). A fresh `hooks-perf-baseline.sh` run is warranted — the current SessionStart chain has grown to ~20 hooks.

- **What:** Two SessionStart hooks (detect-script-candidates 15.6s, goal-billboard-status 7.3s) exceed the 2000ms warn threshold by 8× and 3.6×.
- **Evidence:** `.claude/cache/hooks-perf-baseline-20260526.json` (summary: `slowest_ms:15618, slowest_script:detect-script-candidates.sh`).
- **Risk:** MEDIUM. Pure latency tax on every session start (~23s of the chain). Not a correctness bug. Squarely the "testing/infra must never become the bottleneck" class (bottleneck-restriction chant).
- **Recommendation (PROPOSE):** (a) re-run `hooks-perf-baseline.sh` to get a current-chain measurement; (b) profile `detect-script-candidates.sh` (likely a full session-JSONL scan — consider caching/`--sessions` tightening or moving it off the critical SessionStart path to a daemon); (c) this is itself a strong argument for **wiring `bottleneck-detect.sh`** (orphan #1), whose signals #6/#7 target exactly this.

---

## Ranked cleanup recommendations (ALL PROPOSE — settings.json + hook deploys are A66 high-risk-gated; none applied)

| # | Priority | Action | Type | Risk | Why |
|---|---|---|---|---|---|
| 1 | **HIGH** | Fix `agent-inbox-server.py:46` default `4322`→`4323` | src-adjacent edit (gated) | low change / high payoff | Removes the live port contradiction; kills the silent-click-loss class at root |
| 2 | **HIGH** | Wire `inbox-server-guard.sh --sessionstart` into SessionStart | settings.json (gated) | low | Deploys the built-but-dormant defense for the A45 class; pairs with #1 |
| 3 | **MED** | Re-run `hooks-perf-baseline.sh`; profile/relocate `detect-script-candidates.sh` (15.6s) + `goal-billboard-status.sh` (7.3s) | perf/script | low | ~23s SessionStart tax; bottleneck-chant class |
| 4 | **MED** | Wire `bottleneck-detect.sh` (WARN) after its 24h dry-run | settings.json (gated) | low (WARN) | Immune centerpiece; would auto-flag finding #3's slow hooks; A58/A74 intent |
| 5 | **LOW** | Wire `wrapper-only-write-guard.sh` (WARN) after dry-run | settings.json (gated) | low (WARN) | Closes A57/A48/A61 direct-write class; complements wired inbox-schema guard |
| 6 | **LOW** | Install `heartbeat-independent-tick.sh` via launchd (NOT settings.json) | launchd (gated) | low | Fixes the Heart SPOF (see below); template at `scripts/heartbeat/` ready, just uninstalled |
| 7 | **LOW** | Merge the two `PreToolUse matcher:"Write|Edit"` blocks into one | settings.json (gated) | trivial/cosmetic | Readability only; both fire today |
| 8 | **LOW** | Promote `src-edit-guard.sh` WARN→BLOCK after 24h JSONL review | env flag | low | Routine; no code change (`EVIUM_SRC_EDIT_GUARD_MODE=BLOCK`) |
| 9 | **DEFER** | Fold `bg-ghost-detect-dual-signal.sh` into `bg-auto-stop.sh` | refactor | low | Intentional standby; keep as CLI for now |

### Orphan disposition summary
- **Safe to WIRE (after dry-run where noted):** `inbox-server-guard.sh` (#2), `bottleneck-detect.sh` (#4), `wrapper-only-write-guard.sh` (#5).
- **Safe to wire via LAUNCHD (not settings.json):** `heartbeat-independent-tick.sh` (#6).
- **Intentional standby — keep unwired:** `bg-ghost-detect-dual-signal.sh` (CLI tool; fold-in candidate).
- **Safe to DELETE:** **none.** All 5 are committed, documented, and serve a designed purpose.

### Bonus finding — Autonomic SPOF (corroborates A74)
`launchctl list | grep evium` shows **only `com.evium.daemon10` loaded**. Daemons 2/3/4/5 have plists in `~/Library/LaunchAgents/` but are **NOT loaded** (memory-consolidator, audit-catalog-scan, steering-log-compact, bug-billboard-groom — all dark). This matches the A74 "Heart + 4 event-bus daemons flatlined together; daemon10 sole survivor (own launchd cadence)" finding and is precisely why `heartbeat-independent-tick.sh` was written. → **Surface to user:** 4 autonomic daemons are installed-but-unloaded; `launchctl load` (gated) would revive them, and rec #6 (independent heartbeat) would prevent silent cohort-death recurrence.

---

BG-COMPLETE-SENTINEL
