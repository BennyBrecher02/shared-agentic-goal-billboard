---
kind: audit-findings
audit: loose-ends sweep (the FRESH never-miss pass — post C→A→B arc)
date: 2026-06-08
created_ts: 2026-06-08T13:15Z
trigger: user (napping) "fresh never-miss + loose-ends sweep — catch anything still open after the C→A→B arc"
method: |
  INHERIT the 06:10Z open-asks-reconciliation (the 13-item consolidated open set) + the 4 sibling findings
  docs (floated / half-finished / deferred-decisions / open-asks-reconciliation) + verdict.jsonl, THEN
  RE-RECONCILE every item against on-disk reality AS OF 13:15Z — because the live session ran ~7 more hours
  past the reconciliation doc (06:10Z → 13:10Z) and a large autonomous wave LANDED most of the gated batch.
  THEN scan the live transcript (eb0edcda) for every user ask SINCE the arc started (05:42Z → now) and check
  each for closure. Output: ONE ranked list — ACTIONABLE-BY-CLAUDE vs NEEDS-USER vs DEFER. Read-only + this doc.
constraint: |
  The user HATES phantom gaps AND hates being re-told what's done. This pass DEMOTES everything the wave closed
  with on-disk proof, so the genuinely-open set is tiny + trustworthy. The 06:10Z doc is now itself partly stale
  (it predates the 12:27Z "run the 5 safe steps" greenlight + the 12:53Z goal-remap approval).
sources:
  - audits/findings/open-asks-reconciliation-2026-06-08.md (the 06:10Z 13-item set — the baseline)
  - audits/findings/{half-finished,deferred-decisions,floated-ideas}-audit-2026-06-07.md
  - audits/findings/goal-remapping-proposal-2026-06-08.md (the 13-UNCLEAR / 4 named Decisions)
  - cursor-composer-handoff/thread-b-pipeline-plan.md (B7 — unify ledgers)
  - .claude/cache/alignment/verdict.jsonl
  - eb0edcda session transcript (live asks 05:42Z → 13:10Z)
  - on-disk verification (settings.json==proposed.json, scripts/, goal-billboard/, research/)
verdict_feed: .claude/cache/alignment/verdict.jsonl (1 new drop fed — see §5)
---

# Loose-ends sweep — 2026-06-08 (post C→A→B arc)

## Headline

**The autonomous wave that ran during this session closed almost the entire 06:10Z open set.** Live
`settings.json` is now **byte-identical to `settings.proposed.json`** — i.e. the gated paste batch (open
item #4) LANDED, including the **`status-drift-sweep`** structural cure (4 refs live) that auto-fixes the
stale-flag disease every sibling audit named as the root pathology. **`domino-check` + `domino-consume` are
wired** (B5 done → closes half-finished #3). **The B4 goal-remap was APPLIED** (`belongs_to_goal` now spans
13 goals, no longer a degenerate G2 blob) and a **NEW goal G21** (Portable multi-tool agentic-OS) was created,
resolving UNCLEAR Decision 1.

After re-reconciling, the genuinely-open set is **6 items** (down from 13): **2 actionable-by-Claude now**,
**3 needs-user (decisions / 1 settings gate)**, **1 defer**. Plus **1 newly-caught since-arc ask** whose
delivery is UNVERIFIED — the **stats-page reorg** (the only thing the user asked for after the arc that has
no landed artifact on disk).

**Nothing the user asked was abandoned.** The since-arc window (05:42Z → 13:10Z) is fully serviced or
in-flight, with the single exception flagged in §3 (stats-reorg — work was clearly underway but the user
twice noted "i cant see the updated page," so it needs a verify-and-show, not a re-build).

---

## §1 — What CLOSED since the 06:10Z reconciliation (demote with proof — DO NOT re-flag)

These were OPEN at 06:10Z and are **done on disk now.** This is the second demotion layer on top of the
06:10Z doc's own demotion table.

| 06:10Z open item | On-disk proof @ 13:15Z |
|---|---|
| **#4 — the staged `settings-patches/` merge batch** | `settings.json` == `settings.proposed.json` (0 diff). All 8 orphan-wires LIVE (bg-dispatch-log-gc · bg-ghost-detect-dual-signal · bottleneck-detect · convention-f-closure · cross-software-check · daemon-graduation-check · hook-command-lint · wrapper-only-write-guard — 2 refs each) + **status-drift-sweep (4 refs)**. |
| **half-finished #3 / F4 — domino-check + domino-consume unwired** | Both WIRED in `settings.json` (L177 producer, L486 consumer; B5 — user pasted 12:53Z). The "producer enqueues stubs nothing drains" gap is closed. |
| **#7 — cross-substrate idea-gap sweep** (was "VERIFY") | BUILT: `scripts/detect-gaps-cross-substrate.py` (21.5KB) scans all 3 named substrates — event-bus.jsonl + bg-dispatch-log.jsonl + bug-billboard/inbox/. *(Tail: 0 refs in settings.json → unwired; see actionable #A2.)* |
| **UNCLEAR Decision 1 — does a "Portable multi-tool OS" goal exist?** | RESOLVED **yes**: `goal-billboard/active/G21-portable-multi-tool-agentic-os.md` created 13:00Z; the 8 portable-OS/torch/Cursor-adapter docs now carry `belongs_to_goal: G21`. |
| **B4 goal-remap (the 127 confident re-maps)** | APPLIED backup-first (`goal-billboard.bak-apply-20260608T125516Z/` snapshot exists; `belongs_to_goal` now spans G1/G2/G4/G8/G9/G11/G12/G14/G15/G17/G18/G19/G21). By-goal axis is non-degenerate. |
| **The meta-finding's structural cure** | `status-drift-sweep` is now LIVE (4 refs) — the very mechanism all 3 sibling audits said would close the stale-flag class is wired, so future audits stop re-flagging applied work. |
| **Emulator-vs-responsive research (the dropped rabbit-hole)** | DONE: `research/systems/emulators-vs-responsive-viewer-2026-06-08.md`, `status: complete` (08:27Z). Answers the 12:22Z re-ask. Verdict: ADD a responsive-viewer as an author-time pre-Layer-2 scratch tool, NOT a numbered "done" layer (can't run WebKit). **Its `agentic-device-testing/SKILL.md` edit is a NOTED PROPOSAL, NOT applied** (§7 of the research doc explicitly: "proposals; this task is read-only"). → that skill edit is actionable #A1's neighbor; see §4 DEFER-D1. |

---

## §2 — ACTIONABLE-BY-CLAUDE NOW (safe, no gated ops)

### 🟢 A1 — Add the 5th ALIGN vital sign to `organic-os-pulse.sh`  ·  leverage: HIGH
- **State (VERIFIED 13:15Z):** `organic-os-pulse.sh` has **0 `ALIGN` refs** — still emits only MEMORY / HEART /
  IMMUNE / AUTONOMIC. The §13 AIS design explicitly requires a 5th ALIGN sign so a *dead watchdog shows on the
  pulse strip* (its loud-not-silent invariant). The AIS engine is wired + firing; the pulse sign is the one
  remaining Claude-safe tether (open item #5a; half-finished #1c).
- **Do:** add an ALIGN line reading `verdict.jsonl` freshness + `.last-kick` age (CPU-only, mirrors the 4
  existing signs). No gate.

### 🟢 A2 — Extend `instrument-hooks-usage.sh` to cover wired ROOT hooks  ·  leverage: MEDIUM
- **State (VERIFIED 13:15Z):** `instrument-hooks-usage.sh:52` still hardcodes `HOOKS_DIR="$REPO/scripts/hooks"`
  and globs only that dir → the ~12 SessionStart/Stop hooks in `scripts/` ROOT are invisible → `--dead-only`
  over-reports live hooks as dead (open item #6; half-finished #4). Prerequisite for trusting ANY future
  orphan/dead-hook sweep.
- **Do:** derive the wired-hook set from `settings.json` instead of a fixed dir. Safe.
- *(Bundle note: `detect-gaps-cross-substrate.py` is built but unwired (0 settings refs). Wiring it is a
  SETTINGS edit → user-gated, so it is NOT an A-item; it belongs in the next gated-paste batch. Flagged so it
  doesn't masquerade as Claude-actionable.)*

---

## §3 — NEWLY CAUGHT since the arc (not in any sibling audit) — needs verify/show

### 🟠 N1 — Stats-page reorg (10:09Z ask) — DELIVERY UNVERIFIED  ·  the one genuine since-arc loose end
- **The ask (verbatim 10:09Z):** *"the stats page needs reorganizing, all our previously stressed timing and
  token stats have been flooded by our skills/hooks stats… better organized with sub stats and how theyre
  ordered/displayed by default because all start open which just leaves us an infinite wall of text which is
  bad ui/ux."* User then said **"go"** (10:13Z) and **"go"** (10:21Z).
- **The signal it's not closed:** at **10:33Z** the user said *"i cant see the updated stats dashboard page,
  bring it up for me."* → work was underway but the user never confirmed seeing the result.
- **On-disk state (VERIFIED 13:15Z):** **NO landed artifact** — `build_render_model.py` mtime is **08:34**
  (BEFORE the 10:09Z ask); no stats-reorg commit on any branch since 09:30; no reorg plan/doc written today;
  no worktree (only `dashboard-wip` checked out). So either the reorg landed in an HMR/dev-server-only view
  the user couldn't reach, or it stalled when the session pivoted to the Cursor-cleanup thread at 10:33Z.
- **Why it's the headline loose end:** it is the **freshest** since-arc ask, the user gave an unambiguous "go"
  ×2, and it visibly did NOT reach "user can see it." Everything else from the arc either landed or is a known
  gated/decision item.
- **Next step (Claude, on wake — needs the dev server, so flag for the foreground turn):** bring up the stats
  page (`agentic-preview-tools` / Chrome on :43700), confirm whether the timing/token vs skills/hooks split +
  default-collapsed sub-stats actually rendered; if not, build the reorg (default-collapsed accordion groups,
  timing+token tiers first). **This is the one item to NOT let evaporate.** → fed to verdict.jsonl (§5).

> All OTHER since-arc asks are serviced: the Cursor file/content audit (11:17Z/11:19Z → `cursor-files-audit`
> + `cursor-content-diff` docs exist, 07:23Z); the skill-mirror resolution (11:21Z → confirmed built + in
> use, auto-run wiring folded into A's gated patch); Cursor Run-Mode/allow-list guidance (11:41Z/11:52Z →
> answered + guardrail-mirror allow-list extension agreed); daemon/Heart health (12:22Z → answered live);
> the 5 safe-now B steps (12:27Z "yes" → landing in the live turn at 13:10Z); the goal-remap (12:53Z → APPLIED).

---

## §4 — NEEDS-USER (decisions + the 1 remaining settings gate)

### 🟠 U1 — Instant inbox-acknowledgment (A68) — the 1 still-pending settings gate  ·  recurrence 13× / ~8 sessions
- **State (VERIFIED 13:15Z):** the most-repeated ask in the corpus. Hook shapes exist on disk
  (`ack-latency-measure.sh`, `inbox-on-prompt.sh`, `agent-inbox-*`) but **`instant-ack`/`inbox-ack` = 0 refs
  in BOTH `settings.json` AND `settings.proposed.json`** → A68 was NOT folded into the batch that just landed.
  It is now the **single** genuinely-pending settings gate (open item #3 — survives the wave).
- **Next step:** present the A68 settings line as an explicit yes/no gated paste → arm instant-ack → flip A68
  resolved + bump G6. **Blocked-on-user (1 settings line).**

### 🟢 U2 — The remaining UNCLEAR goal Decisions (2, 3, 4) — 8 files, routing judgment  ·  leverage: LOW
The B4 remap applied the 127 confident maps and G21 resolved Decision 1. The **3 remaining named Decisions**
(8 files) are genuine routing calls only the user can make. Each with a 1-line recommendation:
- **Decision 2 — agentic-system *surveys*: framework (G15) or substrate (G4)?** (3 files: top-starred-agentic-github
  README, non-github-agentic-systems README, A24-meta-loop-eval). **Rec:** the 2 survey READMEs → **G15**
  (they ARE exemplar-pattern surveys, G15's domain); A24 → **G4** (cross-utilization is a skill-substrate eval).
- **Decision 3 — graceful-lifecycle cluster: G10 vs G14 vs G4?** (A30, A31, A32). **Rec:** **A30 → G10**
  (its title is verbatim G10's — "title-match wins" promotes it to H, drops the unclear count); A31 → **G17**
  (6-front master spanning event-bus+bundle+research-v2 leans peak-throughput infra); A32 → **G4** (hook audit).
- **Decision 4 — the 2 leftover process audits** (A2-critical-script-coverage, time-precision-audit). **Rec:**
  A2 → **G9** (script *coverage* = testing toolbelt); time-precision-audit → **G4** (true-time *discipline* is
  substrate, not scheduler-specific G1).
- **Next step:** these are surface-only (do-not-auto-apply per the ideas-lane contract). Present the 3 Decisions
  as a quick A/B/C pick when convenient. **Blocked-on-user (low-stakes routing).**

### 🟢 U3 — Carryover user-decisions still open (unchanged by the wave) — surfaced, not acted
The 06:10Z doc's user-decision tail that the wave did NOT touch (all correctly blocked-on-user):
- **prune-reports.sh `--apply`** (open #1): tool BUILT, reports/ still **1.9 GB**. User said "keep all and lets
  move on" (05:40Z) after protecting the git-rollback archives. **Rec:** treat as DECLINED-for-archives; do NOT
  auto-run. If a future scope-confirm happens, only the regeneratable vq-captures/lighthouse/playwright-temp are
  in-scope — never the archives the user protected.
- **G20 Phase-2 Evium site-reconcile** (open #8): cherry-picks + the `#2dc856` vs `#389834` green decision —
  Cursor-bound, user-gated. **Rec:** re-surface as a one-line note on G2 so it stops hiding in the archived-achieved
  doc; the green hex is a still-open user call.
- **System-prompt A/B** (open #9, pinned P-022): correctly parked by the user; no action until un-parked.
- **Global Claude setup** (open #10, pinned P-022): gated POST-extraction; remind after C lands.
- **`agentic-os-state` → `project-journal` decide** (open #11 / D3): `../agentic-os-state` still exists (26 MB,
  superseded by the kit extraction). **Rec:** delete the superseded cp OR rename — 1-min user-run call (folder
  outside repo → gated mv/rm).
- **I-007 mixed-memory-topic refinement** (open #12): 13 mixed topics to split case-by-case for the portable
  boundary. **Rec:** await per-topic greenlight; relevant to keeping the kit boundary clean.
- **RENAME-001 `agentic-script-design` → `agentic-architecture`** (open #13 / half-finished #6): ~14 days
  unresolved; SKILL.md self-documents the mismatch on load. **Rec:** make the call (rename or keep + close
  RENAME-001) per decision-finality. Cosmetic.

---

## §5 — DEFER

### D1 — B7: unify the two intake ledgers → `request-ledger.jsonl`  ·  leverage: L  ·  the named deferred-hardening item
- **What (thread-b-pipeline-plan B7 / F5 / D6):** collapse `user-asks-pending.jsonl` + `idea-gaps.jsonl` into
  one `request-ledger.jsonl` keyed by prompt-uuid with a single `closed_by:{kind,ref}` + one closure definition;
  both hooks read/write it. Today an ask can be `satisfied` in one ledger and a `gap` in the other for the same
  prompt (D6); 0 rows carry `closed_by`.
- **State:** `request-ledger.jsonl` does NOT exist (verified MISSING). Highest blast-radius item in Thread B
  (two live hooks + the AIS read from these).
- **Recommendation (1-line, per the plan's own §5.4):** **DEFER** — the pipeline is functional with the dual
  ledgers since F1 widened both scans (both now scan all 8 homes); this is hardening, not a hole. Build it
  **only after** B1 (the goal-graph) + B6 (the dashboard alignment surface) prove out **and only if** the
  two-ledger divergence actually bites. Not a drop — a watched, correctly-sequenced defer.

### D2 — emulator-research's `agentic-device-testing/SKILL.md` edit (the responsive-viewer subsection)
- **State:** the research is DONE; §7 of the research doc holds a drafted SKILL.md subsection ("responsive-viewer
  = author-time pre-Layer-2 scratch tool, never a done-gate layer") explicitly marked **proposal, NOT applied**.
- **Recommendation:** LOW-stakes, SAFE skill edit — Claude *could* apply it, but it touches a canonical skill
  surface and the research framed it as proposal-only. **DEFER to a one-line confirm** ("apply the responsive-viewer
  note to agentic-device-testing?") rather than auto-editing the skill. Idea-capture, not urgent.

---

## §6 — Tally

| Bucket | Count | Items |
|---|---:|---|
| 06:10Z open set | 13 | (baseline) |
| **CLOSED by the wave since 06:10Z** | **7** | #4 (settings batch incl. status-drift-sweep) · half-finished #3 (domino) · #7 (cross-substrate, built) · UNCLEAR-Dec-1 (G21) · B4 remap applied · the stale-flag cure wired · emulator research |
| **Genuinely OPEN after re-reconcile** | **6** | A1, A2 (Claude) · U1, U2, U3-cluster (user) · D1 (defer) |
| └ actionable-by-Claude now | 2 | A1 (ALIGN pulse sign) · A2 (instrument-hooks root coverage) |
| └ needs-user | 3 | U1 (A68 settings gate) · U2 (3 UNCLEAR Decisions) · U3 (6 carryover decisions) |
| └ defer | 1 | D1 (B7 unify ledgers) [+ D2 skill-edit confirm] |
| **Newly caught since arc** | **1** | N1 (stats-page reorg — delivery UNVERIFIED) |

**Verdict.jsonl feed:** 1 new row (lobe `drop-through`, the stats-page-reorg verify-gap — the only genuinely
new dropped/unverified ask this sweep surfaced). All other open items already carry verdict rows from the
sibling audits; no duplicates fed.

## §7 — One-line recommendation
Claude can do **A1 + A2** now (no gate). On the foreground turn, **verify-and-show the stats page (N1)** — the
one since-arc ask that didn't reach the user — and build the reorg if it didn't land. Surface **U1 (A68)** as
the single remaining gated settings paste, and **U2's 3 Decisions** as a quick A/B/C pick. Everything else
(U3, D1, D2) is correctly parked/deferred — surfaced, not acted.
