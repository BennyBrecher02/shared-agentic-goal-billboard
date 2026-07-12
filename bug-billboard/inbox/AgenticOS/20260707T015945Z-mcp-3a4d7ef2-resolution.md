---
id: 20260707T015945Z-mcp-3a4d7ef2-resolution
resolves: 20260707T015945Z-mcp-3a4d7ef2
resolved: 2026-07-07T02:14:53Z
agent: claude-code
source: agent
status: fixed-awaiting-consolidation
route: scripts/frame-validate.sh + scripts/ios-simulator-capture.sh + scripts/android-emulator-capture.sh
project: AgenticOS
commit: 129a06382de266efdee3b1ea4cbf16fddbb4a5c0
---
# RESOLUTION — render-validated polling replaces the fixed render waits

Fixed in commit `129a0638` (2026-07-07). The os-tools `bug_log` MCP is append-only (no
status-update parameter), so this is the appended resolution note for the consolidator;
the original inbox entry is untouched per the append-only lane contract.

## What changed
- **scripts/frame-validate.sh** (new, shared by both wrappers): capture → content check →
  retry polling. Content check (python3 + PIL, already a repo dep): crop to the CENTRAL
  60% band (drop top/bottom 20% — status/URL/nav chrome), downscale to ≤64px, luma
  histogram; **≥98% of pixels within ±2 of the modal luma = NOT RENDERED**. The central
  crop is load-bearing: whole-frame histograms measured only 85–91% on the white iOS
  evidence frames (live browser chrome over an unpainted viewport) vs 100.0% in the band.
- **Poll contract**: first attempt after the legacy base wait (iOS 4s, Android 20s — a
  warm device that renders on attempt 1 pays zero extra), then every 3s up to +30s; on
  budget exhaustion the wrapper prints `✗ capture invalid — blank frame after Ns` and
  **exits 2** — never "✓ Captured" on a blank. task-run-matrix consumers therefore see a
  matrix FAIL instead of green-lighting on non-evidence.
- Kill switch `AOS_FRAME_VALIDATE=0` (legacy behavior, loudly labeled unvalidated);
  graceful degrade if the helper or PIL is missing (warn, never brick a capture).

## Verification (all offline — NO devices booted; device-wall v2 live-verify unharmed)
10/10 classification on the 2026-07-06 evidence frames (threshold 98%, central band):

| Frame | Truth | Measured | Verdict |
|---|---|---|---|
| ios/iPhone-SE-(3rd-generation)-20260706-215239.png | blank (white) | 100.0% mode=255 | ✗ BLANK ✓ |
| ios/iPhone-17-Pro-Max-20260706-215259.png | blank (white) | 100.0% mode=255 | ✗ BLANK ✓ |
| ios/iPad-Pro-11-inch-(M5)-20260706-215322.png | blank (white) | 100.0% mode=255 | ✗ BLANK ✓ |
| android/Pixel_4a_API_34-20260706-215245.png | blank (black) | 100.0% mode=0 | ✗ BLANK ✓ |
| ios/iPhone-SE-3rd-gen-validated-20260706-215610.png | good (dark) | 35.5% mode=26 | ✓ RENDERED ✓ |
| ios/iPhone-17-Pro-Max-validated-20260706-215637.png | good (dark) | 35.6% mode=26 | ✓ RENDERED ✓ |
| ios/iPad-Pro-11-M5-validated-20260706-215707.png | good (dark) | 42.1% mode=16 | ✓ RENDERED ✓ |
| android/Pixel_4a_API_34-validated-20260706-215734.png | good (dark) | 34.5% mode=26 | ✓ RENDERED ✓ |
| android/Pixel_9_API_35-20260706-215358.png | good (dark) | 15.8% mode=16 | ✓ RENDERED ✓ |
| android/Pixel_Tablet_API_35-20260706-215451.png | good (dark) | 59.2% mode=16 | ✓ RENDERED ✓ |

The known false-negative risk (a legitimately near-solid dark dashboard, mean luma ~27)
passes with a wide margin: worst good frame 59.2% vs the 98% cutoff; worst blank 100.0%.

Tests: scripts/tests/test-frame-validate.sh — 16/16 pass (~2s), hermetic (generated
solid-white/black stubs + a real dark capture fixture at
scripts/tests/fixtures/dark-dashboard-frame.png), covers blank/rendered/corrupt
classification, the non-zero exit path, late-paint recovery, capture-command failure
(rc 2 distinct from blank rc 1), and the kill switch. Registered in run-tests.sh FAST
tier. `bash -n` clean on both wrappers + helper.
