---
title: "G20 continuation — fresh-session handoff into the new home (FINAL)"
goal_id: G20
status: achieved
created: 2026-05-31
type: handoff
serves_northern_star: G2
related_artifacts:
  - context/markdowns/research/systems/zach-handoff-mechanics-audit.md
  - context/markdowns/goal-billboard/active/G20-repo-decouple-ownership-emergency.md
---

# G20 — Handoff into the new home (FINAL — paste as first message)

> Method is LOCKED (force-push in place, airtight, test-first). Full rationale +
> the airtight sequence: `context/markdowns/research/systems/zach-handoff-mechanics-audit.md`.

---

**VOICE RULE — read first.** "I" / "me" = the human user (Ben), in my own terminal.
"You" = the agent reading this. Safety rule, not a formality.

You are the **agentic-organic-OS**, relocated to `~/Claude/Code/agentic-organic-os`. Clean session
continuing the **G20 repo-decouple**. The plan is fully designed; execute IN ORDER. Nothing
irreversible happens until the tests below pass.

## Task 1 — learn + self-test your guardrails (FIRST, before anything)
1. Read `.claude/settings.json` → `permissions.deny`.
2. Read the guard hooks: `scripts/hooks/{settings-edit-guard,src-edit-guard,no-hardcoded-paths-guard}.sh`.
3. Run `bash scripts/run-tests.sh` (+ guard tests in `scripts/tests/`); confirm a gated command is refused.
4. Report, in your own words, what you may and may not do.

## Task 2 — confirm the migration landed
- Pulse shows **`MEMORY ✓`** (non-zero) at the new slug; `MEMORY.md` auto-loaded. If blank → STOP, tell me.

## Task 3 — prove the transition (the e2e gate)
- Review + run `bash scripts/e2e-migration-verify.sh`. It must report **GO, 6/6**.
- I earlier asked for a "more thorough 2.0" — audit its coverage; if there are gaps, improve it
  (keep it fast-tier), then re-run. This is the proof of 100% transition success.

## Task 4 — build + run the scrub rehearsal (handoff test B)
- Build a script (sibling of `e2e-migration-verify.sh`) that: references zach via
  `git -C ~/Claude/Design/EviumOverhaul` (holds `origin/zach` = `b1a739a`), clones to a TEMP dir,
  builds an **orphan clean root** from zach's tree, then VERIFIES: exactly **1 commit** · **0**
  `/Users/` + **0** email leaks · `npm ci && npm run build` **passes** · then discards the temp dir.
  **NO push.** It must pass before any real force-push. Re-runnable, zero risk to EviumCharging.

## Who runs what (core safety rule)
- **I — the human — run every git mutation + GitHub change myself, in my terminal.**
- **You — the agent — NEVER run git mutations / GitHub changes.** You read state and hand me exact commands.

## The real handoff — force-push in place, airtight (only after Tasks 1–4 pass; I run each step)
1. Eyeball the 4 `secret/key/password` matches; update/remove `.github/CODEOWNERS`.
2. Close the 7 dependabot PRs; delete their branches + any tags. (Refs/PRs pinning old commits block GC.)
3. Disable branch protection on `main`.
4. Build the orphan clean root; `git push --force origin <clean-root>:main`; delete the `zach` branch.
5. Contact **GitHub Support to purge unreachable objects + cached views** (official sensitive-data step).
6. Verify: fresh-clone to a temp dir → 1 commit, clean tree, builds; confirm `621ba18` no longer resolves.
7. Transfer ownership to my boss; ensure I'm re-added as a collaborator.

## The collaborator loop (design + hold)
After handoff I code on EviumCharging alongside my boss. Shared repo = **clean site only** — this OS
tooling never gets pushed there. I work from a **fresh clone**, never the old `~/Claude/Design/EviumOverhaul`
copy. This `agentic-organic-os` repo stays my private toolkit.

## How to work with me
One verified step at a time · don't touch EviumCharging until I explicitly say go · no background agents ·
don't mark anything "done" (that's mine) · no `Co-Authored-By` on commits.

**Start with Task 1.**
