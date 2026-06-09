# Adaptive-immunity retro-sweep roadmap — finishing what A71 started

**Created:** 2026-05-28 (timestamp from system clock at authoring)
**Author context:** A74-D5 retro-sweep follow-up. Read-only investigation; this is the one output file.
**Scope:** the REMAINING 6 of the 9 "make-this-impossible" instances. A71 is the only one at 4/4 (the template). A45 + A56 are being closed separately RIGHT NOW (other concerns own them — untouched here).

---

## Why this roadmap exists

Per `feedback_adaptive-immunity-discipline`, every "make this impossible" / "never again" failure must mutate across **HARDWARE** (immutable barrier) + **SOFTWARE** (executable guard) + **BIOLOGY** (memory/audit/ref) AND carry a **VERIFY** test that fails if the failure can recur. The acid test: **all 3 tethers + VERIFY = durable; any missing = recurrence inevitable.**

A62 retroactively mapped 9 instances and found a **systematic hardware-tier gap** (0/7 deployed at the time) and a **systematic VERIFY skip**. A74-D1 re-confirmed: "only A71 reached 4/4; A40/A44/A45/A47/A48/A56/A57/A61 have ZERO→partial tether coverage; backfill ~89% incomplete." A74-D5 named the live danger: the RESOURCE-LEAK class (A56) recurred *after* its fix was declared landed — proof that skipping VERIFY makes "landed" a lie.

A71's template (`tests/inbox-never-lost-mutation-test.sh`) is the existence-proof for how to close one of these: a sandboxed test that (1) reproduces the OLD-design loss to prove the bug class was real, then (2) asserts the NEW design protects, with the test failing BOTH if the mutation regresses AND if it was never applied.

This roadmap turns "6 instances still open" into a clear, ground-truth-verified, leverage-ranked queue.

---

## Ground-truth verification (filesystem, 2026-05-28 — supersedes A62's 2026-05-27 map)

A62's map is ~24h stale and was partly aspirational. I verified what is ACTUALLY on disk + wired in `settings.json` today. Key deltas from A62:

- **A61 advanced past A62's record**: `scripts/inbox-write.py` wrapper + `scripts/hooks/inbox-schema-validate-pre.sh` are now BUILT and WIRED (settings.json line 188). BUT the hook is **WARN-mode only** (`mode:"dry-run"`, `exit 0` always) — it is *not yet* a hardware barrier (a real barrier blocks the write). And `tests/hooks/test_monkey_hygiene.py` exists but **all 13 assertions are `@skip`** — it verifies nothing today.
- **A40 regressed / was never built**: A62 listed `detect-idea-gaps.py` + SessionStart hook as the software tether. On disk today: **only the plan + memory rule exist.** No `detect-idea-gaps.py`, no `idea-gap-surface.sh`, no `idea-gap-live-check.sh`, no `gaps-pending.jsonl`. A40 is **biology-only** in reality.
- **A47 is the strongest of the six**: `dispatch-bg.sh` + `bg-auto-stop.sh` (wired) + `bg-stuck-warn.sh` (wired) + memory rule + `test_bg_stuck_warn.py` (9 tests, real). BUT `bg-auto-stop.sh` defaults to **dry-run** (no kill) and the test covers the WARN hook's detection logic, NOT the SIGTERM/auto-stop kill path or the sentinel contract.
- **A48 has a real VERIFY already**: `test_sync_monkey_decisions.py` exists (the only one of the six besides A47 with a genuine, non-skipped test). No dedicated memory rule (A62 ruled it folded under the adaptive-immunity umbrella — acceptable).
- **A57 software + biology solid; hook never wired**: `dashboard-content-lint.py` + `post-to-chamber.py` + memory rule all present. The proposed `chamber-post-lint-pre.sh` PreToolUse gate is **not built/wired** (no hardware barrier on data.json writes), and there is no content-lint-specific test.

### Tether legend
- **HW** = immutable structural barrier actually DEPLOYED + ENFORCING (a wired deny / block, a `--dry-run` default that requires explicit `--apply`, a settings deny-list). WARN-mode hooks and dry-run scripts do NOT count as HW until they block.
- **SW** = executable guard that runs (wrapper / linter / hook / detector script) and actually exists on disk + (where relevant) wired.
- **BIO** = persistent memory: `feedback_*.md` rule + audit catalog entry + skill ref.
- **VERIFY** = a test that FAILS if the failure can recur and PASSES only when the mutation prevents it (A71-shaped). Skipped/aspirational tests do NOT count.

---

## The ranked queue

| Instance | Failure class | Current tethers | Missing (to 4/4) | Proposed VERIFY test (A71-shaped) | Effort | Leverage (still hurting?) |
|---|---|---|---|---|---|---|
| **A61** | INBOX-CLOG — schema-invalid files enter inbox + stick, clogging the monkey/banana communication chamber | **2.5/4** — BIO ✅ (audit + recurrence-detect ref) · SW ✅ (`inbox-write.py` wrapper + `inbox-schema-validate-pre.sh` wired) · HW ½ (hook wired but WARN-mode, blocks nothing) · VERIFY ✗ (13 monkey-hygiene tests all SKIPPED) | **Flip hook to BLOCK** (after dry-run review) = real HW barrier; **un-skip the 13 tests** = VERIFY | Un-skip `test_monkey_hygiene.py`: assert a schema-invalid direct Write is REJECTED in block-mode AND that a stub-without-required-fields cannot remain unflagged through a heartbeat scan. Must reproduce the OLD clog (file enters + sticks) then assert NEW design blocks/recovers. | **medium** | **HIGH** — A74-D5 lists INBOX family as the most-recurring class (5×). A71 closed the *loss* sub-class; the *clog* sub-class (A61) is the sibling and its barrier is still WARN-only. Live exposure remains. |
| **A47** | GHOST-BG / SERVER-DEATH — dispatcher BG runs indefinitely (349m ghost) burning compute + stealing A21 slots | **3/4** — BIO ✅ (memory rule + audit + 2 skill refs) · SW ✅ (`dispatch-bg.sh` + `bg-auto-stop.sh` + `bg-stuck-warn.sh` wired) · HW ½ (auto-stop is dry-run; `dispatch-bg.sh` injects timeout = soft barrier) · VERIFY ½ (`test_bg_stuck_warn.py` covers WARN detection, NOT the kill path / sentinel) | **Flip `bg-auto-stop.sh` to live** (post dry-run) = HW (real SIGTERM barrier); **extend test to the kill path** = full VERIFY | New `tests/bg-auto-stop-mutation-test.sh`: synth a ghost (launched, no `completed_at`, artifacts present, >2h, idle >30m) in a sandbox `bg-dispatch-log.jsonl`; assert dry-run LOGS the intended kill AND live-mode emits SIGTERM (mock kill) + writes `status:auto-stopped`. Reproduce OLD (ghost survives forever) → assert NEW stops it. Also assert the 3/hr rate-limit + opt-out sentinel. | **medium** | **HIGH** — SERVER-DEATH flagged "fragile" in A74-D5; ghosts cost real compute every session + collide with the chant ("testing/compute must never bottleneck"). Closest to done, so cheapest HIGH win. |
| **A57** | DASHBOARD-CONTENT — agent ships wall-of-text / jargon / markdown-to-non-markdown-surface to a user-facing field (1500-word chamber post) | **2/4** — BIO ✅ (memory rule + 2 skill refs) · SW ✅ (`dashboard-content-lint.py` + `post-to-chamber.py`) · HW ✗ (proposed `chamber-post-lint-pre.sh` never built/wired; nothing blocks a raw Write to `data.json`) · VERIFY ✗ (no content-lint test) | **Build + wire `chamber-post-lint-pre.sh`** PreToolUse blocking lint-fail on `data.json` writes = HW; **add lint test** = VERIFY | New `tests/content-lint-mutation-test.sh`: feed the ORIGINAL 1500-word B-K7-c-FOLLOWUPS body → assert linter exits non-zero (reproduces the bug: over-length + jargon + literal `===`). Feed a compliant ≤150-word body → assert exit 0. Then assert the PreToolUse hook BLOCKS the over-length Write to `data.json` and ALLOWS the compliant one. | **medium** | **MED-HIGH** — directly user-visible ("absolute incoherent horrendous ui/ux"); every chamber/billboard post is exposure. Linter exists but is advisory — nothing forces it, so a future raw Write reopens the wound. |
| **A48** | SYNC-DRIFT — banana-backlog status manually flipped, drifts from delivered-decision ground truth (some vanish, some don't) | **2.5/4** — BIO ½ (audit + folded under adaptive-immunity umbrella; no dedicated rule, acceptable per A62) · SW ✅ (`sync-monkey-decisions-from-inbox.py` wired into `dashboard-data-prime.sh`) · HW ✗ (nothing prevents a manual `data.json` status write) · VERIFY ✅ (`test_sync_monkey_decisions.py` real, non-skipped) | **HW barrier**: deny direct status writes to `data.json` except via sync script (overlaps A57's `chamber-post-lint-pre.sh` zone) | Already has a real VERIFY. To reach A71-shape, add ONE assertion proving the OLD failure (manual flip leaves drift when a 2nd decision lands mid-edit) is now impossible: assert running sync after N inbox decisions yields data.json == derived-truth regardless of prior manual state (idempotent re-derive). | **small** | **MED** — same INBOX/monkey family as A61 but the *sync* sub-class already has SW + VERIFY. Mostly needs the HW barrier (shared with A57/A61). Lowest residual exposure of the family. |
| **A44** | AGENT-FINDING-SLIP — agent surfaces a critical finding mid-investigation but doesn't auto-spawn structural prevention (reports + moves on) | **1/4** — BIO ✅ (memory rule + skill ref) · SW ✗ (`critical-finding-detect.sh` never built) · HW ✗ (none) · VERIFY ✗ (none) | **Build `critical-finding-detect.sh`** (PostToolUse/Stop scan for the 11 trigger markers; surface "you found a critical-class issue but spawned no protection") = SW; **HW** via Stop-hook block-decision when markers present + no artifact landed in-window; **VERIFY** | New `tests/critical-finding-reflex-mutation-test.sh`: feed a mock agent response containing a trigger marker ("🚨 Critical finding: ...") + NO audit/plan written in the window → assert the hook surfaces/blocks (reproduces the slip). Feed the same marker WITH an artifact landed → assert silent pass. Mirror A40's gap-detector shape. | **medium** | **MED** — meta-reflex; this very roadmap is an instance of the reflex firing manually. Hardest to make structural (depends on parsing agent's own output), so leverage-per-effort is lower. 4.8's better tool-triggering partially compensates (D6). |
| **A40** | IDEA-SLIP — user offers an idea; system doesn't pick it up; no explicit skip/defer logged ("biggest slam dunk" fear) | **1/4** — BIO ✅ (memory rule + audit + plan) · SW ✗ (`detect-idea-gaps.py` + hooks DO NOT EXIST on disk — A62's map was aspirational) · HW ✗ (none) · VERIFY ✗ (none) | **Build the detector + SessionStart surface hook + `gaps-pending.jsonl` substrate** (per the existing plan) = SW; **HW** = Stop-hook that won't let a session end with an un-surfaced gap older than N turns; **VERIFY** | New `tests/idea-gap-mutation-test.sh`: synth a transcript where a prompt with A39 idea-handoff markers gets NO artifact + NO steering skip in its 5-prompt window → assert detector classifies it GAP + emits to `gaps-pending.jsonl` (reproduces the slip). Synth a serviced prompt → assert NOT flagged. A71-shaped: fails if detector absent (which it currently is). | **medium→large** | **MED** — user-named "biggest slam dunk," but the whole SW tier must be built from scratch (not just flipped/un-skipped). Highest absolute effort. The fear is real but no recent recurrence is documented, so it's important-not-urgent. |

---

## Recommended close-order (highest-leverage / still-hurting first)

The ordering principle: **(actively-recurring class) × (cheapness of the remaining gap)**, with a tie-break toward instances where the gap is "flip/un-skip" (cheap) vs "build-from-scratch" (expensive). This matches A74-D5's verdict that the INBOX family + SERVER-DEATH are the live-danger classes and A71/daemon10 are the proven fix patterns to replicate.

1. **A47 — flip auto-stop to live + extend test to the kill path.** *Closest to 4/4 (3/4 today), HIGH leverage, gap is "flip dry-run→live + add one mutation test."* Cheapest HIGH win; ghosts cost compute every session and SERVER-DEATH is flagged fragile. Replicate A71's test shape directly. **Start here.**

2. **A61 — flip schema-validate hook to BLOCK + un-skip the 13 monkey-hygiene tests.** *2.5/4, HIGHEST-recurrence class (INBOX 5×), gap is "flip WARN→BLOCK + un-skip existing tests."* The wrapper + hook + test scaffold already exist — this is unblocking dormant work, not building. Pair with A48 (shares the data.json/inbox HW zone).

3. **A57 — build + wire `chamber-post-lint-pre.sh` + add the content-lint mutation test.** *2/4, MED-HIGH (every user-facing post is exposure), gap requires building the PreToolUse hook.* The linter already exists; this wires it as a real barrier. Its HW zone (block raw `data.json` writes) overlaps A48 and A61 — **build the data.json/inbox write-guard once, claim it for A57 + A48 + A61 together.**

4. **A48 — add the idempotent-re-derive assertion + claim the shared HW barrier.** *2.5/4, MED, smallest residual.* Already has SW + a real VERIFY; only needs the one extra assertion + the shared data.json HW barrier (from step 3). Fast follow.

5. **A44 — build `critical-finding-detect.sh` + the reflex mutation test.** *1/4, MED, must build SW from scratch.* Meta-reflex; harder to make structural (parses agent's own output). 4.8's better tool-triggering softens urgency.

6. **A40 — build the detector + surface hook + substrate + mutation test.** *1/4, MED→large, entire SW tier missing.* Highest absolute effort (A62's map was aspirational; nothing is actually built). User-named fear but no documented recent recurrence — important, not urgent. Close last.

### Three cross-cutting accelerators
- **One HW barrier serves three instances.** A57 + A48 + A61 all need "block direct writes to `data.json` / inbox except via the sanctioned wrapper." Build it ONCE (per A62's deferred hardware-tier proposal + A47 Rule 5's 24h-dry-run-then-block discipline) and three instances claim it. This is the single highest-leverage move in the whole sweep.
- **Two of six already have real VERIFY tests** (A47 partial, A48 full). Three have test scaffolds that are skipped/absent (A61 skipped, A57/A44/A40 absent). The cheap wins are un-skipping/extending (A61, A47) before building from scratch (A40, A44).
- **Every VERIFY follows the A71 contract**: reproduce the OLD-design failure (proving the bug class was real) → assert NEW design protects → test fails BOTH if mutation regresses AND if it was never applied. Sandbox in a tmpdir; never touch real state.

---

## Definition of done for this backlog

Each of the 6 is "closed" (4/4, A71-grade) only when:
1. A wired HW barrier actually BLOCKS the failure path (WARN-mode and dry-run do NOT count until they block/kill).
2. A SW guard exists on disk and runs.
3. BIO is present (already true for all 6).
4. A VERIFY test exists, is NOT skipped, reproduces the old failure, and passes only post-mutation.

When all 6 reach 4/4, the adaptive-immunity backfill A74-D1 flagged as "89% incomplete" closes, and the audit catalog gains 6 more instances eligible for `verified_complete` (joining A71 as the second-through-seventh durably-closed classes).

---

BG-COMPLETE-SENTINEL
