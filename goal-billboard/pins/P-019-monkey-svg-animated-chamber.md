---
pin_id: P-019
title: Animated-monkey SVG chamber — living monkeys + drag-drop banana feeding + glass-with-monkey
created: 2026-05-31T12:42Z
why_paused: user explicit "put a pin in the monkey svg for now" (2026-05-31)
resume_criteria: when the user wants to build the animated-monkey chamber (or explicitly unpins)
estimated_cost: phased build (Phase 0 rig → 1 idle → 2 feeding → 3 accept → 4 glass); ~10–15 KB inline footprint
leverage: chamber delight + the user's "feed the monkeys when bored" want; belongs under G18 (dashboard)
related: context/markdowns/research/systems/monkey-chamber-animated-monkeys-research.md
state: pinned
belongs_to_goal: G18  # by-goal parent (goal-wiring §3 tier-B/C — user-approved)
---

# P-019 — Animated-monkey SVG chamber (PINNED)

## What it is

The Monkey Communication Chamber gets ANIMATED MONKEYS (the agents) that "accept" the user's decisions, PLUS a drag-and-drop banana-FEEDING toy: bananas always available to fling at the monkeys for fun when bored / not deciding anything. **Kept distinct from the "Banana Backlog"** (that name stays for pending decisions; the feeding bananas are a separate playful feature). Plus a glass-pane effect that only reads right WITH a monkey behind it pressing/smudging.

## The research verdict (already done)

Per `research/systems/monkey-chamber-animated-monkeys-research.md`: **2D wins** — animated inline SVG monkeys (CSS keyframes + a ~150-line pointer-events state machine). 3D (Three.js ~600KB + glTF) rejected: it fights the dashboard's 0-fetch inline-everything identity. Inline SVG is the only family that is natively inlinable, recolors per skin (`fill: var(--accent)`), is compositor-cheap, reduced-motion-safe, and buildable in-house.
- **Art:** bespoke geometric SVG monkey rig (separable head/eyes/arm/mouth groups).
- **Feeding:** an "Enrichment Bin" of draggable bananas; pointer events + `setPointerCapture` (NOT HTML5 DnD — dead on mobile); FSM `IDLE → ANTICIPATE → CATCH → EAT → HAPPY`.
- **Glass:** a monkey presses palm/face from inside; the smudge is anchored to the contact point (a smudge needs a visible cause).
- **Phased build:** Phase 0 static rig → 1 idle life → 2 feeding toy → 3 decision-accept reaction → 4 glass. Each shippable, page-scoped to `#page-chamber`, 0-fetch, reduced-motion-safe.

## Why pinned (not built now)

User: "put a pin in the monkey svg for now." The chamber is currently reverted to a clean, simple view (the over-built observation-lab was removed). The animated-monkey feature is deferred until the user wants it — the dashboard's priority threads (git-page overhaul, board goal-wiring, rail redesign) come first.

## Cross-references

- `research/systems/monkey-chamber-animated-monkeys-research.md` (the 2D verdict + phased plan)
- `research/systems/monkey-chamber-observation-lab.md` (the earlier vibe spec — the glass/whiteboard ideas, superseded-by-simple for now)
- G18 — Dev Control dashboard (the home goal)
