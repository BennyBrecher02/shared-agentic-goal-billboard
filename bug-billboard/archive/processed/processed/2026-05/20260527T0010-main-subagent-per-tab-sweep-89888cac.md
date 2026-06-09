# Bug-billboard inbox-form horizontal overflow @ 375px viewport

- **id:** 20260527T0010-main-subagent-per-tab-sweep-89888cac
- **discovered:** 2026-05-27T00:10:00Z
- **agent:** subagent-per-tab-sweep
- **worktree:** main
- **route:** /audit-timelapse/2026-05-25-overnight (dashboard, billboard tab)
- **section:** bb-inbox
- **viewport-bucket:** mobile-375
- **severity:** medium
- **status:** done
- **defect-type:** layout-overflow
- **dedupe-key:** dashboard|bb-inbox|mobile-375|layout-overflow

## What was wrong

`.inbox-form` had `grid-template-columns: 1fr auto auto`. At 375 px viewport the `1fr` track refused to shrink below the input's intrinsic min-content width (the placeholder string was treated as content), so the `Queue ▷` button + select column extended past the form's right edge into the viewport's scroll-overflow region. Result: `document.documentElement.scrollWidth = 420` while clientWidth = 375 → 45 px horizontal overflow visible on EVERY tab at 375px (because the bug billboard inbox renders on every tab as part of the persistent shell).

## Verification (pre-fix, Playwright + browser_evaluate)

```
grid-template-columns: 171.5px 77px 70.9766px  (319px content forced)
form width:            261px  (~74px short)
Queue button right:    420px  (past viewport edge 375)
```

## Fix

`reports/timelapse/2026-05-25-overnight/index.html` line 1996 + input rule at line 1997:

```css
/* before */
.inbox-form { display: grid; grid-template-columns: 1fr auto auto; ... }

/* after */
.inbox-form { display: grid; grid-template-columns: minmax(0, 1fr) auto auto; ... }
.inbox-form input[type="text"] { ...; min-width: 0; ... }
```

Combo of `minmax(0, 1fr)` track + `min-width: 0` on the input lets the 1fr column shrink below its intrinsic placeholder width.

## Verification (post-fix, Playwright sweep)

All 5 dashboard tabs verified at 375 px:

```
timelapse:   scrollW 375 == clientW 375  (no overflow)
scheduler:   scrollW 375 == clientW 375
subagents:   scrollW 375 == clientW 375
improvement: scrollW 375 == clientW 375
stats:       scrollW 375 == clientW 375
```

1440 + 768 widths unchanged (form 1102 px wide on 1440 viewport; all children fit within form right edge 1399).
