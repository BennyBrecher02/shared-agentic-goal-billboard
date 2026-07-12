---
goal_id: G24
title: "Cross-tool activation wave — Cursor + Codex go LIVE (queued behind Pi Day)"
status: active
track: CROSS-TOOL
northern_star: false
priority: P1 (queued — fires the moment G23 Pi Day completes)
created: 2026-07-12
updated: 2026-07-12
gated_on: "G23 (Pi cluster Northern Star) completion + the user saying he's ready to open Cursor"
project: AgenticOS
---

# G24 — Cross-tool activation wave: Cursor + Codex live 🔀

Everything Cursor/Codex has been piling into a scattered "when I'm ready" bucket (user, 2026-07-12:
"this is all gathering into a lot… make some official goal for all this area to be queued up next after
pi day north star is completed"). This goal is that bucket, made official. **Nothing here starts until
G23 is done and the user opens Cursor** — then this becomes the active front.

## Phase 1 — the parked activation pastes (all staged, all user-run)
1. **Cursor task-inbox wiring** (3 pastes, from OS repo root — REGENERATE #3 fresh at run-time, the
   staged 2026-07-05 merge file is stale-by-design by then; the stale-full-file-cp landmine class):
   - `cp portable-kit/templates/cursor-rules/shared-brain-presence.mdc ~/.cursor/rules/`
   - `cp portable-kit/templates/cursor-hooks/cursor-task-inbox-context.sh ~/.cursor/hooks/`
   - a FRESH `~/.cursor/hooks.json` merge (Claude re-stages against the then-current file)
2. **Optional:** the task-watcher plist (or by then, the Pi runs the watcher — likely moot on the Mac).
3. **Codex hooks kit** — the codex-hooks template from portable-kit; Codex CLI login (user's "infinite
   free compute" plan); task-inbox `to: codex` lane goes live.

## Phase 2 — the split-brain proof (agents-talking, for real)
- Claude posts a task → Cursor claims + executes → replies through the lane (and the reverse).
- Then the same round-trip with Codex as the executor (Fable orchestrates, Codex burns the tokens —
  the project_codex-free-compute strategy). Device-ledger already treats codex as a first-class owner.

## Phase 3 — the coworking stress-test project (the trio: Claude + Cursor + Claude Design)
The vetted candidate list from cursor-coworking-fresh-projects-2026-07-09.md, **minus the two
woodshed-duplicate ideas** (Riffbox #1 and Fretlab #2 were rejected 2026-07-09 as overlapping the
already-built guitar app). Valid, still-standing candidates:

| # | Idea | One-liner | Score |
|---|------|-----------|-------|
| 3 | **Groovebox** | browser rhythm/timing game (falling notes, text-pattern charts) | 22/25 |
| 4 | **Effectlab** | CSS gradient/shadow/glass/animation studio | 22/25 |
| 5 | **Patternsmith** | generative SVG pattern/wallpaper studio (seedable, tileable) | 22/25 |
| 6 | **Launchpad** | keyboard-driven start-page / command palette | 20/25 |
| 7 | **Devbox** | local dev-utility multitool PWA (regex/JSON/cron/diff/color) | 19/25 |

User picks ONE at activation time. (Note: a "Theme Forge" name from an earlier chat recap appears in
NO written artifact — treat Effectlab as the closest real candidate; don't chase the ghost name.)

## Definition of done
Both tools live on the lane; one full cross-tool round-trip each (Cursor + Codex); the coworking
project scaffolded with roles split; the standing asks-log Cursor reminder retired into this goal.
