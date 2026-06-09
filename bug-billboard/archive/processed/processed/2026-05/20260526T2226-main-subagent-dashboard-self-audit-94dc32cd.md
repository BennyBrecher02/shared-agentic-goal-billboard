# Lefter rail P0 priority chip — 2.57:1 contrast (WCAG fail)

- **id:** 20260526T2226-main-subagent-dashboard-self-audit-94dc32cd
- **discovered:** 2026-05-26T22:26:00Z
- **agent:** subagent-dashboard-self-audit
- **worktree:** main
- **route:** /audit-timelapse/2026-05-25-overnight (dashboard)
- **section:** lefter-rail
- **viewport-bucket:** all
- **severity:** medium
- **status:** open
- **defect-type:** contrast-fail
- **dedupe-key:** dashboard|lefter-rail|all|contrast-fail

## What's wrong

The P0 priority chip inside `.lefter-ns-mini-pri` renders foreground `rgb(249,169,143)` on background `rgb(200,68,43)` — contrast ratio 2.57:1. WCAG AA minimum is 4.5:1 for small text (9px / weight 700 fails the large-text exception threshold of 18.66px+700).

## Computed style

```
.lefter-ns-mini-pri (P0)
  color: rgb(249, 169, 143);    -- pale orange
  background-color: rgb(200, 68, 43);  -- bright red-orange
  font-size: 9px;
  font-weight: 700;
  contrast-ratio: 2.57 : 1   (WCAG fails 4.5)
```

## Repro

Open the dashboard. The lefter rail (left sidebar) shows mini-NS cards with a colored priority chip (P0 / P1 / P2). On P0 cards, the chip text is hard to read against its red-orange background.

## Suggested fix

Change `.lefter-ns-mini-pri` (P0 variant only) foreground to pure white `#FFFFFF` — would yield ~5.3:1 against the same red-orange. Or darken the chip background to `#9A3220` to push ratio above 4.5 while keeping the orange-warning visual cue.

## Zone

Lefter-rail CSS — owned by Lefter v2 agent. NOT swept in the dashboard-self-audit fix wave because it's outside the typography-tokens / radius-tokens / monkey-copy zones. Logged here for the next agent that touches lefter styling.
