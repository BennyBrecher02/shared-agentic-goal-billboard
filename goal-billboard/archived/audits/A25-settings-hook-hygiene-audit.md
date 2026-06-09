---
id: A25
title: settings.json health + hook-chain hygiene (re-run — current-state, 2026-05-29)
type: audit
status: completed
mode: read-only
created: 2026-05-29
audited_by: subagent (BG dispatch, read-only)
scope: settings.json hook wiring vs scripts/hooks on disk; orphan entries + orphan scripts; structural permission diff-guard wiring state; the "port bug"; hook-chain ordering + per-hook perf budget (per agentic-script-design hook-chain-composition)
feeds: manifesto §7
supersedes_snapshot: context/markdowns/goal-billboard/audits/findings/A25-hook-chain-hygiene.md (2026-05-28 parallel run — reconciled + updated below)
prior_doc: context/markdowns/goal-billboard/audits/A25-settings-json-health-and-hook-chain-hygiene.md (2026-05-26 — describes a PRE-WIRING state that no longer holds)
cross_refs:
  - .claude/settings.json
  - .claude/skills/agentic-script-design/references/hook-chain-composition.md
  - context/markdowns/plans/build-but-surface-settings-wiring-proposals.md
  - context/markdowns/required-services.json
  - scripts/agent-inbox-server.py
  - scripts/hooks/settings-edit-guard.sh
  - scripts/hooks/inbox-server-guard.sh
  - context/markdowns/goal-billboard/audits/A85-git-rollback-safety-audit.md (F5 — tracked settings.json + Cursor blind-spot)
findings:
  - {id: F1, sev: HIGH,   tag: GATED,   what: "agent-inbox-server.py default port 4322 contradicts canonical 4323; :4322 live-squatted → A45 silent-click-loss class still latent"}
  - {id: F2, sev: MEDIUM, tag: GATED,   what: "the guard built to defend F1's class (inbox-server-guard.sh) is unwired — defense exists, not deployed"}
  - {id: F3, sev: MEDIUM, tag: GATED,   what: "goal-billboard-status.sh ~6.9s on SessionStart — alone exceeds the 5s SessionStart budget (1.4×)"}
  - {id: F4, sev: LOW,    tag: MIXED,   what: "9 orphan scripts on disk wired to nothing (up from 5 on 2026-05-28); all intentional-standby or awaiting-gated-wire — none dead/deletable"}
  - {id: F5, sev: LOW,    tag: GATED,   what: "PreToolUse Bash chain order violates hook-chain Rule 1 — hard-reject safety guards run AFTER advisory/enrichment hooks"}
  - {id: F6, sev: LOW,    tag: GATED,   what: "two separate PreToolUse matcher:\"Write|Edit\" blocks — cosmetic; merge for readability"}
  - {id: F7, sev: LOW,    tag: UNGATED, what: "scripts/hooks/settings-patches/ holds 22 JSON patches, ≥5 sampled already wired → stale leftover cruft"}
  - {id: F8, sev: INFO,   tag: NONE,    what: "settings-edit-guard.sh IS wired (PreToolUse Write|Edit, WARN mode) — the parallel-candidates 'guard unwired' flag is STALE"}
  - {id: F9, sev: LOW,    tag: UNGATED, what: "run-deep-audit.sh wired but not +x (advisory only — invoked via 'bash <path>'); its :4399 probe is correct, hook's :4321 is the canonical contradiction (see F1 note)"}
---

# A25 — settings.json health + hook-chain hygiene (re-run, current-state)

**Premise (parallel-candidates hook flag):** *"guard unwired, port bug, 18 orphans."* This audit re-checks all three against the **live 2026-05-29 state** (the original A25 doc is a 2026-05-26 pre-wiring snapshot; a 2026-05-28 findings file already partially reconciled it). **Read-only — nothing applied; settings.json never touched.**

## Bottom line — the three flagged items, re-verified

| Flagged | Verdict NOW | Detail |
|---|---|---|
| **"guard unwired"** | **STALE — guard IS wired** | `settings-edit-guard.sh` is wired at `.claude/settings.json:204` (PreToolUse `Write\|Edit`), running in **WARN mode** (dry-run) by design. The TRUE unwired guard is a *different* one: **`inbox-server-guard.sh`** (F2) — the A45 server-death defense — which is still an orphan. So the flag points at the wrong script but a real unwired guard does exist. |
| **"port bug"** | **REAL & LIVE** | Not the dashboard (`serve-dashboard.sh` :8787 is clean, not a hook). The bug is `agent-inbox-server.py` defaulting to **:4322** while the canonical port (registry + every other component) is **:4323** — and **:4322 is live-squatted by node right now** (PID 63627). This is the A45 silent-monkey-click-loss class, still latent. **F1.** |
| **"18 orphans"** | **OVERCOUNTED — actual = 9** | 0 orphan *entries* (every wired path resolves to a present file). 9 orphan *scripts* (on disk, wired to nothing). The "18" was a 2026-05-26 high-water mark before the wiring sprint; the 2026-05-28 run measured 5; it's back up to **9** because 4 new build-but-surface scripts landed since. All 9 are intentional-standby or awaiting-a-gated-wire — **none is dead/deletable. F4.** |

Net: **structurally healthy chain, no broken wires, no dead code.** Two items carry real risk into the next session: F1 (latent port collision) and F2 (its defense undeployed). Everything else is hygiene/latency/cosmetic.

---

## (1) WIRING INTEGRITY — every referenced path resolves; 0 orphan entries

**Method:** extracted all `scripts/(hooks/)?*.sh` command paths from `.claude/settings.json` (unique basenames), cross-checked existence + exec against disk; reverse-checked every on-disk `scripts/hooks/*.sh` against the wired set.

- **On-disk hook scripts:** 75 (`scripts/hooks/*.sh`).
- **Wired hook scripts (from `scripts/hooks/`):** 66.
- **Orphan ENTRIES (wired → missing file):** **0.** Every command in settings.json resolves to a present file. No renamed/deleted-target breakage.
- **Orphan SCRIPTS (on disk → wired to nothing):** **9** (F4 below).
- **Non-executable wired paths:** **1** — `scripts/run-deep-audit.sh` is `-rw-r--r--` (F9). Advisory only: every hook is invoked as `bash <path>`, so the missing `+x` bit does not break execution. (Also wired from root, all present + exec: `sync-memory.sh`, `audit-recommendations.sh`, `compute-matrix-stats.sh`, `detect-script-candidates.sh`, `bug-billboard-consolidate.sh`, `bug-billboard-status.sh`, `goal-billboard-status.sh`, `lint-skill-staleness.sh`, `verify-memory-sync.sh`, `record-matrix-timing.sh`, `ios-simulator-capture.sh`, `android-emulator-capture.sh`, `mobile-sim-sweep-orchestrator.sh`.)

**77** `type:command` entries total across 6 surfaces (SessionStart 18, UserPromptSubmit 11, PreToolUse 12 across 6 blocks, PostToolUse 15 across 3 blocks, Stop 18, SessionEnd 1, Notification 1). The chain has roughly doubled since the hook-chain-composition ref's "~30 hooks" baseline — the perf budget (§2) matters more than ever.

---

## (2) FINDINGS

### F1 — HIGH — `agent-inbox-server.py` default port (4322) contradicts canonical (4323); :4322 live-squatted  **[GATED — operational script edit]**

The inbox-server's hardcoded default port disagrees with the port every other component settled on after the A45 migration:

| Source | Port | Evidence |
|---|---|---|
| Canonical registry | **4323** | `context/markdowns/required-services.json:7-9` — `"port": 4323`, health `:4323`, start `AGENT_INBOX_PORT=4323 …` |
| `wake-restart.sh` recovery | **4323** | `scripts/wake-restart.sh:37-38` (starts 4323); `:10` calls 4322 *"legacy port from pre-A45 migration"* |
| `inbox-server-guard.sh` | **4323** | `scripts/hooks/inbox-server-guard.sh:144,146` |
| **`agent-inbox-server.py` DEFAULT** | **4322** ⚠ | `scripts/agent-inbox-server.py:46` — `PORT = int(os.environ.get("AGENT_INBOX_PORT", "4322"))`; docstring `:11` `default 4322` |

**Live runtime (probed 2026-05-29):**
- `:4323` → **Python agent-inbox-server PID 9835 LISTEN** (the real server, on the canonical port via explicit env).
- `:4322` → **node PID 63627 LISTEN** — a node/Astro-class process squatting the inbox-server's *default* port. **This is the exact A45 incident shape** (port squatted → POST 404 → monkey-chamber click silently lost).

**The bug:** today it's masked because the server is launched with `AGENT_INBOX_PORT=4323`. But any path that starts it WITHOUT that env — a bare `python3 scripts/agent-inbox-server.py`, a forgotten env in a new wrapper, a fresh-machine setup — binds **4322**, which is **currently occupied**, reproducing silent-click-loss.
**Severity:** HIGH (latent, but the failure mode is exactly the named A45 fear, and its defense is undeployed — F2).
**Fix (GATED — `agent-inbox-server.py` is operational/`src`-adjacent):** change line 46 default `4322` → `4323` (and docstring `:11`) so the default agrees with the registry. One-line change; removes the contradiction at root. Separately surface the node squatter on :4322 (PID 63627) to the user for clearing — out of audit scope.

### F2 — MEDIUM — `inbox-server-guard.sh` is unwired: the defense for F1's class exists but isn't deployed  **[GATED — settings.json wire]**

`scripts/hooks/inbox-server-guard.sh` (A45 server-death closure) detects a down/squatted inbox-server and can auto-restart it on the canonical port — the precise defense for F1. It is an **orphan** (wired to nothing; see F4). The already-wired `sessionstart-services-digest.sh` *reports* a down service but is read-only (never restarts) and checks only :4323 — it would not flag the :4322 squatter because the real server is up on :4323.
**Severity:** MEDIUM — built, tested, has its own dry-run + kill-switch + bounded-recovery contract; just not connected.
**Fix (GATED — settings.json):** wire `inbox-server-guard.sh --sessionstart` into `hooks.SessionStart[]` *after* `sessionstart-services-digest.sh`. Pairs with F1 (F1 removes the contradiction; F2 deploys the live-recovery net).

### F3 — MEDIUM — `goal-billboard-status.sh` alone busts the SessionStart perf budget  **[GATED to relocate; UNGATED to profile]**

Fresh timing (3 samples, this run): **6876 / 6890 / 7110 ms.** The hook-chain-composition ref sets the **SessionStart budget at 5s total** — this single always-on hook is **~1.4× the entire surface budget**, before the other 17 SessionStart hooks run.
- `detect-script-candidates.sh --threshold 3 --sessions 5` measured **3065 ms** on a real (non-short-circuit) run — over the 2000ms warn threshold but far below the stale 15.6s the 2026-05-28 baseline reported (it now caches/dedups; first probe short-circuited at 16ms). Still a watch item, not the headline.
- All other SessionStart hooks sampled <200ms (`compute-matrix-stats` 67ms, `dashboard-data-prime` 134ms, `organic-os-pulse` 79ms).
**Severity:** MEDIUM — pure latency tax on every session open (not a correctness bug), but squarely the "infra must never become the bottleneck" class (bottleneck-restriction chant, A58).
**Fix:** (a) **UNGATED** — profile `goal-billboard-status.sh` (likely a full-billboard + YAML parse on every line; cache the rendered status with a cache-timestamp gate per hook-chain-composition "lazy-fire" §, recompute at most every N min). (b) **GATED** — if it can't be made cheap, relocate it off the critical SessionStart path (a daemon, or Stop-time). (c) note: this is exactly the signal `bottleneck-detect.sh` (orphan, F4) signal #6 "hook latency" is built to auto-flag.

### F4 — LOW — 9 orphan scripts (up from 5); all intentional-standby or awaiting-gated-wire, none deletable  **[MIXED]**

On disk under `scripts/hooks/`, wired to nothing:

| Orphan script | mtime | Disposition | Why unwired |
|---|---|---|---|
| `inbox-server-guard.sh` | 05-28 17:13 | **WIRE (gated)** — see **F2** | A45 server-death defense; high value given F1 |
| `bottleneck-detect.sh` | 05-28 17:19 | WIRE (gated, WARN, after 24h dry-run) | Immune-system centerpiece (A58/A74); would auto-flag F3 |
| `wrapper-only-write-guard.sh` | 05-28 19:05 | WIRE (gated, WARN, after dry-run) | A57/A48/A61 shared barrier; per its own header "highest-leverage move in the retro-sweep" |
| `heartbeat-independent-tick.sh` | 05-28 17:08 | WIRE via **launchd** (NOT settings.json) | Heart SPOF fix (A74); `scripts/heartbeat/heartbeat-launchd.plist.template` exists but is not installed |
| `bg-ghost-detect-dual-signal.sh` | 05-28 05:27 | **KEEP unwired** (CLI/standby) | Refined dual-signal detector; fold-into-`bg-auto-stop.sh` candidate |
| `memory-bloat-guard.sh` | **05-29 05:09** | WIRE (gated, advisory exit-0) | NEW. Write-time half of the MEMORY.md bloat guard; its own header says "GATED — user applies" |
| `domino-check.sh` | **05-29 05:01** | WIRE (gated) | NEW. P2 #3 of build-but-surface-wiring-proposals (impact-domino trigger); built-awaiting-wire by design |
| `convention-f-closure.sh` | **05-29 05:09** | WIRE (gated, Stop) | NEW. P2 #4 of the same plan (capability-closure tether, A83); built-awaiting-wire by design |
| `hook-command-lint.sh` | **05-29 19:09** | WIRE (gated, PreToolUse Write\|Edit) | NEW (hours old). §5 backstop of `hooks-vs-gating-split.md` — lints hook `command` strings (a wired hook bypasses the Bash deny-list, so this matters once hooks{} is agent-editable) |

**Why the count rose 5 → 9:** four build-but-surface scripts landed on **2026-05-29** (`domino-check`, `convention-f-closure`, `memory-bloat-guard`, `hook-command-lint`) — the P2 deliverables of `build-but-surface-settings-wiring-proposals.md` plus the memory-bloat-guard. They are *correctly* built-then-parked: settings.json wiring is A66-gated, so the agent builds the script and surfaces the wire as a GATED proposal rather than self-applying. The orphan count rising is the *expected* signature of that discipline, not rot.
**Severity:** LOW — all 9 are git-committed, executable, documented, purposeful. **None is safe-to-delete.** Each is either a wire-when-gated item (7) or an intentional CLI/standby tool (1) or a launchd-bound script (1).
**Fix:** the 4 P1/P2 wires (`inbox-server-guard`, `bottleneck-detect`, `wrapper-only-write-guard` WARN, + the 4 new ones) are **GATED** settings.json edits — apply via the user or `EVIUM_SETTINGS_EDIT_OK=1` per the wiring-proposals plan. `heartbeat-independent-tick` is a **GATED** launchd install. `bg-ghost-detect-dual-signal` stays unwired (**UNGATED** future refactor: fold into `bg-auto-stop.sh`).

### F5 — LOW — PreToolUse Bash chain order violates hook-chain Rule 1 (cheapest/hard-reject guards first)  **[GATED — settings.json reorder]**

`.claude/settings.json` PreToolUse `Bash` block (lines 138-165), firing order:
```
1. test-e2e-dev-server-check   (advisory; curl probe)        :143
2. snapshot-drift-pre          (enrichment)                  :147
3. worktree-remove-safety      (HARD-REJECT safety guard)    :151
4. scheduler-tick-drift-warn   (advisory)                    :155
5. git-checkout-safety         (HARD-REJECT safety guard)    :159
6. snapshot-baseline-staleness (advisory, env-gated)         :163
```
Per `hook-chain-composition.md` **Rule 1 ("Cheapest hooks first" / hard-reject guards lead so the chain short-circuits)**, the two destructive-op guards (`worktree-remove-safety`, `git-checkout-safety`) should run **before** the advisory/enrichment hooks (`test-e2e-dev-server-check`, `snapshot-drift-pre`). As-is, a `git worktree remove ../EviumOverhaul` that the safety hook will reject still pays the curl-probe + snapshot-drift cost first. Minor (advisory hooks are cheap), but it's a textbook Rule-1 inversion the ref calls out by name.
**Severity:** LOW — correctness is fine (all hooks run; the guard still rejects). Pure ordering hygiene.
**Fix (GATED):** reorder the Bash block to `worktree-remove-safety, git-checkout-safety, scheduler-tick-drift-warn, test-e2e-dev-server-check, snapshot-drift-pre, snapshot-baseline-staleness`. Same minor note in the first Write|Edit block: `settings-edit-guard` (hard-reject) sits after `inbox-schema-validate-pre` — lead with the guard.

### F6 — LOW — two separate PreToolUse `matcher:"Write|Edit"` blocks  **[GATED — settings.json merge]**

`jq '[.hooks.PreToolUse[] | select(.matcher=="Write|Edit")] | length'` → **2**. Block A (lines 196-210): `inbox-schema-validate-pre` → `settings-edit-guard` → `src-edit-guard`. Block B (lines 213-219): `inbox-mutation-guard` alone. Functionally identical to one block (all four fire on every Write/Edit), but two blocks for the same matcher is needless duplication.
**Severity:** LOW — cosmetic/readability.
**Fix (GATED):** merge into a single `Write|Edit` block. (Carries the F5 reorder note: put the hard-reject `settings-edit-guard` / `src-edit-guard` ahead of the advisory schema/mutation validators.)

### F7 — LOW — `scripts/hooks/settings-patches/` is stale leftover cruft  **[UNGATED — directory cleanup]**

The directory holds **22 JSON patch files** (one per pending hook-wire). Sampling shows ≥5 are **already wired** into settings.json and the patch is dead leftover: `ack-latency.json`→`ack-latency-measure.sh` ✅wired, `heartbeat-drain.json` ✅wired, `ns-advance-check.json` ✅wired, `idea-gap-surface.json` ✅wired, `critical-finding-detect.json` ✅wired. The dir is a graveyard of applied patches mixed (possibly) with a few still-pending ones, with no manifest of which is which.
**Severity:** LOW — confusing/misleading (the old A25 doc cited "9-22 parked patches" as a live bottleneck; that framing is now stale), but harmless — nothing reads this dir at runtime.
**Fix (UNGATED — this is a `context`/`scripts` housekeeping op, not settings.json):** audit each patch against the live settings.json; delete the already-applied ones; keep only genuinely-pending patches (with a short README noting which hook each wires). Candidate for a one-shot cleanup script. *Applied nothing this run.*

### F8 — INFO — the structural permission diff-guard IS wired (the "guard unwired" flag is stale)  **[no action]**

`settings-edit-guard.sh` is wired at `.claude/settings.json:204` under PreToolUse `Write|Edit`. It is the **upgraded structural diff-guard** (not the old blanket block): it deep-diffs `.permissions` (jq -S) on-disk vs proposed and ALLOWS hooks{}-only edits while it would-BLOCK any `.permissions` change. It runs in **WARN mode** by default (`EVIUM_SETTINGS_GUARD_MODE:-WARN`, `settings-edit-guard.sh:57`) — the A47-Rule-5 24h dry-run window; flip to BLOCK is itself a gated decision. Fails closed on unparseable/missing perms; `settings.local.json` is fully blocked. **This matches A85's finding (F5/§2e) verbatim — wired, WARN-mode, sound design.** The parallel-candidates "guard unwired" flag is reading a stale state (it was true on 2026-05-26).
**Recommendation (carry-over, not new):** flip WARN→BLOCK after the dry-run JSONL (`.claude/cache/settings-edit-guard.jsonl`) is reviewed for false-positives on real hook edits. Gated env-flag, no code change. (Same WARN→BLOCK promotion applies to `src-edit-guard.sh`, also wired+WARN at :208.)

### F9 — LOW — `run-deep-audit.sh` not executable; its :4399 is correct (the hook's :4321 is the contradiction)  **[UNGATED chmod]**

`scripts/run-deep-audit.sh` is wired-reachable but `-rw-r--r--` (no `+x`). Advisory only — invoked via `bash <path>`, so it still runs. Separately worth noting for the port story: `run-deep-audit.sh:114`, `ios-simulator-capture.sh:31`, `android-emulator-capture.sh:43` and **`playwright.config.ts:29,35,36`** all use **:4399** — and that IS the project's real test/capture dev port (`webServer.command: 'npm run dev -- --port 4399'`). The `test-e2e-dev-server-check.sh` hook (`:37`) probes **:4321** (bare `astro dev` default). So when `npm run test:e2e` runs, Playwright serves on :4399 but the PreToolUse hook checks :4321 → it can mis-warn "dev server not responding" even when the test harness is correctly up on :4399. This is the *inverse* of the original A25 finding (which had the hook on 4399, Astro on 4321) — the earlier fix moved the hook to 4321, but the actual e2e port is 4399. **Not promoted to a higher severity** because the hook is advisory (exit 0, never blocks) and the test:e2e flow brings its own webServer, but the 4321/4399 split is a real inconsistency worth resolving.
**Severity:** LOW.
**Fix:** (UNGATED) `chmod +x scripts/run-deep-audit.sh` (cosmetic). (UNGATED, script-only) align `test-e2e-dev-server-check.sh` probe to **:4399** to match `playwright.config.ts` (or make it read the port from one source) — the hook should check the port the e2e harness actually uses.

---

## (3) HOOK-CHAIN ORDERING + PER-HOOK PERF BUDGET (per hook-chain-composition.md)

| Surface | Budget (ref) | Entries now | Assessment |
|---|---:|---:|---|
| SessionStart | 5s total | 18 | **OVER (F3):** `goal-billboard-status` ~6.9s alone busts the whole-surface budget; `detect-script-candidates` ~3.1s over the 2s/hook watch line. Rest <200ms. |
| UserPromptSubmit | 3s total | 11 | Likely OK — all 10 conditional hooks are env-gated/lazy (`[ -z "$EVIUM_*_OFF" ] && …`); only `inbox-on-prompt` is unconditional. Not freshly timed this run; **re-baseline recommended** (the ref's last UserPromptSubmit measurement is from 2026-05-26 at 5 hooks; it's now 11). |
| PreToolUse | 500ms/hook | 12 / 6 blocks | Ordering finding **F5** (hard-reject guards not leading) + cosmetic **F6** (split Write\|Edit blocks). The `.*`-matcher `agent-inbox-read.sh` (`:191`) fires on EVERY tool call — confirmed-needed (drains mid-response inbox arrivals) but the per-call hot path; keep it cheap. |
| PostToolUse | 1s | 15 / 3 blocks | `scheduler-data-refresh.sh` wired twice (Write\|Edit `:240` AND Bash `:282`) — **legit** (different matchers, both change scheduler state), not a dup. |
| Stop | 2s | 18 | Longest Stop chain in the project's history; all but a few are env-gated/lazy. `agent-inbox-drain-stop.sh` (`:301`) and `agent-inbox-stop-drain.sh` (`:325`) are **two different files** both on Stop — worth a glance for overlap (LOW), not a typo-dup. `verify-memory-sync.sh` on Stop AND SessionEnd is intentional belt-and-suspenders. |
| Notification | — | 1 | clean |

**Rule-by-rule (hook-chain-composition):**
- **Rule 1 (cheapest/hard-reject first):** VIOLATED in PreToolUse Bash (F5). Elsewhere acceptable.
- **Rule 2 (conditional hooks last):** Largely followed — the env-gated `*_OFF` conditionals trail the unconditional hooks on UserPromptSubmit and Stop.
- **Rule 3 (expensive last within category):** VIOLATED on SessionStart — the ~6.9s `goal-billboard-status` is wired mid-chain (`:49`), not last; the expensive always-on hook should trail so cheaper context lands first (and F3 says it shouldn't be on this surface at all).
- **Cascade (producer-before-consumer):** no inversions found; the documented producer/consumer pairs (`state-batch-digest`→`recurrence-detect`, etc.) are in producer-first order.
- **Lazy-fire:** widely + correctly applied (the `[ -z "$EVIUM_*_OFF" ]` env-gate + the `2>/dev/null` file-existence gates inside the heavy hooks). This is the main thing keeping the doubled chain inside budget.

---

## Findings ledger (feeds manifesto §7)

| ID | Sev | Tag | Finding | Fix |
|---|---|---|---|---|
| F1 | HIGH | **GATED [src-adjacent]** | `agent-inbox-server.py:46` default 4322 ≠ canonical 4323; :4322 live-squatted (node PID 63627) → latent A45 silent-click-loss | edit default 4322→4323 (+docstring :11); surface squatter to user |
| F2 | MED | **GATED [settings.json]** | `inbox-server-guard.sh` (F1's defense) unwired | wire `--sessionstart` into SessionStart after services-digest |
| F3 | MED | GATED to relocate / **UNGATED to profile** | `goal-billboard-status.sh` ~6.9s busts 5s SessionStart budget | cache/lazy-gate it (ungated); else relocate off SessionStart (gated) |
| F4 | LOW | **MIXED** | 9 orphan scripts (5→9 since 05-28); all standby/awaiting-gated-wire, none deletable | 7 = gated wires (per wiring-proposals plan); 1 launchd; 1 keep-as-CLI |
| F5 | LOW | **GATED [settings.json]** | PreToolUse Bash order breaks Rule 1 (hard-reject guards after advisory hooks) | reorder guards to lead the block |
| F6 | LOW | **GATED [settings.json]** | two `Write\|Edit` PreToolUse blocks | merge into one |
| F7 | LOW | **UNGATED [dir cleanup]** | `settings-patches/` = 22 JSONs, ≥5 already-wired (stale) | delete applied patches; README the rest; candidate one-shot script |
| F8 | INFO | none | `settings-edit-guard.sh` IS wired (WARN) — "guard unwired" flag is stale | (carry-over) WARN→BLOCK after dry-run review |
| F9 | LOW | **UNGATED [script-only]** | `run-deep-audit.sh` not +x (advisory); hook probes :4321 but e2e port is :4399 | `chmod +x`; align hook probe to :4399 (or single-source the port) |

### Ungated script-fix candidates (listed only — **APPLIED NOTHING this run**)
1. **F3a** — add a cache-timestamp lazy-gate to `scripts/goal-billboard-status.sh` (recompute rendered status at most every N min). Highest-value ungated fix.
2. **F7** — prune `scripts/hooks/settings-patches/` of already-applied patches + add a manifest README (or a `scripts/prune-applied-patches.sh` one-shot).
3. **F9a** — `chmod +x scripts/run-deep-audit.sh`.
4. **F9b** — change `scripts/hooks/test-e2e-dev-server-check.sh:37` probe from `:4321` to `:4399` to match `playwright.config.ts`, or read the port from a single source.
5. **F4 (deferred)** — fold `bg-ghost-detect-dual-signal.sh` logic into `bg-auto-stop.sh` (refactor; intentional-standby today).

### Gated items (require user or `EVIUM_SETTINGS_EDIT_OK=1` — do NOT self-apply)
- **F1** edit `agent-inbox-server.py` default port (operational/`src`-adjacent, A66 high-risk).
- **F2 / F4 wires / F5 reorder / F6 merge** — all `.claude/settings.json` edits (A66 high-risk-always-gated; the `settings-edit-guard.sh` PreToolUse guard also blocks the agent Write/Edit path).
- **F4 heartbeat** — launchd install (A66 high-risk).
- **F8** — flip `settings-edit-guard.sh` + `src-edit-guard.sh` WARN→BLOCK (gated env-flag decision).

## What was NOT done (read-only audit)
No settings.json edit, no script edit, no chmod, no patch-dir cleanup, no guard flip, no port change, no worktree/launchd change. Every recommendation above is a proposal. The only file written is this audit doc.

## Reconciliation note
This supersedes the 2026-05-28 `findings/A25-hook-chain-hygiene.md` snapshot (which found 5 orphans + the same port bug) and the 2026-05-26 `A25-settings-json-health-and-hook-chain-hygiene.md` (a pre-wiring state: guard unwired, port 4399-vs-4321, 18 orphans, 1-hook UserPromptSubmit — **all four of those have since changed**). Current deltas vs the 2026-05-28 run: orphans 5→9 (4 new build-but-surface scripts on 05-29); `detect-script-candidates` latency 15.6s→~3.1s (now caches); `goal-billboard-status` ~7.3s→~6.9s (still over budget); port bug + inbox-server-guard-unwired unchanged (still live).
