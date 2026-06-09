# Audit catalog

A peer to `active/`, `paused/`, and `archived/` inside the goal billboard. Holds **audit definitions** — reflective passes that exist as ready-to-run units we can pick up any time a concern arises.

## Mental model

This is **not** a queue (FIFO, prioritized waiting line). It's a **catalog** — a flat library of named, scoped audits, each one a complete recipe for "if you want to look at X, here's what to look at + when to bother."

Random access. Manual trigger. Any agent or the user can pick any entry at any time. The `trigger:` field describes when an audit *naturally becomes relevant*, but it's a suggestion, not a wait condition.

## What goes in the catalog

An audit is a **scheduled self-reflection** — a check whose value is "did we actually do X correctly across the system?" rather than "build feature X." Examples:
- Did our agentic-vs-scripted decisions hold up after more sessions of data?
- Are our high-risk untested scripts still high-risk, or did they get tested incidentally?
- Are our hooks earning their keep, or do some never fire?
- Are CSS values drifting away from design tokens?

## Distinct from reactionary skills

**Reactionary skills** = event-triggered. They auto-fire when a run group completes (audit, matrix, scheduler tick, etc.). They turn raw output into learning.

**Catalog audits** = on-demand. They sit in the catalog, ready. Picked up when:
- The user explicitly says "audit X"
- An agent notices a concern that matches an existing audit
- A specific trigger condition in the audit's frontmatter is met
- Reflective-pass time arises (rare; usually event-driven is better)

## File schema

```yaml
---
audit_id: A<N>                       # A1, A2, ... monotonic
title: Short noun-phrase title
status: available                    # available | in_progress | completed | verified_complete | cancelled
catalogued: <ISO 8601 UTC>
priority_when_run: P0 | P1 | P2      # how urgent it is WHEN someone picks it up
estimated_effort: small | medium | large
trigger: Concrete condition where this audit naturally becomes relevant (advisory, not a gate)
deferral_reason: Why we're not running it RIGHT NOW (also free-text)
related_goals: [G1, G2, ...]
related_plans: [path1, ...]
related_refs: [path1, ...]
serves_northern_star: G<N>           # What current Northern Star this audit clears the path for. Default: G2 (current NS). Explicit when serving a different NS or pure infra.
serves_guiding_light: G<N>           # Optional. What Guiding Light track this serves. Goals can have multiple GLs; an audit picks the most relevant one.
---

# Title

## Why this audit matters
1-3 sentences.

## What it would look at
Bulleted scope.

## Expected outputs
What would land if we ran it.

## How to trigger
Restate the natural triggers, but ANY agent/user can pick this up at any time.
```

### Northern Star context fields

Two newer (2026-05-26) fields propagate strategic context across artifacts:

- **`serves_northern_star: G<N>`** — what current Northern Star this audit clears the path for. Default is the **current NS** (today: G2). Explicit when the audit serves a different NS (e.g. pure infrastructure → use the goal that NS work blocks on).
- **`serves_guiding_light: G<N>`** — optional. Which Guiding Light track this audit advances. Goals can have multiple GLs; an audit picks the most relevant one.

Rule of thumb: if you can't decide, default to `serves_northern_star: G2` (current NS) and skip `serves_guiding_light:` — the linter is informational, not blocking.

See `.claude/skills/agentic-quality-discipline/references/northern-star-context-propagation.md` for migration tools, linter, and full convention.

## Lifecycle

```
audits/A<N>-…md (status=available)
  ↓ user/agent picks it up
audits/A<N>-…md (status=in-progress)
  + audits/findings/<audit-name>-<date>-phase<N>.md  ← findings docs land here
  ↓ audit completes
archived/audits/A<N>-…md (status=completed)
  + audits/findings/...md  ← findings stay co-located for traceability
  OR
archived/audits/A<N>-…md (status=cancelled, with reason)
```

**Findings docs** for audits-in-progress live at `audits/findings/<name>-<date>-phaseN.md`. Date in filename IS the right call for findings docs because they're time-scoped artifacts — a re-run produces a new file, not an edit. Keep `notes/` reserved for evolving analysis; put audit findings here in the catalog tree.

## How agents should use this

### At SessionStart
The `goal-billboard-status.sh` hook lists available audits. Read the count + titles. **Don't auto-run any.**

### When the user says "let's audit X"
1. Check the catalog for an existing entry matching X. If yes, run it.
2. If no, propose adding one. **Don't add unilaterally** — user owns the catalog.
3. If user approves, write the file in `audits/` with all the metadata, then either run now or leave for later.

### When you notice a trigger condition is met
The audit's `trigger:` field describes natural relevance. If something matches (e.g., 20+ steering log entries since A1 was catalogued), surface it: "trigger met for A1 — run now?"

### When you finish running an audit
1. Append a `## Lessons` section to the body with key findings + recommendations.
2. Update `status:` to `completed` — or `verified_complete` if it cleared the A62 verify-gate (see `_README.md`). **Both are valid terminal "done" states:** `completed` = done/delivered; `verified_complete` = done AND passed the rigorous adaptive-immunity gate. **Close via the wrapper:** `bash scripts/audit-close.sh <N> --completed` (plain done-tier) or `bash scripts/audit-close.sh <N> --verified --verify-test=<path>` (earned tier — refuses without a `verify_test`).
3. Move the file to `archived/audits/`.

## When NOT to catalog an audit

- A one-off question → just ask + answer; don't catalog
- An event-triggered concern → use the reactionary layer
- A bug → use `bug-billboard/`
- Pure speculation → just a `notes/` entry

## Convention origin

Created 2026-05-26 from the user's "build a [catalog] of audits that aren't necessarily needed to happen now but should be written when a concern arises… more of a wider all-available random-access when needed they trigger/get called into effect manually type thing." See `context/markdowns/notes/steering-decisions-log.md`.

## Catalog registry

Monotonic ID assignment. New audits append; never recycle an ID.

| ID  | Title                                                     | Status        |
|-----|-----------------------------------------------------------|---------------|
| A1  | Reactionary automation audit                              | available     |
| A2  | Critical script coverage                                  | available     |
| A3  | Hook effectiveness                                        | available     |
| A4  | Skill graph                                               | available     |
| A5  | Memory consolidation                                      | available     |
| A6  | Worktree hygiene                                          | available     |
| A7  | Style token audit                                         | available     |
| A8  | Missing script wrappers                                   | available     |
| A9  | Deny list                                                 | available     |
| A10 | Scheduler workaround                                      | available     |
| A12 | Oh-shit destructive ops                                   | available     |
| A13 | Visual snapshot baseline drift                            | in_progress   |
| A14 | Stats: time-passed × token-usage correlation              | in_progress   |
| A15 | Research folder utilization                               | resolved      |
| A16 | Agent-idle root cause                                     | in_progress   |
| A17 | 5hr-reset operator alerting                               | in_progress   |
| A18 | Plan/audit creation gap                                   | in_progress   |
| A19 | Cross-system meta-monitoring (recurrence + workflow bypass) | in_progress   |
| A20 | Hedge holdoffs + unrouted blockers (response-side)        | in_progress   |
| A21 | Parallel capacity underuse (response-shape)               | in_progress   |
| A22 | Cluster + NS/GL deep-integration gap                      | in_progress   |
| A23 | Time precision + 3-way integration                        | in_progress   |
| A24 | Meta-loop-eval cross-utilization                          | in_progress   |
| A25 | settings.json health + hook chain hygiene                 | in_progress   |
| A26 | agentic-script-design utilization + expansion             | in_progress   |
| A27 | Testing toolbelt + anti-bottleneck (user 25× stress)      | in_progress   |
| A28 | Skill health framework — usage metrics + refresh cadence  | in_progress   |
| A29 | Lazy regression-to-mean (BG-launch undercount)            | in_progress   |
| A30 | Graceful shutdown/startup overhaul (laptop-suspend-resume) | in_progress   |
| A31 | Pre-restart prep master (6-front parallel upgrade)        | in_progress   |
| A32 | Hook audit + testing + post-shutdown automation           | in_progress   |
| A33 | Local AI mass-wielding research initiative (G12 + G13)    | in_progress   |
| A34 | End-of-session realignment ritual + master system audit   | in_progress   |
| A35 | Rabbit holes research framework + agentic survey          | in_progress   |
| A36 | True-system-time discipline — eliminate hallucinated time | phase_2_patches_applied |
| A37 | Calculation audit — stats/metrics + scheduler/OS arithmetic | phase_2_patches_applied |
| A38 | Master skill-maker from user voice (look through Ben's eyes) | in_progress   |
| A39 | High-potential idea handoff — trigger + workflow + slam-meter | in_progress   |
| A40 | Idea-gap detector + recurring conversation sweep              | in_progress   |
| A41 | Powerful uses of session JSONLs — 25-item brainstorm catalog | in_progress   |
| A42 | Idle emulator resource-saver (7-signal-layer; dynamic threshold) | in_progress |
| A43 | Stats system major upgrade #2 (8 modules; predictive + anomaly) | in_progress |
| A44 | Critical Finding Reflex (agent-side auto-spawn protective workflow) | in_progress |
| A45 | Required-service health + port-conflict protection + auto-recovery | in_progress |
| A46 | Monkey chamber UX de-retardize (Pending/Decided/All filter + reopen flow) | in_progress |
| A47 | BG lifecycle discipline + auto-stop ghosts (5-rule protocol) | in_progress |
| A48 | data.json auto-sync from inbox-history (banana backlog status derived not flipped) | in_progress |
| A49 | Chamber-empty domino + auto-plow pivot (closes the "trigger gap" user named) | in_progress |
| A50 | monkey-click-fan-out stuck in dry-run — 11 real clicks fired 0 live triggers | phase_1_landed (server parser was real bug; F50-1+F50-2 resolved) |
| A51 | Dashboard typing survives HMR full-page reloads (focus/cursor/scroll preservation) | in_progress |
| A52 | Bespoke lefter promoted to top of every dashboard page | proposed |
| A53 | Full Matrix Birdseye dashboard tab (without bloated slop) | proposed |
| A54 | Inbox UNSEND feature (pre-send-only; restore text to field) | proposed |
| A55 | Dashboard tops scrutiny pass #2 (post-expansion) | proposed |
| A56 | Idle test-tool resource leak prevention (audit + killer scripts, foreground subset of A42) | phase_1_landed |
| A57 | Dashboard content quality + UI/UX scrutiny (5-pillar enforcement + linter + wrapper + render fixes BG) | in_progress |
| A58 | Organic OS — Brain (CoT) + Heart (heartbeat) + Immune (tests + chant enforcement) + Phase 2.5 Organs extension | in_progress |
| A59 | Scheduler upgrade brainstorm — 15 ideas in 3 tiers (research only; code changes deferred) | in_progress |
| A60 | Goal billboard × CoT intertwining (schema-additive Phase 1 + plan) | in_progress |
| A61 | Monkey + inbox architecture bottleneck — recurring class (A46/A48/A49/A57 each missed the meta) | in_progress |
| A62 | Adaptive immunity discipline — unifying meta of "make this impossible" (hardware × software × biology triple-tethering) | in_progress |
| A63 | Idle-capacity research-furthering — close the wait-window parallelism gap (7-boundary safety) | in_progress |
| A64 | Stuck-state audit + decoupling integrity + SOLID grade + 5 per-letter skill refs (A45 recurrence confirmed; inbox-server DEAD) | in_progress |
| A65 | Autonomic system + daemon/worker/pool framework (structural fix for recurring idle-laziness; 8-daemon catalog) | in_progress |
| A66 | Decision-handling discipline (scope of directive = scope of permission; high-risk gate list; inferred-permission scope-jump prevention) | in_progress |
| A67 | Impact-domino analysis discipline (triage layer above goal billboard; leverage-ranked stack; daemon-shaped) | in_progress |
| A68 | Poll-gap class fix — agent doesn't surface inbox during text-only/BG-wait turns (3 shapes deployed) | in_progress |
| A70 | Intentionally skipped ID (never claimed; A69→A72 jump) | cancelled |
| A93 | Git page — multi-page change visibility + before/after capture-integrity | in_progress |

Next ID: A94 (A1–A93 assigned; A52–A55 are `proposed` stubs and A70 is a skip placeholder — all five materialized as files so the audit-catalog daemon's file-based gap/xref scan stays clean).
