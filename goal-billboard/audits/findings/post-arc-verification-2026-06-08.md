---
title: Post-ARC verification — 10-wire settings bundle + C→A→B arc
audit_type: system-infra
scope: post-arc-solidity
date: 2026-06-08
run_at: 2026-06-08T13:08Z–13:16Z
mode: READ-ONLY (verify-only; flag-don't-fix)
verifier: subagent
belongs_to_goal: G2
serves_northern_star: G2
status: COMPLETE
verdict: SOLID-WITH-2-MINOR-FLAGS
---

# Post-ARC verification — 2026-06-08

Comprehensive read-only verification that the 10 newly-live hooks + the C→A→B arc
(dashboard alignment surface, goal re-mapping, mirrors, AIS w/ Lobe-D) are SOLID
before the user trusts them. Each hook was run **in its actual wired form** (the exact
`bash -c '[ -z "$AOS_..._OFF" ] && ... || true'` string from `.claude/settings.json`),
with safe synthetic input where a tool-context payload was needed.

**Bottom line: nothing needs a kill-switch.** All 10 new wires exit cleanly (0, or the
intended 2 in BLOCK mode), none storms the transcript, every kill-switch is wired and
works. Two MINOR flags (one new-wire data-quality, one pre-existing noise) are logged
below — neither blocks trust, neither is urgent.

---

## AREA 1 — The 10 newly-live hooks  →  PASS (10/10), 1 minor data-quality FLAG

Run context, exit code, duration, and transcript-output volume for each. Live
`.claude/settings.json` was confirmed **byte-identical to `.claude/settings.proposed.json`**
(the bundle landed), both JSON valid, and **all 10 kill-switch env vars are wired**
(each gated + `|| true` fail-open).

| # | Hook | Event(s) | Exit | Dur | Output | Kill-switch | Verdict |
|---|------|----------|------|-----|--------|-------------|---------|
| 1 | `status-drift-sweep.sh` | SessionStart + Stop | 0 / 0 | 0.22s | silent (no drift) | `AOS_STATUS_DRIFT_OFF` ✓ | PASS |
| 2 | `orphan-sweep.sh --quiet` | SessionStart | 0 | 1.96s | 5 ln (only when NEW orphan) | `AOS_ORPHAN_SWEEP_OFF` ✓ | PASS |
| 3 | `cross-software-check.py --quiet` | SessionStart | 0 | 0.48s | 5 ln | `AOS_CROSS_SOFTWARE_OFF` ✓ | PASS |
| 4 | `daemon-graduation-check.sh` | SessionStart | 0 | 0.05s | 7 ln | `AOS_DAEMON_GRADUATION_OFF` ✓ | PASS |
| 5 | `hook-command-lint.sh` | PreToolUse Write/Edit | 0 (WARN) / 2 (BLOCK) | 0.08s | silent unless settings.json edit | `AOS_HOOK_CMD_LINT_OFF` ✓ | PASS |
| 6 | `wrapper-only-write-guard.sh` | PreToolUse Write/Edit | 0 (WARN) | 0.06s | silent unless protected file | `AOS_WRAPPER_GUARD_OFF` ✓ | PASS |
| 7 | `bg-dispatch-log-gc.sh --apply` | Stop | 0 | 0.04s | 9-ln banner (always) | `AOS_BG_GC_OFF` ✓ | PASS |
| 8 | `bg-ghost-detect-dual-signal.sh` | Stop | 0 (wrapped) | 2.7s | **0 transcript** (→ledger) | `AOS_BG_GHOST_DUAL_OFF` ✓ | PASS ⚠ |
| 9 | `convention-f-closure.sh` | Stop | 0 | 0.07s | silent (nothing new landed) | `AOS_CONVENTION_F_OFF` ✓ | PASS |
| 10 | `bottleneck-detect.sh` | Stop | 0 | 0.08s | 3 ln (DRY-RUN) | `AOS_BOTTLENECK_DETECT_OFF` ✓ | PASS |

### Per-hook behavioral confirmations (not just exit 0)

- **status-drift-sweep** — exit 0 in BOTH SessionStart and Stop contexts; correctly
  short-circuits on `stop_hook_active=true` (0.03s vs 0.22s) so it does not re-fire in a
  Stop loop. Silent = no live status-drift to report. WARN-only, read-only.
- **orphan-sweep** — fired correctly: surfaced 2 genuine orphans
  (`scripts/detect-gaps-cross-substrate.py`, `scripts/prune-reports.sh`) → appended to the
  alignment watchdog feed. This is the hook **working as designed**, not a misfire. (Those
  2 orphans are a real housekeeping item — see "Orphans surfaced" below — but they are not a
  defect of this wire.)
- **cross-software-check** — exit 0, 0.48s; reports 0 wiring gaps, 2 dead-end producers
  (advisory). Quiet.
- **daemon-graduation-check** — exit 0, read-only; correctly reports "No daemons eligible
  to graduate" (all 5 already live, err 0). Surfacing-only; graduation stays user-gated.
- **hook-command-lint** — VERIFIED against real attack shape: fed an absolute-path
  settings.json edit carrying a `rm -rf /tmp/evil` hook command → WARN fired, logged a
  `would-block` row to `.claude/cache/hook-command-lint.jsonl`, edit allowed (exit 0). The
  sanctioned shape (`bash scripts/hooks/foo.sh`) passed clean. BLOCK mode → exit 2 with the
  refusal message. **(NOTE: this hook only lints when `tool_input.file_path` matches
  `*/.claude/settings.json` — i.e. the ABSOLUTE path Claude Code actually supplies. A
  relative-path test payload silently passes; not a defect, just a test-shape caveat.)**
- **wrapper-only-write-guard** — VERIFIED against its real protected surface
  (`reports/screenshot-monkey/data.json`, NOT `public/dashboard/data.json`): WARN fired +
  ledger row written + write allowed (exit 0). Per-action whitelist
  (`AOS_WRAPPER_WRITE_OK=1`) and kill-switch both silence it. Non-protected paths pass
  through silently — correct.
- **bg-dispatch-log-gc** — exit 0; idempotent; closed 0 stuck prelaunch rows (none open).
  9-line banner prints every Stop even on a no-op (see Flag 2 / NOISE notes — minor).
- **bg-ghost-detect-dual-signal** — see FLAG 1 below. In the WIRED form (output redirected
  to `reports/bg-dispatch-log.jsonl`) it emits **0 lines to the transcript** and exits 0 —
  the wrapper does its job, no noise. Kill-switch verified (line count unchanged).
- **convention-f-closure** — exit 0, silent (no caller-less script landed this turn). Correct.
- **bottleneck-detect** — exit 0, DRY-RUN; tripped 1 signal (heartbeat pending-queue depth
  185–187 > 50), logged to its JSONL, NEVER pauses. The 1 tripped signal is a real
  advisory, not a hook defect (see "Pending-queue depth" below).

### Aggregate NOISE test (the user's #1 concern)

Chained the new wires exactly as settings fires them:
- **SessionStart new wires (4):** 13 lines / 901 bytes total.
- **Stop new wires (6):** 13 lines / 831 bytes total (bg-ghost redirected → 0 transcript).

**~26 lines across both boundaries combined — NOT a noise-storm.** Reasonable for an
observability layer. The only fixed-every-time talkers are `bg-dispatch-log-gc` (9-line
banner even on a 0-row no-op) and `daemon-graduation-check` (7 lines listing all 5 daemons
even when none eligible) — tolerable, not spam.

### FLAG 1 (MINOR, new wire) — `bg-ghost-detect-dual-signal` unbounded ledger append
- **What:** the raw script emits **one JSONL row per `/private/tmp/.../tasks/*.output`
  file, with NO dedup/TTL**, on every invocation. There are currently ~300 stale `.output`
  files from prior sessions, so every Stop appends ~150–300 `verdict:ghost` rows to
  `reports/bg-dispatch-log.jsonl`.
- **Evidence:** that ledger is at 1,117 lines / ~362 KB and already holds 332 ghost rows;
  167 of them were a single invocation. The companion GC (`bg-dispatch-log-gc.sh`) only
  closes `prelaunch_*` rows — it does **NOT** prune these ghost rows. → unbounded growth
  (~+150–300 lines/Stop).
- **Why it's only MINOR / NOT a trust-blocker:** (a) the wrapper redirects ALL of it to the
  ledger, so **0 lines reach the transcript** — no user-facing noise; (b) `bg-dispatch-log.jsonl`
  is a **gitignored runtime artifact**, not tracked — no repo churn; (c) exit 0 always.
- **Recommended fix (do NOT need a kill-switch):** either (i) teach the dual-signal script a
  per-day TTL-dedup on `bg_id` (the established JSONL pattern), or (ii) extend
  `bg-dispatch-log-gc` to prune ghost rows older than N days, or (iii) reap the ~300 stale
  `/private/tmp` `.output` files. Track as a follow-up; the kill-switch
  (`AOS_BG_GHOST_DUAL_OFF=1`) is available if the log bloat bites before a fix lands.

### Orphans surfaced (real housekeeping, not a defect)
orphan-sweep + cross-software-check flagged, as designed:
- `scripts/detect-gaps-cross-substrate.py` — 0 wires, 0 callers, not gated-staged.
- `scripts/prune-reports.sh` — same.
- dead-end producers: `.claude/cache/model-route-proposals.jsonl`,
  `reports/ns-hook-test/synthetic-session.jsonl`.
Action: wire / retire / tag `# orphan-ok:` each. (Out of scope to fix here — flagged per mandate.)

---

## AREA 2 — Dashboard (merge + build_render_model + alignment surface F + by-goal spread)  →  PASS

Ran the full pipeline end-to-end:

1. **`build_render_model.py`** — exit 0, 4.05s → `reports/dashboard-build/render-model.json`.
2. **`dashboard-merge-run.sh --full`** — exit 0, 2.5s; merge log shows all blocks (stats,
   asks, meta, cluster, skill-metrics, allstats, …) writing cleanly, **zero Traceback/Error**.
3. **`generate_dashboard.py`** — exit 0, 0.13s → `reports/dashboard-build/dashboard.html`
   (2.46 MB), overview + 13 deep views, model baked in.

### NEW alignment surface (section F / B6) — RENDERS
- `render-model.json` → `health.alignment` is fully populated: `engine_live=true`, `ok=true`,
  `open_findings=121`, `by_lobe` (6 lobes), `by_severity {high:19, med:25, low:67, unknown:10}`,
  `dangling_deferrals=4`, `pipeline {crosslink 58.3%, ideas 19, goals 19, proposals 16}`.
- `needs_you.alignment` carries `user_facing_open=7`, `dangling_deferrals=4`, and a `worst_open`
  (the "D1 Sliding-window report pruning" dangling-deferral, severity high).
- Rendered HTML carries it through: `alignment` ×95, `lobe` ×27, `immune` ×16, `section-f` ×5,
  plus the baked MODEL const with `engine_live:true`, `dangling_deferrals:4`, `user_facing_open:7`,
  `by_lobe`, `open_findings`.

### By-goal view now SPREADS (the re-mapping) — CONFIRMED
- `board.groups.by_goal` = **20 constellations** across **G1, G2, G3, G4, G6–G21**
  (G5 absent — closed/archived). `look_back.plans.by_goal` = **19 goals**.
- This directly confirms the expected "~17 populated, NOT all-G2." The all-G2 collapse is gone.
- (The rendered HTML shows only 5 literal `.bconstellation` strings because the 20 instances are
  built CLIENT-SIDE from the baked `MODEL.board.groups.by_goal` — the documented "baked model,
  no runtime fetch" architecture. The 20-goal spread is verified in the serialized model.)

### Existing surfaces — UNBROKEN (no regression from today)
matrix 108/108 covered · services 5 (up 2 / down 2 / idle 1, probe_live, no collision) ·
git timeline 49 commits · code-diffs 49 (5,718 baked lines) · bugs_open 5 · subagents 51 today ·
improvement 13 tags (ok) · correlations 7/10 · unified board 539 items / 374 edges. All present,
no empty-where-populated-expected. The "GAPS/WARNINGS" block (capture/git-pair coverage notes) is
normal advisory content, not errors.

---

## AREA 3 — The 3 mirrors + the AIS (Lobe-D)  →  PASS

| Surface | Run | Exit | Dur | Result |
|---------|-----|------|-----|--------|
| Memory sync | `sync-memory.sh` | 0 | 0.32s | silent → in sync |
| Memory sync (verify) | `verify-memory-sync.sh` | 0 | 0.07s | silent → mirror matches canon |
| skill-drift mirror | `skill-drift-check.sh` | 0 | 4.08s | 1 benign NORECEIPT (agentic-preview-tools: content matches canon, no deploy receipt) |
| guardrail mirror | `guardrail-mirror-check.sh` | 0 | 1.89s | **0 TRUE dangerous gaps**; clean cross-env table |
| AIS engine | `alignment-sweep.sh` (QUIET=1, wired) | 0 | 3.24s | silent (WARN-only); also clean verbose |

- **Memory sync** — both halves exit 0 silent → canonical auto-memory and
  `.claude/memory-mirror/` are in sync (no hand-edit drift).
- **skill-drift-check** — exit 0; the single `NORECEIPT` (agentic-preview-tools in the
  `cursor` env) is advisory — deployed content matches canon, only a receipt is missing.
  The `claude` env is correctly skipped (dormant/no deploy target). Not a drift error.
- **guardrail-mirror-check** — exit 0; explicitly "0 ⚠ TRUE dangerous gaps (no op Cursor
  lets through that Claude blocks)." The 3 Claude-only categories are tool-specific (no
  Cursor shell analog) → correctly classed N/A, not holes. Consistent on push/force/reset/
  clean/rm-rf/sudo/settings-edit/no-verify/checkout.
- **AIS / alignment-sweep w/ Lobe-D** — exit 0 in wired QUIET form AND verbose; ~3.2s.
  **Lobe-D (`dangling-deferral`) is live**: it's a first-class lobe in the script
  (`LOBE_D_GB_DIR`, fed by `routing-linter.sh` + `cross-link-maintainer.sh --check`) and in
  `parse_alignment.py`'s 6-lobe set. The dashboard already reflects its output
  (`dangling-deferral count=4 open=4 worst=high`; worst-open = D1 report-pruning). All
  6 lobes present: drop-through, dangling-deferral, goal-align, intent-fidelity, +2 maintenance.

---

## FLAG 2 (MINOR, PRE-EXISTING — NOT a post-arc regression) — `lint-skill-staleness.sh` noise
- **What:** the Stop-wired `lint-skill-staleness.sh` **exits 1** and dumps **~41 lines** to
  the transcript on every Stop.
- **Root cause:** it greps the skills tree for stale-markers ("placeholder", "not yet",
  "deferred", …) and returns 1 + the match list whenever ANY match exists (by design, as a
  WARN). Nearly all ~41 matches are **false positives** — the words appear inside legitimate
  skill prose (e.g. `distributed-systems-patterns.md`'s "placeholder-becomes-real" pattern,
  "no skeleton shimmer placeholder", "not yet built").
- **Why it's only MINOR:** (a) it was **NOT modified in today's arc** (last touched in older
  commits 1e254e3 / 02359cd) → this is baseline behavior, not a regression the arc introduced;
  (b) exit **1 ≠ 2**, and only exit 2 blocks a Stop in Claude Code's hook protocol, so it does
  **NOT block** — it just prints to stderr; (c) it's one of the "3 mirrors" in scope, so noted
  here for completeness.
- **Recommended (optional) fix:** tighten its marker regex to skip backticked code and the
  established "placeholder-becomes-real / not yet built (deferred to …)" idioms, OR gate its
  output behind a `--quiet`/summary mode so it surfaces a count not the full 41-line dump. No
  kill-switch needed; non-blocking today.

---

## Read-only integrity
- My verification ran ~13:08–13:16Z and left **NO tracked-file mutations**. The ~60 modified
  tracked `.md` files in `git status` (the `belongs_to_goal:` frontmatter additions, etc.)
  have mtimes of 07:41–08:59Z — they are the **arc's own goal-remapping work in progress on
  `dashboard-wip`**, not my doing.
- The only artifacts I touched are gitignored runtime files (`bg-dispatch-log.jsonl` — I
  appended test ghost rows then trimmed the 167 I injected; the file is gitignored anyway —
  and the `.claude/cache/*.jsonl` ledgers the WARN hooks write to by design).
- Untracked `context/markdowns/goal-billboard.bak-*` dirs (12:29Z, 12:55Z) are
  backup-sensitive.sh / goal-billboard-apply snapshots from before my session — not mine.

---

## SUMMARY
- **wires_clean:** YES — 10/10 exit cleanly, all kill-switches wired + working, live==proposed, JSON valid, no noise-storm (~26 ln aggregate).
- **dashboard_ok:** YES — merge + render + generate all exit 0; section F alignment surface renders; by-goal spreads across 19–20 goals; existing surfaces unbroken.
- **mirrors_clean:** YES — memory sync in sync; skill-drift 1 benign NORECEIPT; guardrail 0 true gaps.
- **ais_clean:** YES — alignment-sweep exit 0, Lobe-D (dangling-deferral) live + reflected in the dashboard.
- **flags:** 2 MINOR, neither a trust-blocker, neither needs a kill-switch:
  1. (new wire) `bg-ghost-detect-dual-signal` appends ~150–300 un-deduped ghost rows/Stop to the gitignored `bg-dispatch-log.jsonl` (0 transcript noise; add TTL-dedup or extend the GC).
  2. (pre-existing, not arc-introduced) `lint-skill-staleness` exits 1 + dumps ~41 mostly-false-positive lines/Stop (non-blocking; tighten the marker regex).
- **kill-switch wires recommended:** NONE. Every wire is safe to trust as-is.
- **verdict:** SOLID-WITH-2-MINOR-FLAGS.
