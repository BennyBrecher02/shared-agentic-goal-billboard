---
kind: audit-findings
audit: orphaned-implementations
date: 2026-06-07
synthesizer: orphaned-implementations synthesizer (subagent)
inspectors: 5
findings_received: 38
findings_confirmed: 38
false_positives: 0
verdict_feed: .claude/cache/alignment/verdict.jsonl (38 lines appended, lobe=orphan-implementation)
---

# Orphaned-implementations audit — 2026-06-07

A built thing that fires nowhere is a **liability, not an asset**: it carries maintenance cost, lies in the
inventory ("we have a bottleneck detector!"), and rots silently. This sweep deduped 38 inspector findings,
**cross-checked every one against on-disk reality (Bash)**, and classified each as **wire / fix / retire /
build / add-memory-pointer**. **All 38 confirmed — zero false positives.** The single highest-confidence
defect (the heartbeat `status.sh` SIGPIPE bug) was independently reproduced on this machine.

## Histogram by class

| State | Count | What it means | Dominant recommendation |
|---|---:|---|---|
| **built-not-wired** | 8 | Script exists on disk, fires nowhere (0 wires in `settings.json`) | wire (×6), retire (×1), leave/gated (×1) |
| **built-broke** | 7 | Wired/dispatched but silently mis-behaves (wrong path, SIGPIPE, dry pipe, frozen id) | fix (×7) |
| **never-built** | 9 | Designed/catalogued, no code on disk | build (×2), retire (×3), leave (×4) |
| **scheduler-holding** | 6 | Real subsystem, idle/holding — no tick driver or gated on user/hardware | leave (×6) |
| **memory-gap** | 9 | Live load-bearing capability with **0 recall path** in canonical memory | add-memory-pointer (×9) |
| **TOTAL** | **38** | | |

Recommendation tally: **wire 7 · fix 7 · retire 4 · build 2 · add-memory-pointer 9 · leave 9** (the
9 "leave" = correctly-idle schedulers + deliberately-deferred/superseded never-builts; flagged, not actioned).

## Load-bearing vs disposable (the prioritization)

**Load-bearing — the system silently depends on these or they deliver real value if revived:**
- **`heartbeat/status.sh`** (built-broke, **fix now**) — actively lies to any human checking Heart health.
- **`sync-monkey-decisions-from-inbox.py`** (built-broke, **fix now**) — auto-fires every SessionStart + every billboard/plan edit, errors and no-ops; the inbox→data.json status sync is fully dead.
- **`hook-usage-stats --dead-only`** (built-broke, **fix**) — the very surface that drives retire-decisions is misleading (over-reports 32 "dead" hooks that are alive). Fixing it is a prerequisite to trusting any future orphan sweep.
- **Heart tier-medium consumer** (never-built, **build**) — the keystone gap: it is the cited firing substrate for A65 Daemons #1/#5/#7/#8/#9 + bottleneck-detect. Its absence is *why* several daemons can't fire even in principle.
- **The 9 memory-gaps** (**add-memory-pointer**) — `migrate-memory-on-move.sh` and `backup-sensitive.sh` are the two highest-blast-radius: forgetting them risks the brain itself (a blank MEMORY.md / an un-backed-up memory edit).

**Disposable — recommend retire to `disposal-bin/` via `scripts/dispose.sh` (recommend, do NOT execute — `dispose.sh` is move-not-delete + git-stage; the user empties the bin):**
- **`scripts/hooks/memory-bloat-guard.sh`** (built-not-wired, **retire**) — its job (24.4KB MEMORY.md ceiling) is **already enforced live** by `scripts/tests/test-memory-within-limit.sh` (wired into `run-tests.sh`). Supplanted. NB: the MEMORY.md index line "Enforced by ... memory-bloat-guard.sh" is **STALE** and should be corrected to name `test-memory-within-limit.sh`.
- **A65 Daemon #7 (snapshot-drift scanner)** (never-built, **retire**) — design-only, self-admittedly near-zero value until `.snapshots/` has baselines (it doesn't). Retire the design or re-scope.
- **A65 Daemon #8 (statistical-correlator)** (never-built, **retire**) — **superseded** by the read-time `scripts/dashboard/correlate_stats.py` (A43 Layer B) + `parse_correlations.py`. The intent is already covered; flip A14 → resolved.
- **`scripts/dispose.sh` proposed disposal command:**
  `scripts/dispose.sh "supplanted by test-memory-within-limit.sh (orphan audit 2026-06-07)" scripts/hooks/memory-bloat-guard.sh`

---

## Classified findings (name · state · evidence · recommendation)

### built-not-wired (8) — exists on disk, fires nowhere

1. **`scripts/hooks/bg-ghost-detect-dual-signal.sh`** — 0 wires in `settings.json`, 0 live callers anywhere, not in any settings-patch or plist. Pure orphan; instrumented but never dispatched so its counter can never increment; invisible to `--dead-only` (which only lists WIRED hooks). → **WIRE** (it is the dual-signal BG ghost detector; if BG ghost-reaping matters, arm it). *Confirmed: `grep -c` settings.json = 0.*

2. **`scripts/hooks/bottleneck-detect.sh`** — 0 wires. The only reference from a wired hook (`organic-os-pulse.sh` ~159-171) is an **existence check** `[[ -e ]]` that renders an IMMUNE glyph; it never EXECUTES. Staged in `settings-patches/organic-os-pulse.json` (never applied). The advertised anti-bottleneck detector is dormant (A58). → **WIRE** (needs a tier-medium firing rhythm — see Heart tier-medium below). *Confirmed.*

3. **`scripts/hooks/convention-f-closure.sh`** — 0 wires, AND not in any settings-patch (unlike the others). The Stop hook (A83 "Convention F") meant to flag NEW scripts landing with no caller — is itself a script that landed with no caller. The orphan-catcher is an orphan. → **WIRE** (this is the natural seed of the standing orphan-auditor below). *Confirmed.*

4. **`scripts/hooks/memory-bloat-guard.sh`** — 0 wires, not in any patch, not called by the pulse. Its one apparent caller (`no-hardcoded-paths-guard.sh:29`) is a **comment only**. CRITICAL: the 24.4KB limit it enforces is **already enforced live** by `scripts/tests/test-memory-within-limit.sh` (wired into `run-tests.sh`). Job supplanted. → **RETIRE** (and fix the stale MEMORY.md "Enforced by" line). *Confirmed: `test-memory-within-limit.sh` exists, independent check.*

5. **`scripts/hooks/domino-check.sh` + `scripts/domino-consume.sh`** — staged producer/consumer pair, both unwired. `domino-check` (0-wired; only in `settings-patches/domino-check.json`) appends `unscored` stubs to `.claude/cache/domino-queue.jsonl`; `domino-consume` is referenced ONLY by `run-tests.sh`. Even if the producer fired, nothing live drains the queue. This is the A67 memory-only item MEMORY.md itself flags. → **WIRE** (both, as a unit — wiring one without the other is pointless). *Confirmed.*

6. **`scripts/hooks/wrapper-only-write-guard.sh`** — 0 wires, not in any patch. Its only code referencer is `convention-f-closure.sh` — itself unwired → transitively unreachable. A passing test on a never-dispatched guard ≠ live. → **WIRE** (alongside convention-f-closure). *Confirmed.*

7. **`scripts/hooks/hook-command-lint.sh`** — 0 wires, not in any patch, no live caller (only a test references it). A lint built to validate hook command strings, never run against the live `settings.json` it guards. → **WIRE**. *Confirmed.*

8. **A83 build-but-surface wiring bundle** (`bottleneck-detect` · `wrapper-only-write-guard` · `domino-check` · `convention-f-closure`) — `plans/build-but-surface-settings-wiring-proposals.md` proposes wiring all 4. All 4 SCRIPTS exist; none in `settings.json` nor `settings.proposed.json`. → **WIRE** — but **A66 high-risk-always-gated**: the user applies `settings.json` (or authorizes `EVIUM_SETTINGS_EDIT_OK=1`). This is the umbrella for #2/#3/#5/#6 above; surface as ONE gated yes/no. *Confirmed: design doc exists, 0 in both settings files.*

### built-broke (7) — wired/dispatched but silently mis-behaves

9. **`heartbeat/status.sh`** — `set -uo pipefail` (line 13) + `launchctl list | grep -qE "...heartbeat"` (line 34). `grep -q` short-circuits on first match → closes pipe → `launchctl list` dies SIGPIPE (141) → pipefail propagates → `if` falls to else → prints **"LaunchAgent: not loaded"** while the Heart is loaded and beating. → **FIX**. **REPRODUCED ON THIS MACHINE:** plain `launchctl list | grep -qE ...heartbeat` MATCHES `com.agentic-os.heartbeat-independent`, but `bash -c 'set -uo pipefail; if launchctl list | grep -qE ...heartbeat; then echo MATCHED; else echo FELL-THROUGH; fi'` prints **FELL-THROUGH**, and `bash scripts/heartbeat/status.sh` prints **"LaunchAgent: not loaded"** — while the label is live (exit 0, ticks at 23:52/00:02/00:12/00:22/00:32, fresh beacon). `daemon-health.sh` reports the SAME domain correctly because it reads `launchctl list` into a var first (no `| grep -q` short-circuit). Fix: read into a var, or `grep -qE ... || true`, or drop `pipefail` for this one read. **Bug isolated to status.sh** (the other 5 reporters have 0 occurrences of the pattern).

10. **`sync-monkey-decisions-from-inbox.py`** — line 88 `DEFAULT_DATA_JSON = REPO/'public'/'screenshot-monkey'/'data.json'`. **`public/screenshot-monkey/` does NOT exist** (data lives at `reports/screenshot-monkey/data.json`, freshly rendered 16:04). Auto-wired into `dashboard-merge-run.sh` (runs every SessionStart via `dashboard-data-prime.sh`, every billboard/plan edit via `billboard-data-refresh.sh`). Live log: `{"error":"data.json not found: /…/public/screenshot-monkey/data.json","flipped":0}`. No `reports/` fallback (unlike `chamber-empty-detect.sh:96-103` which compensates). The inbox→data.json status sync is **fully silent-dead**. → **FIX** (repoint to `reports/screenshot-monkey/data.json` or add the same fallback `chamber-empty-detect` has). *Confirmed: dir absent, reports file present.*

11. **`post-to-chamber.py`** — same phantom `public/screenshot-monkey/data.json` (line 41), with a hard `sys.exit` "data.json not found" (lines 114-116). The "sanctioned path for adding/editing chamber decisions" would refuse on every run. NOT in the auto-fired chain (lower severity than #10), but a broken tool — a path-canon split-brain (the substrate moved `public/`→`reports/`; `build_render_model.py:22` + `monkey-click-fan-out.sh` + `ack-latency-measure.sh` all use `reports/`; these two scripts were left on the abandoned `public/` canon). → **FIX** (repoint to `reports/`). *Confirmed.*

12. **`hook-usage-stats --dead-only`** — over-reports "dead" because the per-hook counter (`instrument-hooks-usage.sh`) instruments **only `scripts/hooks/*.sh`**, missing the ~12 SessionStart/Stop hooks that live in `scripts/` ROOT (`sync-memory.sh`, `verify-memory-sync.sh`, `bug-billboard-consolidate.sh`, `audit-recommendations.sh`, `compute-matrix-stats.sh`, `detect-script-candidates.sh`, `goal-billboard-status.sh`, `lint-skill-staleness.sh` …). Confirmed: `organic-os-pulse.sh` IS instrumented; `sync-memory.sh` is NOT. The tool prints an honest CAVEAT, but as a "retire candidate" surface it would steer toward **retiring live, load-bearing hooks**. NOTE: a measurement/observability defect, **NOT 32 dead hooks** — the hooks work. → **FIX** (extend `instrument-hooks-usage.sh` to cover `scripts/` root hooks, or change `--dead-only` to read the wired-hook set from `settings.json` rather than the instrumentation ledger). **Prerequisite for trusting any future orphan sweep.**

13. **Dashboard timelapse renderers — frozen `DEFAULT_RUN='2026-05-25-overnight'`** — hardcoded in `render-asks-data.py:62`, `render-cluster-data.py:62`, `render-meta-loop-eval-data.py:51`, `heal-timelapse-data.py:28`, `render-stats-data.py`, and `render-billboard-page.py:424` (meta-refresh to `/audit-timelapse/2026-05-25-overnight/`). `build_render_model.py:56` reads that same frozen path. The mergers DO write fresh data (`reports/timelapse/2026-05-25-overnight/data.json` mtime 2026-06-07 20:41) — so NOT a no-op — but "today's" data is merged into a folder named for a session **13 days ago**: the time-lapse/run-history dimension is a single ever-overwritten snapshot, not a rotating series. → **FIX** (decide: pin intentionally → then drop the "rotating timelapse archive" story; OR rotate the run-id per session/day). Lower-confidence "silent fail" (data is current, just mislabeled) — a stale-target smell worth a second look. *Confirmed.*

14. **Event-bus substrate is EMPTY (`.claude/cache/event-bus.jsonl` never created)** — every beat logs `event-bus: cursor 0 -> 0 (advanced); fired tier-high consumers for 0 event(s)` and `pending: 0 line(s)`. **Confirmed both files absent.** So `heartbeat-drain.sh` (UserPromptSubmit) hits `[[ -f "$PENDING" ]] || exit 0` — silent no-op every prompt; `status.sh`'s "Pending queue: not created yet" / "Last drain ack: never drained" are literally true. Producers exist in code (`scheduler.py`, `session-bundle-create.sh`, `agent-inbox-server.py`, `heartbeat-independent-tick.sh`) but **none has ever published a line**. The whole Heart→event-bus→tier-consumer→pending→drain→surface chain runs on a **dry pipe** — "works" (no errors) while carrying zero payload, and the Heart reports success while doing zero real work. → **FIX** (confirm whether a producer — e.g. scheduler `tick_end` events — is SUPPOSED to emit and silently isn't, vs the bus being a not-yet-fed substrate; then either wire a real producer or down-grade the "fired consumers" log to not imply work). Straddles silent-fail / built-not-wired. *Confirmed.*

15. **`heartbeat/status.sh` false-negative (companion to #9)** — beyond the SIGPIPE bug, status.sh reports only the TIERED state files (`last-tick-{high,medium,low,hourly}.txt`) and **ignores the `-independent` label's state file**. The actually-live job is `com.agentic-os.heartbeat-independent` (loaded, exit 0; beacon ts 2026-06-08T00:32:19Z, mode apply, every 10min; `independent-launchd.err` EMPTY). A human gets "not loaded / never run" for a healthy live Heart. Pure observability defect — the Heart is fine. → **FIX** (status.sh must surface the independent tick's beacon + tier state). *Confirmed.*

### never-built (9) — designed/catalogued, no code on disk

16. **A65 Daemon #6 — CoT-ledger consolidator** — catalogued in `audits/A65-...framework.md` (item 6: summarize CoT chains >50 entries → `.claude/cache/cot-ledger-digests/<chain_id>.md`, Heart tier-low). A65: "PHASE 1 LANDED; Phase 2-N deferred." **Confirmed never landed:** no `*cot*` consumer (consumers dir = daemon2/3/4/5/10 + feedback-loop + dashboard-refresh + tier-high only), no `cot-ledger-digests/` dir, no plist, not in `daemon-control` catalog. `cot-ledger.jsonl` has many producers, nothing consolidates. → **LEAVE** (deferred-by-design; revive only when CoT chains actually grow — depends on the Heart tier-low/medium substrate).

17. **A65 Daemon #7 — Snapshot-drift scanner** — 474-line design at `plans/snapshot-drift-scanner-daemon-design.md`, frontmatter "DESIGN ONLY". **Confirmed:** no `snapshot-drift-scanner-daemon.py`, no consumer, no `daemon7` plist, no `reports/snapshot-drift-proposals.md`. (`scripts/hooks/snapshot-drift-pre.sh` is an UNRELATED migration hook.) Design admits near-zero value until `.snapshots/` has baselines (still none). → **RETIRE** the design (or re-scope). 

18. **A65 Daemon #8 — Statistical-correlator** — proposed in A65 item 8 + as A14 sibling. **Confirmed no standalone daemon.** **SUPERSEDED:** `scripts/dashboard/correlate_stats.py` (A43 Layer B) already JOINS the append-only logs → `reports/stats/correlations.json`, rendered by `parse_correlations.py`. A14's resolution doc recommends status→resolved. → **RETIRE** (read-time engine covers the intent; flip A14 → resolved). *Confirmed `correlate_stats.py` exists.*

19. **A65 Daemon #9 — Impact-domino triage daemon** (Heart tier-medium home) — `feedback_impact-domino-analysis.md` (A67) names the persistent home as "A65 Daemon #9 on Heart tier-medium." **The DAEMON was never built** (no Daemon-#9 consumer, no tier-medium firing). Distinct from the lighter `domino-check.sh`/`domino-consume.sh`/`analyzer-domino-effectiveness.py` (which exist but are unwired — see #5). → **LEAVE** (the persistent autonomic daemon is blocked on the Heart tier-medium substrate — build that first, then reconsider).

20. **Heart tier-medium consumer (A58 Heart Phase 2)** — design at `plans/heart-tier-medium-consumer-design.md`, "drafted (awaiting user authorization before any code lands)." **Confirmed:** consumers dir has tier-high + tier-low-daemon10 ONLY — **no tier-medium consumer.** **THE KEYSTONE GAP:** it is the cited firing substrate for A65 Daemons #1/#5/#7/#8/#9 and `bottleneck-detect`, so its absence is *why* those can't fire even in principle. → **BUILD** (highest-leverage never-built; unblocks 6 downstream items). Gated on user authorization per the doc.

21. **Cluster true multi-node dispatch (G7)** — goal paused. `scripts/cluster/dispatch.sh` self-describes as a "task dispatch shim" — **243 lines, 0 `ssh`/`scp`/`rsync`** (verified `grep -cE = 0`): executes locally, placeholder for real nodes. Companion plans all design-grade. The "dispatch shim placeholder-becomes-real" pattern is documented in `agentic-script-design`. → **LEAVE** (deliberate placeholder; revive when real nodes arrive). *Confirmed.*

22. **Local-AI mass-wielding (G12 single-Mac · G13 multi-Mac swarms)** — both goals paused; pin P-010 + audit A33 frame an install/research initiative. **Confirmed nothing built:** `which ollama` → not found; zero `*local-ai*`/`*ollama*`/`*swarm*` scripts. Entirely install/hardware-gated. → **LEAVE** (user/hardware gate, zero agent work possible). *Confirmed.*

23. **Multi-orchestrator slam-dunk wave (G16)** — paused; "design-complete; waiting on Anthropic subscription + M2 setup." 5 SLAM-tier integrations designed, un-coded, gated on a **2nd Anthropic Max subscription (a user purchase) + M2 hardware**. → **LEAVE** (pure user gates per G17 triage item 12). *Confirmed via paused goal + proposal docs.*

24. **BG-coordination memory-mirror substrate (dual-stream expansion)** — `proposals/memory-mirror-dual-stream-and-bg-coordination.md`, user-flagged "its a slam dunk to expand on later... MUST persist." The dual-STREAM half was half-built as a migration byproduct; the load-bearing slam-dunk (memory-mirror as a SHARED BG/spawned-session coordination substrate — every BG appends state so the orchestrator sees them, no chip-cascade, no freeze-in-the-dark) is "NOT yet designed (NEW as of 2026-05-31)." Parked behind G20 — **but G20 is DONE, so its gate has lapsed without the build starting.** MEMORY.md still carries `project_memory-mirror-dual-stream-proposal.md` "DO NOT START (behind G20)" — **now stale.** → **BUILD** (gate lapsed; user explicitly wants this — surface it). *Confirmed proposal exists, G20 closed.*

25. **Seatbelt OS-sandbox for fearless auto-mode (safe-startup, Layer 2)** — `proposals/safe-agent-startup-and-fearless-automode.md`, proposed. Tradeoff table marks the sandbox row "❌ (the missing piece)." Layer 1 (root AGENTS.md) DID land. **Confirmed:** no seatbelt/sandbox-exec/bubblewrap wiring in either settings file or `scripts/`. Proposal's own caveat: a strict sandbox would block legit cross-repo work. → **LEAVE** (genuine deliberate-decision item, not an oversight). *Confirmed.*

### scheduler-holding (6) — real subsystem, idle/holding (no tick driver, or gated)

*(See the dedicated holding-map below for the full table. All 6 → **LEAVE** — correctly idle, not broken.)*

26. **Multi-change scheduler — 22 HOLDING at proposed, 0 in-flight, no tick driver.** → LEAVE.
27. **7-8 parallel-capacity band — genuinely EXHAUSTED, no cron driving `band_check`.** → LEAVE.
28. **research-furthering daemon (autonomic Daemon #1) — NOT installed in launchd.** → LEAVE (A66-gated install).
29. **Tiered heartbeat `tick.sh` (`com.evium.heartbeat.plist`) — never installed (BY DESIGN — A74-D1 SPOF fix).** → LEAVE.

*(items 26-29 are the scheduler-holding entries; #27/#28/#29 detail in the holding-map.)*

### memory-gap (9) — live load-bearing capability with 0 recall path

**All 9 → ADD-MEMORY-POINTER** (one-line index pointer in the CANONICAL auto-memory `MEMORY.md`, per
`feedback_memory-mirror-sync.md` — edit the canonical copy only; the sync mirrors it). **Confirmed: `grep -ril`
over the whole memory dir = 0 for every term below.**

30. **`agentic-flutter-development` (skill)** — full cross-platform skill on disk; **0** memory files name flutter/dart/riverpod; `feedback_skills-split.md` tables only 6 skills and predates it. A skill nobody recalls is a skill nobody invokes. → ADD-MEMORY-POINTER.
31. **`agentic-context-paging` (skill)** — the MMU/residency meta-discipline (even names MEMORY.md's 24.4KB overflow); **0** memory files name it. → ADD-MEMORY-POINTER.
32. **`agentic-design-handoff` (skill)** — governs direct-to-design prompts; **0** memory files name it. `reference_direct-to-design-folder.md` points at the FOLDER but never the SKILL that drives it. → ADD-MEMORY-POINTER.
33. **`scripts/event-bus/` (dispatch subsystem)** — live pub/sub: `dispatch.sh`+`dispatch.py`+8 consumers; wired (`event-bus-dispatch-trigger.sh` in settings; `heartbeat-independent-tick.sh` calls in). **0** memory hits for event-bus. `feedback_autonomic-system.md` covers daemons but never names the bus that wires them — the coordination substrate is invisible. → ADD-MEMORY-POINTER. *(NB: per #14 the bus is currently a dry pipe — the pointer should note "substrate wired, producers not yet emitting.")*
34. **`scripts/backup-sensitive.sh`** — "TOTAL FAILSAFE snapshot of the memory + mirror BEFORE any edit. Run before touching memory." **0** memory refs, not wired, xref=0. A load-bearing safety tool with no recall path → won't run when it matters. → ADD-MEMORY-POINTER (**high blast radius**).
35. **`scripts/migrate-memory-on-move.sh`** — the MOVE HANDLER for the out-of-repo memory dir (when repo path changes, the macOS slug changes and MEMORY.md silently goes BLANK — "the memory-slug time bomb"). `reference_repo-geography.md` is its natural home but doesn't name it. **The single worst thing to forget given memory IS the brain.** → ADD-MEMORY-POINTER (**highest blast radius**).
36. **`scripts/routing-linter.sh` + `scripts/cross-link-maintainer.sh`** — two brand-new (2026-06-07) convention enforcers. routing-linter validates artifact-routing ("what IS this → where it lives"); cross-link-maintainer maintains journal↔goal-billboard traceability (wired into `run-tests.sh`). Neither in settings; `feedback_goal-billboard.md` mentions no routing/traceability terms. A whole new subsystem, 0 memory pointer. → ADD-MEMORY-POINTER. *(NB: real paths are `scripts/` root, NOT `scripts/hooks/` as the finding stated — corrected here.)*
37. **`scripts/daemon-health.sh`** — per-daemon liveness ("loaded yet dead/stuck/crash-looping"; on 2026-05-27 daemons 2-5 sat dark ~2 days). **0** memory hits; xref shows it IS called by the wired `sessionstart-services-digest.sh` (so not fully orphaned at runtime) — but the LESSON ("loaded != working") has no recall path → the dark-daemon failure can recur. → ADD-MEMORY-POINTER. *(Correction: finding said xref=1; it is referenced by a wired hook + a test — runtime-live, memory-blind.)*
38. **`scripts/extract-universal-memory.sh` + portable-kit** — assembles the PORTABLE universal-memory bundle (the mechanism to carry this OS's memory to another project). **0** memory hits, not wired, xref=0. Load-bearing for portability, no recall path. Lower blast radius (port-time only) → a single index pointer suffices. → ADD-MEMORY-POINTER.

---

## Schedulers & resource-handlers holding-map

The recurring shape: **the work-tracking substrate is alive and correct, but nothing drives a tick** (no
launchd timer, no cron) — so backlogs sit, and the *health monitors that would notice are themselves
event-gated on empty event files, so they silently exit.* None of these is broken; all are **LEAVE** (idle-
correct or user/hardware-gated). The systemic gap is **no autonomous tick driver** + monitors that can't see
an empty queue.

| Subsystem | State now (verified) | Driver present? | Why holding | Recommendation |
|---|---|---|---|---|
| **Multi-change scheduler** | proposed 22 · scheduled 0 · **in-flight 0** · verified 20 · wontfix 3; "no events yet"; registry absent; only `main` worktree | **NO** launchd, **NO** cron (`crontab` empty); only the WARN-only `scheduler-tick-drift-warn.sh` (fires *on* a manual tick, doesn't *schedule* one) | No tick driver; `check-scheduler-health.sh` Part-1 reads empty `scheduler-events/` → silently exits → backlog invisible | **LEAVE** (holding is correct without a driver; the user gates any launchd/cron install — A66) |
| **7-8 parallel-capacity band** | `band_check.py` → **EXHAUSTED** "no genuine research left" + "SELF unknown"; ledger 3 lifetime entries (2 of them written by the inspectors' own read) | **NO** cron (docstring says "each cron fire runs THIS" — but `CronList` empty) | EXHAUSTED + "SELF unknown" is the **BY-DESIGN** verdict (floor 7/ceiling 8); orchestrator should PushNotification+CronDelete but there's no cron id. Killswitch + pivot flags ABSENT | **LEAVE** (idle-correct, not gated, just unfed — fed only by an in-session agent) |
| **research-furthering daemon (#1)** | script + plist.template exist; **NOT in launchctl**, **no plist installed**. Output `research/systems/` IS fresh (agent-side); reader `never-idle-refill-surface.sh` IS wired | **NO** (launchd timer uninstalled) | Never-idle is carried by the agent-side `research-furthering` skill (in-session `parallel()` refill). Between sessions the floor is unenforced — the exact "discipline-fragile → structure-permanent" gap | **LEAVE** (`launchctl` install is A66 user-gated) |
| **Tiered heartbeat `tick.sh` (`com.evium.heartbeat.plist`)** | `tick.sh` + 4 `check-*.sh` exist; LaunchAgent **never installed**; status.sh → all tiers "never run" | **NO** (replaced) | **BY DESIGN**: the A74-D1 SPOF fix replaced the scheduler-coupled tiered Heart with the independent 10-min tick. The 4 hourly-tier `check-*.sh` (incl. scheduler-stuck detector) therefore never fire on a timer | **LEAVE** (vestigial/manually-invokable by design) |
| **Independent heartbeat (`com.agentic-os.heartbeat-independent`)** | **LOADED, exit 0, firing every 10min** (ticks 23:52→00:32, fresh beacon, err log EMPTY) | **YES** (600s StartInterval) | Healthy — listed for contrast. The ONLY live cadence. *status.sh mis-reports it (see built-broke #9/#15)* | (healthy — fix the reporter, not the Heart) |
| **Event-bus + pending-queue** | both jsonl files **absent**; cursor 0→0 every beat; drain is a no-op every prompt | producers exist in code, **0 published** | Dry pipe — see built-broke #14 | (covered under #14 — **FIX**) |

**Cross-cutting holding-map recommendation (do NOT auto-apply — A66-gated):** the scheduler backlog (22) and
the never-idle floor both lack an autonomous driver. The *correct* driver is the **Heart tier-medium consumer**
(never-built #20) firing `scheduler tick` + `band_check` on a cadence — which is exactly why #20 is the
keystone build. Until then, **leave** (in-session agents drive both; nothing is broken, just unfed). Also fix
`check-scheduler-health.sh` so a backlog with an EMPTY events file is still visible (today it silently exits).

---

## The standing / auditable part — make this sweep continuous, not one-time

### What already exists (so we extend, not duplicate)
- **`hook-usage-stats.sh --dead-only`** (wired SessionStart) — lists ZERO-fire hooks. **But it's broken (#12):** instruments only `scripts/hooks/*.sh`, over-reports root hooks as dead, and only sees WIRED hooks (so true orphans like #1-#7 are **invisible** to it). Covers a different axis (fire-frequency of wired hooks), not orphan-existence.
- **`scripts/hooks/convention-f-closure.sh`** (A83) — designed precisely to flag NEW scripts landing with no caller. **It is itself unwired (#3).** This is the natural seed.
- **`scripts/routing-linter.sh`** — validates artifact *routing*, not code-reachability. Adjacent, not overlapping.
- **`run-tests.sh`** — already contains an "orphan-test anti-pattern" guard philosophy (lines 505/524/533/546/564/575: tests that don't RUN become orphans) — the same reflex, applied to tests not scripts.

**Gap:** nothing continuously detects a **script on disk with 0 wires AND 0 live callers** (the exact class
that produced items #1-#8). `--dead-only` can't (it only knows wired hooks); `convention-f-closure` could but
is dormant.

### Proposed standing orphan-auditor (concrete design)

**`scripts/orphan-sweep.sh`** — a periodic reachability auditor, fired two ways:

1. **As a Stop hook (cheap, per-session) — by WIRING `convention-f-closure.sh` (#3) and folding the orphan
   check into it.** On Stop, for every `scripts/**/*.{sh,py}`, compute reachability:
   `wired?` = appears in `settings.json` hook commands  **OR**  `called?` = a non-comment, non-self,
   non-test `bash …/X` / `python -m …` / `source` reference exists in another script  **OR**
   `gated-staged?` = present in `settings-patches/*.json` or `settings.proposed.json` (pending the user).
   A script that is none of the three → **candidate orphan** → append one line to
   `.claude/cache/alignment/verdict.jsonl` (`lobe:"orphan-implementation"`, `acted:false`) so the **§13
   Alignment watchdog** tracks it across sessions (the same pipe this audit feeds). Honest filtering: exclude
   `tests/`, `lib-*.sh`, `*.template`, and anything tagged `# orphan-ok:` (a one-line opt-out for
   deliberately-staged/gated scripts like the A83 bundle).

2. **As an autonomic daemon (deep, periodic) — Heart tier-LOW consumer** once the tier substrate exists
   (rides on never-built #20). Weekly (or per-N-beats) it runs the full sweep, plus the **memory-recall axis**
   this audit added: for every `scripts/**` + `.claude/skills/*`, `grep -ril` the term across the canonical
   memory dir; **0 hits on a load-bearing capability → "memory-gap" finding** → same verdict.jsonl pipe.
   Output `reports/orphan-sweep/<run-id>/orphans.json` (rotating run-id — and fix #13's frozen-run-id while
   here so this one actually rotates). Surface the delta (new orphans since last run) in the SessionStart
   digest.

**Why a daemon AND a hook:** the Stop hook catches a *new* orphan the moment it lands (prevents accumulation —
this is convention-f-closure's original A83 intent, finally armed); the tier-low daemon catches *drift*
(a script whose only caller was later deleted → silently orphaned) + the memory-recall axis. Together they
make orphan-detection **living and auditable** instead of a one-time sweep that rots.

**Prerequisite:** fix `hook-usage-stats --dead-only` (#12) first — an orphan-auditor that feeds off a
miscounting instrumentation ledger inherits the lie. The two are complementary axes (fire-frequency vs
reachability) and should agree.

**A66 note:** wiring `convention-f-closure.sh` into `settings.json`, and installing any tier-low/daemon plist,
are **user-gated**. Surface as part of the A83 build-but-surface gated bundle (#8) — one yes/no for the user.

---

## Stale-doc corrections surfaced by this sweep (one-line each — for a follow-up)
- **MEMORY.md `memory-bloat-guard` line** — "Enforced by ... memory-bloat-guard.sh" is STALE; live enforcer is `test-memory-within-limit.sh`. (#4)
- **MEMORY.md `project_memory-mirror-dual-stream-proposal.md` line** — "DO NOT START (behind G20)" is STALE; G20 is closed, gate lapsed. (#24)
- **MEMORY.md `feedback_impact-domino-analysis.md` line** — "memory-only, not yet wired" — the script half (#5) partially landed (unwired); only the persistent daemon (#19) is truly never-built. (#5/#19)
