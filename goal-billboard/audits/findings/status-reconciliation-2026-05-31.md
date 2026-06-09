---
title: "Audit catalog — status reconciliation (2026-05-31)"
audit-id: status-reconciliation
date: 2026-05-31
scope: read-only inventory of audits/ + archived/audits/ + findings/ (the single write is this file)
verified-against:
  - context/markdowns/goal-billboard/audits/                       # 68 active A*.md + findings/ + READMEs (frontmatter status grepped)
  - context/markdowns/goal-billboard/archived/audits/              # 21 archived A*.md (frontmatter status grepped)
  - context/markdowns/goal-billboard/audits/_README.md             # status taxonomy + verified-complete A62 gate
  - context/markdowns/goal-billboard/audits/README.md              # lifecycle + (stale) catalog registry
  - context/markdowns/goal-billboard/audits/CLOSE-CANDIDATES-CHECKLIST.md   # the 13 user-only close-candidates derived from THIS reconciliation
method: grep -m1 '^status:' on every A*.md frontmatter (source of truth = the files, NOT the stale README registry table which stops at A68).
---

> **RECREATED 2026-05-31 — the original was lost in a worktree cleanup (never committed, unrecoverable); this is reconstructed from the preserved catalog state.**
> The archival reorganization it accompanied IS preserved: ≥17 old audits (A11–A89) were moved `audits/` → `archived/audits/`. This doc reconciles the catalog as of 2026-05-31 from that preserved state. It is functionally equivalent to the lost original, not byte-identical. `CLOSE-CANDIDATES-CHECKLIST.md` already back-links to this filename, so recreating it under this exact name restores that reference.

# Audit catalog — status reconciliation, 2026-05-31

## 0. Headline counts

| Bucket | Count | Location |
|---|---:|---|
| **Active** audit definitions (`A*.md`) | **68** | `audits/` |
| **Archived** audit instances (`A*.md`) | **21** | `archived/audits/` |
| Findings docs (time-scoped artifacts) | 34 | `audits/findings/` |
| **Total distinct audit IDs touched** | **A1–A92** (with gaps) | — |

Active breakdown by on-disk `status:` frontmatter (the source of truth — the `README.md` registry table is **stale**, it stops at A68 and predates A69/A72–A74/A80/A90–A92):

| status value | count | meaning |
|---|---:|---|
| `available` | 9 | catalogued, never picked up (A1, A2, A4, A5, A6, A7, A8, A10) + A9 now `in_progress` |
| `in_progress` | 56 | picked up; work landed but not moved to a terminal state / not yet archived |
| `completed` | 1 | A90 (terminal, but still sitting in `audits/` — archival-move pending) |
| `findings-complete` | 2 | A91, A92 (findings written; non-canonical status string — should normalize to `completed` on archival) |

> **Taxonomy note (from `_README.md`).** Canonical status vocabulary is `available | in_progress | completed | verified_complete | cancelled`. Two terminal "done" states exist: `completed` (done/delivered) and `verified_complete` (done AND passed the A62 four-tether adaptive-immunity gate — hardware barrier + software guard + biology rule/ref + a VERIFY test that fails if the failure can recur). The catalog also contains drift values seen in the wild that should normalize: `findings-complete` (A91/A92 → `completed`), `phase_2_patches_applied` / `phase_1_landed` / `resolved` / `proposed` (registry-table free-text → map to `in_progress` until a terminal move). `A71` is the first audit to reach `verified_complete`.

---

## 1. ARCHIVED set (the moved A11–A89 — preserved) + rationale

**21 files** now live in `archived/audits/`, all in a terminal state — **16 `completed`, 5 `verified_complete`**. This is the reorganization that was in flight when the original reconciliation doc was lost; the moves themselves survived.

### 1a. The verified_complete five (cleared the A62 four-tether gate)

| ID | Title | Status |
|---|---|---|
| A71 | Inbox never-lost — adaptive-immunity triple-tether mutation | `verified_complete` |
| A75 | Workflow 4.7-vs-4.8 comparison + conflict-check | `verified_complete` |
| A76 | Semantic code-index adoption — downside analysis | `verified_complete` |
| A77 | Idea-conflict-check capability (generalized the A76 one-off) | `verified_complete` |
| A78 | Dashboard scrutiny + plan-consolidation + creative redesign | `verified_complete` |

### 1b. The completed sixteen

| ID | Title | Status |
|---|---|---|
| A11 | Phase-B concurrency-ceiling investigation | `completed` |
| A12 | Destructive-ops "oh-shit" audit (what clobbers uncommitted/staged) | `completed` |
| A13 | Visual-snapshot baseline drift (matrix fails on intentional changes) | `completed` |
| A15 | Research-folder underutilization | `completed` |
| A25 | settings.json health + hook-chain hygiene (re-run, current-state) | `completed` |
| A45 | Required-service health-check + port-conflict + auto-protect | `completed` |
| A79 | Repo hygiene + exposure audit (PUBLIC-repo exposure surface) | `completed` |
| A81 | Never-idle structural enforcement | `completed` |
| A82 | True agent-plow ceiling / stats / reset / recovery synthesis | `completed` |
| A83 | Internal-build-but-surface-gaps | `completed` |
| A84 | Parallelization ceiling v2 (cloud-light correction, lean-toward-more) | `completed` |
| A85 | Git rollback safety / etiquette (Cursor dual-author era) | `completed` |
| A86 | Spawned-task lifecycle end-to-end | `completed` |
| A87 | Manifesto realignment (re-verify Claude's understanding of Evium) | `completed` |
| A88 | Git rollback — VISIBLE vs INVISIBLE change reclassification | `completed` |
| A89 | Dashboard migration completeness (DEV CONTROL dashboard) | `completed` |

### 1c. Archival rationale

The lifecycle in `audits/README.md` is explicit: a finished audit's file moves `audits/` → `archived/audits/` once it reaches a terminal `status` (`completed` or `verified_complete`), with its findings staying co-located in `audits/findings/` for traceability. The reorganization applied that rule in a batch to the audits that had genuinely reached terminal state — predominantly the **high-A-number recent wave (A75–A89)** plus older finished ones (A11–A13, A15, A25, A45, A71, A79–A84). The intent: stop the active `audits/` directory from masquerading as a backlog when ~20 of its entries were actually done. `.gitkeep` is preserved in `archived/audits/` (so the directory survives empty in git history).

> **ID-in-both-buckets nuance (not a duplication bug).** `A12` and `A25` exist in BOTH `audits/` (active definition) and `archived/audits/` (a completed *instance*), because each was **re-run** under the catalog's "new run → new file, never recycle an ID" rule. The active `A12-oh-shit-destructive-ops-audit.md` (`in_progress`) is the standing definition; `archived/audits/A12-destructive-ops-audit.md` (`completed`) is a finished pass. Same for A25 (`A25-settings-json-health-and-hook-chain-hygiene.md` active vs `A25-settings-hook-hygiene-audit.md` archived). This is expected, not a reconciliation conflict — flagged so a future reader does not "fix" it.

---

## 2. ACTIVE set (remaining in `audits/`) + each one's status

68 files. Grouped by status for the reconciled view (full per-file status was grepped from frontmatter; only groupings shown here for density).

### 2a. `available` — catalogued, never picked up (9)
A1 (reactionary automation), A2 (critical script coverage), A4 (skill graph), A5 (memory consolidation), A6 (worktree hygiene), A7 (style-token), A8 (missing script wrappers), A10 (scheduler workaround). These are the original low-number library entries that remain pure "ready-to-run-if-a-concern-arises" recipes. **No action — correctly parked.**

> `A9` (deny-list) has flipped to `in_progress` on disk (it was `available` in the README registry) — picked up since the registry was last rendered.

### 2b. `in_progress` — the bulk (56), with the close-candidate subset called out
Everything A14–A74 not otherwise terminal, plus A9, A80. Most are "Phase-1-landed, further phases gated on the user" rather than truly mid-flight. The **13 that are basically done except for a user-only step** are enumerated in `CLOSE-CANDIDATES-CHECKLIST.md` (which this doc seeds): **A37, A40, A44, A46, A47, A56, A57, A63, A66, A72, A73, A74, A80**. Each has its built-vs-pending split recorded there; the pending halves are all user-gated (a live-flip, a settings.json/launchd deploy, a subscription pick, or an explicit direction). Claude does not self-close these — the user checks them off (same rule as the manifesto: the user owns "done").

### 2c. Terminal-but-not-yet-moved (3) — the git-page recent wave
| ID | Title | On-disk status | Reconciliation action |
|---|---|---|---|
| A90 | Git rollback page — review-worthiness + per-commit copy + image-route correctness | `completed` | **Move to `archived/audits/`** (terminal; findings already in tree). |
| A91 | Git/rollback before-after picture-vs-description FIDELITY audit | `findings-complete` | Normalize → `completed`, then archive. |
| A92 | Git-history forensic FACT-CHECK (3-way: real diff vs message vs card) | `findings-complete` | Normalize → `completed`, then archive. |

These three are the "branch-cleanup re-audit" lineage's most recent reflective layer (see §3) — they finished as findings docs but their definition files still sit in active `audits/`, which is why the active count reads 68 rather than 65.

---

## 3. OPEN / unresolved items (the live tail as of today)

The genuinely-open work centers on the **Git/Rollback page** — the user's stated MAIN BOTTLENECK — through a chain of audits that supersede each other (A85 safety → A87 manifesto → A88 VISIBLE/INVISIBLE reclassification → A90 logic/content/image → A91/A92 before-after fidelity → the 2026-05-31 multipage + capture-integrity findings). This is the "branch-cleanup / dual-author re-audit" thread the catalog has been iterating.

1. **Broken before/after captures (HIGH — trust/data-integrity).** `findings/git-beforeafter-capture-integrity-audit.md`: **3 of 29 re-rendered route-pairs are byte-identical (0-pixel-diff)** across 3 commits (`382886e`, `3889ae6`, `3d56df0`) on 2 routes (`/products`, `/highway`); 1 more (`3d56df0 /blog`) is real-but-not-representative. Root cause = the capture frame does not contain the pixels the commit changed (two distinct frame-selection bugs in the capture harness), NOT "the change is too subtle." **Unresolved — fix prioritized over swapper polish by severity.**
2. **Multi-page swapper under-emphasized (MED — discoverability).** `findings/git-multipage-swapper-audit.md`: all **5 multi-page items** (`02359cd`/8pp, `3d56df0`/5, `3889ae6`/4, `a553e1a`/3, `382886e`/2) HAVE the per-page swapper and list ALL affected pages (**0 completeness victims**), but the route-picker renders as quiet 10px pills with no priority over the view-mode toggle — the user nearly missed it. This is a salience/hierarchy bug (text passes WCAG AAA), not a contrast bug. **Unresolved — fix = promote the picker to a labelled segmented control.**
3. **Dead `/__rollback-handoff` Path-B route (LOW — fails visibly).** `findings/git-button-persistence-ack-audit.md`: 8/8 rollback action controls persist correctly (NOT the chamber cosmetic-click bug). The single delivery weakness — the served-only "drop to inbox" button POSTs to a route no server in the repo handles — fails *gracefully and visibly* ("writer unavailable"), unlike the chamber's silent loss. The other 7 controls are 0-POST by deliberate, correct design. **Open but low-severity / honestly surfaced.**
4. **Charter still in active `audits/`.** `audits/git-page-multipage-and-capture-audit.md` is the in-progress charter framing items 1+2; it stays active until both fixes land, then it and A90–A92 retire together.
5. **The chamber `.dc-option` cosmetic-click bug** (`generate_dashboard.py:7332-7335`) remains live and open on the chamber page — out of git-page scope but noted by the persistence audit as still-real.

### Fix sequencing (from the charter)
All five git-page audits are read-only; the FIXES (repair captures → make swapper noticeable) land **after** the in-flight Track-A chamber-lockin BG frees `generate_dashboard.py`, batched with the card redesign so the ~8,300-line monolith is edited once, coherently. Capture repair (HIGH) precedes swapper polish (MED).

### Decouple-plan holdoff (steering, not an audit)
The most recent steering commit (`420bfaf`) records that **§4 decoupling is HELD for an explicit user greenlight** (a system-wide + Obsidian-coupled refactor). That is a goal/steering decision, not a catalog audit — noted here so the reconciliation reflects today's true gated state, but it does not change any audit status.

---

## 4. Reconciled snapshot (clean)

```
AUDIT CATALOG — 2026-05-31 reconciled
═════════════════════════════════════
ACTIVE  (audits/)            68
  available .................  9   (A1 A2 A4 A5 A6 A7 A8 A10 — pure library; A9 now in_progress)
  in_progress ............... 56   (incl. 13 user-only close-candidates → CLOSE-CANDIDATES-CHECKLIST.md)
  terminal-not-yet-moved ....  3   (A90 completed · A91/A92 findings-complete → normalize+archive)

ARCHIVED (archived/audits/)  21
  completed ................. 16   (A11 A12 A13 A15 A25 A45 A79 A81 A82 A83 A84 A85 A86 A87 A88 A89)
  verified_complete .........  5   (A71 A75 A76 A77 A78)

OPEN TAIL (live work)
  HIGH  git before/after capture broken — 3/29 pairs byte-identical (capture-frame bug)
  MED   git multi-page swapper complete but under-emphasized (salience, not contrast)
  LOW   git Path-B /__rollback-handoff route unwired (fails visibly, by-contract 0-POST elsewhere)
  open  chamber .dc-option cosmetic-click bug still live (out of git-page scope)
  held  §4 decouple plan — awaiting user greenlight (steering, not an audit)

NEXT AUDIT ID (main range): A93   (A90/A91/A92 are the last claimed; A93 is free)
RECONCILIATION ACTIONS:
  1. Move A90 → archived/audits/ (already terminal).
  2. Normalize A91, A92 status findings-complete → completed, then archive.
  3. Refresh audits/README.md registry table (stale at A68; add A69, A72–A74, A80, A90–A92
     and correct A9 → in_progress).
  4. Leave the 9 `available` library audits parked.
  5. User: walk CLOSE-CANDIDATES-CHECKLIST.md — each [x] authorizes a status:completed + archive move.
```

---

## 5. Provenance / honesty

- Counts and statuses were grepped read-only from each file's frontmatter on 2026-05-31; where a file's status string drifts from the `_README.md` taxonomy (`findings-complete`, `phase_2_patches_applied`, etc.) the drift is flagged, not silently coerced.
- The `audits/README.md` catalog **registry table is stale** (ends at A68, lists pre-reorg statuses); this reconciliation treats the **files themselves** as the source of truth and recommends a registry refresh as a follow-up.
- This file is the **sole write** of this reconciliation pass. No git operations, no code edits, no other file touched.
