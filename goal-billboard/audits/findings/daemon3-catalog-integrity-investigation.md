---
title: daemon3 (audit-catalog-maintainer) CRITICAL — 43-finding integrity spike root-caused to tonight's archiving
type: findings
mode: read-only
status: investigation-complete (recommend-only; nothing edited/committed)
created: 2026-05-31
investigated_by: subagent (BG dispatch, read-only — daemon source read, real catalog scanned, daemon rules reproduced by hand)
hard_timeout: 25min
daemon_under_review: scripts/event-bus/consumers/consumer-audit-catalog-scan-daemon3.sh
poll_source: .claude/cache/daemon3-audit-catalog-poll.jsonl
verdict: >
  41 of 43 findings are FALSE POSITIVES caused by a naive daemon that scans ONLY the active
  audits/ dir and has no concept of archived/audits/. 4 genuinely-stale file PATHS exist in the
  active substrate (worth a one-line cleanup each). The fix is overwhelmingly a DAEMON-RULE change,
  not a catalog clean-up.
---

# daemon3 catalog-integrity spike — root cause + fix

## TL;DR

Tonight's archiving (~20 audits moved `audits/` → `archived/audits/`, 34 YAML files fixed, statuses
reconciled) tripped daemon3 from **10 findings → 43 findings** in two hourly polls. **41 of the 43 are
false positives** the daemon should never have raised: it scans **only** the active `audits/` directory
(`AUDITS_DIR` hard-set at line 52) and has **no awareness that `archived/audits/` exists**, so every
archived A-number reads as a "gap", every findings file whose parent was archived reads as an "orphan",
and every bare `A<N>` token citing a now-archived audit reads as a "broken xref".

**The fix is a DAEMON-RULE change** (teach rules 1/2/4 that archived audits are expected). Only **4
genuinely-stale file paths** in the active substrate are worth cleaning, and **2 of those 4 are
append-only historical log entries that arguably should NOT be rewritten.**

### Poll timeline (proves the archiving is the cause)

```
2026-05-31T06:01Z   gaps=6   orphan=0  xref=4   total=10   (pre-archiving baseline, stable for ~24h)
2026-05-31T07:01Z   gaps=15  orphan=8  xref=8   total=31   (archiving in progress — audits_total dropped 84→65)
2026-05-31T08:01Z   gaps=24  orphan=8  xref=11  total=43   (archiving settled — audits_total 66)
2026-05-31T09:01Z   gaps=24  orphan=8  xref=11  total=43   (steady state — THE CRITICAL the daemon reports)
```

`audits_total` falling 84 → 65 → 66 is the archiving event in the daemon's own log. The finding
counts rose in lockstep. This is conclusive.

---

## How each daemon rule works (from the source)

`consumer-audit-catalog-scan-daemon3.sh` is the **shell deployment shell** of daemon3 (the full 8-rule
Python detector is a deferred "Phase 1" per the header comment). The shell consumer computes 3 coarse
signals, all scoped to **`AUDITS_DIR = $REPO_ROOT/context/markdowns/goal-billboard/audits`** and its
`findings/` subdir **only**:

- **Rule 1 — sequential gaps (lines 105-119):** extract numeric IDs from `audits/A*.md`, take
  `min`..`max`, report `gaps = (max - min + 1) - count_of_distinct_ids`. **Naive:** it assumes the
  active dir holds a contiguous A-number range. Any A-number that is archived, proposed-but-unwritten,
  or simply skipped becomes a phantom gap.
- **Rule 2 — orphan findings (lines 121-134):** for each `findings/A<N>-*.md`, flag it if no
  `audits/A<N>-*.md` exists. **Naive:** it never checks `archived/audits/`. When a parent audit is
  archived, all its findings files instantly read as orphans.
- **Rule 4 — broken xref (lines 136-149):** `grep -hoE 'A[0-9]+'` every active audit body, then set-
  difference against the A-IDs that are files in the active dir; the remainder are "broken". **Naive
  twice over:** (a) it treats a *bare prose ID mention* (`A45 — required-service health`) identically
  to a *file path* — most "xrefs" are narrative, not links, and don't break when a file moves; and
  (b) it has no concept of `archived/audits/`, so a citation of an archived audit always counts as
  broken even when the audit is alive and well one directory over. It also can't see that a path may
  already say `archived/` (e.g. A90 already points correctly) because it only extracts the `A<N>` token.

The daemon is correctly read-only (writes only its JSONL log + a flag in LIVE mode; per A66 it never
edits any audit/finding/README). It correctly ignores `README.md`, `_README.md`, and
`CLOSE-CANDIDATES-CHECKLIST.md` (non-`A`-prefixed names fall outside the `A*.md` glob). The defect is
purely the missing archived-dir awareness + the bare-token xref heuristic.

---

## Rule 1 — 24 "gaps": 19 benign · 0 real-debris · 5 pre-existing

Active range is **A1..A90**; 66 active files. The 24 gap numbers split cleanly:

| Class | Count | A-numbers | Verdict |
|---|---|---|---|
| **(a) benign archiving side-effect** | **19** | A11, A13, A15, A45, A71, A75, A76, A77, A78, A79, A81, A82, A83, A84, A85, A86, A87, A88, A89 | All 19 are present in `archived/audits/`. NOT gaps — the audits exist, just moved tonight. Daemon should ignore. |
| **(c) pre-existing (reserved/skipped numbers)** | **5** | A52, A53, A54, A55, A70 | NOT debris. A52–A55 are listed **`proposed`** in `audits/README.md` (lines 175-178: bespoke-lefter, matrix-birdseye tab, inbox-unsend, dashboard-tops pass #2) — reserved future numbers never written as files. A70 is a fully skipped number (catalog jumps A69→A72; absent from README entirely). These predate tonight and are normal for a sparse, human-curated A-number space. |
| (b) real debris from tonight | **0** | — | none |

Net: **all 24 gaps are non-actionable.** 19 vanish the instant the daemon learns about `archived/`; 5
are legitimate holes in a non-contiguous numbering scheme and should not be "filled."

---

## Rule 2 — 8 "orphan findings": 8 benign · 0 real-debris · 0 pre-existing

Every one of the 8 orphan findings files has a parent audit that was **archived tonight** (parent now
in `archived/audits/`, absent from active `audits/`):

| Findings file (`audits/findings/`) | Parent | Parent now at |
|---|---|---|
| `A13-path-d-implementation-2026-05-26.md` | A13 | `archived/audits/A13-visual-snapshot-baseline-drift.md` |
| `A15-research-utilization-2026-05-26-initial.md` | A15 | `archived/audits/A15-research-folder-utilization.md` |
| `A76-current-system-downsides.md` | A76 | `archived/audits/A76-semantic-index-adoption-downside-analysis.md` |
| `A76-future-system-downsides.md` | A76 | (same) |
| `A76-server-analysis.md` | A76 | (same) |
| `A77-stockpile-conflict-scan.md` | A77 | `archived/audits/A77-idea-conflict-check-capability.md` |
| `A78-pipeline-code-audit.md` | A78 | `archived/audits/A78-dashboard-scrutiny-and-plan-consolidation.md` |
| `A78-ux-scrutiny.md` | A78 | (same) |

**(a) All 8 are benign archiving side-effects.** The findings files were left in the **active**
`audits/findings/` dir while their parent audits moved to `archived/audits/`.

**Optional (b) tidiness (NOT required):** one *could* relocate these 8 findings files to an
`archived/audits/findings/` dir to keep findings co-located with their parents. But this is a judgment
call, not debris that breaks anything — the findings are still readable, and the orphan signal
disappears entirely once the daemon stops treating archived parents as missing. **Recommendation: do
NOT move them; fix the daemon instead.** (If the user later prefers co-location, that's a separate
tidy-up, and the daemon rule should then look for the parent in *either* dir regardless.)

---

## Rule 4 — 11 "broken xrefs": 8 benign · 3 benign (proposed) · 0 real (from the daemon's own list)

The daemon's 11 broken-token set is `A11, A13, A15, A45, A52, A53, A55, A71, A85, A87, A88`. Reproduced
exactly by hand:

| Token | Class | Why |
|---|---|---|
| A11, A13, A15, A45, A71, A85, A87, A88 (8) | **(a) benign — archived** | Audit exists in `archived/audits/`; cited by active audits as a **bare prose ID** (e.g. `A45 — required-service health (auto-recovery cousin)`). These are narrative references, not file links; nothing is actually broken. Daemon flags them only because it can't see `archived/` and treats any cited ID without a same-dir file as broken. |
| A52, A53, A55 (3) | **(c) benign — proposed** | Cited in `A57-dashboard-content-quality-and-uxr.md` as forward-references to **`proposed`** (never-written) audits. Pre-existing; same root cause as the 5 phantom gaps. Not debris. |

**Crucial nuance the daemon misses:** the overwhelming majority of "xrefs" in this catalog are **bare
`A<N>` prose mentions**, not file paths. Moving a file does not break a prose mention of its ID. The
daemon's Rule 4 conflates the two. Example of the daemon being doubly-wrong: `A90-git-page-...md`
**already** has correct cross_refs at lines 15-17 pointing at `archived/audits/A88...`,
`archived/audits/A85...`, `archived/audits/A87...` — whoever archived tonight already fixed A90 — yet
the daemon still flags A85/A87/A88 as broken because it extracts only the bare token and checks the
active dir. **0 of the 11 are real broken references.**

---

## The genuinely-stale FILE PATHS (the only real debris) — 4 total, 2 worth fixing

These are **separate from the daemon's Rule 4 count** (the daemon counts bare tokens, not paths). A
direct path-grep of the **active** substrate (excluding the immutable `archived/` sources) for
`audits/A<N>-...md` links whose target is now archived found exactly 4. They are real stale links a
human reader would hit a 404-equivalent on:

| # | File (active) | Line | Stale ref | Should be | Notes |
|---|---|---|---|---|---|
| 1 | `context/markdowns/goal-billboard/audits/A22-cluster-and-ns-integration-gap.md` | 19 | `context/markdowns/goal-billboard/audits/A11-concurrency-ceiling.md` | `context/markdowns/goal-billboard/archived/audits/A11-phaseB-concurrency-ceiling.md` | **Pre-existing AND archive-broken.** The filename `A11-concurrency-ceiling.md` never existed — the real file was always `A11-phaseB-concurrency-ceiling.md`. So this `related_refs` entry was already a dead link *before* tonight; archiving compounded it (now wrong dir too). Worth fixing — it's an active audit's YAML. |
| 2 | `context/markdowns/goal-billboard/findings-leaderboard.md` | 34 | `audits/A79-repo-hygiene-and-exposure-audit.md` | `archived/audits/A79-repo-hygiene-and-exposure-audit.md` | Italic "_Full ranked findings appending to …_" pointer in a live leaderboard surface. Worth fixing (one-line). |
| 3 | `context/markdowns/goal-billboard/asks-log.md` | 20 | `markdowns/goal-billboard/audits/A79-repo-hygiene-and-exposure-audit.md` | `markdowns/goal-billboard/archived/audits/A79-repo-hygiene-and-exposure-audit.md` | **Append-only historical log** (per the bug-billboard/asks-log convention). Entry is a timestamped record of where the artifact was *at the time* (2026-05-29). Arguably should NOT be rewritten — historical logs record history, not current location. **Recommend LEAVE** (or fix only if the user prefers asks-log paths to stay click-through-live). |
| 4 | `context/markdowns/goal-billboard/asks-log.md` | 21 | `markdowns/goal-billboard/audits/A81-never-idle-structural-enforcement.md` | `markdowns/goal-billboard/archived/audits/A81-never-idle-structural-enforcement.md` | Same as #3 — append-only historical entry (2026-05-29). **Recommend LEAVE.** |

> Stale path-refs that live **inside `archived/audits/*` files** (A12, A25, A77, A84, A86, A87, A88 →
> A82/A85/A86/A87/A76/A25) were intentionally **excluded**: those source files are themselves archived
> and immutable per the archive-immutable convention, AND the daemon never scans them (Rule 4 globs the
> active dir only). They are out of scope — do not touch archived files to fix intra-archive links.

**Net real cleanup: 1 must-fix (#1, A22 — also fixes a pre-existing typo'd filename), 1 nice-to-fix
(#2, leaderboard), 2 leave-as-historical (#3, #4).** Even the maximal interpretation is 3 one-line
edits — trivial next to the 43-finding alarm.

---

## Classification summary

| Rule | Daemon count | (a) benign archiving side-effect | (b) real debris worth cleaning | (c) pre-existing real |
|---|---|---|---|---|
| Rule 1 — gaps | 24 | **19** (archived A-numbers) | 0 | **5** (A52-55 proposed, A70 skipped — *not* debris, normal sparse numbering) |
| Rule 2 — orphans | 8 | **8** (parent archived; optional relocation, not required) | 0 | 0 |
| Rule 4 — broken xref | 11 | **8** (archived) + **3** (proposed) = **11** | 0 (the daemon counts bare tokens) | 0 |
| **TOTAL** | **43** | **38** | **0** | **5** |
| Separate path-grep (real stale links) | — | — | **1 must-fix + 1 nice-to-fix + 2 leave-historical** | — |

So of the 43-finding CRITICAL: **38 are pure daemon-naivety false positives**, **5 are legitimate
non-contiguous-numbering holes (not debris)**, and the **actual repairable debris is 1–3 one-line path
edits that the daemon does not even count.** This is a daemon-rule bug, not a catalog-integrity crisis.

---

## RECOMMENDED FIX

### Primary — teach daemon3 about `archived/audits/` (kills 38 of 43 findings)

Add an archived-dir constant and fold it into all three rules. In
`scripts/event-bus/consumers/consumer-audit-catalog-scan-daemon3.sh`:

1. **New constant** (near line 52-53):
   ```bash
   ARCHIVED_AUDITS_DIR="$REPO_ROOT/context/markdowns/goal-billboard/archived/audits"
   ```

2. **Rule 1 — gaps (lines 105-119):** build the ID set from **active ∪ archived**, then subtract the
   set of numbers known to be intentionally-unwritten. Simplest robust form: a number in `min..max` is
   a *real* gap only if it is absent from **both** dirs **and** not marked `proposed`/reserved in
   `audits/README.md`. Pragmatic minimum (no README parse): treat a number as a gap only if it is in
   neither `audits/` nor `archived/audits/`. That alone drops gaps 24 → 5 (the proposed/skipped set),
   and those 5 are arguably WARN-not-CRITICAL. Best: also read the README "proposed" rows to suppress
   A52-A55, leaving only genuinely-unaccounted numbers (currently just A70, a deliberate skip → treat
   as WARN/info, not CRITICAL).

3. **Rule 2 — orphans (lines 121-134):** change the parent-existence check to look in **either** dir:
   ```bash
   if ! ls "$AUDITS_DIR/${finding_id}-"*.md "$ARCHIVED_AUDITS_DIR/${finding_id}-"*.md 2>/dev/null | head -1 | grep -q .; then
   ```
   This drops orphans 8 → 0.

4. **Rule 4 — broken xref (lines 136-149):** two changes. (i) Add archived IDs to the `existing_ids`
   set so a citation of an archived audit is not "broken":
   ```bash
   existing_ids=$( { ls "$AUDITS_DIR"/A*.md "$ARCHIVED_AUDITS_DIR"/A*.md 2>/dev/null; } | sed -E 's|.*/A([0-9]+)-.*|A\1|' | sort -u )
   ```
   (ii) **Stop counting bare-token mentions as broken xrefs** — the heavy lifting belongs in the
   deferred Python detector, but at minimum the shell rule should only flag tokens that appear in a
   **path context** (a `/A<N>-...md` substring), not every prose `A<N>`. Until the Python detector
   lands, consider downgrading Rule 4's severity from CRITICAL to WARN in the shell layer, since the
   bare-token heuristic is too coarse to be trustworthy (it already over-counts even pre-archiving:
   the stable baseline `xref=4` was itself mostly proposed/archived false positives).

5. **Severity recalibration (lines 158-163):** with the above, archived-aware orphan/xref counts go to
   0 and severity returns to `ok`. The 5 proposed/skipped gaps should map to `warn` or be suppressed —
   they are not a CRITICAL condition. Recommend: CRITICAL only when a findings file's parent is in
   **neither** dir, or a path-context xref resolves to **no file in either** dir.

**Expected post-fix poll:** `gaps≈0-5 (warn/info), orphan=0, xref=0, severity=ok`.

### Secondary — the small real-path cleanup (separate, optional, NOT done here)

- **Fix #1 (must):** `audits/A22-cluster-and-ns-integration-gap.md:19`
  `…/audits/A11-concurrency-ceiling.md` → `…/archived/audits/A11-phaseB-concurrency-ceiling.md`
  (also corrects a pre-existing wrong filename).
- **Fix #2 (nice):** `findings-leaderboard.md:34` `audits/A79-…md` → `archived/audits/A79-…md`.
- **Leave #3/#4:** `asks-log.md:20-21` — append-only historical entries; recommend not rewriting
  (record-of-history, not live index). Confirm with user; trivial to flip if they want live links.
- **Do NOT** edit intra-`archived/` links (immutable + unscanned).

### Note on `findings/` co-location (optional, deferred)
If the user wants findings to travel with archived parents, create
`archived/audits/findings/` and move the 8 orphan files there. This is purely organizational; the
daemon rule-2 fix (look in both dirs) makes it unnecessary for signal-cleanliness. **Recommend skipping
unless the user asks.**

---

## What was verified (read-only)
- Read the daemon source in full; reproduced rules 1/2/4 by hand against the live catalog — counts
  match the daemon's poll exactly (gaps 24, orphans 8, xref 11; findings_files 21; audits 66).
- Cross-checked every gap number against `archived/audits/` and `audits/README.md`.
- Confirmed each orphan findings file's parent lives in `archived/audits/`.
- Path-grepped the active substrate (excluding `archived/`) for stale `audits/A<N>` links → 4 hits.
- Confirmed A90's cross_refs are already archive-correct (daemon false-positives it anyway).
- Confirmed no active goal-billboard `active/*.md` entry carries a stale archived-audit path.
- **Nothing was edited, moved, or committed.** Recommend-only.
