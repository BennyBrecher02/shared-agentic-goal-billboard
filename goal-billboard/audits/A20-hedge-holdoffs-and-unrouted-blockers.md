---
audit_id: A20
title: "Hedge holdoffs + unrouted blockers — clear actions deferred when agent could act + holdoffs buried in chat invisible to user"
status: in_progress
catalogued: 2026-05-26T22:35:00Z
priority_when_run: P0
estimated_effort: |-
  medium (3 categories of fix: hedge detection + classifier + monkey routing)
trigger: |-
  2026-05-26 22:34Z — user audit of 5 declared holdoffs revealed 2 were CLEAR (ack-latency improvement attempt + inbox-archive row-by-row) and shouldn't have been deferred; the other 3 (settings patches, LaunchAgent install, monkey decisions) were genuine but BURIED in chat instead of routed to the monkey chamber where they'd be visible. User framing: *"for all the things you stopped for holding off for user input, i totally missed all that and wouldnt have known had i not asked, why isnt this in our billboard??"*
deferral_reason: NONE — running immediately. This is the natural extension of A18 (per-prompt artifact gap) + A19 (cross-system meta-monitoring). A20 catches the response-side hedge pattern.
related_goals: [G4, G6]
related_plans: [context/markdowns/plans/automation/holdoff-routing-and-monkey-expansion-plan.md]
related_refs:
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
  - context/markdowns/goal-billboard/audits/A16-agent-idle-root-cause.md
  - context/markdowns/goal-billboard/audits/A19-cross-system-meta-monitoring.md
findings: []
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'dashboard' -> serves NS
belongs_to_goal: G14
---
# A20 — Hedge holdoffs + unrouted blockers

## Why this audit matters

A16 catches IDLE (agent fully stopped). A18 catches MISSING ARTIFACTS (per-prompt). A19 catches RECURRENCE + WORKFLOW BYPASS. **A20 catches the third response-side failure: hedge-holdoffs.**

Hedge-holdoff = the agent COULD act on a clear instruction but PHRASES IT AS A QUESTION ("want me to surface row-by-row?", "let me know if you want X", "happy to do Y when ready"). This violates the standing-protocols "Auto Mode" + the user's standing instruction "act without asking." User explicit framing today: *"are there things here where its clear what to do and you just havent done it when you could?"* — answer: yes, 2 of 5.

The companion failure: **unrouted blockers** — when something GENUINELY needs user input, it gets written into a long chat response where it's easy to miss, instead of being routed to a SHARED surface (monkey chamber, goal billboard, lefter mini-block). User: *"i totally missed all that and wouldnt have known had i not asked, why isnt this in our billboard??"*

## Receipts (this session)

### CLEAR holdoffs (should have acted)

| Time | Holdoff in chat | Why CLEAR | What I did instead |
|---|---|---|---|
| 22:30Z | "want me to surface row-by-row [the 20 inbox archive entries] so you can flag which were real misses vs heuristic false-positives?" | User had asked "list all inboxes ever" twice. Implication = the WHOLE thing. Hedge phrasing = "want me to". | Asked permission. Should have just done it. |
| 22:00Z (since 21:08Z) | "Try a new best idea on how to shorten the gap" — never executed | User said "act without asking." Standing protocol says "Don't sit idle when parallel work is queued." | Treated as future-work; should have attempted in same response. |

### SAFETY / TASTE that needed routing not chat-burying

| Time | Item | Class | Was visible? |
|---|---|---|---|
| 22:30Z | 4 settings patches needing user nod | SAFETY (settings-edit-guard is deliberate) | NO — buried in chat |
| 22:30Z | Heartbeat LaunchAgent install | SAFETY (intrusive system change) | NO — buried in chat |
| various | 3 monkey-chamber decisions | TASTE (design call) | YES — already in monkey chamber ✓ |

## Root causes (response-side failures)

1. **Hedge-smell phrasings unflagged**: "want me to / let me know if / happy to / I can do X if you want / standing by for / ready when you say" — these are reflexes after pushback or in uncertain moments. None are caught by current hooks.

2. **Over-cautious post-pushback**: After user critique, the agent defaults to MORE asking, less acting. Wrong direction — user wants MORE action, less asking.

3. **Conflating user questions with input-needed**: A user question is a question; a holdoff item is a different thing. Mixing them in the same chat response loses the structure.

4. **No SHARED SURFACE for SAFETY/TASTE holdoffs**: Monkey chamber exists for design A/B/C only. Settings patches + system installs + general yes/no asks have NO designated surface — they get text-buried.

5. **Standing-protocols memory rule "don't sit idle when parallel work is queued" is REACTIVE not PROACTIVE**: only enforced at Stop hook (after response written), never PRE-RESPONSE.

## Connection to prior audits

- **A16 (agent-idle)** — catches "stopped doing anything." A20 catches "doing things but asking instead of acting."
- **A18 (artifact-creation gap)** — catches "user asked for plan; no plan landed." A20 catches "user asked for action; agent asked instead."
- **A19 (recurrence + workflow bypass)** — catches "topic repeats" and "edit without scrutiny." A20 catches "ask instead of act."

Same family. Three sibling response-side disciplines.

## Fix (designed + scaffolded same turn)

See `plans/automation/holdoff-routing-and-monkey-expansion-plan.md`. Three layers:

### Layer 1 — Holdoff classifier (decision rule, agent-side)

Every time the agent considers writing a question-flavored statement about a pending action, **classify it first**:
- **CLEAR** → ACT immediately. No question.
- **SAFETY** → route to monkey chamber (NEW category: settings-patch / system-install / risk-tradeoff). NOT chat.
- **TASTE** → route to monkey chamber (existing design-A/B/C category).
- **AMBIGUITY** → small ONE-LINE clarification + pick reasonable default + log.

### Layer 2 — Hedge-phrase detector (Stop hook scan)

Scan outgoing agent response for hedge-smell phrasings:
- "want me to"
- "let me know if"
- "happy to"
- "ready when you say"
- "I can do X if you want"
- "standing by for"
- "shall I"

Surface as `<system-reminder>` at next prompt: "Last response contained hedge: '<phrase>'. Re-evaluate: was this a CLEAR holdoff you should have acted on?"

### Layer 3 — Monkey chamber expansion + Billboard "Waiting on you" surface

- Monkey chamber gains 3 new decision categories beyond existing TASTE: `settings-patch`, `system-install`, `yes-no-customize`
- Pre-populate with 5 current holdoffs (4 settings patches + heartbeat install)
- New Billboard tab section: "🤚 Waiting on you" — table of all pending holdoffs across all categories
- New lefter mini-block: "N items need you" — count badge, collapses to icon
- Heartbeat tier-high check: if any "needs-user" item has been pending >30min, surface in next UserPromptSubmit drain

## Verification (after BG lands hooks + dashboard expansion)

1. Hedge-phrase scan over a synthetic agent response containing "want me to" → confirms detection + system-reminder
2. Holdoff classifier UI: 4 categories visible in monkey chamber + Billboard "Waiting on you" panel
3. Pre-populated settings/install decisions render with proper action options
4. Lefter mini-block shows count badge
5. Heartbeat tier-high detects stale holdoff + surfaces

## Status

IN PROGRESS. Will flip to `resolved` once:
- Hedge detection hook wired (Stop)
- Monkey chamber expansion deployed (BG launching this turn)
- Billboard "Waiting on you" panel rendering
- One full cycle: detected hedge → corrected → no recurrence in 5+ subsequent responses

## Lessons (preliminary)

- "Don't sit idle" needs a PRE-RESPONSE counterpart — outgoing-response scan for hedge smell before send.
- The monkey chamber is the right primitive — make it general-purpose for any user-input gate, not just design A/B/C.
- A SHARED surface (dashboard panel) is the only way invisible-to-user blockers become visible-to-user. Chat-buried is the same failure mode as memory-rule-only.
- Three sibling disciplines now: A18 (artifact-side) + A19 (process-side) + A20 (response-side). They cover the full response loop.

## Cross-references

- `kpi-self-eval-workflow.md` — the 6-step loop applied to ack-latency improvement attempt (executed this turn)
- A18 / A19 — sibling structural-enforcement audits
- `feedback_standing-protocols.md` — receiving hedge-detection + holdoff-routing rules
- Monkey communication chamber — being extended (was design-only; becomes general user-input gate)

