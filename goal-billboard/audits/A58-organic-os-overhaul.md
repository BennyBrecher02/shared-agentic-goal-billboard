---
audit_id: A58
title: Organic OS — Brain (Chain of Thought) + Heart (Heartbeat) + Immune (tests + bottleneck enforcement)
status: in_progress
catalogued: 2026-05-27T15:50:00Z
phase_1_landed_at: 2026-05-27T15:50:00Z
priority_when_run: P0
estimated_effort: large (foundation + 3 skill refs + chant memory rule landing this turn; full implementation across multiple turns)
trigger: 2026-05-27 user explicit — *"we were just plowing, and reached a monkey bottleneck which caused us to sidetrack, which is fine, but we need to be able to track this process, like some actual historic chain of thought, so that when we're plowing and reach bottlenecks and get sidetracked correctly that we then dont just keep diving in the wrong direction... also reminder to flesh out the heartbeat system too since i forgot about fleshing that myself hands on so much... a heartbeat in a system like ourrs can mean many things... also maybe lets add an imune system which will handles our tests and also enforce my bottleneck restriction chant i repeated so many times."*
deferral_reason: NONE — foundation + 3 skill refs + chant memory landing this turn. Full implementation (hooks + scripts) deferred to subsequent turns.
related_goals: [G2, G9, G14]
related_plans: [context/markdowns/plans/organic-os-overhaul-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - context/markdowns/organic-os/README.md (manifesto)
  - context/markdowns/organic-os/brain/README.md (Brain index)
  - context/markdowns/organic-os/heart/README.md (Heart index)
  - context/markdowns/organic-os/immune/README.md (Immune index)
  - .claude/skills/agentic-quality-discipline/references/chain-of-thought-discipline.md (NEW)
  - .claude/skills/agentic-quality-discipline/references/heartbeat-system-discipline.md (NEW)
  - .claude/skills/agentic-quality-discipline/references/immune-system-discipline.md (NEW)
  - .claude/memory-mirror/feedback_bottleneck-restriction-chant.md (NEW; the chant generalized)
  - .claude/memory-mirror/feedback_testing-cost-discipline.md (parent; testing-specific chant origin)
  - A57 audit (sibling; content + UX discipline)
  - A47 audit (BG lifecycle; immune-cell precedent)
  - A27 audit (testing toolbelt + anti-bottleneck; chant origin)
  - A18-A29 family (meta-monitoring; Brain absorbs A18/A19/A40)
findings: []
---

# A58 — Organic OS overhaul

## What triggered this audit

User asks (verbatim quotes from 2026-05-27):

1. **Brain (Chain of Thought):** *"we need to be able to track this process, like some actual historic chain of thought, so that when we're plowing and reach bottlenecks and get sidetracked correctly that we then dont just keep diving in the wrong direction like we did in between e2e's after work lastnight/thismorning when we were confused and course corrected, so we should enforce some way to make it that that can never happen and the chain of thought system sounds like it can be a huge slam dunk to flesh out."*

2. **Heart (Heartbeat):** *"reminder to flesh out the heartbeat system too since i forgot about fleshing that myself hands on so much and let you handle it in bg mostly, a heartbeat in a system like ourrs can mean many things and idk if youre actually fully utilizing our heartbeat to its fullest potential so expound on that too."*

3. **Immune (Tests + chant enforcement):** *"also maybe lets add an imune system which will handles our tests and also enforce my bottleneck restriction chant i repeated so many times."*

4. **Conceptual framing:** *"fleshing both of these out will help this system not only be more cohesive but it will feel more human/organic seeing as we have a chain of thought brain process and then also a cardiovascular heartbeat keeping us going."*

5. **Folder structure question:** user open to alternatives — *"if you think making a whole new folder system for the organic operating system is a bad idea you can do a better improvement after a brainstorm session"*.

## The constitutional chant (memorized + cited)

> ***"i dont ever want testing becoming a time/tokencost bottleneck."***
> — user, 2026-05-26T23:18Z, repeated 25 times in one message

Origin: `context/markdowns/notes/steering-decisions-log.md` entry for A27 audit + G9 goal. Currently encoded as `feedback_testing-cost-discipline.md`. This audit GENERALIZES it: any system substrate, not just testing.

## Folder structure brainstorm — verdict

The user offered the new `organic-os/` folder as an option AND offered to consider better alternatives. My verdict (in the manifesto README):

**HYBRID:** `context/markdowns/organic-os/` exists as an INDEX layer (manifesto + 3 sub-folder READMEs). Actual skill refs, memory rules, scripts, and hooks live in their canonical homes (`.claude/skills/`, `.claude/memory-mirror/`, `scripts/hooks/`, etc.). Each sub-folder README cross-links to the real artifacts.

Why hybrid not full split:
- The existing folder taxonomy (audits / plans / skills / memory / hooks / scripts) is where future-me looks
- Splitting hooks into `organic-os/heart/hooks/` would force duplicate paths or symlinks
- The metaphor (brain/heart/immune) is GREAT for framing + discoverability
- Combining gives both: conceptual coherence (manifesto) + canonical storage

## The three subsystems

### Brain (Chain of Thought)

**Problem it solves:** the May-25 → May-26 confusion where the agent dove into wrong-direction work after hitting an unspecified bottleneck. User: *"in between e2e's after work lastnight/thismorning when we were confused and course corrected."*

**Mechanism:** 7-state ledger (PLOWING → BOTTLENECK → SIDETRACK-INTENT → SIDETRACK → RESUME-READY → PLOWING-RESUMED → DONE). Every state transition lands a JSONL entry at `.claude/cache/cot-ledger.jsonl`. A PreToolUse hook (proposed: `scripts/hooks/cot-checkpoint-pre.sh`) checks: if the most recent CoT entry is BOTTLENECK and the next tool invocation isn't a SIDETRACK-INTENT followup, surface a warning. Cross-session continuity via SessionStart hook reading the last entry.

**Key invariant:** no plow-then-sidetrack happens unrecorded. Future Claude (post-compaction or post-restart) reads the CoT ledger and knows exactly what state the prior session left in.

### Heart (Heartbeat)

**Problem it solves:** under-utilization. The substrate exists (scheduler-heartbeat-coordinator + heartbeat-drain + 6 event-bus-dedup caches + launchd template) but it's wired only to scheduler ticks. The agent doesn't reach for it for cross-cutting "is X still alive?" checks.

**Mechanism:** 3-tier heartbeat hierarchy:
- **High** (every scheduler tick, sub-minute): drain pending events, flush dedup caches, hook health
- **Medium** (every ~15 min): idle-tool audit, banana staleness, capacity throttle
- **Low** (every hour or session-boundary): cross-substrate health, session-bundle, cross-session liveness

Existing consumer is `consumer-heartbeat-tier-high.sh`. Medium + low consumers are the expansion target. 6 under-utilized roles enumerated in heart README (hook health, cross-session liveness, idle-tool ticks, banana-staleness, capacity self-throttle, cross-substrate health).

**Safety:** LaunchAgent installation requires user authorization. The launchd template stays a template; the heartbeat continues firing on scheduler ticks and hook chains.

### Immune (Tests + chant enforcement)

**Problem it solves:** the bottleneck-restriction chant has been violated in spirit several times — testing infra is the explicit named target but the same dive-past-substrate-friction pattern shows up elsewhere (e.g. wall-of-text chamber post per A57; ghost BGs per A47; etc.).

**Mechanism:** two parts.

1. **Existing immune cells** — the system already has defenses (Playwright matrix, bug billboard, BG auto-stop, settings edit-guard, snapshot drift pre-check, content lint). This audit catalogs them under the immune banner so the relationship is visible.

2. **Bottleneck-detector** — new hook (proposed: `scripts/hooks/bottleneck-detect.sh`) runs on heartbeat tier-medium tick. Detection signals: cold-cache matrix > 15min; ghost BGs > 2h; pending-queue depth > 50; sustained burn > target; same audit ID 3+ times in steering log within a week; dashboard hook latency > 5s. When triggered → writes `bottleneck_active.flag` + CoT BOTTLENECK entry + steering log + dashboard surfacing.

**24h dry-run mandatory** before any bottleneck-detect hook ships in BLOCK mode (per A47 precedent).

## Implementation plan (foreground this turn + deferred next-turn)

**Foreground this turn:**
- ✅ organic-os/ scaffold (README + brain/heart/immune sub-READMEs)
- ✅ A58 audit + plan
- ✅ 3 skill refs (chain-of-thought-discipline / heartbeat-system-discipline / immune-system-discipline)
- ✅ feedback_bottleneck-restriction-chant.md memory rule
- ✅ Steering log entry + audit catalog update

**Deferred to subsequent turns (after user confirms framing):**
- CoT ledger format finalization + initial seed entries
- cot-checkpoint-pre.sh hook (24h dry-run first)
- cot-resume.sh SessionStart hook
- bottleneck-detect.sh hook (24h dry-run first)
- Medium + Low tier heartbeat consumers
- Optional dashboard panels for each subsystem (Brain CoT timeline, Heart pulse meter, Immune defense-status)

## Cross-references

- A57 (dashboard content + UX overhaul; runs in parallel and Track B BG is in flight at audit time)
- A47 (BG lifecycle; immune-cell precedent — auto-stop is the model for bottleneck-detect)
- A27 (testing toolbelt + anti-bottleneck; chant origin)
- A36/A37 (true-time + calculation audit; Heart's foundation)
- A18-A29 family (Brain absorbs A18/A19/A40 thinking around user-asks + agent-finds)
- A45 (required-services health; overlaps Immune's substrate check)
- A56 (idle test-tool prevention; Heart ticks the audit per Heart Rule 3 expansion)

## Status

PHASE 1 LANDED 2026-05-27T15:50Z. Foundation + 3 skill refs + chant memory + manifesto + sub-folder READMEs all in foreground. Phase 2 deferred to subsequent turns (hooks + dashboard panels + CoT ledger seeding).

## Lessons

- The metaphor (organic biology) gives the user a tangible framing. Future audits can extend with nervous system (hooks), digestive (inbox processing), circulatory (event bus), etc.
- Hybrid folder structure (index layer + canonical storage) is the right pattern when adding a cross-cutting conceptual layer to an existing folder taxonomy. Don't duplicate; cross-link.
- The chant has been LIVING in `feedback_testing-cost-discipline.md` but its generalized form ("no substrate may become THE bottleneck") wasn't codified. A58 fixes that.

## Phase 2.5 — Organs extension (2026-05-27T16:30Z addendum)

User extension: *"should we maybe make our opsys resources/shards/clusters into our internal organs for our organic operating system?"*

Landed: `context/markdowns/organic-os/organs/README.md` with the three primary mappings:
- **Resources** → LIVER (multi-lobed regulator; sync layers = lobes)
- **Shards** → LUNG ALVEOLI (parallel filtration units; Playwright projects = alveoli)
- **Clusters** → CIRCULATORY SYSTEM (network connecting nodes; Heart = pump)

Plus extended catalog (9 future mappings) and organ-analog table for A59's 15 scheduler ideas. Coupling insights from the mapping clarify which scheduler upgrades need to land together vs which are standalone.

The organs extension is conceptually-coherent with A58's framing AND informs A59 execution sequencing. Per the foreseen-in-manifesto note, this was always the trajectory.
