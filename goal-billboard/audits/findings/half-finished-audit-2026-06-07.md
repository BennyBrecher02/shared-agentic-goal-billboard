---
kind: audit-findings
audit: half-finished
angle: STARTED-but-never-COMPLETED (design done / fix half-applied / "I'll do X next" / staged / PARTIAL verdicts)
date: 2026-06-07
created_ts: 2026-06-08T02:07:42Z
method: |
  grep ALL 35 session JSONLs (~660MB) for (a) my own promissory language ("I'll do X / next /
  staged / pending / design-only / awaiting / half-built / build-gated") = 1313 hits and (b) the
  user's flesh-out/finish/expand/come-back-to asks = 316 hits. Deduped to 72 strong-partial signals,
  then CROSS-REF'd every survivor against on-disk reality (Bash) + the goal-billboard + the
  MASTER-REALIGNMENT-MANIFESTO §12 spawned-work tracker + PENDING-WIRES.md + the asks-log.
constraint: |
  The user HATES phantom gaps. EVERYTHING that turned out built was demoted to "DONE — not a partial"
  with the on-disk proof. Only genuinely-still-half-done items are ranked below. This angle is DISTINCT
  from the same-day full-history-request-audit (dropped asks), session-request-audit (per-session
  delivery), and orphaned-implementations-audit (built-but-unwired code) — it hunts the
  *design-landed-but-build-didn't* / *fix-half-applied* / *PARTIAL-verdict* class specifically.
verdict_feed: .claude/cache/alignment/verdict.jsonl (lobe=half-finished)
sources:
  - /Users/bennybrecher/.claude/projects/-Users-bennybrecher-Claude-Code-agentic-organic-os/*.jsonl (35 sessions)
  - context/markdowns/MASTER-REALIGNMENT-MANIFESTO.md (§12 tracker + §13)
  - scripts/hooks/settings-patches/PENDING-WIRES.md
  - context/markdowns/plans/mega-prompt-2026-06-07-execution-tracker.md
  - context/markdowns/goal-billboard/{asks-log.md,ideas/,proposals/,plans/}
---

# Half-finished audit — 2026-06-07

## Headline

The system is in much better shape than a raw promissory-language sweep suggests. Of **1313** "I'll do X /
staged / pending / design-only" assistant hits and **316** user flesh-out asks, after dedup + on-disk
cross-ref **almost everything STARTED in the last 10 days actually LANDED.** The same-day full-history audit
already proved this for *dropped* asks; this sweep proves it again for *started-but-unfinished* work. The
big multi-goal sessions (666b8e4c, 699bce48, 23f93bfd, d19973ac) generated nearly all the promissory noise,
and their promises were overwhelmingly fulfilled in-session.

**The genuine survivors cluster into ONE high-leverage item and a short tail.** The keystone is the **§13
Alignment Immune System (AIS)** — its engine is built AND now wired LIVE, but four surrounding tethers are
unfinished *and its own status docs still say "not applied" when it IS applied* (a stale-status trap that
will make every future audit re-flag it). The rest are design-only daemons + one slam-dunk that lost its
gate + small bookkeeping staleness.

**Demoted to DONE (the would-be partials that actually shipped — proof on disk):**

| Earlier "partial" signal | Source | Reality on disk |
|---|---|---|
| `inbox-write.py` wrapper "NEVER actually built" (adaptive-immunity software arm) | 372afa3d 05-27 | **DONE** — `scripts/inbox-write.py` exists |
| before/after matrix-coverage dashboard page "aggregation half-built" | 666b8e4c 05-31 | **DONE** — `build_matrix()` wired into render model (line 3054) + `test_matrix_page.py` |
| `idea-gap-close.sh` "referenced everywhere but does not exist" (F2) | tracker 06-07 | **DONE** — built 06-07 17:59, 10.7KB |
| `findings-life-thread-surface.sh` "missing surfacer for classify-findings.py" | tracker 06-07 | **DONE** — built + wired (2 refs live settings.json) |
| `daemon-control.sh` wrapper "in flight" | tracker 06-07 | **DONE** — built 06-07 18:59, 17.6KB |
| the new GIT/rollback dashboard page (§3 Cursor-handoff unblocker) | manifesto §3 | **DONE** — parser + render model + 5 tests (A90-A93) + `public/git-rerender` |
| chip-loop fix #1 (broaden spawn-guard to 3-signal, nested→BLOCK) | tracker 06-07 PM | **DONE** — `spawn-task-guard.sh` 3-signal detection, exit 2 nested |
| chip-loop fix #2 (mid-session chip-result substrate) | tracker 06-07 PM | **DONE** — `spawned-task-result-surface.sh` reads `spawned-task-recovery.jsonl`; surfaced-log has entries |
| "THE ONE PASTE" — settings.proposed → settings.json | session-request-audit 06-07 | **APPLIED** — live `settings.json` (uncommitted, 06-07 19:24) now == proposed.json (0 diff); the 4 high-value hooks ARE wired live |

> **Why so many demotions:** the curated **asks-log** has ZERO open/partial rows, and the manifesto **§12
> tracker** shows items 1-11 all "DONE." The promissory hits are mostly *intra-turn* promises ("I'll wire X")
> that were kept the same turn. The honest still-open set is small and lives in the design-only / gated tier.

---

## Ranked findings (genuinely still half-done)

### 1. §13 Alignment Immune System — engine built + WIRED LIVE, but 4 tethers unfinished AND status docs are STALE  ·  leverage: HIGH  ·  state: PARTIAL
**What was started:** the 6th Organic-OS subsystem. The reflex-tier watchdog `scripts/hooks/alignment-sweep.sh`
(26KB, 3 lexical lobes drop-through/goal-align/intent-fidelity + self-kicking heartbeat, 12/12 VERIFY) is
built **and now wired LIVE** (2 refs in `.claude/settings.json`, SessionStart + Stop). It IS firing: `.last-kick`
is fresh (2026-06-08T02:00:36Z) and it writes verdicts (the very `verdict.jsonl` this audit feeds; 98 entries,
3 drop-through from the AIS itself). The acid test names **four tethers** (hardware/software/biology/VERIFY).

**What's LEFT (the unfinished tethers — all VERIFIED missing on disk):**
- **5th pulse vital sign (ALIGN):** `organic-os-pulse.sh` emits MEMORY / HEART / IMMUNE / AUTONOMIC — **no ALIGN sign.** The §13 design explicitly requires it ("a 5th vital sign (ALIGN) on the SessionStart pulse"). MISSING.
- **Canonical memory topic `feedback_alignment-immune-system.md`:** MISSING in BOTH the canonical auto-memory AND the mirror; 0 hits in MEMORY.md. So the whole 6th subsystem has **no recall path** — the same memory-gap class the orphan audit flagged for 9 other capabilities, but for the keystone alignment pillar itself.
- **Tier-2 LLM-as-judge sweep:** CONFIRMED a STUB — `alignment-sweep.sh` only *references* "run the Tier-2 judge" as a suggested action (lines 451, 570); no actual agent-side LLM-judge sweep exists. Intent-fidelity (lobe C) therefore can only lexically *suspect*, never *confirm*.
- **launchd free-tick:** not installed (user-gated — correctly so).

**The STALE-STATUS trap (this is what makes it insidious):** THREE canonical docs still say the wiring is NOT applied when it IS:
  - Manifesto **§13** header: "STATUS: PARTIAL (engine + surfaces built WARN-only; ... launchd user-gated)" + body "Built so far (WARN-only, **unwired**)" — wiring IS now live.
  - Manifesto **§12 item 12**: "GATED remainder PROPOSED not applied: settings wiring" — IS applied.
  - **PENDING-WIRES.md**: items 1-4 (incl. alignment-sweep) still sit under `## PENDING`; the `## Applied` section is EMPTY — they were pasted but never moved.

**Concrete finish-step:** (a) move PENDING-WIRES items 1-4 → `## Applied` with the 06-07 date; (b) flip
manifesto §13 header + §12 item-12 to reflect "wired live; remainder = ALIGN pulse sign + memory topic +
Tier-2 judge + launchd"; (c) add the ALIGN 5th vital sign to `organic-os-pulse.sh` (reads
`verdict.jsonl` drop-through/goal-align freshness + `.last-kick` age — CPU-only, mirrors the existing
4 signs); (d) write `feedback_alignment-immune-system.md` into the CANONICAL memory + a one-line MEMORY.md
pointer (backup-first per memory-mirror-sync); (e) the Tier-2 LLM-judge + launchd stay user-gated.
**Why HIGH:** this is the subsystem whose entire job is "no request falls through the cracks" — and it
itself has a dangling memory-gap + a stale status that guarantees re-flagging + a missing pulse sign that
means a dead watchdog wouldn't show on the SessionStart strip (defeating its "loud-not-silent" invariant).

### 2. The 30-row AIS change-list across 9 canonical surfaces — designed, build-gated, never applied  ·  leverage: HIGH  ·  state: design-only / build-gated
**What was started:** the AIS manifesto pillar's *companion* — a 30-row change-list spanning 9 canonical
surfaces (manifesto pair, memory, Organic-OS READMEs, dashboard render-model/builder/parsers/HTML, the 5th
pulse vital sign, Obsidian, AGENTS.md/CLAUDE.md, an audit). The tracker (06-07): "Drafted; touches canonical
surfaces → needs your green-light before I apply."
**What's LEFT:** the bulk of the 30 rows. The SAFE surfaces (alignment-sweep + manifesto/checklist/AGENTS/
CLAUDE/dashboard *mentions*) landed; the canonical-surface edits (memory topic — see #1, the dashboard
render-model NEEDS-YOU bucket + HEALTH tile, the 5th pulse sign — see #1, Obsidian lens) did not.
**Concrete finish-step:** surface the 30-row list to the user as ONE gated yes/no batch; the rows that
overlap #1 (memory topic, ALIGN sign) can be folded together. Distinct from #1 in that #1 is the *minimum*
to stop the stale-status bleed; #2 is the *full* canonical-surface integration the user explicitly asked for
("make this into a master realignment manifesto... come back to as much as possible").

### 3. Request/RH → goal-board pipeline — design ✅, BUILD pending (F4 domino unwired is the live残)  ·  leverage: MEDIUM-HIGH  ·  state: design-only (2 point-fixes shipped)
**What was started:** mega-prompt Part 4 — a reliable redesign of the request/research-hole → goal-board
pipeline so ideas stop "falling through." Design landed (`research/systems/request-to-goalboard-pipeline-design-2026-06-07.md`,
13-type artifact dictionary + 4-gate promotion ladder). Two safe point-fixes shipped (F1 widen closure scan,
F2 `idea-gap-close.sh`).
**What's LEFT:** the full data-model build + **F4: `domino-check.sh` + `domino-consume.sh` still unwired**
(VERIFIED 0 refs in both settings.json AND settings.proposed.json) — the tracker itself marked F4 "✅ safe to
do now," and it was NOT done. The producer enqueues `unscored` stubs nothing drains. (This overlaps the
orphan audit's #5, but here the angle is: it was *explicitly promised as a safe same-session fix and left
half-done.*)
**Concrete finish-step:** wire `domino-check.sh` (producer) + `domino-consume.sh` (consumer) as a UNIT into
settings.json (gated paste) — or, if the impact-domino daemon (#19 in orphan audit) is the intended home,
fold both into the Heart tier-medium consumer build. The full pipeline data-model is build-gated on the
same canonical-surface sign-off as #2.

### 4. `hook-usage-stats --dead-only` instrumentation gap — fix "in flight," not landed  ·  leverage: MEDIUM  ·  state: half-applied
**What was started:** the tracker lists "hook-usage-counter" under "🏗 in flight" (06-07 PM). The orphan
audit (#12) diagnosed it: `instrument-hooks-usage.sh` instruments **only `scripts/hooks/*.sh`** (HOOKS_DIR),
missing the ~12 SessionStart/Stop hooks in `scripts/` ROOT → `--dead-only` over-reports live hooks as dead.
**What's LEFT (VERIFIED):** `instrument-hooks-usage.sh:52` still hardcodes `HOOKS_DIR="$REPO/scripts/hooks"`
and globs only that dir — the root-hook coverage was NOT added. The fix is started-in-intent, unwritten-in-code.
**Concrete finish-step:** extend `instrument-hooks-usage.sh` to also instrument the wired root hooks (derive
the wired set from `settings.json` rather than a fixed dir), OR change `--dead-only` to read the wired-hook
set from settings.json. **Prerequisite for trusting any future orphan/dead-hook sweep** (an auditor reading a
miscounting ledger inherits the lie).

### 5. memory-mirror dual-stream BG-coordination substrate — slam-dunk, gate LAPSED, build never started  ·  leverage: MEDIUM  ·  state: design-only (gate stale)
**What was started:** the user explicitly flagged this "its a slam dunk to expand on later... MUST persist"
(`proposals/memory-mirror-dual-stream-and-bg-coordination.md`). The dual-STREAM half was half-built as a
migration byproduct.
**What's LEFT:** the load-bearing slam-dunk — memory-mirror as a SHARED BG/spawned-session coordination
substrate (every BG appends state so the orchestrator sees them; no chip-cascade, no freeze-in-the-dark) —
is "NOT yet designed." Parked behind G20, **but G20 is now CLOSED** so the gate has LAPSED without the build
starting. MEMORY.md still carries the stale "DO NOT START (behind G20)" pointer.
**Concrete finish-step:** the gate lapsed → surface to the user that this slam-dunk is now unblocked; design
the deliberate dual-stream model + the BG-coordination append schema; fix the stale MEMORY.md pointer.
(Overlaps orphan audit #24 — same item, here framed as "user-flagged slam-dunk whose gate lapsed unnoticed.")
**Note:** this directly addresses the 06-07 chip-loop incident root cause (a successful chip was invisible to
the orchestrator) — it is the structural fix the tracker folded into the AIS design but never built.

### 6. agentic-script-design → agentic-architecture rename — deferred "post-deadline," never re-raised  ·  leverage: LOW  ·  state: deferred-decision (orphaned)
**What was started:** the rename was flagged 05-25 ("Real rename deferred to post-deadline") and is open as
monkey-chamber RENAME-001. The SKILL.md itself admits "Name is legacy... actual scope is broader system
architecture (rename candidate `agentic-architecture`)" and "Decision deferred to user."
**What's LEFT:** the actual call + (if yes) the rename. The deadline it was deferred past has long passed; the
decision has sat unresolved across ~13 days with the skill self-documenting the mismatch on every load.
**Concrete finish-step:** make the call (rename or explicitly keep-the-name + close RENAME-001). Per
decision-finality discipline, a deferral whose trigger ("post-deadline") has fired should be resolved, not
left dangling. LOW leverage (cosmetic) but it's a textbook "deferred-and-forgotten" partial.

### 7. Standing "report-don't-spawn" line in EVERY spawn prompt (chip-loop fix #3)  ·  leverage: LOW  ·  state: functionally-covered, literally-incomplete
**What was started:** the third chip-loop fix — a standing line in every spawn-task prompt forbidding recursive
spawning.
**What's LEFT:** `spawn-task-guard.sh` now hard-BLOCKS nested spawns (exit 2) and prints the rule, which
*functionally* enforces it. But there is NO line injected into the spawn-prompt template itself (the literal
deliverable as framed). The guard supersedes the need, so this is borderline-done.
**Concrete finish-step:** either (a) declare it covered-by-the-guard and close it, or (b) add the one standing
line to the spawn-prompt convention (belt-and-suspenders). Recommend (a). Flagged only for completeness — the
behavior IS guarded.

---

## Design-only daemons (correctly deferred, NOT bugs — flagged so they don't re-surface as "started")

These were each "designed but not built" and that is the CORRECT state (gated on the un-built Heart
tier-medium consumer or on user authorization). They are NOT half-finished oversights — listing them so a
future sweep doesn't re-flag them:
- **research-furthering daemon** — design-only, no consumer; the agent-side `research-furthering` skill carries the in-session floor; launchd install is A66 user-gated. LEAVE.
- **snapshot-drift-scanner daemon** — design-only; orphan audit recommends RETIRE (near-zero value until `.snapshots/` has baselines). RETIRE the design.
- **A69 Phase-3 25-orphan-hook wave-wire** — the inventory/plan is COMPLETE (design done by intent); the plan explicitly scopes itself as "does NOT execute the wiring" (user-gated per A66). The *plan* is finished; the *wiring* is a separate gated initiative, not a half-done deliverable.

---

## Meta-finding — the stale-status trap is the real disease here

The single recurring failure mode this angle surfaced is NOT abandoned work — it's **status docs that lag the
build**. The AIS (#1) is built + wired + firing, yet THREE canonical docs (manifesto §13, §12 item-12,
PENDING-WIRES "Applied") still describe it as "not applied / unwired." The memory-mirror slam-dunk (#5) has a
stale "behind G20" gate after G20 closed. Every one of these will make the NEXT audit (and the AIS's own
drop-through lobe) re-flag completed work as "still open" — manufacturing exactly the phantom gaps the user
hates. **The cheapest highest-value action across this whole audit is a status-reconciliation pass:** when a
gated thing gets applied, flip its status in the manifesto + PENDING-WIRES + MEMORY.md in the SAME turn.
(This is itself an argument for the AIS intent-fidelity lobe + a "status-drift" check — I-004
audit-status-drift-sweep already exists as an idea; this audit is direct evidence it's worth building.)

## Closure-path note
None of these is Claude-auto-applicable without a user gate EXCEPT: #1(a) PENDING-WIRES bookkeeping move,
#1(b) manifesto status flips, #4 instrumentation extension, #5 stale MEMORY.md pointer fix, and #6/#7 the
decisions — those are safe doc/script fixes. #1(c) ALIGN pulse sign + #1(d) memory topic + #2 the 30-row
canonical-surface batch + #3 domino wiring + #5 the design are the build-gated remainder.
