---
audit_id: A46
title: Monkey chamber UX de-retardize — hide decided cards from default view + clean reopen-for-re-answer flow + empty-pending celebration
status: in_progress
catalogued: 2026-05-27T04:45:12Z
phase_1_landed_at: 2026-05-27T04:59:05Z
phase_1_apply_report: reports/audit-findings/A46-phase-1-apply.md
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-27T04:45Z — user explicit (frustration markers + flesh): *"this ui/ux feels genuinely retarded, how can we upgrade our monkey chamber safely?... the answered questions should go away but maybe we should allow some neat clean way to be able to go back and give another answer like we have now but that can only be after we de-retardize the monkey communication chamber."* A19 RECURRENCE — chamber UX has had multiple upgrades (task #35, #42, #75) and is STILL frustrating; structural redesign needed not band-aid.
deferral_reason: NONE — running in parallel with user finishing chamber-mania; built via BG while user works
related_goals: [G14, G6]
related_plans: [context/markdowns/plans/automation/monkey-chamber-ux-de-retardize-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - Task #42 "Add 'Other' option + text input" — existing offline queue path
  - Task #75 "A20 hedge-detect + monkey chamber expansion" — category tabs predecessor
  - Task #35 "Screenshot Monkey 5th page + capture decisions" — original chamber build
  - bug-billboard `20260527T0410-main-claude-main-dashboard-reset.md` (companion: anti-jump fix)
findings: []
---

# A46 — Monkey chamber UX de-retardize

## The user's frustration (verbatim)

*"this ui/ux feels genuinely retarded, how can we upgrade our monkey chamber safely?... the answered questions should go away but maybe we should allow some neat clean way to be able to go back and give another answer like we have now but that can only be after we de-retardize the monkey communication chamber."*

## What's broken today

1. **Decided cards STILL render** in main view — just sort to bottom and get `.decided` CSS class (faded). Clutters the pending queue.
2. **No "I'm done" feedback** — when chamber empties, no clear visual celebration.
3. **Inline-only re-answer** — current "Other" text input on a decided card works, but the workflow is confusing (looks like the prior answer is still active).
4. **No filter axis for status** — only category tabs (all / design / settings-patch / system-install / yes-no-customize). No `Pending / Decided / All` orthogonal filter.

## What's GOOD today (preserve)

- Category tabs work well
- Long-pending flag (>24h) bubble-up is good
- Urgency-first sort
- Previous-answer note via `loadMonkeyHistory` (audit trail of past responses)
- "Other" text input with offline-first queue (task #42)
- mkSent in-memory tracker

## The redesign

### Filter dimension: STATUS (new — orthogonal to category)

Add 3 status chips next to category tabs:
- `Pending (N)` — default selected; hides decided cards entirely
- `Decided (M)` — shows only decided cards, sorted decided_at DESC, with answer summary inline
- `All (N+M)` — current behavior preserved as fallback

Layout: `[ Pending | Decided | All ]   ⋅⋅⋅   [ all | design | settings | install | yes-no ]`

Two orthogonal filter axes. Both filters apply together (e.g., Decided × design = decided design decisions).

### Decided view shape

In Decided filter:
- Cards stack chronologically (most-recently decided first)
- Each card shows: question (smaller) + LATEST answer (prominent) + "previous answers: N" badge if reopened before
- `↺ Re-answer` button (top-right) — clicks to REOPEN

### Reopen flow (`↺ Re-answer`)

1. Click reopen → card transitions back to Pending state
2. Card behavior:
   - mkSent.delete(decisionId) — removes session-local "sent" tracking
   - `.decided` class removed
   - Options re-enabled
   - "Other" textarea cleared + ready
   - "↺ Reopened — N prior answer(s) on record" badge added
3. Inbox message auto-posted: `DECISION <id>: REOPEN — superseding prior N answer(s)` (urgent)
4. Card appears in Pending view + auto-scrolls to it
5. User submits new answer normally
6. Old answers REMAIN in inbox history (NOT deleted; `loadMonkeyHistory` shows all)

### Empty-pending celebration

When Pending count hits 0:
- Empty-state panel: `🐒 The troop is at rest. 0 pending decisions.`
- Cute touch: small banana/vine SVG; subtle pulse
- Below: link "Show all decided" → switches to Decided filter

### Audit trail preservation

EVERY answer EVER given persists in:
- `context/markdowns/agent-inbox/processed/<ts>-<hash>.md` files (server-side)
- `loadMonkeyHistory` queries these by `DECISION <id>:` prefix
- "previous answers: N" badge on reopened cards shows count

The user can ALWAYS see the full chronology of their answers per decision. Reopen doesn't erase; it appends.

## Implementation phases

### Phase 1 — Foreground UI build (BG; ~30min)

5 distinct edits in `public/audit-timelapse/2026-05-25-overnight/index.html`:

1. **HTML**: add status-filter chip row to `#mk-tabs` area (3 chips: Pending / Decided / All)
2. **CSS**: chip styling matching existing `.mk-tab`; decided-view card layout (compact); reopen button (`.mk-reopen`); empty-pending celebration (`.mk-empty-troop-rest`)
3. **JS state**: `mkActiveStatus = 'pending'` (3-state); read from URL `?monkey-status=X` for deep-link
4. **JS render**: `mkRenderForActiveTab()` extends to filter by status × category; render variant for decided cards (compact + reopen button)
5. **JS handlers**: status-chip click handlers; reopen handler (mkSent.delete + class remove + state reset + inbox POST `REOPEN` message + scroll-to)

### Phase 2 — Safety + tests

- Hash-skip (A19-related fix) extended to include `mkActiveStatus` in the hash
- Unit tests for filter logic + reopen + empty-pending
- Manual smoke-test plan: pending→decide→decided→reopen→pending again

### Phase 3 — Polish

- 🐒 emoji + vine SVG for empty-troop-rest
- Subtle decided-count animation when a card flips pending→decided (so it FEELS like progress)
- Deep-link support: `?tab=monkey&monkey-status=decided&monkey-focus=SET-005`

## Safety constraints

- **NEVER delete answer history** — `processed/` inbox files are immutable audit trail
- **NEVER reopen without explicit user click** — no auto-reopen
- **Idempotent reopen** — clicking ↺ N times only triggers ONE inbox post + UI flip (debounce)
- **Settings.json untouched** — pure dashboard HTML/CSS/JS change; no hook chain edits
- **Backward-compat URL params** — existing `?tab=monkey&monkey-cat=X&monkey-focus=Y` still works; `?monkey-status=Z` adds dimension

## Cost gates

Phase 1 BG: <250k tokens, <30min wall-clock. Phase 2 tests: <100k. Phase 3 polish: <100k. Total <450k.

## Cross-references

- A19 — recurrence detection (this is the 4th-5th chamber UX iteration; A19 says structural not band-aid)
- A20 — hedge-detect + monkey chamber expansion (category-tabs predecessor)
- A40 — idea-gap detector (reopen flow PREVENTS the "I want to revisit my answer" gap)
- A38 — voice analysis (reopen audit trail is rich training data: same person, multiple answers across system states)
- task #42 — offline "Other" path (preserved + extended)
- task #75 — A20 monkey chamber expansion
- task #35 — original Screenshot Monkey 5th page build
- bug-billboard `20260527T0410-main-claude-main-dashboard-reset.md` — sibling stability fix in same area

## Lessons (preliminary)

- The chamber has had ~4 prior UX iterations and is STILL frustrating user — that's an A19 recurrence signal. Structural redesign (orthogonal filter axis) > another band-aid.
- The "previous answer note" feature was correctly built but USER VISIBILITY into it was poor — decided cards stayed mixed with pending. UX problem, not data problem.
- The reopen-with-audit-trail pattern generalizes — any future "I want to change my mind" feature should follow same shape (append, never delete; user-explicit reopen; idempotent).

## Status

**PHASE 1 LANDED** (2026-05-27T04:59:05Z, ~12min wall-clock). All 6 deliverables in place: HTML chip row, CSS (chips/decided-view/reopen/empty-troop-rest), JS state (`mkActiveStatus` w/ URL+localStorage), JS render (status × category filter, render-hash extended), JS handlers (chip clicks, `mkOnReopenClick` w/ idempotency + offline-queue), URL deep-link (`?monkey-status=X` + decided-card auto-show). 11/11 in-browser smoke-tests pass; reopen POST verified (priority=urgent, tag=monkey-reopen, message includes prior-answer count). Audit trail preserved — reopen APPENDS to inbox; prior answers stay in `processed/`. See `reports/audit-findings/A46-phase-1-apply.md` for full per-deliverable + per-test detail. Pre-existing TDZ errors on startup (`BB_DATA_URL` / `MK_DATA_URL` declared after IIFE calls them) noted but NOT in A46 scope — harmless on auto-refresh; sibling investigation recommended.

Phase 2 (safety + tests) + Phase 3 (polish) remain. Open for either to be triggered.

