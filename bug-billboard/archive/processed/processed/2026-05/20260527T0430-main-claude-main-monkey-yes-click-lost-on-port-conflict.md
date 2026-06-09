# Monkey YES button silently loses clicks when inbox server unreachable + port 4322 squatted (FIXED + PREVENTED)

- **id:** 20260527T0430-main-claude-main-monkey-yes-click-lost-on-port-conflict
- **discovered:** 2026-05-27T04:30:35Z
- **agent:** claude-main
- **worktree:** main
- **route:** dashboard /audit-timelapse/2026-05-25-overnight/ — monkey panel
- **section:** monkey card "YES/NO/DEFER" option-button click
- **viewport-bucket:** all
- **severity:** CRITICAL
- **status:** done
- **defect-type:** interactive-broken
- **dedupe-key:** monkey-chamber|option-click|no-offline-queue|all

## Description

User reported (2026-05-27T04:10ish): pressed YES on SET-003 monkey decision and got `POST failed: http 404. Is agent-inbox-server.py running on port 4322?`. User had been doing "Other" text entries for the entire backlog — those queued safely (offline path uses localStorage). The first single button-click was lost.

## Two-part root cause

1. **Port conflict**: Port 4322 is occupied by a second Astro dev server (PID 63944 = `node_modules/.bin/astro dev`). The agent-inbox-server.py was NOT running. Astro returns 404 on `/agent-inbox` because it doesn't know that route. Status 404 ≠ network error → fetch resolves with !res.ok rather than throwing → the offline-queue fallback never triggered.

2. **Asymmetric offline-queue coverage**: `wireMonkeyOther` / `onMonkeyOtherSubmit` (text-entry path) used `inboxEnqueue` + localStorage. `onMonkeyOptionClick` (button-click path) did NOT — it just threw on failure. **An asymmetry the user couldn't see until they happened to click a button while the server was down.**

## Two-part fix (both landed 2026-05-27T04:30Z)

### Fix 1 — Non-destructive port migration (4322 → 4323)
- Started `AGENT_INBOX_PORT=4323 python3 scripts/agent-inbox-server.py` in background
- Verified POST works: `HTTP 200 · filename 20260527T043013Z-e8azc.md`
- Updated 3 dashboard constants in `index.html` (INBOX_API + INBOX_RECENT_API + MK_INBOX_API) from `:4322` → `:4323`
- Server log at `/tmp/agent-inbox-server-4323.log`
- CORS-allow remains `http://localhost:4321` — same dashboard origin, different backend port = still works

### Fix 2 — `onMonkeyOptionClick` now uses the same offline queue as `onMonkeyOtherSubmit`
- On any failure (404, 5xx, network error), enqueues to localStorage via `inboxEnqueue(message, 'urgent', 'monkey-decision')`
- Calls `inboxDrainQueue()` immediately + `inboxRenderAll(null)` + `inboxUpdateQueueBar()` to refresh UI
- Marks card decided (with offline status row) — so user sees the click WAS captured
- Status row shows: `⧗ server unavailable — click saved locally and will deliver when it comes back: <option>`
- Only hard-fails if localStorage itself is unavailable

## Verification

After user reloads dashboard:
1. Dashboard fetches recent inbox at new port 4323 → succeeds
2. Retry loop notices server is back → drains all queued "Other" entries from localStorage
3. Each queued message becomes a file in `context/markdowns/agent-inbox/`
4. User re-clicks SET-003 YES (the one lost click) → POST works directly OR queues if interrupted

## What was lost vs preserved

| Item | Status |
|---|---|
| All user-typed "Other" entries (visible "server is offline — saved locally and will deliver" status) | ✅ SAFE in browser localStorage; drains on reload |
| SET-003 YES button click (single button-click that errored) | ❌ Lost — no queue at time of click; needs re-click |
| Future clicks during server-down moments | ✅ Now queue automatically per Fix 2 |

## Cross-references

- Task #25 (Dashboard offline-first inbox queue) — offline queue infra; A40 audit confirms substrate-built-but-not-used pattern (queue existed but `onMonkeyOptionClick` didn't use it)
- Task #42 (Add 'Other' option + text input) — proper offline path implementation
- A40 idea-gap detector — this exact pattern: "feature built but only partial code paths use it"
- INSTALL-001 (heartbeat LaunchAgent opt-in) — adjacent pattern: agent-inbox-server.py also wants a launch-on-boot mechanism so this doesn't recur after restarts
- A36 BG-stuck-warn hook — companion: detects another class of "thing should be running but isn't"
