---
audit_id: A51
title: Dashboard typing survives Vite HMR full-page reloads (Layer 4 — the actual root cause user keeps reporting)
status: in_progress
catalogued: 2026-05-27T06:00:00Z
priority_when_run: P0
estimated_effort: small (inline JS additions to existing dashboard)
trigger: |-
  2026-05-27T05:52Z — user 3rd-time report: *"the dashboard is still jumpy and constantly reseting which takes me out of typing in textboxes, how can we FINALLY get rid of this behavior from our dashboard, launch investigation and testing."* My prior Layer 1+2+3 fix (typing guard + render-hash skip + in-flight capture) addressed INTRA-PAGE auto-refresh; this audit addresses the OTHER root cause — HMR full-page reloads.
deferral_reason: NONE — user reported 3rd time; structural fix not band-aid
related_goals: [G14, G6]
related_plans: [context/markdowns/plans/automation/dashboard-textarea-hmr-survival-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - A46 chamber UX (where the textareas live)
  - bug-billboard `20260527T0410-main-claude-main-dashboard-reset.md` (Layer 1+2+3 fix; this is Layer 4)
  - A19 recurrence-detection (3rd time = mandatory structural)
findings:
  - finding_id: F51-1
    severity: P0
    title: Vite/Astro HMR full-page reload wipes textarea state on every index.html edit
    evidence: Astro dev server on PID 76491 (port 4321); index.html mtime updates trigger HMR; full-page reload destroys in-memory DOM state
---

# A51 — Dashboard textarea survives HMR reload (Layer 4)

## The 3rd recurrence — root cause finally found

Prior Layer 1 (typing guard at tick) + Layer 2 (render-hash skip) + Layer 3 (in-flight value capture/restore) all addressed **intra-page** refreshes. They didn't help across **full-page reloads** triggered by Vite/Astro HMR.

Every time I edited `public/audit-timelapse/2026-05-25-overnight/index.html` (5+ times this session: port fix, A46 P1, Clear-Stuck button, etc.), Astro dev server told the browser to reload the whole page → in-memory textarea values wiped.

**That's why the user kept reporting "still jumpy."** My fixes didn't reach this code path.

## 🚨 CORRECTION (post-Read inspection)

**Layer 4 VALUE persistence ALREADY EXISTS** at lines 7127-7187: `wireMonkeyOther` reads `monkey-draft-${decisionId}` from localStorage, persists on input, clears on send. Built 2026-05-26 per user inbox at 20:14Z.

**The actual issue is NOT value loss** — it's the FULL-PAGE-RELOAD experience itself: blink, focus loss, cursor position reset, scroll jump. Value persists but the typing FLOW is destroyed.

**Real triggers:**
1. **Agent edits to `index.html`** during user typing sessions (this is the BIG one — happened 5+ times today)
2. Any other file Astro is HMR-watching in `public/`

## Real fix paths

### Fix A — Behavioral (immediate, today)
- Agent commits to NOT editing `index.html` during user-active typing sessions
- Use Edit tool only when no user typing is in progress
- All future dashboard JS changes batched at session boundaries

### Fix B — Structural (Layer 4-extended)
- **Focus + cursor + scroll persistence** to complement value persistence
- On `beforeunload` (HMR fires this): save `{activeElement.id, selectionStart, selectionEnd, window.scrollY}` to localStorage `dashboard-typing-state`
- On `DOMContentLoaded`: if `dashboard-typing-state` present + recent (<30s): restore focus to that element + cursor position + scroll
- Combined with existing Layer 4 value persistence = full state preservation across HMR
- Localized impact; ~30 lines of additional JS in `wireMonkeyOther`

### Fix C — Move dashboard out of HMR scope
- Symlink `public/audit-timelapse/2026-05-25-overnight/index.html` to a non-watched location
- Or serve it via separate static server (different port; no HMR)
- Eliminates HMR-triggered reloads entirely
- Bigger blast radius; more risky

**Recommendation: Fix A immediately + Fix B in next batch.** Fix C deferred unless A+B prove insufficient.

### Storage budget
- Max 5KB per decision × ~20 active decisions = 100KB localStorage worst case (well under 5MB limit)
- Auto-cleanup on send + on reopen
- Optional: garbage-collect drafts older than 7 days on page load

### Why this works across HMR
- HMR reload preserves localStorage (only DOM is destroyed)
- Restore runs after card render → textareas get their drafts back
- User can't tell the page reloaded

## Phase 1 — Inline foreground fix (5-min edit in dashboard JS)

Edit `public/audit-timelapse/2026-05-25-overnight/index.html`:
1. Add 4 helper functions: `mkDraftLoad(id)`, `mkDraftSave(id, value)`, `mkDraftClear(id)`, `mkDraftRestoreAll()`
2. Wire `input` event listener on `.mk-other-input` (debounced 300ms) → save
3. Call `mkDraftRestoreAll()` in `loadMonkey()` after `cardsEl.innerHTML = ...`
4. Call `mkDraftClear(decisionId)` in `onMonkeyOtherSubmit` after marking decided + in `mkOnReopenClick` (A46 reopen handler)

~50 lines of JS added. No new tools or scripts needed.

## Phase 2 — Verification

After Layer 4 lands:
- Type into a chamber textarea
- Trigger HMR by editing `index.html` (any whitespace edit)
- Browser reloads page
- Textarea value PERSISTS via localStorage restore
- ✅ user-named bug eliminated

## The meta-lesson

The user reported the same class 3 times. Each prior fix addressed a different layer:
- Layer 1+2: auto-refresh tick (Stream B + my fix earlier this session)
- Layer 3: in-flight render hash (Stream B + my fix earlier)
- **Layer 4: HMR full-page reload survival** ← THIS

The 3-layer model wasn't complete. **4 is the structural answer.**

## Cross-references

- bug-billboard `20260527T0410-main-claude-main-dashboard-reset.md` (Layers 1+2+3 fix)
- A19 recurrence-detection
- A46 Phase 1 (chamber UX redesign; respects all 4 layers)

## Status

IN PROGRESS — design landed; inline foreground fix landing this turn.

