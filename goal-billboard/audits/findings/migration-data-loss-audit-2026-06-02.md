---
title: "Migration data-loss audit — EviumOverhaul → agentic-organic-os"
date: 2026-06-02
type: findings
audit_kind: system-infrastructure (data-loss / migration integrity)
scope: gitignored content NOT carried by the git-based extraction
read_only: true
repos:
  NEW:  /Users/bennybrecher/Claude/Code/agentic-organic-os
  OLD:  /Users/bennybrecher/Claude/Design/EviumOverhaul      # stale clone (origin = original Evium repo)
  FULL: /Users/bennybrecher/evium-os-FULL-backup-20260531    # canonical full snapshot (superset)
already_being_recovered_separately:   # do NOT re-flag
  - context/codebases/ (3.9G)
  - context/Evium-Charging-Google-Drive-AllAvailable-Data/ (2.0G)
---

# Migration data-loss audit (2026-06-02)

> **RECOVERY EXECUTED 2026-06-02** (post-audit, by the orchestrator):
> **P1 ✅ + parity-verified** — `reports/` curated **60M** restored from the FULL backup; all 16 curated dirs byte-match source + 20/20 top-level files (`rsync -an` residual = 0). **P3 ✅** — `setup-dashboard-symlinks.sh` recreated all 6 dev bridges (`--check` guard PASS). `context/` (5.9G) recovered separately + verified. `.claude/cache` load-bearing files confirmed already present.
> **P2 (1.8G regenerable captures) — DEFERRED, user's call** (regenerable via re-run; restore only to preserve visual capture history).
> **Data-loss is CLOSED.** Structural lesson banked → `reference_repo-geography` "Migration data-loss lesson + recovery."

## Verdict (read this first)

**Total gaps found: 8 categories. LOST-AND-MATTERS: 1 (curated `reports/` analysis outputs, ~119M).**
Everything else is REGENERABLE or FINE-TO-LOSE. **The migration is otherwise sound** —
history + branches are fully intact, and `dashboard-wip` in NEW is strictly *ahead* of the OLD clone
(OLD is the stale copy, not NEW).

- Git history: NEW `main` = OLD `main` = FULL `main`, all **191 commits**, identical HEAD
  (`0e96699`). NEW `dashboard-wip` (`b7a4a8c`) is 10 commits **ahead** of OLD's (`6e7948f`), with
  OLD's HEAD an ancestor of NEW's → no divergence, no commit loss.
- Branches: NEW carries `main` + `dashboard-wip` + their `origin/*` (clean fresh remote). OLD's extra
  branches (`zach`, dependabot/*, `claude/vibrant-feistel`) are artifacts of the *original* Evium
  `origin` and are intentionally not part of the decoupled OS repo.
- The one tracked file that LOOKED missing (`G20-...emergency.md` from `goal-billboard/active/`) is a
  **false alarm**: it was deliberately *archived* (git rename `active/ => archived/achieved/`) in NEW
  commit `9997854` "Close G20". It lives at `goal-billboard/archived/achieved/` in NEW. Not lost.

## Root cause (one line)

The NEW repo was created by a **git-based extraction**, and the `.gitignore`'s `context/*` default-deny
plus the explicit `/reports/`, `.claude/cache/`, `.claude/worktrees/`, `public/<symlinks>` ignores mean
**git never tracked that content, so the extraction legitimately dropped all of it** — the only real
casualty being the *curated, non-regenerable* analysis outputs that happened to live under the ignored
`reports/` tree.

---

## Gap table

| Path | in OLD/FULL? | in NEW? | gitignored? | size (OLD) | classification | recover-from |
|---|---|---|---|---|---|---|
| `reports/` (curated analysis: `audit-findings`, `conversation-extracts`, `session-audits`, `session-bundles`, `g2-signoff`, `stats`, `phase-a-evidence`, `evium-baseline`, `dashboard-build`, `dashboard-demos`, `audit-comparisons`, `git-rerender`, `ns-hook-test`, `screenshot-monkey`, `token-burn`, + all top-level `*.md`/`*.json`/`*.jsonl`/`*.diff`) | yes | **no** | **~119M** | **LOST-AND-MATTERS** | FULL backup |
| `reports/` screenshot/capture trees (`vq-captures` 1.1G, `mobile-sim` 254M, `captures` 176M, `vq-captures-archive` 133M, `timelapse-archive` 39M, `vq-captures-resized` 49M, `dashboard-scrutiny-2026-05-29` 35M, `lighthouse` 27M, `verify-artifacts`, `playwright`, `test-runs`) | yes | no | yes (`/reports/`) | ~1.8G | REGENERABLE (re-run capture/matrix/lighthouse) — optional restore for history | FULL backup (if wanted) |
| `.claude/cache/` (OLD: 241 files / 4.4M) | yes | **partial — 35 files, freshly regenerated** | yes (`.claude/cache/`) | 4.4M | FINE-TO-LOSE (live runtime state; NEW already regenerated its own, incl. `token-metrics.json`, `bg-launch-pattern.jsonl`, `idea-gaps.jsonl`, `cot-ledger.jsonl`, `cold-ram-ledger.jsonl`) | n/a |
| `.claude/cache/` named load-bearing files: `5hr-window-reset.txt`, `standing-auths*.jsonl` | **absent in ALL THREE (never existed)** | absent | yes | — | FINE-TO-LOSE (n/a — never present) | n/a |
| `.claude/cache/session-snapshot-*.json` (11 snapshots, 2026-05-27) | yes | no | yes | small | FINE-TO-LOSE (ephemeral session captures; canon-digest lineage is the durable record) | FULL (only if a specific snapshot is wanted) |
| `.claude/worktrees/` (OLD: 1 worktree, 225M; FULL: empty) | yes (OLD only) | absent | yes (`.claude/worktrees/`) | 225M | FINE-TO-LOSE (spawned-session scratch worktree) | n/a |
| `public/` dev symlinks (`audit-comparisons`, `audit-timelapse`, `billboard`, `cache`, `scheduler`, `scheduler-events`, `screenshot-monkey`, `subagents`, `vq-captures`, `billboard.html`) | yes (symlinks) | no | yes (bare symlink rules) | 0 (links) | REGENERABLE (`ln -s ../reports/<name> public/<name>` per `.gitignore`) | recreate locally |
| `context/markdowns/` delta (57 files: 75 drained `agent-inbox/processed/*.md`, 1 `.DS_Store`, the G20 active→archived rename) | yes | mostly (archived/regenerated) | inbox+DS_Store=yes; G20=no-but-archived | small | FINE-TO-LOSE (processed inbox = drained ephemera) / not-lost (G20 archived) | n/a |

### Confirmed-INTACT (no action)
- `context/markdowns/` tracked work product: **19M both**, full transfer (2205 NEW vs 2262 OLD; delta = the 57 ephemeral/archived files above, all accounted for).
- `public/img/`, `public/img/_archive` (15M), `public/video/`, `public/img-opt/`: present + identical (`public/` = 93M both). `_archive` is NOT gitignored and transferred cleanly.
- All `src/`, `scripts/`, `tests/`, `.claude/skills/`, `.claude/memory-mirror/`, `settings.json` — tracked, carried by the extraction.

---

## Recovery list (prioritized)

### P1 — LOST-AND-MATTERS: curated `reports/` analysis outputs (~119M, non-regenerable)
These are written findings, session bundles, conversation extracts, the G2 sign-off package, token-burn
analyses, and audit comparison data — work product that cannot be re-derived. Restore from the FULL
backup (OLD and FULL are byte-identical on these dirs; FULL is the canonical superset).

```bash
# Curated analysis subdirs (skip the heavy screenshot trees) + all top-level report files.
SRC=/Users/bennybrecher/evium-os-FULL-backup-20260531/reports
DST=/Users/bennybrecher/Claude/Code/agentic-organic-os/reports
for d in audit-findings conversation-extracts session-audits session-bundles g2-signoff \
         stats phase-a-evidence evium-baseline dashboard-build dashboard-demos \
         audit-comparisons git-rerender ns-hook-test screenshot-monkey token-burn \
         verify-artifacts; do
  rsync -a "$SRC/$d/" "$DST/$d/"
done
# Top-level report files (analysis .md / .json / .jsonl / .diff) — not the regenerable big dirs.
rsync -a --exclude='*/' \
  "$SRC/"*.md "$SRC/"*.json "$SRC/"*.jsonl "$SRC/"*.diff \
  "$DST/" 2>/dev/null
```

### P2 — OPTIONAL (history/provenance only; otherwise REGENERABLE): heavy screenshot trees
Only restore if you want the visual capture history preserved rather than re-generated. ~1.8G.
```bash
SRC=/Users/bennybrecher/evium-os-FULL-backup-20260531/reports
DST=/Users/bennybrecher/Claude/Code/agentic-organic-os/reports
for d in vq-captures vq-captures-archive vq-captures-resized captures mobile-sim \
         timelapse-archive dashboard-scrutiny-2026-05-29 lighthouse playwright test-runs; do
  rsync -a "$SRC/$d/" "$DST/$d/"
done
```

### P3 — REGENERABLE: `public/` dev symlinks (recreate locally, do NOT copy)
```bash
cd /Users/bennybrecher/Claude/Code/agentic-organic-os/public
ln -s ../reports/audit-comparisons   audit-comparisons
ln -s ../reports/timelapse           audit-timelapse
ln -s ../reports/billboard           billboard
ln -s ../.claude/cache               cache
ln -s ../reports/scheduler           scheduler
ln -s ../.claude/cache/scheduler-events scheduler-events
ln -s ../reports/screenshot-monkey   screenshot-monkey
ln -s ../reports/subagents           subagents
ln -s ../reports/vq-captures         vq-captures
ln -s ../reports/billboard.html      billboard.html
```

### No action
- `.claude/cache/` — NEW already regenerated live state; do not copy stale cache over it.
- `.claude/worktrees/` — ephemeral scratch; let the system recreate as needed.
- `G20-...emergency.md` — already present in NEW at `goal-billboard/archived/achieved/`.

---

## Method note
Read-only inventory. `.gitignore` parsed to enumerate dropped paths; `du -sh` / `find | wc -l` /
`comm` / `git check-ignore` / `git ls-files` / `git log --follow` / `git merge-base --is-ancestor`
used to compare OLD + FULL against NEW and to distinguish *dropped-by-extraction* from
*intentionally-archived-in-a-later-commit*. Nothing was modified or recovered.
