# Dashboard auto-refresh nukes user's typing in monkey-other textbox (FIXED)

- **id:** 20260527T0410-main-claude-main-dashboard-reset
- **discovered:** 2026-05-27T04:10:35Z
- **agent:** claude-main
- **worktree:** main
- **route:** dashboard (/audit-timelapse/2026-05-25-overnight/)
- **section:** monkey panel + global auto-refresh
- **viewport-bucket:** all
- **severity:** high
- **status:** done
- **defect-type:** interactive-broken
- **dedupe-key:** dashboard|monkey-panel|auto-refresh-nukes-input|all

## Description

The monkey chamber's "Other" text input lost typed text every 10 seconds because `cardsEl.innerHTML = filtered.map(d => renderMonkeyCard(d)).join('')` in `mkRenderForActiveTab()` blew away the DOM unconditionally on every auto-refresh tick. Prior "anti-jump" fix (Stream B, 2026-05-26) only addressed scroll-position restoration; never addressed the DOM-nuke.

User report (2026-05-27T03:54:06Z, urgency=urgent): *"the dashboard is still jumpy and constantly reseting which takes me out of typing in textboxes, how can we FINALLY get rid of this behavior."*

## Root cause

3 layers of bug:
1. `wireAutoRefresh.tick()` called PAGES.monkey.fn() every 10s with no typing-awareness
2. `mkRenderForActiveTab()` always called innerHTML even when data unchanged
3. No persistence/restore of in-progress textarea values around re-render

## Fix (3 layers, all in `public/audit-timelapse/2026-05-25-overnight/index.html`)

### Layer 1 — `isUserActivelyTyping()` guard at tick level
`wireAutoRefresh.tick()` now checks if focused element is a typeable input/textarea inside the active dashboard page. If yes → skip this tick + show `.ar-deferred` visual cue (yellow ⟳ glyph + "typing" label). Manual ⟲ Refresh still works.

### Layer 2 — render-hash skip in `mkRenderForActiveTab()`
Hash `(id, status, queued_at, urgency, __longPending, mkActiveCategory)` of all filtered decisions. If hash matches `cardsEl.dataset.lastRenderHash` → skip innerHTML entirely. Cuts no-op re-renders to zero.

### Layer 3 — in-flight textarea capture + restore around legit re-renders
For the edge case where data legitimately changed while user was typing: before re-render, snapshot `{value, selectionStart, selectionEnd, hadFocus}` for any non-empty `.mk-other-input`. After re-render, restore values by `decision-id` match + re-trigger count update + restore focus + cursor position.

### Plus CSS
New `.ar-indicator.ar-deferred` styling — yellow glyph + " typing" label so user can SEE the tick was held (not silently dropped).

## Verification

1. Reload `dashboard?tab=monkey` (or `:/audit-timelapse/2026-05-25-overnight/index.html?tab=monkey`)
2. Click in any "Other" textbox + start typing
3. Watch `⟳ auto` indicator — should turn YELLOW + show "typing" during your input
4. Pause typing for >10s — indicator returns to spinning blue (auto-refresh resumes)
5. Type 100+ chars then walk away 30s + come back — text is still there

## Source

Logged in same response that fixed it. User explicit: *"FINALLY get rid of this behavior."*

## Cross-references

- Stream B "anti-jump" fix (2026-05-26 PM) — predecessor; addressed scroll, missed DOM nuke
- A19 recurrence detection — repeated user frustration → structural fix not band-aid
- A20 monkey chamber expansion (where "Other" textarea was originally added)
- A40 idea-gap detector — meta-pattern of "build-but-not-test-the-actual-usage"
