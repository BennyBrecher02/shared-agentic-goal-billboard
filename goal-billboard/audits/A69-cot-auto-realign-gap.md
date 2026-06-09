---
audit_id: A69
title: CoT auto-realign gap — Brain discipline written but never executes
status: in_progress
catalogued: 2026-05-27T21:55:00Z
priority_when_run: P0
estimated_effort: medium (3 hook scripts + ledger seed + settings.json wiring + 24h dry-run per A47)
trigger: 2026-05-27T21:50Z user — "we need a memory reread for full realignment before we start. its annoying to ask this, what ever happened to our self aligning CoT we fleshed out?"
related_plans:
  - context/markdowns/plans/cot-auto-realign-fix-plan.md
serves_northern_star: G2
belongs_to_goal: G4
related_refs:
  - .claude/skills/agentic-quality-discipline/references/chain-of-thought-discipline.md
  - context/markdowns/organic-os/brain/README.md
  - .claude/memory-mirror/feedback_critical-finding-reflex.md
  - .claude/memory-mirror/feedback_bottleneck-restriction-chant.md
  - .claude/memory-mirror/feedback_adaptive-immunity-discipline.md
findings:
  - cot-ledger-never-written-this-session — .claude/cache/cot-ledger.jsonl does not exist; zero PLOWING/BOTTLENECK/DONE entries since the discipline ref landed (A58 Phase 1 commit fd7264c, 2026-05-27)
  - three-cot-scripts-never-built — chain-of-thought-discipline.md cites scripts/hooks/cot-checkpoint-pre.sh + cot-resume.sh + cot-log.sh as "proposed"; none exist on disk anywhere outside markdown
  - settings-json-has-no-cot-entry — zero `cot` matches in .claude/settings.json across SessionStart / UserPromptSubmit / PreToolUse / PostToolUse / Stop
  - cot-is-not-isolated-instance — 25 of 53 hook scripts in scripts/hooks/ are orphaned (written but not referenced in settings.json); class-broken substrate, not one-off
  - memory-rule-mismatches-reality — feedback_standing-protocols.md states "Hooks at user-ask-detect.sh (UserPromptSubmit) + user-ask-artifact-check.sh (Stop) catch + enforce"; both scripts exist but neither is wired in settings.json
---

# A69 — CoT auto-realign gap

## What this audit names

The Chain-of-Thought (CoT, the Brain of the Organic OS — A58) is documented as a STANDING discipline across two skill refs and a Brain README. It defines a 7-state model, JSONL ledger schema, mandatory write triggers, and SessionStart/PreToolUse hook protections. None of it executes. The ledger file does not exist. The hook scripts do not exist. The settings.json wiring does not exist. The discipline has produced zero entries in any session since it was written.

This was flagged by user 2026-05-27T21:50Z: *"we need a memory reread for full realignment before we start. its annoying to ask this, what ever happened to our self aligning CoT we fleshed out?"*

This is a **literal A44 critical-finding-reflex trigger** ("Built but not [used/queried/tested]"). It is also the canonical anti-pattern of `feedback_adaptive-immunity-discipline.md`: software-tier tether documented; hardware-tier tether (the actual hook) never deployed. VERIFY step skipped.

## Classification

**Class: substrate-broken** (most severe of the three A44 categories).

Reasoning: the broken substrate is **the gap between skill-ref-discipline and hook-enforcement**. CoT is the instance that surfaced. Cross-check below shows 25 sibling orphans. If we fix only CoT, the class recurs at the next "I wrote a discipline that doesn't fire" moment.

## Why the gap exists (forensic)

### 1. The discipline ref proposes mechanism but uses passive voice

`chain-of-thought-discipline.md` §"The dive-past-bottleneck protection":

> A PreToolUse hook (proposed: `scripts/hooks/cot-checkpoint-pre.sh`) on every `Edit`/`Write`/`Bash` invocation checks the most recent CoT entry.

The word "proposed" describes intent. There is no acceptance criterion that flips "proposed" → "deployed." A reader (human or agent) cannot distinguish "shipped" from "queued" from "never built." 

### 2. The Brain README echoes the same proposed-status without a deployment log

`organic-os/brain/README.md` §"The check that prevents dive-past-bottleneck":

> Hook `scripts/hooks/cot-checkpoint-pre.sh` (proposed; deploy with 24h dry-run): on every `Edit`/`Write`/`Bash` tool invocation, the hook checks the recent CoT ledger.

Same problem. Two documents declare a proposal in parallel. Neither tracks acceptance.

### 3. No write-side tool

There is no `cot-log.sh` (or `cot-write.py`, or any CLI). To write a CoT entry the agent would have to hand-craft JSONL, generate an epoch-seeded ID, find the parent_id, and append to a file that does not exist with no directory pre-created. The cost of compliance is so high that even an agent that remembers the discipline will skip it under load — which is exactly what has happened.

### 4. No read-side tool either

There is no `cot-resume.sh` (the documented SessionStart helper). The skill ref includes example `jq` queries but no shell wrapper. SessionStart has 13 hooks wired; none consult the CoT state. The "cross-session continuity" promise in the discipline ref is unreachable.

### 5. The class is invisible from inside the discipline

The skill ref does not require checking "does this hook actually run on every Edit?" There is no health check on whether the Brain wired up correctly. The system does not detect its own broken substrate.

### 6. Memory rules confidently claim wired status

`feedback_standing-protocols.md` (A18 family) states: *"Hooks at user-ask-detect.sh (UserPromptSubmit) + user-ask-artifact-check.sh (Stop) catch + enforce."* Both scripts exist. Neither is wired in settings.json. The memory rule is wrong but reads as authoritative — so a future agent reading memory believes the protection is live when it is not.

## What should happen (per the discipline)

Every prompt-handling cycle:

1. **SessionStart**: read tail of `cot-ledger.jsonl`; if last state is BOTTLENECK, surface the blocker to user. (`cot-resume.sh` — does not exist)
2. **UserPromptSubmit** or first user-correction marker (*"we strayed"*, *"re-think this"*, *"go back"*, *"realign"*): write USER-CORRECTION entry + transition state. (no wrapper exists)
3. **Plow start**: write PLOWING entry. (no wrapper)
4. **Encountering an unexpected blocker**: PAUSE, write BOTTLENECK entry, then decide pivot vs continue. (no wrapper)
5. **PreToolUse on Edit/Write/Bash**: if last state == BOTTLENECK and tool call is not a SIDETRACK-INTENT followup, surface WARN. (`cot-checkpoint-pre.sh` — does not exist)
6. **DONE**: when primary goal closes, write DONE + steering log entry.

What actually happens: agent reads memory → notices CoT discipline → has no script to write entries → writes none → next session repeats.

## What's actually happening

| Phase | Documented behavior | Real behavior |
|-------|---------------------|---------------|
| SessionStart | `cot-resume.sh` surfaces last state | No hook; no ledger; nothing surfaced |
| First user prompt | USER-CORRECTION entry on realignment trigger | No entry |
| Plow start (e.g. THIS bg dispatch) | PLOWING entry | No entry |
| Tool calls under bottleneck | WARN from PreToolUse hook | No warning; no checking |
| Session end | Ledger persisted to reports/ for next session | Nothing to persist |
| Cross-session resume | Read tail; resume awareness | Nothing to read |

## Verification of the gap

```
$ ls -la .claude/cache/cot-ledger.jsonl
ls: ...: No such file or directory

$ ls scripts/hooks/cot-*
zsh: no matches found

$ grep -i cot .claude/settings.json
(zero matches)
```

## A62 acid test (triple-tethering check)

| Tier | Status |
|------|--------|
| Hardware (immutable barriers) | **MISSING** — no hooks wired; no settings.json entry |
| Software (executable guards) | **MISSING** — no scripts exist; cot-log.sh / cot-checkpoint-pre.sh / cot-resume.sh all unbuilt |
| Biology (persistent memory) | PARTIAL — discipline ref + Brain README exist but mark mechanism as "proposed" indefinitely |
| VERIFY | **MISSING** — no test asserts "writing a fake BOTTLENECK entry causes the WARN to fire on next Bash call" |

0 of 4 tiers deployed. Worst-scored adaptive-immunity response in our catalog. Recurrence guaranteed until structurally fixed.

## Cross-check — the broader class of "built but not wired"

`scripts/hooks/` contains **53 .sh files**. `.claude/settings.json` references **28** of them. **25 are orphaned**:

```
ack-latency-measure.sh
bg-auto-stop.sh                 (A47 — memory rule claims wired)
bg-stuck-warn.sh
chamber-empty-detect.sh
established-workflows-check.sh  (A19 — memory rule claims wired)
event-bus-dispatch-trigger.sh
heartbeat-drain.sh
hedge-detect-drain.sh
hedge-detect.sh
hooks-health-check.sh           (the meta-check that would have caught this)
lost-work-drain.sh
lost-work-token-cost.sh
ns-advance-check.sh
parallel-capacity-check.sh
pending-parallel-work-scan.sh
recurrence-detect.sh            (A19 — memory rule claims wired)
research-cross-link-harvest.sh
research-folder-advise.sh
research-surface-on-resume.sh
scheduler-heartbeat-coordinator.sh
snapshot-baseline-staleness-check.sh
state-batch-digest.sh           (A19 — memory rule claims wired)
test-e2e-dev-server-check.sh
user-ask-artifact-check.sh      (A18 — memory rule claims wired)
user-ask-detect.sh              (A18 — memory rule claims wired)
```

Six of these orphans are explicitly claimed as "wired" by memory rules (marked above). Memory is lying to the agent. The agent reads "user-ask-detect.sh is wired" and trusts it; the hook never fires; the user-asks-don't-slip protection from A18 is illusory in practice.

Additionally, the following scripts named in memory rules **do not exist on disk at all**:

- `scripts/detect-idea-gaps.py` (A40 / never-miss-an-idea)
- `scripts/hooks/idea-gap-surface.sh` (A40)
- `scripts/hooks/idea-gap-close.sh` (A40)
- `scripts/bottleneck-detect.sh` (A58 / bottleneck-restriction-chant)
- `scripts/hooks/cot-checkpoint-pre.sh` (this audit)
- `scripts/hooks/cot-resume.sh` (this audit)
- `scripts/hooks/cot-log.sh` (this audit)

So the substrate-broken pattern is at least 3 forms deep:
1. Script exists, not wired (25 cases)
2. Script doesn't exist, memory claims it does (3 idea-gap cases)
3. Script doesn't exist, discipline-ref calls it "proposed" indefinitely (3 CoT cases + bottleneck-detect)

## Why this is the same pattern as A40 / A44

- **A40 (never-miss-an-idea)**: "every user prompt MUST reach one of 4 closure paths." Mechanism cited: `scripts/detect-idea-gaps.py` + SessionStart hook `idea-gap-surface.sh`. Neither file exists. Same pattern.
- **A44 (critical-finding-reflex)**: "EVERY agent response — Stop hook scans output for trigger markers." There is no Stop hook scanning agent output for the 11 trigger markers. Same pattern.
- **A58 (bottleneck-restriction-chant)**: "Enforced via Brain (CoT BOTTLENECK state) + Heart (tier-medium bottleneck-detect fire) + Immune (`bottleneck-detect.sh` proposed)." None of the three enforcement mechanisms exist as scripts. Same pattern.
- **THIS audit (A69 / CoT)**: same pattern.

The meta-pattern: **STANDING PROTOCOLS that the agent commits to in memory are not paired with executable enforcement.** When session pressure rises, the agent skips the discipline. The user notices. The user has to reissue the directive. Recurrence is the substrate.

## What the fix must do

1. Build the missing CoT scripts (write-side `cot-log.sh`, read-side `cot-resume.sh`, guard-side `cot-checkpoint-pre.sh`) — the SOFTWARE tether.
2. Wire them into `.claude/settings.json` (SessionStart + PreToolUse + on-realign trigger) — the HARDWARE tether.
3. Seed the ledger with an initial PLOWING entry so the first read doesn't fail — the BIOLOGY tether continuity.
4. Update discipline refs to remove "proposed" status when deployed; add an acceptance log — the BIOLOGY tether honesty.
5. Add a `hooks-health-check.sh` wire that runs at SessionStart and surfaces orphan hooks (close the meta-gap) — the VERIFY tether.
6. Build a regression test that asserts: writing a BOTTLENECK entry to a fresh ledger triggers the WARN on the next Bash call — the VERIFY tether.

All six are in the plan.

## Honesty about scope of THIS audit

This audit names CoT specifically (per user prompt) but documents the class. The plan handles CoT end-to-end and proposes a **second plan** (parent: A69) to close the 25-orphan substrate class. That parent-class plan is sketched in the cross-check section of `cot-auto-realign-fix-plan.md` but not authored as a separate artifact in this BG (out of file-zone scope; ask gate before authoring).

## Cross-references

- A18 — trigger-phrase rule (user-side; same pattern: claims wired, isn't wired)
- A19 — recurrence detection + established-workflows + state-batch-digest (same pattern, 3 instances)
- A40 — never-miss-an-idea (same pattern, 3 instances; scripts don't exist)
- A44 — critical-finding-reflex (the meta-trigger that fired on this)
- A47 — bg-lifecycle (the 24h dry-run precedent for new hooks; bg-auto-stop also orphaned despite the discipline)
- A58 — bottleneck-restriction-chant + organic-os overhaul (Brain/Heart/Immune triplet — Brain is THIS audit's domain)
- A60 — goal-billboard × CoT intertwining (frontmatter has `cot_chain_id` field; nothing populates it)
- A62 — adaptive-immunity (the triple-tethering test this audit fails on 0/4 tiers)
- A66 — decision-handling (the agent's bias toward documenting-without-wiring is a decision-handling failure pattern)
- A68 — poll-gap class (analogous broken substrate, also still has user-action gates pending)
- Plan: `context/markdowns/plans/cot-auto-realign-fix-plan.md`
- Hook scratch: `context/markdowns/plans/scratch/cot-auto-realign-hook.sh`
- Memory rule update: `.claude/memory-mirror/feedback_critical-finding-reflex.md` + `~/.claude/projects/.../memory/feedback_critical-finding-reflex.md` (new "discipline-without-mechanism" anti-pattern section)
