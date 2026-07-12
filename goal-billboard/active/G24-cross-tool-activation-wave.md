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
REFINED 2026-07-12 (user: "genuinely useful devtool stuff, exclude all guitar/music"). All
music/guitar ideas are OUT of this list (Riffbox/Fretlab = woodshed dupes; Groovebox = music →
its rhythm-game concept moved to the woodshed backlog instead). **Theme Forge RECOVERED from the
session transcript** (a compaction had dropped the artifact): it was the 🏆 recommendation, aimed
at the user's actual paying work. Current candidates, best first:

| Idea | One-liner | Why it's genuinely useful |
|------|-----------|---------------------------|
| 🏆 **Theme Forge** | ingest a **Claude Design export** → live-preview the component set → tweak design tokens → export production Tailwind/CSS | in the paying wheelhouse (Astro + Claude Design client sites); Claude Design literally produces the input — the perfect 3-way test |
| **Breakpoint Hunter** (app-ified) | URL → width sweep 320–1600 + bisection on cracks (h-scroll/overflow/nav-wrap) → visual crack report | already spec'd in mission-control-plan §Also-folded-in; Cursor=report UI, Claude=sweep engine; we'd use it on every client site |
| **Effectlab** | layered CSS gradient/shadow/glass/keyframe studio with copyable output | daily web-design tool |
| **Regex/Selector Lab** | pattern → live-highlighted matches + plain-English explanation | classic engine↔UI ping-pong (from the recovered transcript table) |
| **Patternsmith** | seedable tileable SVG pattern studio | client-site background assets |
| **Launchpad / Devbox** | command-palette start page / dev-utility multitool | solid but lower keep-value — bench options |

User picks ONE at activation time; Theme Forge is the standing recommendation.

## Definition of done
Both tools live on the lane; one full cross-tool round-trip each (Cursor + Codex); the coworking
project scaffolded with roles split; the standing asks-log Cursor reminder retired into this goal.
