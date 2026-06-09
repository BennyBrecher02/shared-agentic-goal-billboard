---
audit: A25 (re-run)
kind: READ-ONLY hook-chain hygiene audit
ran: 2026-06-07T19:29Z
scope: .claude/settings.json wiring ↔ scripts/hooks/ ; orphans ; missing wires ; should-be-wired guards ; port-literal rule ; safety-guard health (4 + 2 new)
prior: context/markdowns/goal-billboard/audits/findings/A25-hook-chain-hygiene.md (2026-05-28, found 5 orphans)
verdict: STRUCTURALLY HEALTHY. 0 dangling wires, 0 non-exec, 4/4 safety guards wired+sound. 10 orphans (was 5; +5 net new — all git-tracked, documented, intentional-standby or pending-wire, NONE dead). 1 real defect (idea-gap dead-end). 1 should-be-wired prevention guard (memory-bloat write-time half). Port rule: CLEAN.
---

# A25 re-run — hook-chain hygiene (2026-06-07)

**One-line:** the chain is in good structural health — every wired command resolves to a present,
executable script; all four named safety guards are wired and mode-correct. The "~18 orphans" catalog
hint is **stale/overcounted** (as the prior 2026-05-28 A25 already established); the true current count is
**10** orphans, all git-tracked and documented, none deletable. Net change since the last A25: +5 (one
prior orphan — `inbox-server-guard.sh` — got WIRED; six new standby/library scripts appeared). The two
"new guards being built this session" (`spawned-task-result-surface`, the widened idea-gap pipeline) are
**designed, not yet built** — no files exist to wire. One genuine LOW defect found (an idea-gap dead-end),
and one prevention-guard that should be wired but isn't (`memory-bloat-guard.sh` write-time half).

Reconciliation method: `jq` over `.claude/settings.json` → 88 distinct wired script paths (74 in
`scripts/hooks/`, 14 in `scripts/` root). `scripts/hooks/` holds 85 `.sh`/`.py` files. Cross-`comm` →
10 present-but-unwired, **0** wired-but-missing.

---

## (a) ORPHANS — scripts in `scripts/hooks/` not wired anywhere — **10** (catalog said ~18: STALE)

All 10 are **git-tracked, executable, header-documented**. Disposition matches the prior A25's "intentional
standby — keep unwired" finding, extended for the 6 that appeared since. **None is safe to delete.**

| # | Orphan (`scripts/hooks/`) | Class | Why unwired (intent) | Severity |
|---|---|---|---|---|
| 1 | `bottleneck-detect.sh` (19KB) | immune centerpiece | A58/A74 — designed for a 24h `--dry-run` window BEFORE wiring (A47 Rule 5). `organic-os-pulse.sh` checks its *existence* only, never execs it. | INFO (standby) |
| 2 | `heartbeat-independent-tick.sh` (18KB) | launchd-bound | Heart SPOF fix — meant for **launchd, NOT settings.json**. Template at `scripts/heartbeat/`, not installed. | INFO (standby) |
| 3 | `wrapper-only-write-guard.sh` (10KB) | dry-run guard | A57/A48/A61 shared write-barrier; designed for dry-run review before WARN-wiring. | INFO (standby) |
| 4 | `bg-ghost-detect-dual-signal.sh` (4KB) | CLI/standby | Dual-signal refinement of `bg-stuck-warn`/`bg-auto-stop`; fold-in candidate. | INFO (standby) |
| 5 | `convention-f-closure.sh` (17KB) | designed, unbuilt-wire | Capability-closure Stop-hook (P2 of build-but-surface plan §4); the wire was never applied. | INFO (pending-wire) |
| 6 | `domino-check.sh` (7KB) | designed, unbuilt-wire | Impact-domino UserPromptSubmit trigger (P2 §3). Built but **0 wired refs**; enqueues stubs nothing consumes yet (its consumer is part of the still-unbuilt widened idea-gap pipeline — see §d). | LOW |
| 7 | `findings-life-thread-surface.sh` (15KB, **2026-06-07**) | PENDING-WIRE (staged) | Built+tested this session (12/12). Already present in `.claude/settings.proposed.json`; **not** in live `settings.json`. The single delta between proposed and live. | LOW (awaiting paste) |
| 8 | `hook-command-lint.sh` (8KB) | meta-guard, dormant | Backstop for when `hooks{}` becomes Claude-editable (the §5 backstop of `hooks-vs-gating-split.md`). Not yet needed. | INFO (standby) |
| 9 | `memory-bloat-guard.sh` (2KB) | **should-be-wired guard** | WRITE-TIME half of the memory-bloat safeguard — see **(b)** below. | LOW |
| 10 | `cross-software-check.py` (21KB) | periodic/CLI | The cross-software-overlap + wiring-gap scanner; designed as a periodic/on-demand check, not a per-event hook. | INFO (standby) |

**Net vs prior A25 (2026-05-28):** prior found 5 orphans (`bottleneck-detect`, `heartbeat-independent-tick`,
`inbox-server-guard`, `bg-ghost-detect-dual-signal`, `wrapper-only-write-guard`). Since then:
`inbox-server-guard.sh` was **WIRED** (SessionStart, `--apply` mode — good; closes the prior A25's HIGH rec #2),
and six new scripts landed unwired (#5–#10). The count grew because the build velocity outran the gated
settings.json paste cadence — which is *by design* (PENDING-WIRES.md: "nothing waits on a paste").

---

## (b) Guard that SHOULD be wired but isn't — `memory-bloat-guard.sh` (LOW–MED)

- **What:** the guard's own header declares it a *"PreToolUse(Write|Edit) advisory guard for the auto-load
  MEMORY.md … the WRITE-TIME half of the memory-bloat safeguard."* It has **zero invokers** (the only repo
  reference is a comment in `no-hardcoded-paths-guard.sh` citing its *shape*, not calling it).
- **Why it matters:** the DETECTION half IS wired (`organic-os-pulse.sh` VITAL SIGN 5, SessionStart, limit
  24400B). But detection fires only at the *next* SessionStart — a write that pushes `MEMORY.md` over the
  silent-truncation ceiling is caught *after the fact*, never blocked at write time. The prevention half
  sits dark, so the loader can silently truncate the index tail between sessions (the exact failure
  `feedback_memory-bloat-guard.md` exists to prevent).
- **Fix (one line, GATED — user pastes):** add to `hooks.PreToolUse[]` matcher `Write|Edit`, WARN-mode with
  kill-switch, mirroring the existing guard block:
  `bash -c '[ -z "$EVIUM_MEMORY_BLOAT_GUARD_OFF" ] && bash scripts/hooks/memory-bloat-guard.sh || true'`

Other "designed-to-wire" orphans (`bottleneck-detect`, `wrapper-only-write-guard`, `convention-f-closure`,
`domino-check`) are tracked in `build-but-surface-settings-wiring-proposals.md` + `PENDING-WIRES.md` and are
correctly gated; they are deferred-by-intent, not accidental gaps. `heartbeat-independent-tick.sh` should be
wired via **launchd, not settings.json** (still uninstalled — carries the Heart SPOF the prior A25 flagged).

---

## (c) Hardcoded port literals violating the 437xx-from-`project-config.json` rule — **NONE** ✅

Scanned all `scripts/hooks/*.{sh,py}` for 4–5-digit port literals. Every hit is one of:
- **Python string-slice false positives** (`[:3000]` in `ns-advance-check`, `recurrence-detect`,
  `pending-parallel-work-scan`, `parallel-capacity-check`, `state-batch-digest`) — not ports.
- **Migration-history comments** documenting the retired `4321/4322/4323` → `437xx` renumbering
  (`inbox-server-guard.sh`, `test-e2e-dev-server-check.sh`) — annotations, not bindings.
- **Correctly-derived ports with a 437xx fallback literal:**
  - `inbox-on-prompt.sh:326` → `printf '43720'` only *after* trying `$AGENT_INBOX_PORT` then the
    services-registry (`ports.agent_inbox`). Correct derive-then-floor pattern.
  - `test-e2e-dev-server-check.sh:41` → `ASTRO_PORT="${PORT_ASTRO_DEV:-43700}"` — sources
    `project-config.sh` first, falls back to the in-band 43700.
  - `inbox-server-guard.sh:162-215` → `43720` canonical + a `WRONGPORT_SCAN` list that *intentionally*
    includes legacy `4321 4322 4323` (it's scanning for squatters on the dead ports — correct by design).

**No retired AI-default port (4321/5173/8080/8000/8787/3000) is used as an actual bind/connect value in any
hook.** The 437xx-band rule holds across the hook layer.

---

## (d) Safety-guard health

### The 4 core guards — ALL WIRED, mode-correct, kill-switched ✅

| Guard | Wired (event/matcher) | Default mode | Override / kill-switch | `exit 2` path | Verdict |
|---|---|---|---|---|---|
| `settings-edit-guard.sh` | PreToolUse `Write\|Edit` | WARN (`EVIUM_SETTINGS_GUARD_MODE`) | `EVIUM_SETTINGS_EDIT_OK=1` | yes | HEALTHY |
| `src-edit-guard.sh` | PreToolUse `Write\|Edit` | WARN (`EVIUM_SRC_EDIT_GUARD_MODE`) | `EVIUM_SRC_EDIT_OK=1` | yes | HEALTHY |
| `no-hardcoded-paths-guard.sh` | PreToolUse `Write\|Edit` | enforcing (block) | `EVIUM_HARDCODED_PATH_GUARD_OFF=1` | yes | HEALTHY |
| `spawn-task-guard.sh` | PreToolUse `mcp__ccd_session__spawn_task` | WARN (`EVIUM_SPAWN_GUARD_MODE=BLOCK`) | `EVIUM_SPAWN_GUARD_OFF=1` | yes | HEALTHY |

All four resolve to present, executable scripts, fire on the right event/matcher, default to a non-blocking
mode with a documented BLOCK promotion, and carry a per-action override. `spawn-blocking-prompt-interceptor.sh`
(PENDING-WIRES item #1) is **also now wired** (PreToolUse `AskUserQuestion|ExitPlanMode`) — that item can move
to "Applied."

### The 2 "new guards being built this session" — DESIGNED, NOT YET BUILT (nothing to wire)

Per `plans/mega-prompt-2026-06-07-execution-tracker.md` §"Part 1 follow-on — the chip-loop incident":

1. **`spawned-task-result-surface`** — a *"chip-result substrate the orchestrator reads each turn"* (the
   mid-session result-loop that closes hole #2: a *successful* spawned chip is currently invisible until
   SessionStart harvest). **Status: DESIGNED, build pending user go.** No `scripts/**/*result-surface*` file
   exists. The adjacent BUILT pieces are `spawned-task-pipeline.sh` (wired, SessionStart `--surface-and-stage`)
   + `harvest-spawned-tasks.sh` (wired, SessionStart) — but both are SessionStart-only, which is exactly the
   gap this new guard is meant to fill. → **Not an orphan; an unbuilt design.** Severity: tracked-design.

2. **Widened idea-gap pipeline** — tracker fixes **F1** (closure detectors scan only 2 of 8 goal-billboard
   artifact homes → false "slipped" flags), **F2** (create the missing `scripts/idea-gap-close.sh`), **F4**
   (wire `domino-check.sh` + build its consumer). **Status: classified "✅ safe bug-fix," build pending.**
   See the LOW defect below — F2 is live-confirmed.

---

## Real defects (ranked)

| # | Severity | Finding | One-line fix |
|---|---|---|---|
| 1 | **LOW** | **idea-gap dead-end (tracker F2, live-confirmed).** `idea-gap-surface.sh:189` prints `Close/defer/ignore: scripts/idea-gap-close.sh <…>` but `scripts/idea-gap-close.sh` **does not exist** → every surfaced idea-gap is a permanent dead-end (no close path). Also referenced in `feedback_never-miss-an-idea.md`. | Build `scripts/idea-gap-close.sh <close\|defer\|ignore> <N> [rationale]` (ungated; part of the widened idea-gap pipeline). |
| 2 | **LOW–MED** | **memory-bloat WRITE-TIME guard unwired** (see §b). Prevention half dark; only post-hoc SessionStart detection runs. | Paste the one-line PreToolUse `Write\|Edit` WARN wire (GATED). |
| 3 | **LOW** | **`domino-check.sh` wired-nowhere + dangling consumer** (tracker F4). Built, 0 wired refs; even if wired it enqueues stubs whose consumer isn't built. | Wire to UserPromptSubmit **and** build the consumer together (don't wire half). |
| 4 | **LOW** | **`findings-life-thread-surface.sh` staged but not live.** Sole delta between `.claude/settings.proposed.json` and `.claude/settings.json`; both files show as modified in `git status`. | User pastes the SessionStart wire from `PENDING-WIRES.md` §3 (GATED); then move it to "Applied." |
| 5 | **INFO** | **Heart SPOF persists** (carried from prior A25). `heartbeat-independent-tick.sh` still uninstalled in launchd; only `com.evium.daemon10` was loaded at last check. | `launchctl load` the heartbeat plist template (GATED — launchd). |
| 6 | **COSMETIC** | **Two separate `PreToolUse` `matcher:"Write\|Edit"` blocks** in settings.json (one holds inbox-schema/settings-edit/src-edit/no-hardcoded-paths; the other holds `inbox-mutation-guard` alone). Both fire; readability only. | Merge into one block (GATED, trivial). |

**No CRITICAL or HIGH findings.** The prior A25's HIGH items both resolved: `inbox-server-guard.sh` is now
wired, and the `agent-inbox-server` port contradiction was fixed by the 2026-06-01 port-uniqueness-initiative
(canonical `43720`; the old `4322/4323` are retired and only appear in squatter-scan lists + history comments).

### Meta-observation (INFO)
The wired meta-guard `hooks-health-check.sh` (SessionStart) verifies **wired → present + executable** but does
**not** verify the reverse (present → wired) nor catch *referenced-but-missing helper scripts* like defect #1.
That blind spot is precisely what `convention-f-closure.sh` (orphan #5) and `cross-software-check.py`
(orphan #10) were built to cover — both unwired. Wiring one of them would have auto-flagged the
`idea-gap-close.sh` dead-end. Reinforces the standing "build-but-surface" disease pattern.

---

## Verification basis
- `jq` extraction of all hook commands from `.claude/settings.json`; `comm` against `ls scripts/hooks/*.{sh,py}`.
- Per-orphan repo-wide `grep` for callers/wires; mtime + `git ls-files` tracked-status on each.
- Per-guard: wiring grep + header mode-line + `exit 2` presence.
- Port scan: `grep -E '(4[0-9]{3}|5173|8080|8000|8787|3000)'` over hooks, manually triaged each hit.
- Cross-checked against prior `A25-hook-chain-hygiene.md` (2026-05-28), `PENDING-WIRES.md`,
  `build-but-surface-settings-wiring-proposals.md`, `mega-prompt-2026-06-07-execution-tracker.md`,
  `.claude/settings.proposed.json`.

ALL fixes touching `.claude/settings.json` or launchd are A66 high-risk-GATED — surfaced as PROPOSE, none applied
(this was a READ-ONLY audit; no settings/hook/spawn_task action taken).

BG-COMPLETE-SENTINEL: (issues_found=6, critical_count=0, status=HEALTHY)
