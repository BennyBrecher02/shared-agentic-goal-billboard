---
audit_id: A59
title: Scheduler upgrade brainstorm — 15 ideas in 3 tiers (research-only; code changes deferred)
status: in_progress
catalogued: 2026-05-27T16:15:00Z
priority_when_run: P1
estimated_effort: small (brainstorm + audit only); large if code-changes greenlit
trigger: 2026-05-27 user explicit — *"brainstorm how we can upgrade our scheduler and better implement it in our system. lmk wether or not it would be a bad move to start making changes to the scheduler tho."*
deferral_reason: Code changes RISKY this turn (A57 Track B BG live; Phase B pending; A58 hooks not yet deployed; 4 recent commits; bash policy denies some git ops). This audit captures the ideas for safe execution AFTER dependencies settle.
related_goals: [G1, G2]
related_plans: [context/markdowns/plans/multi-change-scheduler-implementation-plan.md, context/markdowns/plans/multi-change-scheduler-phase-A3-A5-plan.md, context/markdowns/notes/scheduler-upgrade-brainstorm-2026-05-27.md]
serves_northern_star: G2
belongs_to_goal: G1
serves_guiding_light: G14
related_refs:
  - scripts/scheduler/ (existing implementation)
  - .claude/skills/agentic-quality-discipline/references/chain-of-thought-discipline.md (CoT integration target)
  - .claude/skills/agentic-quality-discipline/references/heartbeat-system-discipline.md (Heart integration target)
  - .claude/skills/agentic-quality-discipline/references/immune-system-discipline.md (Immune integration target)
  - A21 audit (parallel-capacity ceiling)
  - A27 audit (testing toolbelt + anti-bottleneck — Tier 2 idea #8/#9 are G9 cost moves)
  - A36 audit (timing pair gap — Tier 1 idea #2)
  - A37 audit (priority routing bug — Tier 1 idea #1)
  - A33 audit (cluster — Tier 3 idea #12)
  - A47 audit (BG lifecycle — model for change-completion sentinel)
  - A58 audit (Organic OS — Brain/Heart/Immune integration targets are Tier 1 ideas #3/#4/#5)
findings:
  - priority-routing-bug-unresolved: scheduler.scheduled[0] picks oldest-by-mtime; ignores priority field. A37 finding, not yet fixed.
  - timing-pair-gap: no structured (launched_at, completed_at) anywhere. A36 dispatcher canary applies to scheduler too.
  - organic-os-integration-missing: A58's Brain/Heart/Immune are perfect dependencies for scheduler but scheduler doesn't reach for them yet.
  - cost-discipline-gap: G9 affected-tests-only mode + result caching not yet wired into scheduler tick flow.
  - file-zone-implicit: parallel safety relies on agent judgment; explicit `affects_files` per change with conflict checker would formalize.
---

# A59 — Scheduler upgrade brainstorm (research only)

## Verdict on code changes RIGHT NOW

**Moderately risky.** 6 specific reasons it's safer to wait:

1. A57 Track B BG live (touching dashboard files that scheduler downstream depends on)
2. Phase B full e2e milestone pending — refactoring scheduler now would obscure Phase B's diagnostic signal
3. A58 Phase 2+ hooks (CoT checkpoint + bottleneck-detect) are the perfect dependencies; building scheduler against them is cleaner than retrofitting
4. 4 commits in last few hours (`3d56df0` / `3889ae6` / `31329fe` / `fd7264c`); mid-flight refactor on a moving stack invites diff-apply pain (memory: "scheduler uses `git worktree add HEAD`; drift can break diff-apply")
5. Bash policy denies some git ops (e.g. `git rm --cached`); we can't undo certain scheduler changes cleanly mid-session
6. Token budget at ~33% (16.5M / 50M); a real scheduler refactor BG would burn 2-5M tokens — spend after alignment, not before

**Safe execution order** (next session ideally):
1. A57 Track B BG completes
2. Run Phase B full e2e (lock current state)
3. Ship A58 Phase 2 (CoT + bottleneck-detect hooks in 24h dry-run)
4. Restart session (clean state)
5. THEN scheduler refactor with dependencies in place

## The 15 ideas

(Full details in `notes/scheduler-upgrade-brainstorm-2026-05-27.md`.)

### TIER 1 — Already-identified bugs (high priority, well-scoped)

1. **Priority routing fix** — read priority field; FIFO+aging WITHIN tier (A37 #4)
2. **Structured timing pairs** — every tick + change writes (launched_at, completed_at) to dispatch-log (A36)
3. **CoT integration** — scheduler writes Brain CoT entries on PLOWING / BOTTLENECK / DONE
4. **Heart integration** — formalize `scheduler-heartbeat-coordinator.sh` as Heart's tier-high consumer
5. **Immune integration** — scheduler exposes 3 bottleneck signals (tick latency, queue depth, ghost-change rate) to `bottleneck-detect.sh`

### TIER 2 — Workflow & cost (medium effort, big payoff)

6. **Dynamic A21 ceiling** — adaptive parallelism from burn rate + 5hr window + queue depth
7. **Per-change CoT sub-chain** — each scheduled change has its own CoT chain for failure-tracing
8. **Affected-tests-only mode** — compute affected tests per change; skip unaffected projects (G9 win)
9. **Test result caching** — hash test inputs; reuse cached outputs (G9 win)
10. **File-zone affinity tracker** — explicit `affects_files` per change; conflict-checker before dispatch

### TIER 3 — Advanced (research-required first)

11. **Speculative dispatch** — high-confidence changes dispatch matrix run in parallel with change
12. **Cluster offload** — M2/RPi cluster takes matrix runs (G12/G13 substrate)
13. **Multi-tier scheduling** — short/long/queue tiers explicit
14. **Adaptive backoff** — Immune-triggered tick throttle
15. **Cross-session resume** — scheduler state survives session boundaries

## Implementation gating

Each idea is gated on dependencies:

| Idea | Depends on | When |
|------|-----------|------|
| #1 priority routing | Nothing | Immediate after dependencies (A57 Track B + Phase B) |
| #2 timing pairs | Nothing | Immediate after dependencies |
| #3 CoT integration | A58 Phase 2 (CoT ledger + hooks) | After A58 Phase 2 lands |
| #4 Heart integration | A58 Phase 3 (Heart tier-medium/low consumers) | After A58 Phase 3 |
| #5 Immune integration | A58 Phase 4 (bottleneck-detect.sh) | After A58 Phase 4 |
| #6-10 (Tier 2) | Mostly nothing; some need Tier 1 first | After Tier 1 lands |
| #11-15 (Tier 3) | RH-014 research outputs; cluster substrate (G12/G13) | After RH-014 returns + cluster operational |

## Phase 1 — captured (this turn)

- ✅ This audit
- ✅ `notes/scheduler-upgrade-brainstorm-2026-05-27.md` (full ideas + design sketches)
- ✅ Audit catalog updated
- ✅ Steering log entry

## Phase 2 — execution gate (next session, post-Phase-B)

When ready to execute, this audit transitions from `in_progress` to `phase_2_kickoff`. Phase 2 is a separate work block that:
1. Re-verifies dependencies are in place (A57 Track B done; Phase B done; A58 Phase 2 deployed)
2. Selects which Tier 1 ideas to land first (likely #1 + #2 + the relevant subset of #3/#4/#5 that don't require deferred A58 phases)
3. Writes implementation plan against the existing `multi-change-scheduler-implementation-plan.md`
4. Dispatches as paired BGs with file-zone discipline

## Cross-references

- `notes/scheduler-upgrade-brainstorm-2026-05-27.md` (the full brainstorm doc)
- A58 audit + plan (Organic OS; Tier 1 #3-#5 integrate here)
- A37 audit (priority routing bug)
- A36 audit (timing pair gap)
- A21 audit (parallel-capacity ceiling)
- A27 audit + G9 (testing cost discipline; Tier 2 #8/#9)
- A33 audit + G12/G13 (cluster; Tier 3 #12)
- A47 audit (BG lifecycle precedent)
- RH-014 (async + token-saves research; informs Tier 3 #11/#13/#14)
- `plans/multi-change-scheduler-implementation-plan.md` (existing implementation plan; A59 extends)

## Status

PHASE 1 LANDED 2026-05-27T16:15Z (brainstorm captured). Phase 2 (execution) deferred to next session post-dependencies.

## Lessons (so far)

- Honest verdict before action: when user asks "is it a bad move to change X right now," the answer must be specific about RISKS not just hedging. 6 named risks here.
- Brainstorm-as-audit pattern: when the ideas are valuable but execution is gated, capture as audit + brainstorm doc; revisit when dependencies settle. Avoids both losing the ideas AND premature changes.
