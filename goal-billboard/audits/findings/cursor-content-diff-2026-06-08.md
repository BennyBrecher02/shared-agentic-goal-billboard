# Cursor CONTENT-level diff audit — deployed `~/.cursor` vs repo canon

- **Date:** 2026-06-08
- **Auditor:** read-only content-diff sweep (subagent)
- **Scope:** byte/line `diff` of EVERY deployed file in `~/.cursor` against its repo-canonical
  counterpart. Complements the parallel `~/.cursor` names/risk audit — that one inspects *what
  exists*; this one inspects *internals* (stale pre-edit bodies, un-propagated hand-edits, drift).
- **Repo:** `/Users/bennybrecher/Claude/Code/agentic-organic-os`
- **User's concern:** since the Jun-3 transport, the OS has manually edited skills/memories in the
  source, so deployed copies in `~/.cursor` may have DIVERGED in content (stale, forked, drifted).
- **Constraints honored:** read-only — did NOT modify `~/.cursor` or canon; no git commit; no spawn_task.

---

## VERDICT — NO CONTENT DRIFT

Every deployed file with a canonical counterpart is **byte-for-byte IDENTICAL to canon.** Zero stale
bodies, zero un-propagated hand-edits (no fork ever back-propagated to canon), zero divergence.
The user's specific worry — that this session's canonical edits (incl. the **I-007 Principle/Example
memory refactor**) never reached the deployed copies — is **disproven**: the refactored bodies are
present and identical on the deployed side.

| Surface | Deployed counterpart files | IDENTICAL | DIVERGED | DEPLOYED-ONLY | CANONICAL-ONLY |
|---|---|---|---|---|---|
| `skills/agentic-*` bodies (SKILL.md + refs) | 167 | **167** | 0 | 0 (only per-skill `.deploy-receipt.json`, by design) | 0 |
| `universal-memory/*.md` | 38 | **38** | 0 | 0 | 0 |
| `rules/*.mdc` | 4 | **4** | 0 | 0 | 0 |
| `hooks/*.sh` | 3 | **3** | 0 | 0 | n/a (canon has 88 hooks; only 3 are *meant* to deploy to Cursor) |
| `cli-config.json` vs its `.bak` | 1 | — | **1 (intentional OS edit)** | n/a | n/a |
| **Totals (canon-counterpart files)** | **212** | **212** | **0** | — | **0** |

Authoritative cross-check: the repo's own `scripts/skill-drift-check.sh` (sha256-fold of the
deployable set vs the deploy receipt vs live canon) reports **no TAMPER, no STALE, no FORK** — its
only flag is one benign `NORECEIPT` (below).

---

## Per-surface detail

### 1. Skills — `~/.cursor/skills/agentic-*/**` vs `$REPO/.claude/skills/agentic-*/**`  → IDENTICAL ×13

`diff -rq` across all 13 deployed `agentic-*` skills, then a **per-file byte diff of all 167 shared
files** (SKILL.md + every `references/*.md` + assets): **0 diverged, 0 canonical-only, 0
deployed-only** content files.

- The only thing `diff -rq` flagged is a `.deploy-receipt.json` present in each deployed skill and
  absent from canon. **This is deploy metadata, not content drift** — it is *expected* deployed-only
  (records `kit_commit` / `canon_hash` / `deployed_at`). Example
  (`agentic-safety/.deploy-receipt.json`): `deployed_at: 2026-06-08T02:07:44Z`, `source: repo-canon
  .claude/skills`, `env: cursor`.
- Canon skills are **git-clean** (`git status --porcelain -- .claude/skills` empty) — no uncommitted
  hand-edit sitting only in the repo.
- Newest canon skill mtime = **2026-06-07T22:18Z**, which **predates** the deploy
  (2026-06-08T02:07Z) — i.e. the deploy captured the latest canon, including this session's edits.

> Note on a false positive during the audit: an ad-hoc `find | xargs shasum | shasum` recompute of
> the receipt `canon_hash` mismatched the stored value for all skills. That is a **methodology
> artifact** — `skill-drift-check.sh` computes the fold differently (`relpath \0 per-file-sha`,
> `LC_ALL=C` sorted, double-fold). The authoritative `diff` (byte-identical) and the tool's own
> verdict (no TAMPER/STALE) are the ground truth; the receipt hashes are an opaque internal
> fingerprint and were not independently reproducible by hand.

### 2. Cursor rules — `~/.cursor/rules/*.mdc` vs `$REPO/portable-kit/cursor-rules/*.mdc`  → IDENTICAL ×4

`house-conventions.mdc`, `ports-and-config.mdc`, `substrate-respect.mdc`, `web-conventions.mdc` —
all byte-identical. No canonical-only rule left undeployed; no deployed-only rule.

### 3. Deployed hooks — `~/.cursor/hooks/*.sh` vs `$REPO/scripts/hooks/<same name>`  → IDENTICAL ×3

The three gate scripts Cursor runs are the **current** canon versions:
- `artifact-landed-check.sh` — IDENTICAL
- `no-hardcoded-paths-guard.sh` — IDENTICAL
- `screenshot-janitor.sh` — IDENTICAL

(Canon `scripts/hooks/` holds 88 scripts; only these 3 are intended for the Cursor deploy target —
the rest are Claude-Code-harness hooks, correctly NOT in `~/.cursor/hooks`. Not a gap.)

### 4. Universal memory — `~/.cursor/universal-memory/*.md` vs `$REPO/portable-kit/universal-memory/*.md`  → IDENTICAL ×38

⚠ Special attention per the prompt: the canonical memory topics were **refactored this session**
(the I-007 **Principle / Example split**). Result: **the deployed copies carry the refactored
bodies, byte-identical to canon — NOT a stale pre-refactor body.**

- `diff -rq` recursive: **all identical.** sha256 of every `*.md` on both sides: **0 mismatches.**
- The 15 files bearing the `Principle` / `Example:` split markers are the **same 15** on both sides.
- Spot-verified `feedback_decision-handling-discipline.md` (the largest, 11038 B): byte-identical,
  and the deployed copy contains the post-refactor `Example (from A66 trigger):` structure — proving
  the new split body deployed, not the old one.
- `MEMORY.md` index (6617 B) identical on both sides; deployed mtime 2026-06-08T06:48Z (a later
  re-sync), still equal to canon.

### 5. `cli-config.json` vs `cli-config.json.bak-20260603T183906Z`  → DIVERGED (intentional, benign)

The only intentional content change anywhere. The OS added an `attribution` block on top of the
Jun-3 baseline:

```diff
   "network": {
     "useHttp1ForAgent": false
   },
+  "attribution": {
+    "attributeCommitsToAgent": false,
+    "attributePRsToAgent": false
+  }
 }
```

- **What changed:** +4 lines adding `attribution.attributeCommitsToAgent=false` and
  `attributePRsToAgent=false`. Nothing removed.
- **Authoritative side:** the **current** `cli-config.json` (the `.bak` is the pre-change snapshot).
- **Assessment:** correct and desirable — this enforces the **No-AI-co-authorship** convention
  (`feedback_commit-authorship.md`: commits/PRs carry no agent attribution) at the Cursor-CLI level.
  No action needed. This is a deliberate OS edit, not drift.

---

## Items flagged but OUT of canon-counterpart scope (no fix needed)

These exist under `~/.cursor` but have **no repo-canonical counterpart**, so "content drift vs
canon" does not apply. Listed for completeness so the parallel names/risk audit can reconcile.

- **`~/.cursor/skills-cursor/`** (17 skills: `automate`, `babysit`, `canvas`, `create-hook`,
  `create-rule`, `create-skill`, `create-subagent`, `loop`, `migrate-to-skills`, `sdk`, `shell`,
  `split-to-prs`, `statusline`, `update-cli-config`, `update-cursor-settings`, …). These are
  **Cursor's own built-in / Cursor-managed skills** (its `.cursor-managed-skills-manifest.json`
  lists them under `builtinSkillIds` / `managedSkillIds`). **None has a counterpart in the repo** —
  searched the repo for each name, zero matches. NOT OS-canonical content; out of scope.
- **`~/.cursor/ai-tracking/ai-code-tracking.db`** — a Cursor-internal SQLite db. No canon
  counterpart; out of scope.

---

## Recommended (non-mutating) follow-ups

The user drives every mutating step. None of these is content drift; all are hygiene-only:

1. **(optional) Record a deploy receipt for `agentic-preview-tools`.** `skill-drift-check.sh`
   reports it `NORECEIPT — content matches canon but no receipt`. Content is correct and identical;
   it simply lacks a `.deploy-receipt.json` (it was the one skill that showed IDENTICAL on the very
   first `diff -rq`, i.e. it was copied without the receipt-writing path). A re-deploy via the kit
   would stamp a receipt. **No content risk** — purely closes the bookkeeping gap.
2. **No re-sync required for any other surface** — skills, rules, hooks, and universal-memory are
   all already in lockstep with canon. Do **not** re-copy; there is nothing to fix.
3. **No promote-to-canon action** — no deployed-only hand-edit exists, so nothing needs back-porting
   into the repo.

---

## How this was verified (method)

- `diff -rq` per skill, then a per-file byte `diff` loop over all 167 shared skill files (receipts
  excluded), plus canonical-only / deployed-only detection both directions.
- `diff` per `.mdc` rule and per deployed `.sh` hook against the same-named canon file.
- `diff -rq` + per-file `sha256` over universal-memory; targeted grep for the I-007
  `Principle`/`Example` markers on the deployed side to confirm the refactored (not pre-refactor)
  body shipped.
- Line `diff` of `cli-config.json` against its `.bak`.
- Cross-checked against the repo's authoritative `scripts/skill-drift-check.sh` (sha256-fold vs
  receipt vs live canon).
- `git status --porcelain` over canon skills + portable-kit; mtime comparison of newest canon file
  vs deploy timestamp.

---

BG-COMPLETE-SENTINEL files_diffed=212 identical=211 diverged=1(cli-config:intentional-attribution-block) deployed_only=0(content;13 .deploy-receipt.json are by-design metadata) top_divergences="cli-config.json +attribution{attributeCommitsToAgent=false,attributePRsToAgent=false} (current=authoritative, enforces no-AI-co-authorship, no fix); NO skill/memory/rule/hook content drift; agentic-preview-tools NORECEIPT(content matches canon, missing receipt only)" doc_path=context/markdowns/goal-billboard/audits/findings/cursor-content-diff-2026-06-08.md status=CLEAN-NO-DRIFT
