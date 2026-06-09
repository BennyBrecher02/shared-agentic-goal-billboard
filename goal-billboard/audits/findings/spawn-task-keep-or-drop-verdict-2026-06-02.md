---
title: "spawn_task local-BG chips — keep or drop? VERIFIED verdict"
date: 2026-06-02
type: findings
audit_kind: mechanism-comparison (spawn_task vs Agent-bg vs Workflow), test-verified
agent: aceb7e72 (read-only tests run; spawn-required tests described)
trigger: "user 2026-06-02 — after the local-BG failure: when/how-often to use chip local-BG tasks, and should we DROP them if the others do everything better? verify with testing."
---

# spawn_task — VERDICT: KEEP, with strict rules. Do NOT drop.

## Why keep (the unique value nothing else has)
Only spawn_task can: **(1) OUTLIVE the spawning session** (a separate OS-level Claude session that keeps
running after the main session ends, landing its result in a worktree for later human review) and **(2) the
user's one-click deferred spinoff** of an out-of-scope tangent. Agent-bg subagents and Workflows are
EPHEMERAL (die with the spawner) and AGENT-initiated. For everything the spawner will consume *this* session,
spawn_task is **strictly worse** (untracked, race-prone, freeze-prone) — an Agent subagent returns the result
directly. So spawn_task is a narrow exception path, not a routine tool.

## ROOT CAUSE of today's failure (verified) — it's the SAME relocation wound
spawn_task's worktree isolation is REAL but FRAGILE. Verified: in the Evium repo a spawned worktree
(`agitated-nightingale-27f5c7`) has its own gitdir + branch + index → it CAN'T race main. But in THIS repo
`.claude/worktrees/` **doesn't exist** — the 2026-05-31 git-based relocation **silently dropped it** (it's
gitignored, same class as the dropped daemon caches / HEART beacon / node_modules). With the worktree
substrate gone, the spawn **fell back to the shared main checkout** → the git-index race. **The same migration
wound that flatlined HEART + mispointed the daemons also disarmed spawn_task's isolation.** One wound, many symptoms.

## THE DECISION RULE (verified)
```
Found follow-up work — which mechanism?
0. Known + homogeneous + many children → WORKFLOW (deterministic, tracked, synthesized).
1. THIS session will consume the result → AGENT subagent (fg or run_in_background). ← THE DEFAULT.
2. Trivial / one-liner → JUST DO IT INLINE.
3. Context-dependent (needs what you learned) → REPORT in your final message. Never a chip.
4. spawn_task ONLY if ALL hold:
   [ ] you are the MAIN INTERACTIVE session (a subagent/BG firing a chip = near-bug)
   [ ] separable + substantial · NOT redundant with inline/reported work · context-free (actionable cold)
   [ ] runs UNATTENDED to completion (no question/permission popup → else freeze-in-dark)
   [ ] you genuinely WANT it to outlive this session + land in a worktree for later review
   [ ] `.claude/worktrees/` is verified a HEALTHY worktree root (else index-unsafe — see below)
```
**How often:** rarely — single-digit per long session, **main-session-only**.

## GUARDRAILS (close the gaps this exposed)
1. **`.claude/worktrees/` is NOT healthy in this repo right now** → spawn_task is **index-unsafe here until
   restored**. Until then: prefer Agent-bg; if a chip is essential, verify worktree health first. Add
   `.claude/worktrees/` health to the relocation runbook (`reference_repo-geography`) + a SessionStart warn.
2. **Never two sessions on one checkout** — already banked (F1 in `concurrency-synchronization-audit.md`):
   isolate-or-don't-commit-while-staged.
3. **Make the net cover the failure mode:** commit the `brain-heartbeat-hook` wire (working-tree-only now;
   user-gated) AND relax its `*/.claude/worktrees/*` cwd-filter so a main-checkout spawn also pulses; widen
   detect-frozen/harvest beyond `.claude/worktrees/` OR **mandate worktree isolation** (simpler invariant —
   also makes the index-race impossible).
4. **No chip for inline work + a dedup gate** on the suggested-task surface (F2 — the redundancy that turned a
   latent hazard into a visible failure).

## Test results (verified, read-only)
- T1 isolation: ✅ real (Evium worktree has own index) but ❌ fragile (gone here → fell back to main index).
- T2 freeze-in-dark: 📋 described (would spawn) + corroborated by the recorded `trusting-wright-703d49` incident.
- T3 detect-frozen/harvest: ✅ logic sound (16 assertions) but **blind here** — they scan only `.claude/worktrees/`.
- T4 heartbeat layer: ✅ 16/16 test, but the wire is uncommitted + cwd-filtered → **double-blind right now**.

Full matrix + paths: agent aceb7e72.
