---
kind: audit-findings
audit: deferred-decisions (the DANGLING-DEFERRAL angle)
angle: a DECISION the user deferred or asked me to punt ("revisit when X" / "park it" / "decide later" / "behind GX" / "for now X but…" / "monkey-chamber it") whose revisit-TRIGGER has since FIRED but was never revisited
date: 2026-06-07
created_ts: 2026-06-08T02:27:26Z
method: |
  grep ALL 35 session JSONLs (~491MB, 2026-05-21 → 2026-06-07) for the deferral register
  (defer/park/revisit/behind-GX/decide-later/for-now/punt/table/come-back/post-deadline/
  hold-off/monkey-chamber/dont-build-yet/snooze/TBD). 3125 raw hits → noise-filtered + scored
  (user-voice + conditional-trigger weighted) → 474 high-signal, 159 trigger-bearing, 98 with an
  explicit conditional trigger. Then for EACH strong candidate: did its TRIGGER/condition FIRE and
  get resolved, or is the condition past with no revisit? Cross-ref'd every survivor against on-disk
  reality (Bash) + the goal-billboard (active/archived/paused/ideas/pins) + the closed goals
  (esp. G20) + the steering-decisions-log + deferred-followups.md + verdict.jsonl.
constraint: |
  The user HATES phantom gaps. EVERYTHING that turned out resolved / cleanly-gated / already-tracked
  was demoted with the evidence. The proven failure mode (this session) = the dual-stream proposal sat
  "behind G20" long after G20 CLOSED — the revisit-trigger fired, nobody revisited. This audit hunts
  that exact class: TRIGGER-FIRED-BUT-NOT-REVISITED.
dedup: |
  DISTINCT from the same-day siblings. The dual-stream (#5) and the agentic-script-design→architecture
  RENAME-001 (#6) are ALREADY in verdict.jsonl from half-finished-audit; G19-behind-G2 + the parked
  goals are ALREADY in stale-goals-triage-2026-06-07.md. Those are cross-referenced here (Section C),
  NOT re-fed. Only NET-NEW dangling deferrals are ranked + fed to verdict.jsonl.
verdict_feed: .claude/cache/alignment/verdict.jsonl (lobe=dangling-deferral)
sources:
  - /Users/bennybrecher/.claude/projects/-Users-bennybrecher-Claude-Code-agentic-organic-os/*.jsonl (35 sessions)
  - context/markdowns/notes/deferred-followups.md (the user's explicit parked-with-trigger ledger)
  - context/markdowns/notes/steering-decisions-log.md (the ad-hoc deferral ledger)
  - context/markdowns/goal-billboard/{active,archived,paused,ideas,pins}/
  - context/markdowns/goal-billboard/archived/achieved/G20-repo-decouple-ownership-emergency.md
  - context/markdowns/goal-billboard/stale-goals-triage-2026-06-07.md
  - context/markdowns/goal-billboard/audits/findings/half-finished-audit-2026-06-07.md
---

# Deferred-decisions audit — 2026-06-07 (the dangling-deferral angle)

## Headline

The deferral register is **far healthier than the raw 3125 hits suggest** — most "defer/park/post-deadline"
language was either (a) clean technical phrasing, (b) a decision whose trigger has NOT yet fired (correctly
parked, watched), or (c) already surfaced by a same-day sibling audit. **After cross-ref, only THREE genuine
net-new dangling deferrals survive** — a decision whose revisit-condition has demonstrably PASSED and was
never revisited.

**The keystone is D1: the user-filed `deferred-followups.md` report-pruning TODO.** The user explicitly wrote
the trigger criteria himself ("revisit when `reports/` exceeds 500 MB") and said "do not re-raise until a
trigger fires." **`reports/` is now 1.9 GB (3.8× the trigger) and `prune-reports.sh` was never built.** This
is the cleanest possible dangling deferral: a numeric trigger the user set, fired hard, with the discipline
(`feedback_decision-finality.md`) explicitly telling us to keep silent until it fires — and it fired.

**The meta-finding mirrors the sibling half-finished audit:** the disease isn't abandonment, it's that
**status/triggers go stale silently**. The good news — the brand-new `status-drift-sweep.sh` (I-004, built
2026-06-07, 15/15 tests, `gate-lapsed` detector) is the structural cure for the `behind-GX` sub-class and
already flipped 2 G20 status drifts. It does NOT yet cover the **numeric-disk-trigger** class (D1) or the
**user-filed `deferred-followups.md`** ledger — extending it there closes this whole audit's gap.

---

## Tally

| | Count | Notes |
|---|---:|---|
| **Raw deferral hits (all 35 sessions)** | **3125** | defer 1190 · snooze 394 · monkey-chamber 287 · park 268 · not-now 243 · freeze 231 · for-now 139 · TBD 82 · post-deadline 74 · … |
| **High-signal (noise-filtered + scored + deduped)** | **474** | user-voice 319 · conditional-trigger 98 |
| **Trigger-bearing (behind-GX/revisit/decide-later/after-X/post-deadline)** | **159** | the dangling-risk pool |
| **DANGLING — trigger fired, never revisited (NET-NEW)** | **3** | ranked below (D1–D3) |
| **Cross-referenced — already in a sibling audit / verdict.jsonl** | **2** | dual-stream, RENAME-001 (Section C) |
| **Already tracked / triaged elsewhere** | **~4** | G19-behind-G2 + parked goals (in stale-goals-triage); G20-status (status-drift-sweep) |
| **CLEAN defers — trigger NOT yet fired (correctly parked)** | **~9** | listed Section D (watched, no action) |
| **RESOLVED — trigger fired AND was revisited** | **most** | e.g. android-layer-4, TheFreezer freeze, Path-D apply-strategy, the 3 stale MEMORY.md pointers |

---

## DANGLING — ranked by leverage (the gold)

### 🔴 D1 — Sliding-window report pruning (the user-filed TODO whose numeric trigger fired 3.8×)

- **Deferred:** 2026-05-25T05:24Z (session 2842f478), by EXPLICIT user instruction.
- **User's words (verbatim):** *"TODO note creation — sliding-window report pruning (deferred, do not
  implement now). Add a deferred-followup note to the project… file the TODO with clear trigger criteria for
  when to revisit… do NOT re-raise this in reports or summaries until a trigger fires."*
- **The decision:** build `scripts/prune-reports.sh --keep N` (default 5) for `reports/lighthouse/*.{html,json}`
  + `reports/vq-captures/{timestamp}/` + `.playwright-mcp/`, idempotent, wired into the SessionStart hook
  list. Deferred because "slow-burning, no urgent disk pressure, mid-audit focus."
- **The trigger the user SET (any one fires):**
  1. `reports/` exceeds **500 MB** (`du -sh reports/`).
  2. Path-A + full-system audit both green AND a session opens with no active fix queue.
  3. A future cleanup pass explicitly addresses workflow hygiene.
- **TRIGGER STATE — FIRED HARD:** `du -sh reports/` = **1.9 GB** (3.8× the 500 MB trigger). Breakdown:
  `vq-captures/` **1.1 GB** (60 run dirs, zero retention), `mobile-sim/` 254 MB, `captures/` 176 MB,
  `vq-captures-archive/` 133 MB, `timelapse/` 89 MB, `lighthouse/` 27 MB (55 files). Trigger #2 ALSO fired
  (G2 is awaiting-push/parked, no active fix queue). **TWO of three triggers fired; revisited: never.**
- **Confirmed genuinely dropped (not a phantom gap):** `prune-reports.sh` does NOT exist
  (`scripts/` has only `calibration-cleanup.sh`, `prune-spawned-worktrees.sh`, `scheduler-cleanup-orphan.py`
  — none touch `reports/`). No plan, idea (I-001…I-019), goal, or proposal covers report retention. The
  ONLY record is the parked TODO in `deferred-followups.md` itself.
- **What's needed NOW:** the discipline says the trigger firing UNPARKS it → build `prune-reports.sh` per the
  spec the user already wrote, wire it SessionStart, and run it once (it would reclaim ~1.5 GB on first pass).
  This is a ~30-min build with a fully-specified design already on disk. **Highest-leverage, lowest-risk item
  in this audit.**
- **Structural fix:** `status-drift-sweep.sh` (or a tiny `deferred-followups-trigger-check.sh`) should
  evaluate the `deferred-followups.md` triggers each SessionStart so a fired numeric trigger surfaces itself
  — the ledger currently relies on a human remembering to `du -sh`.

### 🟠 D2 — G20 Phase 2: the prepared Evium site-reconciliation (cherry-picks + green decision)

- **Deferred:** the site-reconcile half of G20 was scoped "Phase 2 — in Cursor" (2026-05-31), then on
  2026-06-02T16:27Z the user parked Evium itself: *"putting a temporary pin into evium, **i still intend on
  getting to its prepared rollbacks and plan improvements**… now that my boss has the repo it's a
  collaborative effort… high priority although no longer the main north star."*
- **The decision (prepared, never executed):** G20 Phase 2 = reconcile the site against the boss's `zach`
  branch — **cherry-pick `9f9d53c`** (brand green `#2dc856→#389834`, astro-icon+lucide, logo hero) **+
  `b1a739a`** (header: remove Home link + close-X, green Contact accent), **SKIP `4c6d816`** (the OS-deletion
  commit), **decide the green**, **decline the mobile-fullPage iOS-fix revert**, keep astro-icon + logo.png.
- **TRIGGER STATE — partially fired / lapsed:** **G20 sits in `archived/achieved/` but every Phase-2
  checklist box is still `[ ]` unchecked** (`[ ] Phase 1`, `[ ] Phase 2: reconcile`, `[ ] Phase 3/4`). The
  brand-green hex grep over `src/` returns EMPTY — i.e. the site never took zach's `#389834`, so the green
  decision + cherry-picks were NOT applied. The `status-drift-sweep.sh` (commit ff18a77) flipped G20's
  *status field* to `achieved` (the ownership-decouple side genuinely landed: repo split, kit-extraction
  plan, backups), but that **conflated the achieved DECOUPLE with the unfinished SITE-RECONCILE** — Phase 2
  is the live dangling tail.
- **Why it's a true dangling (not just "parked"):** the user said he "still intends to get to" the prepared
  rollbacks; G20 is filed achieved; and there is **no OPEN item anywhere** carrying "cherry-pick 9f9d53c /
  decide green / Phase-2 reconcile" forward — it lives only inside an *archived-achieved* doc's unchecked
  checklist, which by `feedback_user-owns-done.md` means it's NOT done but is filed as if the goal is.
- **What's needed NOW:** this is correctly **user-gated + Cursor-bound** (site work moved to Cursor; the boss
  co-owns the repo) — so the action is NOT to execute it, but to **stop it hiding inside an archived doc**:
  re-surface G20 Phase 2 as an explicit open user-gated line (e.g. a pin or a one-line note on G2) so the
  "prepared rollbacks" the user intends to return to don't evaporate with the archived goal. The green
  decision (`#2dc856` vs `#389834` vs the BLS-era `#d2a05f`/`#C5A059` gold threads) is a still-open user call.

### 🟡 D3 — `agentic-os-state` → `project-journal`: a rename decided ≥3× and never executed

- **Deferred / re-decided repeatedly:** 2026-06-03. Built overnight as `../agentic-os-state` (a copy-only
  state-hub, cutover GATED). Through that day the user pushed back twice on deleting it
  (*"what was it really for, you didn't just spend time on that for nothing?"* / *"why do you keep writing
  off the agentic-os-state repo… you're genuinely concerning me"*), and the agreed call landed on **keep it,
  rename `agentic-os-state → project-journal`** (the agent: *"it's a dormant copy, so renaming is a one-liner
  whenever — or I fold it into the cutover"*).
- **TRIGGER STATE — fired but unexecuted:** the rename was committed-to ≥3× across the day and queued
  ("Journal rename — `agentic-os-state → project-journal`, finalized"). On-disk: **`../agentic-os-state`
  still exists** (26 MB, 1 commit, NO remote, `Jun 3` mtime) and **`../project-journal` does NOT exist** —
  the rename was never run. The user's broader trigger (*"if it ever becomes one — you living in Cursor
  daily — we'll revisit"*) HAS fired ("you're going to Cursor today, which changes everything").
- **The complication (why it's 🟡 not 🔴):** the whole `agentic-os-state` concept was **superseded** on
  2026-06-07 by the `agent-dev-env-kit` extraction (`cursor-composer-handoff/extraction-build-2026-06-07.md`),
  which produces the neutral portable repo properly. The agent's latest read (2026-06-07T23:33Z) calls
  `agentic-os-state` *"the overnight experiment… the architecture has since moved past it… a naive cp
  (1 commit, no history)."* So the rename decision is **stale-but-unclosed**: the journal IS still wanted
  (un-retired), but `agentic-os-state` is the wrong artifact for it.
- **What's needed NOW:** make the call the discipline (`feedback_decision-finality.md`) demands — either
  (a) **delete `../agentic-os-state`** as the superseded naive-cp and let the kit-extraction produce the real
  journal, OR (b) **rename it `project-journal`** if it's the interim home. It can't keep sitting as a 26 MB
  no-remote orphan that 3 separate turns promised to rename. **This is a 1-minute decision the user must
  ratify** (it's a folder *outside* the repo → user runs the `mv`/`rm`, per gated-ops).

---

## Section C — Cross-referenced (ALREADY surfaced by a sibling; NOT re-fed to verdict.jsonl)

These ARE dangling deferrals of exactly this class, but a same-day audit already caught + fed them. Listed
so this audit is complete and so nobody double-counts.

- **C1 — Memory-mirror dual-stream / BG-coordination slam-dunk** — parked "behind G20"; G20 CLOSED so the
  gate LAPSED without build starting. The literal proven failure mode that motivated this audit.
  **Status: already in `verdict.jsonl` (lobe=half-finished #5)** and the stale MEMORY.md pointer was ALREADY
  FIXED this session (now reads *"G20 CLOSED — unblocked; revisit"* — verified line 59). The build itself
  remains the open follow-on, tracked by the sibling.
- **C2 — `agentic-script-design → agentic-architecture` rename (RENAME-001)** — deferred 2026-05-25
  "post-deadline"; trigger long fired; open as monkey-chamber RENAME-001; the SKILL.md self-documents the
  mismatch on every load. **Status: already in `verdict.jsonl` (lobe=half-finished #6).** Decision-finality
  says resolve it (rename or explicitly keep + close RENAME-001) — owned by the sibling.

## Section D — CLEAN defers (trigger has NOT fired — correctly parked, NO action, listed for completeness)

These were checked and the revisit-condition genuinely has not arrived. Do NOT re-raise (per
`feedback_decision-finality.md`).

- **Semantic-index / embeddings adoption (A74-D3)** — "revisit when G12 local-AI lands and owns embeddings."
  **G12 is `paused`** → trigger NOT fired. Clean. (Also has its own archived downside-analysis A76.)
- **"Bro" apply-strategy (scheduler Option C)** — explicitly "future plugin, Phase B+"; `VALID_APPLY_STRATEGIES`
  lists it; `scheduler.py:583` raises `NotImplementedError` intentionally. A watched defer, not dangling.
- **Skill cross-link rebuild → stale-only switch** — "switch from rebuild-ALL if/when skill count exceeds 20."
  13 top-level skills (under 20). Trigger NOT fired (a per-file perf optimization on a non-per-turn rebuild).
- **`linked_goals:` cross-goal field** — "revisit when a 3rd subtle cross-goal dependency appears." Borderline
  (a few `serves_*`/`depends_on G7` edges exist + the field is partially present) but no clear 3rd *subtle*
  case forced it; the `status-drift-sweep` gate-lapsed detector now covers the visible cross-goal deps.
- **G19 / ClickUp blog automation (RH-015)** — "deferred behind G2." Its gate technically lapsed when G2 was
  demoted from NS (2026-06-03), **but `stale-goals-triage-2026-06-07.md` already covers this** and recommends
  pausing G19. Tracked — not net-new.
- **Pixel_Fold device coverage (P-017)** — `state=deferred_dropped_from_matrix`, `resume_criteria=deferred-until-asked`,
  user-confirmed-drop 2026-05-28. AVD on disk (parked, not deleted) — exactly as the user chose. Clean.
- **P-018 Cursor-with-agents deep-dive** — re-pointed resume trigger ("only if the benchmark shows Composer
  is worth integrating deeper"); deliberately re-parked 2026-06-03. Clean.
- **CSS Tier-B family consolidation** — "post-G2, deferred under the design LOCK"; the design LOCK still
  holds (G2 awaiting push). Clean.
- **Heartbeat-polling inbox-latency pattern** — "spec next session post-reset"; later subsumed by the
  heartbeat-system-plan + autonomic daemon work. Effectively absorbed.

## Section E — Notable RESOLVED (trigger fired AND was revisited — the healthy baseline)

- **Android Layer-4 activation** — parked "post-scheduler-MVP"; trigger fired AND it was activated
  (plan now in `plans/completed/`; AVDs live since 2026-05-24). ✅
- **TheFreezer Evium freeze** (2026-06-02 user ask "save sourcecode to `context/codebasesTheFreezer/`") —
  DONE: `codebasesTheFreezer/evium/` is 93 MB on disk. ✅
- **Path-D apply-strategy** ("post-sprint build D properly") — landed: `scheduler.py` uses `git apply --3way`
  + a real `apply_strategy` (manual/diff). ✅
- **The 3 stale MEMORY.md pointers** (dual-stream / impact-domino / memory-bloat-guard, flagged 2026-06-08
  in-session) — ALL three already corrected on disk (verified lines 6, 50, 55, 59). ✅
- **MyZmanim / Vercel wiring research** — "post-deploy, researched not built"; the two research docs are
  referenced by the active BLS plan (the current NS) → consumed, not orphaned. ✅

---

## Recommendation (one line)

Build **D1** now (it's fully specced, the user filed it, the trigger fired 3.8×, ~1.5 GB reclaim, zero-risk);
extend `status-drift-sweep.sh` to evaluate the **`deferred-followups.md` numeric triggers** + **archived-goal
unchecked-checklists** each SessionStart (the structural cure for the D1 + D2 classes); ratify **D3** (delete
or rename) as a 1-minute user call. D2's *execution* stays user-gated/Cursor-bound — just stop it hiding in an
archived doc.
