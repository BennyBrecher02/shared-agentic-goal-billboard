---
audit_id: A40
title: Idea-gap detector + recurring conversation sweep — never-again the "I offered an idea and it slipped" fear
status: in_progress
catalogued: 2026-05-27T03:47:54Z
priority_when_run: P0
estimated_effort: large
trigger: |-
  2026-05-27T03:47Z — user explicit: *"also mine it for gaps of requests never truly serviced, maybe set up hooks to check a window of our convo every so often so i can never again have to worry about an idea i offer not getting picked up by our system. this is the biggest slam dunk ive had in a while maybe."* USER LABELED THIS AS A SLAM-DUNK SEED — explicit calibration data for A39 retrospective.
deferral_reason: NONE — structural fix to a named recurring fear; running immediately
related_goals: [G14, G4]
related_plans: [context/markdowns/plans/automation/idea-gap-detector-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - reports/conversation-extracts/ALL-PROMPTS.md (substrate from A38 — 296 prompts)
  - A18 audit + user-ask-detect.sh / user-ask-artifact-check.sh (predecessor; catches WITHIN-session; A40 catches CROSS-session)
  - A19 audit (state-substrate-read + recurrence detection; sibling family)
  - A39 (slam-meter retrospective; A40 is the gap-detection cousin)
  - context/markdowns/goal-billboard/audits/findings/adaptive-immunity-retro-sweep-roadmap-2026-05-28.md (A74-Z — found A40 biology-only; this build is the response)
findings:
  - "2026-05-28T21:26Z — A40 is NO LONGER biology-only. The SW tether now EXISTS on disk: scripts/detect-idea-gaps.py (the gap detector) + tests/idea-gap-test.sh (A71-shaped VERIFY, 6/6 assertions pass). A74-Z's roadmap finding (A62's shipped-claim was aspirational) is now closed for the SOFTWARE tether. Remaining to reach 4/4: HW (Stop-hook that blocks session-end with an un-surfaced stale gap) + the SessionStart surface hook (A40 Phase 2). Those live in scripts/hooks/ (other concerns own that zone) — NOT built here."
  - "2026-05-28T21:26Z — Detector validated against REAL sessions. Newest session (666b8e4c): 0 gaps in last 20 prompts (artifact-heavy plow; closure-signal distribution 19 artifact_landed / 5 substantive-answer / 1 explicit-skip — discriminating, not rubber-stamping). A conversational session (b92834fe): 3 genuine idea-handoff gaps surfaced (e.g. 'why dont we integrate tasks into the os heartbeat system? flesh this out' got no artifact in its 5-prompt window) — proof the detector finds real IDEA-SLIP instances, not just always-zero. One false positive (<task-notification> harness injection) was caught + filtered during validation."
  - "2026-05-28 (adaptive-immunity-closure workflow) — The SURFACE TETHER now EXISTS: scripts/hooks/idea-gap-surface.sh (SessionStart hook that runs the detector + emits a WARN <system-reminder> naming the gap count + idea-handoff sub-count + the idea-gaps.jsonl substrate when gaps are found; SILENT when clean). WARN-mode by construction (SessionStart hooks cannot block — they only inject context, so there is no BLOCK path to flip). Kill-switch EVIUM_IDEA_GAP_SURFACE_OFF=1; exit 0 ALWAYS; full EVIUM_IDEA_GAP_* test-override surface so the VERIFY sandboxes without touching real state. VERIFY: tests/idea-gap-surface-test.sh — A71-shaped, fully sandboxed, 10/10 pass. Proven non-vacuous: with the hook hidden the test exits 1 ('MUTATION NOT APPLIED'); with it present, exit 0. Assertion 1 reproduces the OLD silent-slip (gap in substrate but no user-facing surface); Assertion 2 proves the hook breaks the silence. LIVE-FLIP REMAINING (user-gated): wire idea-gap-surface.sh into .claude/settings.json hooks.SessionStart — confirmed NOT YET wired (grep of settings.json = no match), so it is staged + tested but does not run automatically until the user enables it."
---

# A40 — Idea-gap detector + recurring conversation sweep

## The user's fear (named verbatim)

*"i can never again have to worry about an idea i offer not getting picked up by our system."*

This is the most direct articulation of the recurring fear that A18 / A19 / A20 / A21 / A29 each chased one shape of. A40 is the **structural anti-fumble** — once it's wired, the user STRUCTURALLY cannot have an idea slip without the system surfacing the gap.

## What "serviced" means (the gap definition)

A user prompt is **serviced** if AT LEAST ONE of these is true within 5 prompts after it:
1. **Artifact landed** — a new file (audit / plan / skill ref / hook / memory rule / code change) was written
2. **Explicit skip logged** — a steering-log entry says "skipped because X" with rationale
3. **Explicit deferral** — a steering-log entry says "deferred to <trigger>" with rationale
4. **User affirmation** — user's next prompt confirms the response was sufficient ("yes proceed", "thanks", "good")

A prompt is a **GAP** if NONE of the above happens. The user offered the idea; the system didn't pick it up; no explicit reason was logged.

## The mechanism (high-level)

```
[user prompt]
    ↓
[A40 detector reads next 5 prompts' span]
    ↓
[for each user prompt, classify: serviced | gap | unknown]
    ↓
[gap candidates emit to gaps-pending.jsonl]
    ↓
[SessionStart hook surfaces pending gaps to user]
    ↓
[user responds: address-now | defer-to | mark-ignored]
    ↓
[gap is closed (one of the 4 serviced-paths) OR moved to explicit-defer]
```

## Phases

### Phase 1 — Build the gap-detector script (foreground; ~30 min)

`scripts/detect-idea-gaps.py`:
- Reads `reports/conversation-extracts/ALL-PROMPTS.md` (A38 substrate)
- For each prompt, finds the 5-prompt window after it
- Cross-references against:
  - Filesystem `find` for files modified between prompt-1 mtime and prompt-6 mtime
  - Steering log entries with timestamps in that window
  - User's next prompts for affirmation markers ("yes", "thanks", "good", "proceed")
- Classifies each historical prompt: serviced | gap | unknown
- Outputs:
  - `reports/audit-findings/A40-historical-gaps.md` — list of historical GAPS with prompt# + timestamp + classification reason
  - `reports/idea-gaps/gaps-pending.jsonl` — append-only queue of currently-unresolved gaps

### Phase 2 — Wire as SessionStart + on-demand hook

`scripts/hooks/idea-gap-surface.sh` (SessionStart):
- Reads `reports/idea-gaps/gaps-pending.jsonl`
- If ≥1 unresolved gap: surface a `🚨 N idea gap(s) pending from last N sessions:` block in SessionStart digest
- Each gap shows: prompt# / 1-line summary / when offered / why no artifact landed
- User can: `gap-close <N>` (mark as serviced), `gap-defer <N> <trigger>` (move to defer), `gap-ignore <N> <reason>` (mark won't-do with rationale)

`scripts/idea-gap-close.sh` — CLI for the above ops. Each emits steering log entry to maintain audit trail.

Wire goes into `scripts/hooks/settings-patches/A40-gap-surface-wire.json` for SET-005-style batch.

### Phase 3 — Live in-session detection (PostToolUse on Stop)

`scripts/hooks/idea-gap-live-check.sh`:
- On every Stop, look back at last 5 user prompts in current session JSONL
- For each, apply gap classification
- If gap detected on a prompt OLDER than 3 turns ago AND not yet flagged: emit `🚨 in-session gap candidate:` warning
- Doesn't block; just surfaces

### Phase 4 — Recurring weekly sweep

Optional: cron-like trigger (post-shutdown bundle analyzer extension):
- Once per day OR on every graceful shutdown, run `detect-idea-gaps.py` against newly-accumulated prompts
- Surface any new gaps in next session's SessionStart
- Trend tracker: per-session gap-rate (gaps / total user prompts) — KPI for master writeup

### Phase 5 — User-facing dashboard panel

Optional: add "Idea Gaps" panel to the billboard dashboard:
- Live list of pending gaps
- Click to drill into the prompt + spawned-artifacts (or non-)
- Manual close / defer / ignore controls

## The gap classification difficulty

Some prompts are TRULY questions, not idea-handoffs — those don't need an "artifact." Need to distinguish:
- **Question** ("how does X work?") — serviced by a textual answer; no artifact required
- **Status check** ("where are we on X?") — serviced by a status response
- **Decision** ("X or Y?") — serviced by user picking
- **Idea handoff** ("flesh out X") — REQUIRES artifact landing per A39 workflow
- **Tangent** ("by the way, X") — depends on whether tangent calls for action

The detector uses A39's trigger-detection rules as the primary "idea handoff" classifier. Only idea-handoff-classified prompts NEED artifact landing.

For other types, the detector verifies a TEXTUAL response was given (response not empty). The user can still flag "this question wasn't well-answered" but that's separate (handled by future calibration).

## What this catches (concrete examples from past sessions)

Phase 1 BG will produce the actual list. Expected pattern (from memory):
- "make a new skill called X" type prompts where I wrote the skill ref but never wired discovery
- "we should also do Y" tangent that I acknowledged but never created an audit for
- "this should integrate with Z" cross-link asks I forgot to honor
- "remember to do A later" deferrals where the deferral wasn't logged

## Why this matters

The user's exact words: *"i can never again have to worry."* The system today makes them worry because they have to manually track whether each idea got picked up. The structural fix moves that tracking to the system. **Worry-elimination is the user-visible promise.**

This audit is the COMPLETION of the A18-A29 family:
- A18 catches trigger-phrase asks within-session
- A19 catches state-substrate gaps
- A20 catches hedge holdoffs
- A21 catches parallel-capacity floors
- A29 catches lazy regression
- **A40 catches all of them cross-session, retroactively, with explicit user closure ops.**

## Cross-references

- A18 — within-session ask detection (artifact-discipline hook); A40 extends cross-session
- A19 — state-substrate inline reads; A40 closes the cross-session gap
- A29 — output-side discipline (slam-meter cousin)
- A38 — substrate (extracted prompts); A40 consumes
- A39 — trigger detection rules; A40 uses for classification
- G14 — master audit owner (Phase 4 surfaces gap-rate KPI)
- G11 / A34 — session bundle (Phase 4 sweep integration point)

## Lessons (preliminary)

- The user calling this "biggest slam dunk in a while maybe" is itself a labeled training example for A39's retrospective. EVERY future "this is a slam dunk" claim should be checked against A39's meter.
- The fear is real and named. The fix has to be structural, not behavioral. Behavioral promises ("I'll be more careful") never closed the A18-A29 family. Hooks + scripts will.
- The substrate (JSONLs at `~/.claude/projects/`) was always there. We just never queried it for gaps. **Built-but-not-queried** is a sub-pattern of build-vs-use.

## Status

IN PROGRESS — **Phase 1 detector BUILT + VALIDATED (2026-05-28T21:26Z); Phase 2 SessionStart surface hook BUILT + VERIFIED (2026-05-28, adaptive-immunity-closure workflow).** All three software/biology tethers + both VERIFY tests are in place; the only remaining step is the user-gated live-flip (wiring the surface hook into settings.json — see "LIVE-FLIP" below).

### What now exists on disk (the SW tether — A40 is no longer biology-only)

- **`scripts/detect-idea-gaps.py`** — reads the recent session transcript (`~/.claude/projects/<project>/*.jsonl`) read-only; classifies the last-N user prompts against the 4 closure paths within a K-prompt window; emits gaps to `.claude/cache/idea-gaps.jsonl` + a human summary; true-time discipline (system clock); exit 0 always. Idea-handoff prompts (A39 trigger vocabulary) REQUIRE an artifact / explicit skip / explicit deferral; non-handoff questions are closed by a substantive textual answer (path-4b). Flags: `--transcript` / `--last-n` / `--window` / `--steering-log` / `--cache-dir` / `--artifact-root` / `--no-write` / `--json` / `--quiet`.
- **`tests/idea-gap-test.sh`** — A71-shaped VERIFY, fully sandboxed (synthetic transcripts + synthetic steering log + synthetic artifact root; never touches real state). 6 assertions: serviced-by-artifact / **the GAP reproduction** / serviced-by-affirmation / serviced-by-steering-skip / non-handoff-question / substrate-write-integrity. **Confirmed it FAILS if the detector misses a gap** (mutation check: a sabotaged detector that never flags gaps makes assertions B+F fail) — a genuine VERIFY, not a rubber stamp. Current run: **6/6 pass**.

### Validation against real sessions (2026-05-28T21:26Z)

- Newest session (`666b8e4c`): **0 gaps** in last 20 prompts. Closure-signal distribution: 19 artifact_landed / 5 substantive_answer / 1 explicit_skip_logged — discriminating, consistent with this session's artifact-heavy plow (steering log shows multiple artifacts per turn).
- Conversational session (`b92834fe`): **3 genuine idea-handoff gaps** surfaced — including *"why dont we integrate tasks into the os heartbeat system? flesh this out"* which got no artifact in its 5-prompt window. Proof the detector finds real IDEA-SLIP instances.
- One false positive found + fixed during validation: `<task-notification>` harness BG-completion injections arrive as `type==user` records; added to the noise filter (475+ occurrences in the corpus).

### Tether status (updated 2026-05-28 — adaptive-immunity-closure workflow)

| Tether | State | Evidence |
|--------|-------|----------|
| **BIO** | ✅ | `feedback_never-miss-an-idea.md` + this audit + `idea-gap-detector-plan.md` |
| **SW** (detector) | ✅ | `scripts/detect-idea-gaps.py` + `tests/idea-gap-test.sh` (6/6) — validated against real sessions |
| **SW/Surface** (the surfacing layer) | ✅ | `scripts/hooks/idea-gap-surface.sh` (SessionStart) + `tests/idea-gap-surface-test.sh` (10/10) — built + VERIFY-tested in this task |
| **VERIFY** | ✅ | both A71-shaped, sandboxed, proven non-vacuous (fail when mutation absent) |
| **HW / live enforcement** | ⏳ **user-gated** | the surface hook is built + tested but **NOT yet wired** into `.claude/settings.json` |

**The surface hook (A40 Phase 2 — `idea-gap-surface.sh`) is now BUILT + VERIFIED.** It runs the detector and surfaces any GAPS as a WARN `<system-reminder>` ("N user idea(s) may have slipped — see idea-gaps.jsonl"); it is SILENT when the transcript is clean. By construction it is WARN-only: a SessionStart hook cannot block — it only injects context — so there is no BLOCK/kill path to flip. The remaining live-flip is purely the WIRING.

### LIVE-FLIP — the single user-gated action that remains

To flip A40 from "built + tested, dormant" to "live enforcing," add `scripts/hooks/idea-gap-surface.sh` to `.claude/settings.json` under `hooks.SessionStart`. This is **deliberately NOT done in this task** — wiring a SessionStart hook that surfaces text mid-onboarding could annoy the user, so the user decides when to enable it (consistent with the adaptive-immunity discipline that live enforcement is user-gated). Until wired, the hook exists + passes its VERIFY but does not run automatically.

Suggested settings.json wiring (for the user to apply when ready):

```jsonc
// under "hooks": { "SessionStart": [ { "hooks": [ ... add: ... ] } ] }
{ "type": "command", "command": "bash \"$CLAUDE_PROJECT_DIR/scripts/hooks/idea-gap-surface.sh\"" }
```

Kill-switch after wiring: `export EVIUM_IDEA_GAP_SURFACE_OFF=1` (silent short-circuit, no detector run).

### Still genuinely future work (NOT this task's scope)

- **Stronger HW tether** — a Stop-hook (`idea-gap-live-check.sh`, A40 Phase 3) that won't let a session end with an un-surfaced gap older than N turns. The SessionStart surface is the Phase-2 leading-edge; the Stop-hook is the trailing-edge backstop. Lives under `scripts/hooks/` — a different file zone owned by other concerns.
- **Close/defer/ignore CLI** (`scripts/idea-gap-close.sh`) — the surface reminder references it; not built here.

