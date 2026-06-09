---
pin_id: P-017
title: Pixel_Fold capture corruption — emulator screencap returns invalid PNG
created: 2026-05-28T10:12Z
narrowed_in: layer-4 expanded coverage during sleep-plow (2026-05-28T06:09Z sweep)
why_paused: substrate bug observed; not blocking — 3 other Android profiles capture cleanly (Pixel_9 / Pixel_Tablet / Pixel_9a)
resume_criteria: deferred-until-asked (user 2026-05-28T15:38Z: "thats overhead we dont need to handle rn"); do not pick up autonomously
estimated_cost: ~30k tokens debug + maybe a small script patch
leverage_g2: 2.0 (substrate; cosmetic — affects only fold form factor coverage)
ns_relation: substrate; not G2-blocking (other 8 device profiles cover layer 3/4 adequately)
last_touched: 2026-05-28T10:12Z
state: deferred_dropped_from_matrix
user_confirmed_drop: 2026-05-28T16:10Z (user: "yeah based on your findings i agree this is the way to go")
related_audit: A73
belongs_to_goal: G2  # by-goal parent (goal-wiring §3 tier-B/C — user-approved)
---

# P-017 — Pixel_Fold capture corruption

## Symptom

After sleep-plow Phase I.8 extended Android coverage to Pixel_Fold AVD:
- Orchestrator sweep RESULT: all captures succeeded (9/9 reported OK with file sizes shown)
- Auto-shrink IMMEDIATELY failed: "audit-shrink FAILED (non-fatal; raw captures still on disk)"
- Manual `python3 audit-capture-shrink.py`: 9 errored / 0 processed
- `file Pixel_Fold-blog.png` reports: "data" (not "PNG image")
- PIL: `UnidentifiedImageError: cannot identify image file`

## Diagnosis hypothesis

Pixel_Fold AVD's `adb exec-out screencap -p` is emitting non-PNG content. Possibly:
- Folded vs unfolded state confusion (Pixel_Fold has 2 displays)
- AVD GPU rendering returning raw framebuffer instead of PNG
- Display index mismatch in capture script

## Fix path (for next cycle)

1. Reproduce manually: `~/Library/Android/sdk/platform-tools/adb exec-out screencap -p > /tmp/pixel-fold-test.png` and inspect bytes
2. Check `~/Library/Android/sdk/platform-tools/adb shell dumpsys SurfaceFlinger | head -50` for display setup
3. May need to specify display ID: `adb shell screencap -d <display_id>` for folded device
4. Update `scripts/android-emulator-capture.sh` with Pixel_Fold-aware display selection

## Workaround

- 3 other Android profiles (Pixel_9 / Pixel_Tablet / Pixel_9a) capture cleanly — layer-4 coverage is adequate without Pixel_Fold
- Not blocking G2-CLOSE verdict (already YES based on 8 working device profiles + Playwright matrix)

## Cross-references

- `scripts/mobile-sim-sweep-orchestrator.sh` (orchestrator that surfaces auto-shrink failure non-fatally)
- `scripts/android-emulator-capture.sh` (the underlying screencap script)
- P-001 (wave 3 audit) — Pixel_Fold not in wave 3 device list, so unrelated
