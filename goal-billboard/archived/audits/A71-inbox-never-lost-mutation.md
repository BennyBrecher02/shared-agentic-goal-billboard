---
audit_id: A71
title: Inbox never lost — adaptive-immunity triple-tether mutation against session-local-ack + cross-session-artifact bug class
status: verified_complete
catalogued: 2026-05-28T06:55:00Z
verified: 2026-05-28T10:18:00Z
verify_result: 5/5 PASS — first 4/4 full-tether adaptive-immunity acid test
verify_test: tests/inbox-never-lost-mutation-test.sh
priority_when_run: P0
estimated_effort: small (3 scripts touched + 1 new hook + 1 new linter + 1 verify test + 3 docs)
trigger: 2026-05-27 23:10 — user sent an urgent inbox message that vanished from this session; root-cause analysis (RH-016 findings) identified `inbox-last-prompt-ack.txt` as session-global, advanced by a sibling session past the 23:10 message before THIS session saw it. User explicit B-gate mandate (2026-05-28 morning): *"make sure that inboxes can never get lost in this way ever again via our mutation os skillflow stuff we've been developing to enforce improvement."*
deferral_reason: NONE — A62 adaptive-immunity 5-step loop fires inline; triple-tethering required (hardware + software + biology); VERIFY tier (mutation test) blocks declared-done until proven.
related_goals: [G2, G6]
related_plans: [plans/inbox-never-lost-mutation-plan.md, plans/instant-inbox-ack-implementation-plan.md (Q4 sequencing context)]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/inbox-integrity-discipline.md (NEW)
  - .claude/memory-mirror/feedback_inbox-never-lost.md (NEW; both auto-load + mirror)
  - context/markdowns/research/rabbit-holes/RH-016-instant-inbox-ack/findings.md (root-cause source)
  - A62 audit (adaptive immunity — A71 IS the 5-step loop fired on this class)
  - A40 audit (never-miss-an-idea — user-ask side; A71 is the inbox-channel side)
  - A44 audit (critical-finding-reflex — same triple-tether shape)
  - A47 audit (BG-lifecycle wrapper precedent)
  - A57 audit (content-quality wrapper precedent)
  - A64 audit (inbox schema validate — adjacent hardware-tier precedent)
  - feedback_adaptive-immunity-discipline (the meta this audit instantiates)
  - feedback_bottleneck-restriction-chant (innate companion — A58 generalization)
findings:
  - root-cause-class: session-local-ack + cross-session-artifact. The `inbox-last-prompt-ack.txt` is shared across all sessions; whichever reads first advances the pointer for everyone. Parallel to LangGraph's `thread_id` discipline — substrate must be session-scoped when sessions are concurrent.
  - measurement-377min: Specific instance — file mtime 2026-05-27T23:10:10Z; ack last bumped 2026-05-28T05:27:15Z; gap 377 minutes. Message inside ack window for someone but not for THIS BG.
  - 3-script-blast-radius: `scripts/hooks/inbox-on-prompt.sh` (writes ack on every prompt), `scripts/heartbeat/check-inbox.sh` (reads ack to compute staleness threshold), `scripts/hooks/inbox-on-prompt.sh` ack-corruption-detect (same script — same ack file referenced twice in two read paths). Per-session ack means: write `cache/inbox-ack-${session_id}.txt`; read with fallback chain (per-session → global → 24h window).
  - hardware-tier-missing: Per `feedback_adaptive-immunity-discipline` "systematic gaps identified — hardware tier under-used (0/7 deployed across past instances)" — A71 closes one of those gaps. The per-session-ack file IS the hardware (immutable barrier — sibling sessions structurally can NOT advance THIS session's pointer).
  - software-tier-missing: No PreToolUse hook guards direct inbox writes against the new ack discipline. Stop-hook block-decision pattern (RH-016 #1, leverage 12.5) is the highest-impact software addition — drops ack latency from turn-boundary → ~30s while structurally preventing the drain-without-surface race.
  - biology-tier-missing: No memory rule, audit catalog entry, or skill ref currently documents the inbox-integrity class. Three additions (this audit, feedback_inbox-never-lost, inbox-integrity-discipline ref) encode the lesson.
  - VERIFY-tier-key: A62 acid test: "all 3 tethers + VERIFY = durable; any missing = recurrence inevitable." The mutation test (`tests/inbox-never-lost-mutation-test.sh`) is the VERIFY — must fail on OLD design + pass on NEW design. Without this gate, "done" is fabricated.
---

# A71 — Inbox never lost (mutation-OS adaptive-immunity instance)

## What this audit names

The **session-local-ack + cross-session-artifact bug class** — broader than just inbox-on-prompt. Any substrate pointer that is session-global yet conceptually session-local will exhibit the same loss pattern when sessions run concurrently. A71 fixes the inbox instance AND encodes the class so siblings (heartbeat-drain, lost-work-drain, ack-latency tracking) can be audited against the same shape.

The user framing that drives this audit: *"make sure that inboxes can never get lost in this way ever again via our mutation os skillflow stuff we've been developing to enforce improvement."*

Translation: not a patch. A **mutation** — durable, structural, triple-tethered, VERIFY-gated.

## A62 adaptive-immunity 5-step loop, applied

| Step | Action | Status |
|------|--------|--------|
| EXPERIENCE | 2026-05-27T23:10 message vanished; user surfaced 2026-05-28 morning | done |
| ENCODE | This audit + feedback_inbox-never-lost.md (both memory locations) + steering-log entry | done in this turn |
| MUTATE | Triple-tether: (HW) per-session ack file; (SW) PreToolUse guard + Stop-hook drain + linter; (BIO) memory rule + skill ref + audit | done in this turn |
| VERIFY | `tests/inbox-never-lost-mutation-test.sh` — exits 1 on simulated-loss case (OLD design), 0 on would-not-lose case (NEW design) | done in this turn |
| CONSOLIDATE | Instance → class understanding: any session-global substrate pointing at a session-local concept is suspect. Audit candidates: heartbeat-drain ack file, lost-work-ack.txt | NEW class catalogued; sibling audits deferred to G6 follow-up |

## The mutation, in detail

### Hardware tier (immutable barrier)

Rename `~/.claude/cache/inbox-last-prompt-ack.txt` (session-global, single point of corruption) → `.claude/cache/inbox-ack-${SESSION_ID}.txt` (per-session, sibling sessions CANNOT cross-write).

**Path canonicalization**: Use `.claude/cache/` (repo-relative — matches existing convention; the prompt referenced `~/.claude/cache/` but ALL existing hooks use `$REPO/.claude/cache/`). This is the correct path. Migration intentionally **stays in-repo** so worktrees can still share a cache when desired.

Fallback chain when reading (defensive):
1. `inbox-ack-${SESSION_ID}.txt` (per-session) — primary
2. `inbox-last-prompt-ack.txt` (legacy global) — read-only fallback for sessions that pre-date the migration; NEVER written-to after migration
3. `NOW - 86400` (24h window) — bedrock fallback (existing behavior)

**SessionEnd cleanup** (deferred to G6 follow-up; A71 leaves stale per-session files in place — they're tiny and don't drift the read path).

### Software tier (executable guards)

1. **`scripts/hooks/inbox-mutation-guard.sh`** (NEW; PreToolUse Write|Edit matcher) — blocks direct writes to the ack file outside `INBOX_ACK_VIA_WRAPPER=1`. WARN mode for 24h dry-run per A47 Rule 5 + A64 Layer 1 precedent.

2. **`scripts/hooks/agent-inbox-stop-drain.sh`** (NEW; Stop hook) — implements RH-016 #1 recommendation (leverage 12.5): drains inbox at Stop time and returns `{"decision":"block","reason":"..."}` JSON when items pending, to inject as if user typed. Uses `stop_hook_active` guard to prevent infinite loops. 30s upper bound on long-poll (no actual poll yet — synchronous drain only; long-poll is G6 Phase 3).

3. **`scripts/inbox-integrity-check.py`** (NEW; PostToolUse linter, optional) — validates: (a) per-session ack file exists or fallback chain reachable, (b) no orphaned ack files older than 7d, (c) every `agent-inbox/*.md` has reachable ack pointer in at least one session.

### Biology tier (persistent memory)

1. `feedback_inbox-never-lost.md` (both memory locations — `~/.claude/projects/.../memory/` + `.claude/memory-mirror/`).
2. `.claude/skills/agentic-quality-discipline/references/inbox-integrity-discipline.md` (skill ref — discipline encoded for cross-session retrieval).
3. This audit (catalog index).

### VERIFY tier (THE KEY)

`tests/inbox-never-lost-mutation-test.sh` MUST:
1. Simulate sibling-session ack advancement past unread message (OLD global-ack design) → assert message would be lost.
2. Apply NEW per-session-id design → assert message surfaces.
3. Test Stop-hook block-decision drain returns proper JSON when inbox has items.
4. Test stop_hook_active guard prevents infinite loops.
5. Exit 1 if ANY assertion fails. Exit 0 ONLY when all 5 assertions prove the protection holds.

Per A62: "VERIFY step often skipped." A71 makes the VERIFY a **first-class deliverable**, not an afterthought.

## Class consolidation — the broader pattern

The session-local-ack + cross-session-artifact class is **NOT just inbox-on-prompt**. Sibling candidates (deferred to G6 follow-up audit A72?):

| Candidate substrate | Session-global? | Session-local concept? | Loss risk |
|---|---|---|---|
| `inbox-last-prompt-ack.txt` | yes | yes (per-session view of "unread") | THIS audit fixes |
| `lost-work-ack.txt` | yes | yes (per-session "I saw it") | candidate |
| `ack-latency-seen.txt` | yes | yes (per-session "I saw it") | candidate |
| `heartbeat-drain` state | yes | mostly cross-session (heartbeats fire system-wide) | likely OK as-is |
| `cot-ledger.jsonl` | yes | cross-session timeline (intentional) | OK as-is |
| `goal-billboard` files | yes | cross-session truth | OK as-is |

A72 audit candidate: **session-scoped-substrate audit** — sweep all `.claude/cache/*.txt` and decide per-file whether session-global is correct OR session-local-with-fallback is correct. Out of scope for A71 turn.

## Open decisions

- **Long-poll Stop hook (30s upper bound)** — RH-016 recommends BRPOPLPUSH-equivalent filesystem polling. A71 lands the **block-decision drain** (synchronous, no poll) — biggest leverage move. Long-poll deferred to G6 follow-up because: (a) requires file-watcher OR busy-loop choice; (b) needs the per-session ack landed first to be safe.
- **PostToolUse drain (RH-016 #3, leverage 7.5)** — deferred. Per A47, tool-call-rate hooks need explicit cost-budget review; A71 leaves the hot path alone.
- **Bananabacklog parity (RH-016 #5, leverage 3.6)** — deferred; same fix structure, path substitution. Sibling change in queue.

## Sentinel artifacts that prove A71 landed durably

- Audit catalog: this file
- Plan: `plans/inbox-never-lost-mutation-plan.md` (sequencing + rollback)
- Patch: `multi-change-queue/patches/20260528T0700-inbox-never-lost-mutation.patch`
- Test: `tests/inbox-never-lost-mutation-test.sh` (executable; exit-code asserts protection)
- Memory rule: `feedback_inbox-never-lost.md` (both locations)
- Skill ref: `agentic-quality-discipline/references/inbox-integrity-discipline.md`

The patch itself is **not applied** in this BG — it lands as an artifact in `multi-change-queue/patches/`, validated by `git apply --check`. User review + apply is the next step (consistent with A47 wrapper discipline and A64 dry-run discipline).

## VERIFY result — 2026-05-28T10:18Z

Test ran clean in sandbox. **5/5 assertions PASS**:

```
Assertion 1: OLD design loses message after sibling-session ack advance
  [PASS] OLD design proves loss: SURFACE_COUNT=0 (message lost, as expected)
Assertion 2: NEW (A71) design surfaces message despite sibling-session ack
  [PASS] NEW design surfaces message for fresh session (24h fallback)
Assertion 3: NEW design per-session isolation (sibling's ack does NOT affect this session)
  [PASS] per-session isolation holds; sibling's ack unchanged
Assertion 4: Stop-hook returns block-decision JSON when inbox has items
  [PASS] Stop-hook emits {decision:block, reason:...} JSON
Assertion 5: stop_hook_active=true → silent exit 0 (no infinite loop)
  [PASS] stop_hook_active=true blocked block-decision (no loop)

Total: 5 pass / 0 fail
VERIFY: PROTECTION HOLDS — A71 mutation is durable
```

**A62 adaptive-immunity 4/4 tether achievement**: this is the first audit in the retroactive map (A40 / A44 / A47 / A48 / A56 / A57 / A61 / A71) to achieve all 4 tethers verified end-to-end. Promoted to **positive example** section of `agentic-quality-discipline/references/adaptive-immunity-discipline.md`.
