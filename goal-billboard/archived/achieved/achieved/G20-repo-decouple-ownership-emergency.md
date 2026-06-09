---
goal_id: G20
title: "🚨 RED ALERT — Repo decouple + ownership transition (boss deleted the agent OS)"
priority: P0-RED-ALERT
status: achieved
created: 2026-05-31
type: emergency
serves_northern_star: G2
belongs_to_goal: G20
related_artifacts:
  - context/markdowns/research/systems/boss-zach-branch-analysis.md
  - context/markdowns/research/systems/repo-decouple-ownership-transition-plan.md
  - context/markdowns/research/systems/branch-cleanup-safety-reaudit.md
  - context/markdowns/research/systems/zach-self-sufficiency-audit.md
  - context/markdowns/goal-billboard/audits/findings/status-reconciliation-2026-05-31.md
project: OS-core  # stamped by migrate-billboard-to-shared (shared store)
---
# 🚨 G20 — RED ALERT: Repo decouple + ownership transition

## The emergency (what happened)
The user's boss (`zlowy1 <zach@justsomepeople.com>`) judged the agentic-OS tooling as "mess" and pushed a branch **`zach`** to origin (github.com/BennyBrecher02/EviumCharging):
- **`4c6d816`** — DELETES the entire agent OS (`.claude/`, `scripts/` minus the image pipeline, `tests/`, `context/markdowns/`, `.github/workflows/` — 2,509 deletions). **SKIP — never merge.**
- **`9f9d53c`** — real site changes (brand green `#2dc856→#389834`, astro-icon+lucide, **reverts the mobile-fullPage iOS fix**, logo hero, stripped eyebrows). Cherry-pick; decide green; decline the fullPage revert.
- **`b1a739a`** — header (removes Home link + close-X, green Contact accent). Cherry-pick.

zach is **FROZEN** (boss stopped, confirmed via 2 read-only re-analyses 2026-05-31), CI green 2/2. He wants to become repo owner — **not until after decouple**.

## End-state architecture (LOCKED with user)
- **New PRIVATE repo (user's)** = ALL the user's work + **full git history**, **MINUS `zach`**. Full history is essential — the git/rollback page's keep/undo decisions only work against real commits.
- **Original repo (boss's)** = `main` becomes zach's clean site; boss owns it.
- **NEVER** transfer the original repo or scrub-and-force-push it (carries all tooling + history + 85 `/Users/bennybrecher` path leaks). Boss gets a **fresh** clean repo.

## 4-phase plan + safety gates
- **Phase 0 — Preserve** ✅ ~done: `dashboard-wip` commit `887a55a`; 3 backups (see don't-forget). TODO: off-machine copy.
- **Phase 1 — Build new repo** from the bundle (zach excluded), restore gitignored artifacts, **VERIFY rollback page + keep/undo work** (HARD GATE).
- **Phase 2 — Reconcile site** (Cursor): rollback decisions + cherry-pick `9f9d53c`+`b1a739a`, SKIP `4c6d816`, decide green, decline fullPage revert, keep astro-icon + logo.png.
- **Phase 3 — Clean original repo** → zach-only.
- **Phase 4 — Hand off** the fresh clean repo to boss.
- SAFETY: original repo untouched until the new repo passes Phase 1 verify; every phase gated on user's explicit go; user runs all git mutations.

## ✅ TODO (red-alert checklist)
- [ ] **Get a backup OFF this laptop** (external drive / cloud) — all 3 backups are on one machine.
- [ ] **DO NOT run the minus-zach re-bundle** (`git update-ref -d refs/remotes/origin/zach` + re-bundle). The current bundle (incl. `origin/zach`) is a fine COMPLETE backup; "minus zach" is enforced where it matters — in **Phase 1** when building the new repo. One less step, zero downside. *(per user 2026-05-31)*
- [ ] **zach self-sufficiency audit** (does zach build standalone as the new main, no broken refs to deleted tooling) — LAUNCHED 2026-05-31 → `zach-self-sufficiency-audit.md`.
- [ ] Phase 1: new PRIVATE repo from the bundle, zach excluded, artifacts restored, rollback VERIFIED.
- [ ] Phase 2: reconcile.
- [ ] Phase 3/4: clean original → handoff.

## 🧠 Don't-forget facts (context that may be lost between sessions)
- **Backups (2026-05-31, all on the user's MacBook — NOT yet off-machine):**
  - `~/evium-os-backup-20260531/evium-full-repo.bundle` — 211 MB, full history; **but INCLUDES `refs/remotes/origin/zach`** → exclude at Phase 1.
  - `~/evium-os-backup-20260531/{cache, dashboard-build, memory-autoload}` — the gitignored artifacts.
  - `~/evium-os-FULL-backup-20260531/` — 8.2 GB full rsync copy; a working git repo, HEAD `887a55a`, 191 commits, both branches.
- **The bundle includes `origin/zach`** — it got fetched into local refs at some point (after an earlier check found it absent). Exclude it when building the new repo, NOT by re-bundling.
- **Gitignored artifacts** (`reports/dashboard-build/`, `.claude/cache/`, the auto-load memory dir at `~/.claude/projects/-Users-bennybrecher-Claude-Design-EviumOverhaul/memory/`) are NOT in git → must be restored manually into the new repo.
- **85 files leak `/Users/bennybrecher`** + emails in history → new repo MUST be private; boss gets a fresh clean repo, never a transfer/scrub.
- **Misstep + lesson:** lost a regenerable doc (`status-reconciliation-2026-05-31.md`) to a `--force` worktree remove (the recover script missed loose docs). Recreated. RULE: before `--force`-removing a HOLD worktree, verify recovery captured it or `cp -R` the dir first.
- **Local branches now:** `main` (123 ahead of origin/main, 191 total) + `dashboard-wip` (HEAD `887a55a`, tonight's build). All `claude/*` + `verified/*` cleaned up.

## Linked artifacts
- `research/systems/boss-zach-branch-analysis.md` — zach branch analysis (re-analyzed 2026-05-31, frozen, 3 commits).
- `research/systems/repo-decouple-ownership-transition-plan.md` — full decouple plan + ours-vs-site split (~660 site / ~2,860 tooling).
- `research/systems/branch-cleanup-safety-reaudit.md` — 27-branch cleanup audit + the eager-sutherland landmine.
- `research/systems/zach-self-sufficiency-audit.md` — (launching) does zach build standalone as main.
- `audits/findings/status-reconciliation-2026-05-31.md` — recreated audit-catalog reconciliation.
