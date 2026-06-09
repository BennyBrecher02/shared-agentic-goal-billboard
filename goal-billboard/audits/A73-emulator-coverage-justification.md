---
audit_id: A73
title: Emulator + simulator coverage justification — which sim/emu profiles belong in the matrix, which should be cut
status: in_progress
catalogued: 2026-05-28T16:02:00Z
priority_when_run: P1
estimated_effort: small (inventory + per-profile justification + recommendation)
trigger: |-
  2026-05-28T16:00Z — user verbatim: *"last night while i was sleeping you downloaded a bunch of new emulators. long ago i asked you for research on how many screen sizes was worth caring about logically and you did a full deep dive and came to a normal sounding solution that we've been using since then so justify every new emulator you got and decide if they make sense to be in the full 100% coverage e2e matrix because if some are too similar or too obscure(like fold) then we should hold off, so do an research rabbit hole+audit for this"*
deferral_reason: NONE — explicit user research+audit request
related_goals: [G2 (cost-efficient peak throughput; cutting redundant sims = cheaper sweeps)]
related_plans: [plans/multi-orchestrator-new-subscription-plan.md (matrix shard partition impact), plans/m2-ownership-tradeoff-analysis.md (W2 shard scope depends on this)]
serves_northern_star: G2
belongs_to_goal: G9
related_refs:
  - feedback_device-matrix-discipline (5-layer rule)
  - feedback_parallel-coverage-discipline (per-device 100%)
  - .claude/skills/agentic-device-testing/SKILL.md (the doctrine)
findings:
  - prior-research-conclusion: |-
      Original 9-project Playwright matrix decided 2026-05-24 based on engine coverage × form-factor matrix. Settled at 12 projects (currently): 10 distinct viewports × 4 engines mix.
  - sleep-plow-addition-count: 8 new sim/emu profiles added 2026-05-27 → 2026-05-28 sleep-plow (5 iOS + 4 Android — but only 2 Android (Pixel_9_API_35 + Pixel_Tablet_API_35) are actually USED as defaults; Pixel_9a + Pixel_Fold are listed in AVDs but Pixel_Fold capture is broken per P-017).
  - duplication-finding: iOS sims iPhone 17e + iPhone Air duplicate viewport coverage already in iPhone 17 + iPhone 17 Pro Max. Android Pixel_9a duplicates Pixel_9 form-factor (older API only). Pixel_Fold form-factor is genuinely distinct but currently broken AND user explicitly flagged as "too obscure".
  - user-explicit-flag: User called out "(like fold)" as example of "too obscure".
---

# A73 — Emulator + simulator coverage justification

## The prior research, recapped

Original decision (2026-05-24, captured in `feedback_device-matrix-discipline.md`):

12 Playwright projects covering 10 distinct viewports across 4 engines:

| Project | Viewport | Engine | Form factor |
|---|---|---|---|
| chromium-desktop | 1440×900 | Chromium | desktop |
| chromium-laptop | 1366×768 | Chromium | laptop |
| webkit-desktop | 1440×900 | WebKit | desktop |
| firefox-desktop | 1440×900 | Firefox | desktop |
| chromium-android | 393×873 (Pixel) | Chromium-android | phone |
| chromium-android-narrow | 360×800 | Chromium-android | phone-narrow |
| chromium-android-tablet | (Galaxy Tab S9 default) | Chromium-android | tablet |
| webkit-iphone | 402×874 (iPhone 17) | WebKit-mobile | phone |
| webkit-iphone-large | 440×956 (iPhone 17 Pro Max) | WebKit-mobile | phone-large |
| webkit-iphone-small | (iPhone SE 3rd gen) | WebKit-mobile | phone-small |
| webkit-ipad | (iPad Pro 11 default) | WebKit-mobile | tablet |
| webkit-ipad-mini | (iPad Mini default) | WebKit-mobile | tablet-small |

**Rationale (verbatim from device-matrix-discipline)**: "The 9-project matrix catches WebKit-engine bugs, Android Chrome differences, tablet form-factor issues, and Firefox regressions in one ~3-min parallel run."

**This was the "normal sounding solution"** — engine-coverage × form-factor, deliberately bounded to keep wall-clock at ~3-min.

## Layer 3+4 purpose (the sim/emu layer)

Per the 5-layer hard rule (2026-05-28 anti-drift addition):

- **Layer 2** = Playwright matrix (engine emulation — what's above)
- **Layer 3** = iOS Simulator (real Mobile Safari shell — catches 100dvh / safe-area / URL-bar / font-substitution gaps Layer 2 misses)
- **Layer 4** = Android Emulator (real Android Chrome — hardware accel / paint diffs / touch behaviors)

**Layer 3+4 value = REAL OS shell**, not viewport diversity. The viewport diversity is Layer 2's job.

This is the lens for justifying each sim/emu profile.

## Inventory: what's currently configured

### iOS (Layer 3)

`scripts/ios-simulator-capture.sh` + orchestrator defaults: **iPhone 17**. User mentioned manually using these in BG context:
- iPhone 17 (default; comments cite as "current flagship baseline, iOS 26.5")
- iPhone 17 Pro Max
- iPhone 17e
- iPhone Air

All 4 are available simctl device types.

### Android (Layer 4)

`emulator -list-avds` returns: **4 AVDs**
- Pixel_9_API_35 (default; orchestrator)
- Pixel_9a_API_34
- Pixel_Fold
- Pixel_Tablet_API_35

## Per-profile justification

### iOS profiles

| Profile | Viewport (~) | Distinct from Layer 2 webkit-iphone-*? | Distinct from another iOS sim? | Verdict |
|---|---|---|---|---|
| iPhone 17 | 402×874 | ✅ adds real Mobile Safari shell to webkit-iphone | baseline | **KEEP** — the standard real-iOS baseline |
| iPhone 17 Pro Max | 440×956 | ✅ adds real Mobile Safari shell to webkit-iphone-large | distinct viewport from iPhone 17 | **KEEP** — large-iPhone real-iOS coverage |
| iPhone 17e | ~390×844 (close to iPhone 17) | ✅ technically; but viewport ≈ iPhone 17 | ❌ duplicate viewport of iPhone 17 | **DROP** — viewport coverage already provided by iPhone 17 |
| iPhone Air | ~437×934 (close to Pro Max) | ✅ technically; but viewport ≈ Pro Max | ❌ duplicate viewport of iPhone 17 Pro Max | **DROP** — viewport coverage already provided by Pro Max |

**iPhone Air note**: it's a real iOS device type (iOS 18+). Positioned between standard and Pro Max. But for Layer 3 purposes — real-iOS-shell verification — having both Pro Max AND Air is redundant. The shell behavior is identical; only viewport differs marginally.

### Android profiles

| Profile | Viewport / form factor | Distinct from Layer 2 chromium-android-*? | Distinct from another Android AVD? | Verdict |
|---|---|---|---|---|
| Pixel_9_API_35 | ~412×915 phone, API 35 | ✅ adds real Android Chrome shell | baseline | **KEEP** — standard Android phone real-shell |
| Pixel_9a_API_34 | ~412×915 phone, API 34 | ✅ technically; older API marginal value | ❌ same form-factor as Pixel_9; only API differs | **DROP** — API-version diversity is low-value vs the cost |
| Pixel_Fold | foldable 6.2"/7.6" hinge | ✅ unique form factor (not in Layer 2) | ✅ truly distinct | **DROP** (per user flag + P-017 capture corruption) — see analysis below |
| Pixel_Tablet_API_35 | ~600×960 tablet | ✅ tablet form in real Android shell | ✅ tablet vs phone is distinct | **KEEP** — Android tablet real-shell complement to webkit-ipad |

**Pixel_Fold deep analysis** (the user-flagged one):

Arguments FOR keeping:
- Genuinely distinct form factor (Layer 2 doesn't emulate fold hinges)
- Foldable market growing (Samsung Z Fold, Google Pixel Fold)
- Astro app should at least RENDER on a fold

Arguments AGAINST (winning):
- **Currently broken** (P-017 — `screencap` returns invalid PNG; not a viewport bug, a capture-infra bug)
- **Obscure form factor** for this client's audience (Evium EV charging — not a foldables-first user base)
- **Time/token cost**: each sweep including Pixel_Fold = additional ~30-45s wall-clock + ~9 capture attempts that auto-shrink fails
- **User explicit flag**: "(like fold)" was the user's own example of too-obscure
- **No active defect tied to fold-specific rendering** in the bug billboard

**Verdict on Pixel_Fold: DROP from active matrix**. Park the AVD on disk (don't delete) so it's recoverable if foldables become relevant for a future client. P-017 stays as `deferred-until-asked` — no debug investment.

## Recommended matrix (post-cut)

### iOS (Layer 3) — 2 profiles (down from 4)

- iPhone 17 (standard real-iOS shell)
- iPhone 17 Pro Max (large real-iOS shell)

### Android (Layer 4) — 2 profiles (down from 4)

- Pixel_9_API_35 (standard Android phone real-shell)
- Pixel_Tablet_API_35 (Android tablet real-shell)

### Layer 2 (Playwright) — unchanged at 12 projects

The Playwright matrix is the engine-coverage × form-factor backbone. No changes recommended.

## Wall-clock impact

| Layer | Before | After | Saved |
|---|---|---|---|
| 2 — Playwright matrix | ~3 min (parallel) | ~3 min | 0 |
| 3 — iOS sweep (per profile ~20-25s) | 4 × 25s ≈ 100s | 2 × 25s ≈ 50s | **~50s** |
| 4 — Android sweep (per AVD ~30-40s plus boot) | 4 × 40s ≈ 160s | 2 × 40s ≈ 80s | **~80s** |
| **Total per full sweep** | ~5.3 min | ~3.5 min | **~1.8 min (34% saving)** |

Per A72 cost-tracking — sweep wall-clock savings compound across the multi-orchestrator era when M2 also runs sweeps.

## What I'm NOT recommending

- **Don't delete the AVDs/sims from disk** — keep them parked for future use. Removal would mean re-download if needed.
- **Don't touch Playwright matrix Layer 2** — it's the well-justified backbone.
- **Don't add MORE sim/emu profiles** — the 5-layer rule says "Layer 3 catches what Layer 2 can't" not "Layer 3 covers every device variant." Two-per-OS is sufficient.

## Action items (this audit, to ship the recommendations)

| Item | Effort | Status |
|---|---|---|
| Document recommendation in this audit | done | ✓ |
| Steering log entry | done | will land with commit |
| Update `mobile-sim-sweep-orchestrator.sh` to default-skip Pixel_Fold + Pixel_9a + iPhone 17e + iPhone Air | 5 min | NOT done — awaiting user confirm of recommendation |
| P-017 stays deferred (already) | done | ✓ |
| Update `feedback_device-matrix-discipline` memory rule with Layer 3+4 cap (2-per-OS) | 5 min | NOT done — awaiting user confirm |

## Open questions for you

1. Accept the 2-per-OS Layer 3+4 cap (drop iPhone 17e + iPhone Air + Pixel_9a + Pixel_Fold from active matrix)?
2. Default-skip in the orchestrator OR delete the AVD/sim references entirely? Recommend default-skip (recoverable).
3. Future trigger for re-adding fold form factor — when (if ever)? Recommend: only if a foldables-targeting client comes in.

## Cross-references

- `feedback_device-matrix-discipline` — 5-layer hard rule
- `goal-billboard/pins/P-017-pixel-fold-capture-corruption.md` — deferred-until-asked
- `plans/multi-orchestrator-new-subscription-plan.md` — matrix shard scope when M2 goes live
- `plans/m2-ownership-tradeoff-analysis.md` — W2 matrix shard partition impact
- `goal-billboard/audits/A72-anthropic-cost-comparison-and-money-tracking.md` — sweep wall-clock savings compound with multi-account era

