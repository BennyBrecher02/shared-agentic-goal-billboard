---
kind: audit-findings
audit: open-asks-reconciliation (the CURRENT-open-set consolidation)
date: 2026-06-08
created_ts: 2026-06-08T06:10Z
trigger: user 2026-06-08T05:55Z "i asked for a lot of things that fell to the wayside so run an audit" (post-4hr-movie return)
method: |
  INHERIT (not re-discover) the known open set from verdict.jsonl (118 rows / 6 lobes) + the 5 same-day findings
  docs (full-history-request / half-finished / deferred-decisions / floated-ideas / session-request). THEN
  RECONCILE every inherited item against on-disk reality — because the autonomous wave (2026-06-07 22:xx, logged
  in notes/while-you-were-out-2026-06-08.md) ACTED on many items that verdict.jsonl still flags acted:false.
  THEN scan the live session transcript (eb0edcda, 2026-05-31->2026-06-08T05:59Z) for asks made SINCE the user
  returned (~05:13Z) + the pre-departure wave (00:39Z-01:57Z) that aren't in any sibling audit. Output: ONE
  deduped, ranked, blocked-vs-actionable list.
constraint: |
  The user HATES phantom gaps AND hates being re-told about things already done. So this pass does the OPPOSITE
  of the siblings: it DEMOTES every inherited verdict row that the autonomous wave has since closed (with on-disk
  proof), so the genuinely-open set is small + trustworthy. verdict.jsonl acted-flags are STALE post-wave; this
  doc is the reconciled truth.
sources:
  - .claude/cache/alignment/verdict.jsonl (118 rows, all acted:false — STALE post-wave)
  - audits/findings/{full-history-request,half-finished,deferred-decisions,floated-ideas,session-request}-audit-2026-06-07.md
  - context/markdowns/notes/while-you-were-out-2026-06-08.md (the wave digest = what actually landed)
  - context/markdowns/goal-billboard/pins/P-022-pinned-threads-2026-06-07.md (the durable don't-drop pins)
  - eb0edcda session transcript (live asks 00:39Z->05:59Z)
  - on-disk verification (scripts/, .claude/skills/, settings.json, ~/Library/LaunchAgents/, memory/)
---

# Open-asks reconciliation — 2026-06-08

## Headline

The 4hr-break autonomous wave **closed most of what the 5 sibling audits flagged** — verdict.jsonl's 118
`acted:false` rows are now badly stale. After reconciling every inherited item against on-disk reality and
folding in the since-return asks, **the genuinely-open set is 13 items**, of which **9 are blocked-on-USER
(gated pastes / decisions) and 4 are actionable-by-Claude**. Only **2 items are newly-surfaced since the
user returned at 05:13Z** — and both came out of the live 05:55Z exchange, neither is dropped, both are
in-flight this turn.

**The single most important open thing** is the **C→A→B handoff sequence itself** — the user explicitly said
"better guide me from c to a to b, lets start yes" (05:42Z) and C (kit extraction) is **mid-flight right now**,
paused at the kit-content-boundary gate. That is not "fell aside" — it is the live foreground task. Everything
below is the *background* the user asked me to reconcile ("but thats all bg lets keep at the important stuff").

**Biggest demotion:** the daemon-relabel (`com.evium.*`→`com.agentic-os.*`), which session-request-audit
listed as gated-on-user, is **already DONE** — live launchd labels are `com.agentic-os.*` (the `com.evium.*`
are `.bak` files). Don't re-surface it.

---

## Reconciliation tally

| | Count |
|---|---:|
| Inherited verdict.jsonl rows (6 lobes) | 118 |
| └ orphan-implementation (its own dedicated audit + own follow-up; NOT re-litigated here) | 82 |
| └ split-breaker (13) — folds into open item #2 (the split) | 13 |
| └ cross-audit curated set (drop-through/half-finished/dangling/floated) | 23 |
| **CLOSED by the autonomous wave (on-disk proof) — DEMOTE, do not re-surface** | **9 of the 23** |
| **Genuinely OPEN after reconcile (this doc's ranked list)** | **13** |
| └ blocked-on-USER (gated paste / decision) | 9 |
| └ actionable-by-Claude (safe doc/script fixes) | 4 |
| **Newly-surfaced SINCE return (05:13Z+)** | **2** (both in-flight, not dropped) |

---

## CLOSED BY THE WAVE — demoted with proof (so they stop re-flagging)

These are flagged `acted:false` in verdict.jsonl but are **done on disk.** Do NOT re-raise.

| Inherited item | verdict lobe | On-disk proof |
|---|---|---|
| D1 sliding-window report pruning | dangling-deferral (high) | `scripts/prune-reports.sh` BUILT (16.8KB, 06-07 22:34). *(Running it `--apply` is the only open tail → item #1 below.)* |
| Floated #1 `agentic-preview-tools` skill | floated (high) | `.claude/skills/agentic-preview-tools/` EXISTS |
| Floated #2 master skill-maker / A38 | floated (high) | `notes/look-through-bens-eyes.md` EXISTS; A38 unfrozen, corpus → 1,415 prompts |
| Safety-skills cluster (00:44Z ask) | (00:44Z) | `.claude/skills/agentic-safety/` EXISTS (7 refs) |
| Skill-mirror / cross-env drift (01:20Z ask) | (01:20Z) | `scripts/skill-drift-check.sh` BUILT + live Cursor drift fixed (11 stale copies refreshed); design `research/systems/cross-env-skill-consistency-2026-06-07.md` |
| AIS memory topic (half-finished #1 tether d) | half-finished (high) | `feedback_alignment-immune-system.md` in canonical memory + mirror + 1 MEMORY.md pointer |
| Memory recall-gaps (9 load-bearing tools) | orphan (various) | MEMORY.md now has pointers (slug-time-bomb, event-bus, daemon-health, etc.) |
| Daemon-relabel `com.evium`→`com.agentic-os` | session-request (gated) | **DONE** — live `~/Library/LaunchAgents/` labels are `com.agentic-os.*`; `com.evium.*` are `.bak` only |
| Longitudinal "getting-better" metric (floated #6) | floated (med) | Built; verdict TRENDING_BETTER (+27% skill-coverage/6d) |

---

## THE OPEN SET — ranked by leverage

### 🔴 #1 — Run `prune-reports.sh --apply` (the ~985 MB reclaim) — BLOCKED-ON-USER (mass-delete gate)
- **What:** D1's tail. Script is built + dry-run-verified (~985 MB / 109 units, all regeneratable: vq-captures
  bulk + lighthouse + playwright temp; keeps newest 5 each; soft-deletes to ~/.Trash).
- **Status:** the user's 500 MB trigger fired at 1.9 GB. At 05:22Z the user **rejected** an inferred run
  ("this sounds extreme... you deleted archives i repeatedly said we should keep even though theyre
  recreatable!!! dont mess up my git rollback page stuff"). At 05:40Z: "keep all and lets move on."
- **Next step:** this is **soft-gated and arguably now declined for the archives.** Re-confirm scope with the
  user EXPLICITLY (which dirs are safe vs the keep-forever archives + the git-rollback-page captures he just
  protected) before any `--apply`. Do NOT infer it. **Blocked-on-user; do not auto-run.**

### 🔴 #2 — The C→A→B handoff + the shared-`_core` invariant — IN-FLIGHT (the live foreground)
- **What:** C = kit extraction (`cursor-composer-handoff/extract-kit-do-it.sh`); A = Cursor bridge-kit +
  guided handoff; B = journal pipeline (done). User: "better guide me from c to a to b, lets start yes" (05:42Z).
- **Status:** **C is mid-flight right now**, correctly paused at the kit-content-boundary gate
  (`--path context/markdowns/` would leak ~20 MB of Evium-specific billboard/manifest/audit material into the
  "portable" kit; `portable-kit/MANIFEST.md` is the authoritative boundary that resolves it).
- **NEW hard invariant surfaced 05:55Z (item #12 below):** the shared dashboard + event-bus + journal **must**
  land in shared `_core`, NEVER inside claude-os. Bake into the split design as non-negotiable.
- **Next step:** finish settling the kit boundary against `portable-kit/MANIFEST.md`, get user sign-off on the
  GOES-vs-STAYS table, then run C's gated push. **Actionable up to the gated `gh repo create`/push.**

### 🟠 #3 — Instant inbox-acknowledgment (A68) — BLOCKED-ON-USER (1 settings line)  ·  recurrence 13× / ~8 sessions
- **What:** the most-repeated ask in the whole corpus ("I sent an inbox ages ago, why haven't you acked??" ×12
  + "most important throughput improvement"). 3 hook shapes deployed + dry-run-tested; `A68.status:
  in_progress` because all 3 await a `settings.json` line.
- **Next step:** surface the A68 settings.json gate as an explicit yes/no → arm instant-ack → flip A68 resolved
  + bump G6 phase. **Blocked-on-user (gated paste).** (verdict drop-through #1, still genuinely open.)

### 🟠 #4 — The staged `settings-patches/` merge batch — BLOCKED-ON-USER (gated pastes)
- **What:** `PENDING-WIRES.md` + 11 unwired `settings-patches/*.json`. VERIFIED still-pending in live
  settings.json: **status-drift-sweep** (the I-004 structural cure, 15/15 tested) + **8 orphan-wires**
  (bg-dispatch-log-gc · bg-ghost-detect-dual-signal · bottleneck-detect · convention-f-closure ·
  cross-software-check · daemon-graduation-check · hook-command-lint · wrapper-only-write-guard) +
  **domino-check** (= half-finished #3 / F4 — producer enqueues `unscored` stubs nothing drains).
- **Next step:** present as ONE gated paste batch grouped by array (SessionStart / PreToolUse / Stop). Each
  ships with a kill-switch; nothing waits on the paste. **Blocked-on-user.** (Folds verdict half-finished #3.)

### 🟠 #5 — AIS finish: ALIGN 5th pulse sign + status-doc flips — MIXED (Claude-actionable parts + gated parts)
- **What (half-finished #1, the keystone):** AIS engine is wired + firing, but **VERIFIED still missing:**
  (a) the 5th ALIGN vital sign in `organic-os-pulse.sh` (0 `ALIGN` refs) — so a dead watchdog wouldn't show on
  the pulse strip, defeating its loud-not-silent invariant; (b) **Tier-2 LLM-judge** is DESIGNED-only (build-
  gated on a judge Workflow); (c) launchd free-tick uninstalled (user-gated, correct).
- **STALE-STATUS tail:** the wave fixed the memory topic, but check whether manifesto §13 / §12-item-12 still
  say "unwired" (they describe applied wiring as pending → guarantees re-flagging).
- **Next step:** (a) add ALIGN to `organic-os-pulse.sh` (CPU-only, reads verdict.jsonl freshness + `.last-kick`
  age) — **Claude-actionable**; (b) flip any stale manifesto status prose — **Claude-actionable**; (c) Tier-2
  judge + launchd stay **user-gated**.

### 🟡 #6 — `instrument-hooks-usage.sh` root-hook coverage — ACTIONABLE-BY-CLAUDE
- **What (half-finished #4):** VERIFIED `instrument-hooks-usage.sh:52` still hardcodes
  `HOOKS_DIR="$REPO/scripts/hooks"` and globs only that dir → the ~12 SessionStart/Stop hooks in `scripts/`
  ROOT are invisible → `--dead-only` over-reports live hooks as dead. *(Note: the hook-usage-COUNTER is wired
  live; the instrumentation SCAN is what under-covers.)*
- **Why it matters:** prerequisite for trusting ANY future orphan/dead-hook sweep — an auditor reading a
  miscounting ledger inherits the lie.
- **Next step:** derive the wired-hook set from `settings.json` instead of a fixed dir. **Actionable, safe.**

### 🟡 #7 — Cross-substrate idea-gap sweep — ACTIONABLE (or confirm done)
- **What (floated #4):** extend `detect-idea-gaps.py` (session-JSONL-only) to also scan `event-bus.jsonl` +
  `dispatch-log.jsonl` + `bug-billboard/inbox/`. The wave claims "cross-substrate idea-gap detector built" —
  **VERIFY** before re-surfacing (if built, demote).
- **Next step:** confirm on disk; if not built, extend the detector. **Actionable.**

### 🟡 #8 — G20 Phase 2 Evium site-reconcile — BLOCKED-ON-USER + stop it hiding
- **What (dangling D2):** the prepared cherry-picks (zach `9f9d53c` green/icons/logo + `b1a739a` header, SKIP
  `4c6d816`, decide the green, decline mobile-fullPage revert) live ONLY in an `archived/achieved/` doc with
  every Phase-2 box still `[ ]`. brand-green hex grep over `src/` is EMPTY — never applied. User "still
  intends to get to its prepared rollbacks" but it's Cursor-bound + boss co-owns the repo.
- **Next step:** execution stays user-gated/Cursor-bound — but **re-surface it as an explicit open line**
  (a pin or one-line note on G2) so it doesn't evaporate with the archived goal. The green decision
  (`#2dc856` vs `#389834`) is a still-open user call. **Blocked-on-user.**

### 🟡 #9 — System-prompt A/B test — BLOCKED-ON-USER (explicitly PARKED, pin held)
- **What:** user 00:50Z asked where this dropped to; 01:20Z "put a pin in the system prompt for now." Research
  DONE (`research/systems/system-prompt-value-and-metrics-2026-06-07.md`: scalpel-not-slab, small
  `--append-system-prompt` for 3-4 directives). Tracked in **P-022**.
- **Next step:** NONE until the user un-parks (per decision-finality). Listed so it isn't re-flagged as dropped
  — it is correctly parked. **Blocked-on-user (by the user's own choice).**

### 🟡 #10 — Global Claude setup — BLOCKED-ON-USER (post-extraction build, pinned)
- **What:** user 00:50Z + 01:20Z "the global setup shouldnt be forgotten." `deploy claude` → `~/.claude/`
  global (skills/CLAUDE.md/settings/rules global; memory per-project). Doc:
  `research/systems/global-claude-setup-2026-06-07.md`. Pinned in **P-022**; explicitly POST-extraction.
- **Next step:** build AFTER C lands (it's gated behind the extraction). REMIND the user post-C. **Blocked-on-sequence.**

### 🟢 #11 — `agentic-os-state` → `project-journal` decide (delete vs rename) — BLOCKED-ON-USER (1-min call)
- **What (dangling D3):** `../agentic-os-state` still exists (26 MB, 1 commit, no remote, Jun 3);
  `../project-journal` does NOT. Rename committed-to ≥3× on 06-03, never run. SUPERSEDED 06-07 by the
  agent-dev-env-kit extraction (the naive cp is the wrong artifact for the journal).
- **Next step:** make the call — delete the superseded cp OR rename it. It's a folder OUTSIDE the repo → user
  runs the `mv`/`rm`. **Blocked-on-user (1-minute ratify).**

### 🟢 #12 — I-007 mixed-memory-topic refinement — BLOCKED-ON-USER (case-by-case greenlight)
- **What:** 13 "mixed" memory topics (universal principle welded to Evium example) to split case-by-case so the
  universal half promotes into `portable-kit/universal-memory-manifest.md`. User: "do this later, case-by-case
  — NOT in bulk." Surface-only ideas-lane contract; nothing auto-starts. Relevant NOW because C/the split
  needs the portable boundary clean.
- **Next step:** await user greenlight per topic. **Blocked-on-user.** (verdict orphan/session-request tail.)

### 🟢 #13 — RENAME-001 `agentic-script-design`→`agentic-architecture` — BLOCKED-ON-USER (1 decision)
- **What (half-finished #6):** deferred 05-25 "post-deadline" (long fired); open monkey-chamber RENAME-001;
  SKILL.md self-documents the name mismatch on every load. ~13 days unresolved.
- **Next step:** make the call (rename or keep + close RENAME-001) per decision-finality. LOW (cosmetic), but a
  textbook deferred-and-forgotten. **Blocked-on-user (1 decision).**

---

## NEWLY SURFACED SINCE RETURN (05:13Z+) — both in-flight, neither dropped

1. **The shared-`_core` cross-env invariant (05:55Z).** User: "will this block our system from being truly
   cross environment agnostic? ...check our single core shared dashboard and see its research progressing the
   same [in cursor/composer] as if i asked in claude?" → answered live (3-layer model: portable kit / shared
   `_core` dashboard+event-bus+journal / per-env driver-hooks). **Open build:** the **Cursor-side adapter that
   publishes into the shared bus** (= Thread A + the `_core` split) is NOT built; event-bus + dashboard +
   convo-torch protocol exist. The invariant ("dashboard+bus+journal land in shared `_core`, never claude-os")
   must be baked into the split design. → **folded into open item #2.**
2. **The 3 "bg" health checks (05:55Z):** ① Heart/Immune/Autonomic post-break, ② hook health + increment
   counting, ③ "run an audit" (= THIS doc). ①+② answered live (Heart beating, Immune detector live, Autonomic
   5/5 daemons healthy, counter checks out). ③ is this reconciliation. **All three serviced this turn — not open.**

> **Nothing the user asked SINCE returning was dropped.** The since-return window (05:13→05:59Z) is fully
> serviced or in-flight. The "fell to the wayside" items are all from BEFORE the break, and 9 of the loudest
> were closed by the autonomous wave (demotion table above).

---

## The meta-finding (consistent with both sibling audits)

The disease is **not abandonment — it's stale status flags.** verdict.jsonl carries 118 `acted:false` rows;
after on-disk reconcile, **9 of the curated 23 are already done.** An auditor (human or the AIS drop-through
lobe) reading verdict.jsonl cold would re-flag 9 completed items as gaps — manufacturing exactly the phantom
gaps the user hates. **Cheapest high-value fix:** a reconcile step that flips `acted:true` (with the proof
path) the moment a verdict item lands on disk — which is precisely what the staged-but-unwired
`status-drift-sweep` (open item #4) automates. Wiring it closes this whole class.

## One-line recommendation
Finish C's boundary (item #2) — it's the live task the user said "lets start yes" to — then present items
#3+#4 as ONE gated paste batch (instant-ack line + the settings-patches merge incl. status-drift-sweep, which
auto-cures the stale-flag disease). Claude can do #5(a/b), #6, #7 now without a gate. Re-confirm #1's scope
before any prune (the user just protected the archives). #8–#13 are user decisions/pins — surfaced, not acted.
