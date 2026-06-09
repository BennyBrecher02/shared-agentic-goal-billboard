# Lefter rail P0 priority chip — 2.57:1 contrast (WCAG fail) [INVALID — measurement error]

- **id:** 20260527T0010-main-subagent-per-tab-sweep-33e498ed
- **discovered:** 2026-05-27T00:10:00Z
- **agent:** subagent-per-tab-sweep
- **worktree:** main
- **route:** /audit-timelapse/2026-05-25-overnight (dashboard)
- **section:** lefter-rail
- **viewport-bucket:** all
- **severity:** medium
- **status:** wontfix
- **defect-type:** contrast-fail
- **dedupe-key:** dashboard|lefter-rail|all|contrast-fail

## Resolution: invalid — the original audit miscomputed contrast

The earlier audit (BG #74, inbox entry `20260526T2226-main-subagent-dashboard-self-audit-94dc32cd`) reported `rgb(249,169,143)` on `rgb(200,68,43)` = 2.57:1.

That used the **raw chip color** (`200,68,43`) as the background. But the actual CSS rule is:

```
background: rgba(200,68,43,0.18);
```

which renders as alpha-composited over the dark lefter rail parent (`rgb(11,15,23)`). The composited rendered background is `rgb(45,25,27)` (computed) / `rgb(48,32,38)` (sampled from rendered pixels).

## Measured contrast (this audit, 2026-05-27)

Method: Playwright `browser_evaluate` + WCAG formula + PIL pixel sampling of the actual rendered chip screenshot.

```
fg:  rgb(249, 169, 143)         (matches declared color)
bg:  rgb(48, 32, 38)            (PIL-sampled, alpha-composite)
contrast ratio: 8.16 : 1        (WCAG AA-large + AA-normal both pass; AAA-normal passes)
```

The 8.16:1 figure has headroom against the 4.5:1 normal-text threshold and the 7:1 AAA threshold. No CSS change required.

## Methodology note for future audits

When a CSS rule uses `background: rgba(...)`, `getComputedStyle.backgroundColor` returns the **declared** rgba — not the rendered solid. To get the actual rendered contrast you must either:

1. Walk parent chain to find first opaque background + alpha-composite (Player-supplied math).
2. PIL-sample a screenshot of the element.

Bug billboard entries for contrast should always note BOTH the declared chip color AND the parent's solid color when the chip is semi-transparent.

Logged as a recommendation note to `context/markdowns/agent-recommendations/page-scrutinization.md`.
