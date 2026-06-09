---
title: Autonomic daemon flip-readiness audit (the 5 dry-run-forever daemons)
generated: 2026-06-07T23:07:59Z
type: audit-finding
scope: READ-ONLY — no daemon was flipped; flip is the orchestrator's gated step via daemon-control.sh
subject: com.agentic-os.daemon{2,3,4,5,10} (relabeled from com.evium.*) on DRY_RUN=1
verdict_summary: FLIP-NOW × 5 / FIX-FIRST × 0 (all safe; see efficacy caveat E1)
---

# Autonomic daemon flip-readiness — 2026-06-07

**The ask.** 5 launchd-loaded autonomic daemons (`com.agentic-os.daemon{2,3,4,5,10}`) have run
`DRY_RUN=1`, hourly, observe-only since **2026-06-02** — the A50 "dry-run-forever" anti-pattern (A47
discipline = 24h dry-run THEN flip; these are 5.5 days in). Determine which are SAFE to flip to live.

## The headline finding (why all 5 are safe)

These daemons are **not the kind of autonomic worker that mutates substrate in apply mode.** I verified
by grep that **every one of the 5 deployed shell consumers contains ZERO mutation verbs** (`rm` / `mv` /
`cp` / `sed -i` / `PlistBuddy` / `git` / `launchctl` — the one `launchctl` hit was a code *comment*).
The **entire** difference between dry-run and apply mode is a single line: in apply mode each daemon does
one **idempotent `>` overwrite of a `<name>-findings.flag`** file under `.claude/cache/` for the next
agent turn to read. **No source substrate (MEMORY.md, feedback_*.md, audits, steering log, bug billboard,
skill refs) is ever touched, in either mode** — enforced in code and reinforced by A66 in every header.

So "flip to apply" here ≠ "let a robot edit my files." It means "let the daemon drop a flag when it finds
something, instead of only logging to stderr." That is the safest possible apply semantics. The dry-run
histories are clean (60-61 fires each, **0 malformed JSON lines**, all genuinely *ran* with live-varying
metrics, all `dry_run:1`, 0 cost/pivot skips). **All 5 are FLIP-NOW.**

### Caveat E1 (efficacy, NOT safety — does not block any flip)
The flag files the daemons would drop currently have **no live reader hook** wired in `.claude/settings.json`
for daemon2/4/5 (and daemon3/10's only "readers" are docs/worktree copies, not runtime hooks). This is the
exact "**daemon WRITES a flag but NO hook READS it**" gap that `scripts/hooks/never-idle-refill-surface.sh`
was built to close for Daemon #1 (research-furthering). Flipping is still safe (worst case: flag sits
unread, overwritten next fire — idempotent), but **flipping yields no surfaced action until a reader hook
exists.** Recommend: wire a SessionStart flag-surfacer (mirror `never-idle-refill-surface.sh`) either before
or right after the flip, so the live signal actually reaches the agent. This is a follow-up, not a gate.

> **E1 RESOLVED (2026-06-07, build):** the reader is built — `scripts/hooks/daemon-findings-surface.sh`
> reads all 5 flags (`memory-consolidator`/`audit-catalog`/`steering-compact`/`bug-billboard-groom`-findings.flag
> + `inbox-tier-low-drain.flag`) and surfaces one concise WARN system-reminder at SessionStart + Stop, with an
> append-only ack ledger (`.claude/cache/daemon-findings-surfaced.jsonl`) so each finding surfaces once and a
> fresh fire re-surfaces. Kill-switches `AOS_DAEMON_FINDINGS_OFF` / `_QUIET`; CPU-only/zero-token; modeled 1:1
> on the never-idle + spawned-task-result surfacers. VERIFY: `tests/daemon-findings-surface-test.sh` (13/13 pass).
> Wiring is STAGED (user-gated) in `scripts/hooks/settings-patches/daemon-findings-surface.json` — NOT applied
> to `settings.json`. Wire it (SessionStart, optionally Stop), then flip the 5 daemons; the hook is silent until
> a flag appears, so order is flexible.

---

## Per-daemon rows

### daemon2 — `consumer-memory-consolidator-daemon2.sh`
1. **Catalog identity:** **#2 Memory-consolidator** (A65 catalog — exact match).
2. **Apply-mode behavior / mutation / reversibility / cost:** Read-only scan of memory substrate (mirror vs
   auto-load feedback counts, mirror/auto-load drift, skill-ref count, audit count, phantom MEMORY.md index
   entries). Apply mode drops `memory-consolidator-findings.flag` only when `severity != ok`. **Mutates state:
   no** (flag file only). **Reversible:** trivially (flip back; flag is idempotent overwrite). **Cost:**
   ~0 (no LLM, `duration_sec:0`, pure `ls`/`grep`).
3. **Dry-run history (2026-06-02 → 06-07):** 60 fires, 0 malformed, **60/60 `severity:ok`**, drift=0,
   phantom=0. Cleanest possible record — never even *wanted* to surface.
4. **7-gate status:** ✓ all 7. dry-run default; wrapper/launchd-only; cost gate (reads cost-state.json);
   pivot gate (30-min sentinel); idempotent; CoT-on-fire (if ledger present); steering-entry deferred to
   Python (correct — never writes steering).
5. **VERDICT: FLIP-NOW** — 60/60 clean `ok`, apply-side-effect is a flag-only drop, zero mutation/cost.

### daemon3 — `consumer-audit-catalog-scan-daemon3.sh`
1. **Catalog identity:** **#3 Audit-catalog maintainer** (exact match).
2. **Apply-mode behavior / mutation / reversibility / cost:** Read-only scan of `goal-billboard/audits/`:
   sequential-ID gaps, orphan findings files, broken intra-catalog A<N> xrefs (archived audits correctly
   excluded). Apply mode drops `audit-catalog-findings.flag` when `severity != ok`. **Mutates state: no.**
   **Reversible:** yes. **Cost:** ~0 (`duration_sec:1`).
3. **Dry-run history:** 60 fires, 0 malformed, **60/60 `severity:warn`** (gaps=5, orphan=0, xref=3). I
   independently reproduced these: gaps at audit IDs **52, 53, 54, 55, 70**; broken xrefs **A52, A53, A55**.
   These are **TRUE POSITIVES** — real catalog-hygiene findings, exactly what the daemon is for. The
   persistent `warn` is the daemon working correctly, **not** a malfunction. NOTE: the matching
   `daemon3-launchd.err` (5100 B) is **not an error log** — it is stderr capturing the by-design
   "[daemon3] DRY-RUN: … would surface" notices. Benign.
4. **7-gate status:** ✓ all 7 (same harness as daemon2).
5. **VERDICT: FLIP-NOW** — findings are real + flag-only apply. (On flip it will immediately drop one
   `warn` flag for the A52-55/A70 gaps; that's the intended surfacing. Pairs well with wiring E1's reader so
   the gap actually gets actioned. Resolving the gaps themselves is separate catalog work, not a flip blocker.)

### daemon4 — `consumer-steering-log-compact-daemon4.sh`
1. **Catalog identity:** **#4 Steering-log compactor** (exact match).
2. **Apply-mode behavior / mutation / reversibility / cost:** Read-only scan of
   `notes/steering-decisions-log.md`: byte size, section/entry counts, per-actor grep buckets, week-over-week
   byte-growth %. Apply mode drops `steering-compact-findings.flag` ONLY on growth >50% WoW or size >5 MB.
   **Mutates state: no.** Despite the scary name "**compactor**," the deployed Phase-2 consumer does **not**
   compact/edit anything; the design plan (`plans/steering-log-compactor-daemon-design.md`) is explicit that
   even future Phase-1/Phase-3 logic produces a **digest + monthly-archive SNAPSHOT (copy, not move),
   append-only-grow, source log NEVER edited.** **Reversible:** yes. **Cost:** ~0 (`duration_sec:0`).
3. **Dry-run history:** 60 fires, 0 malformed, **60/60 `severity:ok`**, byte_growth 0%, current size 176 KB
   (far under the 5 MB guard). Clean.
4. **7-gate status:** ✓ all 7 — incl. the gate-7 **special case** (must NOT write steering entries in normal
   digest mode to avoid an infinite-loop; correctly deferred/suppressed).
5. **VERDICT: FLIP-NOW** — 60/60 `ok`, flag-only apply, "compaction" is non-destructive by design.

### daemon5 — `consumer-bug-billboard-groom-daemon5.sh`
1. **Catalog identity:** **#5 Bug-billboard groomer** (exact match).
2. **Apply-mode behavior / mutation / reversibility / cost:** Read-only scan of `bug-billboard/`: inbox count,
   oldest-inbox age, stale-inbox (>24h), master open count, archived-fixed-30d, broken A<N> xref vs audit
   catalog. Apply mode drops `bug-billboard-groom-findings.flag` when `severity != ok`. **Mutates state: no**
   (header: "Daemon NEVER moves files"). **Reversible:** yes. **Cost:** ~0 (`duration_sec:0`).
3. **Dry-run history:** 60 fires, 0 malformed, **60/60 `severity:ok`** (inbox empty, no stale, no broken
   xref). Clean.
4. **7-gate status:** ✓ all 7.
5. **VERDICT: FLIP-NOW** — 60/60 `ok`, flag-only apply, explicitly never moves files.

### daemon10 — `consumer-heartbeat-tier-low-daemon10.sh`
1. **Catalog identity:** **NOT one of the 8-daemon catalog (#1-8).** Its own header: "**A65 Daemon #10 —
   periodic inbox poll (A68 Shape C: Heart tier-low daemon)**." It is the **Heart tier-low consumer that polls
   `context/markdowns/agent-inbox/`** — the inbox-surfacing sibling, not a grooming daemon. (The catalog's #6
   CoT-consolidator / #7 snapshot-scanner / #8 statistical-correlator are **not deployed** as launchd plists
   at all — they remain on the gated path / unbuilt.)
2. **Apply-mode behavior / mutation / reversibility / cost:** Counts inbox `*.md`; apply mode drops
   `inbox-tier-low-drain.flag` when items>0. **Mutates state: no.** **Reversible:** yes. **Cost:** ~0
   (smallest of the 5). Note: a *notification*-trigger sibling (`hooks/agent-inbox-drain-notification.sh`)
   already drops an equivalent `inbox-notification-drain.flag`, so daemon10's apply-mode surfacing is
   well-precedented and even partially redundant (additive, not conflicting).
3. **Dry-run history:** **61 fires** (one extra tick), 0 malformed, all `dry_run:1`, items_pending=0
   throughout (no schema has a `severity` field — it's a pure count poll). Clean.
4. **7-gate status:** ✓ all 7 (header self-certifies; matches the others' harness; A62 acid 4/4 per A68).
5. **VERDICT: FLIP-NOW** — clean poll history, flag-only apply, lowest cost of the set.

---

## How to flip (the orchestrator's gated step — NOT done here)

Use the safe-by-construction wrapper (already covers these labels in `KNOWN_SUFFIXES`):
```
bash scripts/daemons/daemon-control.sh flip com.agentic-os.daemon2  apply
bash scripts/daemons/daemon-control.sh flip com.agentic-os.daemon3  apply
bash scripts/daemons/daemon-control.sh flip com.agentic-os.daemon4  apply
bash scripts/daemons/daemon-control.sh flip com.agentic-os.daemon5  apply
bash scripts/daemons/daemon-control.sh flip com.agentic-os.daemon10 apply
```
Each `flip … apply` backs up the live plist (`.bak-<ISO>`), `plutil -lint`s it, sets `DRY_RUN=0` via
PlistBuddy, re-validates, and re-bootstraps. Fully reversible: `flip <label> dry-run`. Preview without
executing first via `AOS_DAEMON_CONTROL_DRY=1`. (Editing a LaunchAgent plist is on the A66 gated list — the
wrapper is the sanctioned, allowlistable path; the actual run is still the orchestrator's call, by design.)

**Recommended sequencing:** flip all 5 (all safe), and either before or immediately after, wire a SessionStart
flag-surfacer hook (mirror `never-idle-refill-surface.sh`) so the dropped flags reach the agent (caveat E1).
Without it the flip is safe but inert.

---

BG-COMPLETE-SENTINEL
per_daemon_verdicts:
  daemon2  (memory-consolidator #2):    FLIP-NOW — 60/60 ok, flag-only apply, 0 cost
  daemon3  (audit-catalog maintainer #3): FLIP-NOW — 60/60 warn are TRUE gaps (A52-55,A70 / xref A52,A53,A55); flag-only apply; .err is benign stderr
  daemon4  (steering-log compactor #4):  FLIP-NOW — 60/60 ok; "compact" is non-destructive (copy-not-move, source never edited)
  daemon5  (bug-billboard groomer #5):   FLIP-NOW — 60/60 ok, never moves files, flag-only apply
  daemon10 (heart tier-low inbox poll; NOT in #1-8 catalog): FLIP-NOW — clean poll, flag-only apply, lowest cost
flip_now_count: 5
fix_first_count: 0
status: COMPLETE — READ-ONLY (nothing flipped); 1 efficacy follow-up (E1: wire a flag-reader hook so apply-mode surfacing isn't inert); catalog daemons #6/#7/#8 are NOT deployed (gated/unbuilt)
