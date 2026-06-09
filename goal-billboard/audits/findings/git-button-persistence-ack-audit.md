---
title: "Git/Rollback page action controls — persistence + agent-delivery audit"
audit-id: git-button-persistence-ack-audit
subject: "Do the Git/Rollback page action buttons lose progress on tab-swap and fail to reach the agent, like the chamber cosmetic-click bug?"
date: 2026-05-31
verified-against:
  - scripts/dashboard/generate_dashboard.py   # live source, grepped + line-read
  - context/markdowns/research/systems/chamber-decision-lockin-design.md
  - scripts/services-registry.json
  - scripts/agent-inbox-server.py
verdict-class: NOT-the-chamber-bug
produced-via: bracketed-research-audit workflow
---

# Git/Rollback action controls — persistence + delivery audit

## 1. HEADLINE VERDICT

**NO — the Git/Rollback page buttons are NOT cosmetic like the chamber bug. None of the audited controls lose progress on a tab-swap, a flatten-toggle, or a full reload.**

- **Controls examined: 8** (Flag to undo · Flag to keep · per-change note · always-on feedback dataset + copy/download/clear · re-categorize badge "mark routine / needs my eye" · tray per-row untick · "show me the undo commands / copy as Cursor note" · the served-only "drop to inbox" Path-B handoff).
- **Persist correctly: 8 / 8.** Every control writes `localStorage` synchronously on interaction and is **repainted FROM that store on every render path**. This is the exact discipline whose ABSENCE defines the chamber bug. The chamber's `.dc-option` handler (`generate_dashboard.py:7332-7335`) only does `b.style.borderColor=…` + inserts a ✓ — pure DOM mutation, no store, no repaint — so a re-render wipes it. The rollback controls do the opposite.
- **Cosmetic bug: 0 / 8.** No control's body is "mutate the DOM and stop."
- **Deliver to the agent: 1 / 8 attempts it** (the served-only Path-B "drop to inbox" at L7834), and that one is **half-wired** — it POSTs to `/__rollback-handoff`, **a route no server in this repo serves** (the only matches are the emitter itself + two test files; no Python/JS handler exists). The other 7 controls are **0-POST by deliberate design**, and that is **correct, not a bug** (see §3).

**Why this is a different verdict from the chamber.** The chamber bug is a *triple* failure (no persist + no deliver + false "drafts persist" prose). The rollback page fails **none** of those: persistence is real and verified, and the 0-POST stance is the page's *stated contract*, not a broken promise. The chamber decision **is a message the agent must act on**; a rollback flag is a **private working-set selection the human applies by hand in Cursor**. Same surface family, opposite contract.

> The chamber's own dead `.dc-option` handler at L7332-7335 is still live and unchanged on the chamber page — that bug is real and remains open. It is simply **not present** on the Git/Rollback page.

---

## 2. PER-CONTROL VERDICT TABLE

All line numbers are `scripts/dashboard/generate_dashboard.py` unless noted. "Repaint-from-store" = the property the chamber bug lacks: on every (re)render the view reads the localStorage key and re-applies state, so a tab-swap / flatten-toggle / reload cannot lose it.

| # | Control | localStorage key | Persists | Delivers to agent | Cosmetic bug | Evidence (file:line) |
|---|---------|------------------|:--------:|:-----------------:|:------------:|----------------------|
| 1 | **Flag to undo** (per-card checkbox `[data-flag]`) | `dash-rollback-flagged` | ✅ yes | ❌ no — *by design* | ❌ no | Wired at `ccWireVerdictScope` (7994-7997) → `ccApplyVerdict(…, 'undo', …)` (7969) → `setFlagged(sha,on)` writes the key synchronously. **Repaint-from-store:** `ccWireCards` reads `loadFlagged()` on every (re)render (8025-8029) + initial HTML bakes the checked state from `isFlagged()`. Survives tab-swap (showPage caches DOM, 7315), flatten swap (re-runs `ccWireCards`), reload (re-baked). |
| 2 | **Flag to keep** (per-card checkbox `[data-keep]`) | `dash-rollback-keep` | ✅ yes | ❌ no — *by design* | ❌ no | Wired at 7999 → `ccApplyVerdict(…, 'keep', …)` → `setKept(sha,on)`. Mutually exclusive with undo (7973-7976). Repaint-from-store identical to #1 via `loadKept()` (8022,8028). Records a positive judgment into the local feedback dataset; nothing for the agent to act on. |
| 3 | **Per-change note** (free-text `[data-note-for]`) | `dash-rollback-notes` | ✅ yes | ❌ no — *by design* | ❌ no | Wired 8001-8008: debounced `input` (350ms) + `blur` → `setNote(commit,inp.value)` + `touchFeedbackNote`. Repaint-from-store: initial HTML bakes `value="${esc(note)}"` from `getNote()`; note-field open state restored in `ccApplyFlagState` on every render. Placeholder "saved with your decision" is backed by a real write — claim-honest. |
| 4 | **Feedback dataset** (always-on `recordFeedback` on each verdict) + **copy-JSONL / download-.jsonl / clear** | `dash-rollback-feedback` | ✅ yes | ❌ no — *export-only by design* | ❌ no | `recordFeedback` writes the key on each tick-ON; `dropFeedback` on untick. `ccUpdateFeedbackBar` repaints the live count/preview FROM `loadFeedback()` on every render. copy/download emit `feedbackJSONL()`; clear does `localStorage.removeItem`. Comment L4920-4921: *"0-runtime-fetch holds: localStorage + a client-side export only; nothing is ever POSTed to a backend."* The user keeps the JSONL — claim-honest. |
| 5 | **Re-categorize badge** ("mark routine / needs my eye" `[data-recat]`) | `dash-rollback-categories` | ✅ yes | ❌ no — *by design* | ❌ no | Two wire sites (page-level + relocated-detail) both `saveCatOverrides(ov)`; back-to-default deletes the key. Repaint-from-store: initial badge baked via `ccEffectiveCategory(sha,baseCat)` reading `loadCatOverrides()`; `ccUpdateCardBadge` is only the in-session DOM sync after the persist. Survives reload. Purely a local view re-categorization. |
| 6 | **Tray per-row untick** (`[data-ft-untick]`, delegated on `#flag-tray`) | `dash-rollback-flagged` | ✅ yes | ❌ no — *by design* | ❌ no | Delegated click → `setFlagged(sha,false)` + `dropFeedback` (if not kept) + repaint. **Delegated** on the tray so it survives `ccUpdateTray()` repaints; the tray is itself repainted FROM the store by `ccUpdateTray` reading `flaggedShasOrdered()`→`loadFlagged()`. Persists across reload. |
| 7 | **Show me the undo commands / copy-cmds / copy as Cursor note** (`[data-ft-show-cmds]`, `[data-copy-flagged-cmds]`, `[data-copy-flagged-md]`) | reads `dash-rollback-flagged` (no own key) | ✅ yes (reads the durable set) | ➖ N/A — *the delivery channel itself* | ❌ no | Builds the `git revert <sha>` block + Cursor-ready markdown from `flaggedShasOrdered()` and copies to clipboard (7826-7829). **This IS the page's intended delivery rail** — the human's hand carries the block into Cursor. Not a control that should POST. |
| 8 | **"Drop to inbox" (Path-B handoff)** (`[data-drop-flagged-inbox]`, served-mode only) | — | ➖ n/a (transient action) | ⚠️ **attempts, but DEAD** | ❌ no (not cosmetic — it tries) | L7834: `fetch('/__rollback-handoff', {method:'POST', …, body: md})`. **No server in the repo serves `/__rollback-handoff`** (only this emitter + 2 test files reference it; no Python/JS route handler). On the live dashboard the `.catch` just sets the button to "writer unavailable" — so it fails *gracefully and visibly*, not silently. This is the one genuine delivery gap, and it is **honestly surfaced**, unlike the chamber's silent loss. |

**Net:** 8/8 persist, 0/8 cosmetic. The single delivery weakness (#8) is an unwired endpoint that fails *visibly*, categorically unlike the chamber's silent-loss class.

---

## 3. Why "delivers = false" is CORRECT here (the reframe)

The page's contract is written into the source as a deliberate safety stance, not an oversight:

- L4838-4846: *"KEEPING is the DEFAULT… Ticking it does NOTHING to the code… The page NEVER executes git; there is no 'do it now' anywhere."*
- L4920-4921 & L6577: *"0-runtime-fetch holds: this is localStorage + a client-side export only; nothing is ever POSTed to a backend."*

A rollback flag is **not** a "do this now" instruction. The whole reason the page exists is that the human curates a shortlist and applies the `git revert` themselves in Cursor. Forcing every flag to POST like a chamber click would **break the page's reason to exist** and contradict its safety stance. So the audit verdict is: **0-POST on controls 1-6 is by-design-correct; the only true gap is the dead `/__rollback-handoff` route behind the opt-in Path-B button (#8).**

---

## 4. FIX-PLAN

**Scope is far smaller than "make every button POST."** Two corrections to the originating brief, both verified against source:

1. **The brief's "port 4322" is the documented footgun.** `services-registry.json` (`_notes`, L22) and `agent-inbox-server.py` (docstring L11-14) both state the canonical inbox port is **4323**; **4322 is the dead pre-A45 default that "silently lost monkey-chamber clicks on 2026-05-27" and must never be reintroduced.** All work below uses **4323**.
2. **The brief's premise that "the chamber-lockin inbox queue already physically lives in `generate_dashboard.py` (blob L6058…)" is FALSE.** The chamber-lockin design doc is explicitly `status: design / plan-only — DO NOT build` (doc L6), and a grep of the generator for `INBOX_API` / `inboxEnqueue` / `inboxDrainQueue` / `localhost:4323` / `agent-inbox-queue` returns **zero hits**. Those "blob L6xxx" citations point at the design doc's *proposed* code, not landed code. The `scripts/agent-inbox-server.py` sidecar (real `POST /agent-inbox` at L257) **does** exist; the dashboard-side client transport **does not yet**.

So "ONE shared persist+deliver layer, not two parallel ones" is achieved by **building the chamber-lockin client transport once** and making the rollback page its **second consumer** — but it is a *build*, gated on the chamber-lockin greenlight, not a free reuse.

### 4a. The shared persist+deliver layer (build once in `generate_dashboard.py`)

Land the client transport block the chamber-lockin doc specifies (doc L254-265, §"Port the inbox subsystem"), inside the single `<script>` (`SCRIPT = r"""…"""`, L3137):

- Constants: `INBOX_API='http://localhost:4323/agent-inbox'`, `INBOX_QUEUE_KEY='agent-inbox-queue'`, `INBOX_MAX_ATTEMPTS=30`, `INBOX_RETRY_INTERVAL_MS=5000`. **Port 4323.**
- Offline-first queue: `inboxLoadQueue` / `inboxSaveQueue` (localStorage), `inboxEnqueue(message,priority,tag)`, `inboxSendOne`, `inboxDrainQueue`, a 5s retry `setInterval`, and a queue-status bar with "retry now" / "clear stuck".
- This is the SAME layer the chamber decisions use (`tag:'monkey-decision'`). The rollback page becomes the **second consumer** with its own tag (`tag:'rollback-handoff'`).

### 4b. What actually changes on the rollback page (minimal)

Keep controls **1-7 local-only** — their 0-POST contract is correct (§3). Only **two** delivery surfaces change, both **opt-in and explicit** so the silent-curation model is preserved:

- **Control #8 — replace the dead `fetch('/__rollback-handoff')` with `inboxEnqueue(md,'normal','rollback-handoff')`.** This is the single concrete bug-fix: today it POSTs to a route nothing serves. Routing it through the shared queue gives it a real destination (the :4323 sidecar), durable retry, and a visible queued/failed state — replacing "writer unavailable" with a true offline-safe enqueue. **Priority `normal`, not `urgent`:** a rollback handoff is a review note, not an act-now command — do not spam the urgent fan-out.
- **Optional second sink for the feedback dataset (#4):** add an opt-in "also send a copy to the feedback sink" affordance that enqueues the JSONL via the same layer. Default stays **export-only**; the silent local dataset is untouched unless the user opts in. (Lowest priority; ship #8 first.)

### 4c. The G6 ack tie

The shared layer's POST target — `agent-inbox-server.py`'s `POST /agent-inbox` (L257) — already parses incoming messages (`_parse_monkey_decision` L115) and spawns the fan-out (`_spawn_monkey_fanout` L202). The rollback handoff rides the **same inbox → G6 ack loop** the chamber decisions feed: enqueue → sidecar files it → G6 picks it up → ack flows back. **No server-side change required** for the rollback consumer; it reuses the existing endpoint. (Add a `rollback-handoff` branch to the parser only if rollback notes need distinct routing from monkey-decisions — otherwise the generic path suffices.)

### 4d. Server-down fallback

This is the whole point of routing through `inboxEnqueue` rather than a bare `fetch`:

- On POST failure (sidecar down / wrong port / offline), `inboxEnqueue` **writes the message to `agent-inbox-queue` in localStorage** — durable across reloads.
- The 5s `setInterval` `inboxDrainQueue` retries up to `INBOX_MAX_ATTEMPTS` (30); the queue-status bar shows pending count + "retry now" + "clear stuck."
- **Nothing is silently lost** — the exact failure mode that killed chamber clicks on 4322. The user always sees queued-vs-sent state. (And because the persist layer (controls 1-7) is independent localStorage, a dead sidecar **never** costs the user their flagged set — only the optional handoff delivery waits in the queue.)

### 4e. Touch-points

| File | Change |
|------|--------|
| `scripts/dashboard/generate_dashboard.py` | (1) Add the shared inbox transport block (constants + queue + retry + status bar) inside `SCRIPT` (L3137+), **port 4323** — per chamber-lockin doc L254-265. (2) L7834: swap dead `fetch('/__rollback-handoff')` → `inboxEnqueue(md,'normal','rollback-handoff')`. (3) Optional: opt-in feedback-sink enqueue near `ccUpdateFeedbackBar`. (4) Wire the chamber's `onMonkeyOptionClick` to the SAME layer when the chamber-lockin work lands (replaces the dead L7332-7335 cosmetic handler) — that is the shared-layer's first consumer. |
| `scripts/agent-inbox-server.py` | No change required (reuses `POST /agent-inbox` L257). Optional: add a `rollback-handoff` tag branch in `_parse_monkey_decision` (L115) only if distinct routing is wanted. |
| `scripts/services-registry.json` | No change (already canonical 4323). |
| `context/markdowns/research/systems/chamber-decision-lockin-design.md` | Note that the rollback page is now a **second consumer** of the planned shared layer (informs the §4 decoupling currently held for user greenlight, per commit 420bfaf). |

### 4f. Sequencing / gating

The shared transport is **gated on the chamber-lockin greenlight** (doc L6 `DO NOT build`; commit 420bfaf holds §4 decoupling for user greenlight). Recommended order:
1. **Now, ungated:** the `/__rollback-handoff` route is dead today — either (a) wire a `served`-mode writer for that endpoint as a stopgap, or (b) fold #8's fix into the shared-layer build. Prefer (b) to avoid building throwaway transport.
2. **On greenlight:** build the shared layer once; make chamber decisions consumer #1 (kills the L7332-7335 cosmetic bug) and rollback handoff consumer #2 in the same pass. This satisfies the brief's "ONE shared layer, not two."

---

*Produced via the bracketed-research-audit workflow.*
