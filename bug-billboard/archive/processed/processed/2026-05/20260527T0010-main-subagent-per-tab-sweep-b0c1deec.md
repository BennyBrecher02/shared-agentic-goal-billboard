# Stats Correlation tab skips H2 heading level (semantic + visual hierarchy break)

- **id:** 20260527T0010-main-subagent-per-tab-sweep-b0c1deec
- **discovered:** 2026-05-27T00:10:00Z
- **agent:** subagent-per-tab-sweep
- **worktree:** main
- **route:** /audit-timelapse/2026-05-25-overnight (dashboard, stats tab)
- **section:** page-stats
- **viewport-bucket:** all
- **severity:** low
- **status:** open
- **defect-type:** a11y-violation
- **dedupe-key:** dashboard|page-stats|all|a11y-violation

## What's wrong

The Stats Correlation tab page (`#page-stats`) has:
- 1 H1 (`Stats Correlation`, 28 px)
- 0 H2 elements
- 8 H3 elements (14 px) acting as the visible section heads ("Burn rate", "Recent sessions", etc)

Every other tab (subagents, scheduler, time-lapse) uses H2 (20 px) for top-level section headings. Stats skipping the H2 level breaks two contracts:

1. **Visual hierarchy** — top-level sections on this tab are 14 px (H3) versus 20 px (H2) on every sibling tab. Tab feels lower-priority / less polished by comparison.
2. **A11y / SR navigation** — screen readers expose heading levels as a navigation tree. Skipping H1 → H3 violates WCAG SC 1.3.1 (Info & Relationships, level A) and breaks the "next heading" keyboard shortcut for non-sighted users.

## Repro

1. Open dashboard
2. Click "Stats Correlation" in left nav
3. Inspect `#page-stats > h3` — there are 8, no H2 between H1 and them

## Suggested fix

Either:
- (a) Promote the 4 top-level group heads ("Burn rate", "Recent sessions", "Pushback rate", "BG-launch Pattern") from H3 to H2, OR
- (b) Add wrapper H2s like "Time & spend", "Session activity", "Frustration signals", "Concurrency patterns" above the existing H3 sub-heads.

Option (b) better matches the multi-section feel of other tabs.

## Zone

Stats Correlation HTML — owned by A14 agent. NOT fixed in this BG because the structural change merits a layout decision (which is option-(a)-or-(b)). Logged for next A14-zone touch.
