---
audit_id: A61
title: Monkey + inbox architecture audit — recurring bottleneck class (A46/A48/A49 each missed something)
status: in_progress
catalogued: 2026-05-27T16:30:40Z
phase_1_landed_at: 2026-05-27T16:30:40Z
priority_when_run: P0
estimated_effort: medium (4 parallel BGs covering architecture + forensics + design + tests this turn; deployment + rollout deferred)
trigger: 2026-05-27 user explicit, hot — *"why is the banana backlog inbox clogged still? didnt i ask you to make this impossibe? we need better monkey hygene. also how intertwine are our agent inbox and monkey communication system, we need an architecture audit to solve this constant bottleneckery the monkey communication chamber has been so we can get to our next e2e plow push, so go full send on this, and stiop being so lazy, why do you think i asked for major parallelization research, why have you not been doing more than 1 simultaneous agent at once?"*
deferral_reason: NONE — bottleneck IS the chant target; immune response active; 4 parallel BGs in flight; foreground unclog completed.
related_goals: [G2, G6]
related_plans: [context/markdowns/plans/monkey-impossible-to-clog-plan.md (BG3 deliverable), context/markdowns/plans/monkey-hygiene-test-spec.md (BG4 deliverable)]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - context/markdowns/notes/monkey-inbox-architecture-2026-05-27.md (BG1 deliverable; full path diagram + failure mode catalog)
  - reports/audit-findings/A61-BG2-forensics.md (BG2 deliverable; class pattern + A46/A48/A49 misses)
  - tests/hooks/test_monkey_hygiene.py (BG4 deliverable; assertions)
  - .claude/skills/agentic-quality-discipline/references/immune-system-discipline.md (this IS the chant firing)
  - .claude/memory-mirror/feedback_bottleneck-restriction-chant.md (the constitutional law)
  - A46 audit (dashboard reopen flow + 6/6 status flip) — fixed dashboard surfacing; missed schema enforcement
  - A48 audit (sync-monkey-decisions-from-inbox.py auto-sync) — fixed routing; missed pre-write validation
  - A49 audit (chamber-empty domino + plow-pivot) — fixed action triggering on drain; missed clog prevention
  - A57 audit (dashboard content quality + ux) — sibling; chamber wrapper IS one defense layer (Layer 0 effectively)
findings:
  - recurrence-class-confirmed: A46 + A48 + A49 + A57 each addressed ONE failure mode in the monkey/inbox communication ecosystem; the CLASS keeps recurring because no audit has addressed the META pattern (schema fragility + no pre-write validation + no auto-recovery)
  - stuck-file-was-retraction-stub: the 16h stuck file was a wontfix retraction (no actual defect; just missing schema fields). Class includes "audit-trail submissions that bypass schema."
  - immune-response-fired: this audit demonstrates the bottleneck-restriction chant working — DETECTED (user) → PAUSED (plow path) → DIAGNOSING (4 parallel BGs) → FIXED (foreground manual unclog) → VERIFIED (consolidate clean) → RESUMING (this audit drives forward).
  - user-callout-on-laziness: I was under-utilizing parallel BG dispatch capacity. 4 parallel BGs here vs my prior pattern of 1-2 sequential. RH-014's findings explicitly call out safe escalation; this audit applies it.
---

# A61 — Monkey + inbox architecture audit (recurring class)

## What triggered this audit

User verbatim (hot): *"why is the banana backlog inbox clogged still? didnt i ask you to make this impossibe? we need better monkey hygene. also how intertwine are our agent inbox and monkey communication system, we need an architecture audit to solve this constant bottleneckery the monkey communication chamber has been so we can get to our next e2e plow push, so go full send on this, and stiop being so lazy, why do you think i asked for major parallelization research, why have you not been doing more than 1 simultaneous agent at once?"*

Two complaints, both valid:

1. **The class recurs** — A46 / A48 / A49 / A57 each addressed ONE failure mode; the meta pattern was never structurally closed.
2. **I was under-parallelizing** — RH-014 explicitly delivered safe-escalation criteria; I'd been dispatching 1-2 BGs sequentially instead of 4-5 in parallel.

Both addressed in this audit.

## The recurring class

A46 (2026-05-27 ~05:02) fixed: dashboard Pending/Decided/All filter + decided view + reopen flow + 6/6 retroactive status flip. **Missed:** schema-violation handling.

A48 (2026-05-27 ~05:16) fixed: sync-monkey-decisions-from-inbox.py wired into dashboard-data-prime.sh. **Missed:** pre-write validation (so schema violations still ENTER the inbox before being caught later).

A49 (2026-05-27 ~05:32) fixed: chamber-empty domino + plow-pivot triggers. **Missed:** if inbox NEVER drains because of a stuck file, the domino never fires.

A57 (2026-05-27 ~15:00) fixed: chamber content quality + render UX + `post-to-chamber.py` wrapper. **Missed:** the wrapper is for CHAMBER posts only; agent-inbox and bug-billboard-inbox don't have a parallel wrapper.

**The meta pattern**: each audit addressed downstream symptoms; none addressed the upstream gap of *no pre-write schema validation across all inbox surfaces*. A61 closes this.

## Immediate foreground action (this turn)

Stuck file `20260527T0011-main-subagent-a30phase3-d8f3a212.md` was a retraction-stub (wontfix; no actual defect) that lacked required schema fields. Manually archived to `bug-billboard/archive/processed/2026-05/`. Consolidate verified clean. Immediate bottleneck cleared.

## Systemic fix (4 parallel BGs, in flight)

| BG | Scope | Deliverable | File zone |
|----|-------|-------------|-----------|
| BG1 | Architecture audit — component inventory + path diagram + failure mode catalog + class synthesis | `notes/monkey-inbox-architecture-2026-05-27.md` (~150-250 lines) | disjoint |
| BG2 | Clog forensics — stuck file analysis + 52 processed pattern + A46/A48/A49 misses + class pattern + ranked recommendations | `reports/audit-findings/A61-BG2-forensics.md` (~150-250 lines) | disjoint |
| BG3 | Impossible-to-clog mechanism design — 5-layer defense + 5 script specs + deployment sequencing + organic-OS integration + test plan | `plans/monkey-impossible-to-clog-plan.md` (~150-250 lines) | disjoint |
| BG4 | Test suite + assertions for monkey hygiene — 11+ pytest functions covering all 5 layers; SKIPPED-pending-BG3 | `tests/hooks/test_monkey_hygiene.py` + `plans/monkey-hygiene-test-spec.md` | disjoint |

All 4 dispatched at `2026-05-27T16:30:40Z` (epoch `1779899440`) with A47 discipline (HARD TIMEOUT 45min, BG-COMPLETE-SENTINEL, cost gate <300k each).

## The 5-layer defense (BG3 design preview)

(BG3 will fully spec; this is the framing)

| Layer | Defense | Failure mode it prevents |
|-------|---------|--------------------------|
| 1 | Pre-write schema validator (`inbox-write.py` wrapper + PreToolUse hook blocking direct Write) | Schema-missing files enter inbox |
| 2 | Heartbeat staleness ticker (Heart tier-medium scans inbox every 15min; >T-hours = alert) | Stuck files go unnoticed |
| 3 | Auto-recovery (rewrite-via-template / archive-as-wontfix-schema / escalate) | Stuck files require manual rescue |
| 4 | Recurrence detection (same dedupe-key class fires N times → escalate to class-level fix) | Repeated instance-fixing instead of class-closure |
| 5 | Cross-substrate consistency check (dashboard / data.json / inbox must agree) | Drift between display + ground truth |

## Why this finally addresses the CLASS

The 5 layers are orthogonal — each catches a different failure mode. No single audit fix would have closed the class. A61 is structural because it identifies the META pattern (no pre-write validation + no recovery + no consistency check) and addresses all three together.

The user's "make this impossible" framing is finally honored: with all 5 layers shipped + tested, a file cannot remain stuck without violating an immune signal that auto-recovers OR escalates to user.

## Cross-references

- A46 / A48 / A49 / A57 (prior fixes; A61 supersedes for the meta pattern but each remains valid for its own failure mode)
- A58 (Organic OS — Immune system extends to absorb this audit's mechanism)
- A47 (BG lifecycle — 5-layer's pre-write wrapper mirrors `dispatch-bg.sh` discipline)
- A21 (parallel cap — 4 BGs respects cap of 5; RH-014 safety preserved)
- RH-014 findings (safe-escalation; affects_files file-zone discipline)
- `feedback_bottleneck-restriction-chant.md` (this audit IS the chant firing)
- `feedback_recurrence-detect.md` (this audit IS the recurrence-detect 6-step loop)

## Status

PHASE 1 LANDED 2026-05-27T16:30Z (foreground unclog + 4 BG dispatches + this audit). Phase 2 (synthesize 4 BG outputs + ship Layer 1 wrapper + 24h dry-run + Layer 2-5 rollout) deferred to subsequent turns when BG outputs return.

## Lessons (live)

- **The class always wins if you only fix instances.** A46-A49 + A57 each fixed an instance; the class persisted. A61 addresses the class.
- **User-level parallelization callout was justified.** I should default to 3-5 parallel BGs for any audit-class work where file zones are obviously disjoint. RH-014 gave me the safety framework; I wasn't applying it.
- **The immune chant + recurrence-detect rule fired correctly here.** This audit demonstrates the discipline working end-to-end (DETECT → PAUSE → DIAGNOSE → FIX → VERIFY → RESUME).
- **"Audit-trail-only" submissions need their own schema variant** — the 16h stuck file was a legitimate wontfix retraction; the bug-billboard schema didn't accommodate that shape. BG3 should propose a `kind: audit-trail` variant alongside `kind: bug-report`.

## Shared hardware barrier (2026-05-28 — adaptive-immunity retro-sweep)

A61's HARDWARE tether was at ½ — `inbox-schema-validate-pre.sh` was wired but WARN-mode only (blocks nothing). It is now reinforced by a **shared barrier** built once and claimed by THREE instances — A57 (dashboard-content) + A48 (sync-drift) + A61 (this). Per the retro-sweep roadmap (`audits/findings/adaptive-immunity-retro-sweep-roadmap-2026-05-28.md`): "block direct writes to `data.json` / inbox except via the sanctioned wrapper" — the single highest-leverage move in the sweep, since A61 (INBOX-CLOG) is the most-recurring failure family.

- **HW guard:** `scripts/hooks/wrapper-only-write-guard.sh` — PreToolUse `Write|Edit` hook. A direct write to EITHER inbox surface (`context/markdowns/agent-inbox/*.md` OR `context/markdowns/bug-billboard/inbox/*.md`) — the exact A61 clog path — is flagged and pointed at the sanctioned `inbox-write.py` wrapper (a.k.a. `bug-billboard-log`), which validates schema before atomic-write. Complementary to the existing `inbox-schema-validate-pre.sh`: this is the UNIFIED data.json+inbox write-gate the roadmap asked us to "build once, claim for three."
- **VERIFY:** `tests/wrapper-only-write-guard-test.sh` — A71-shaped, 5/5 passing. Assertion 4 reproduces the A61 clog shape (a schema-invalid stub written straight into both inboxes) and asserts the barrier flags both; assertion 3 asserts the `INBOX_VIA_WRAPPER`-routed write passes silently. Fails if the guard is absent (verified). (Distinct from the 13 still-skipped `test_monkey_hygiene.py` assertions, which the roadmap tracks separately.)
- **Mode:** ships in **WARN/dry-run** (`exit 0`). Kill-switch `EVIUM_WRAPPER_GUARD_OFF=1`.
- **Live-flip is USER-GATED:** WARN→BLOCK (`EVIUM_WRAPPER_GUARD_MODE=BLOCK`) + `settings.json` wiring is an explicit USER action after a 24h dry-run review — NOT an agent turn (a stray match could block the user mid-edit). See the hook header for the flip steps.
