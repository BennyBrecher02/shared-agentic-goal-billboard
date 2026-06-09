---
pin_id: P-020
title: Zoomed-out board mode (birds-eye) — the home for the red-thread constellation links
created: 2026-05-31T14:26Z
why_paused: user "put a pin in the zoomed out board mode with threads" — deferred; the literal threads were scrapped at 1:1 scale (clutter) and only become meaningful zoomed out
resume_criteria: when the user wants the zoomed-out / constellation-map board view (or explicitly unpins)
estimated_cost: design + build of a node-graph/minimap board view + reviving the I-001 threads at that scale
leverage: turns the board into a structure-revealing map; gives the (now-scrapped) threads a home where they're actually useful
related: context/markdowns/goal-billboard/ideas/I-005-zoomed-out-board-view.md
state: pinned
belongs_to_goal: G18
---

# P-020 — Zoomed-out board mode + threads (PINNED)

## What it is

A birds-eye / zoomed-out board view: cards collapse to dots/nodes, goals become hubs, the whole ~449-item / 18-goal structure fits one screen — and the goal-constellation connections (the red-thread yarn, scrapped at 1:1) are drawn at THIS scale, where they're meaningful (they reveal the shape of the work: dense vs isolated goals, how research/plans cluster).

## Why pinned (not now)

The literal red threads were built at the current 1:1 card scale and were *"a cute concept but… meaningless clutter slop"* (user, 2026-05-31) — scrapped, with the logical `belongs_to_goal` connections kept. The threads aren't a bad idea; they're at the wrong **zoom level**. The zoomed-out view is their real home — but it's a meaty new view (node-graph / minimap), parked until the user prioritizes it.

## Full concept + research-design

See `goal-billboard/ideas/I-005-zoomed-out-board-view.md` — the zoom control, node/hub collapse, the revived threads, click-to-zoom interaction, and the SVG-vs-minimap tradeoffs. The `belongs_to_goal` data (now wired) is the substrate it needs, so it's buildable when wanted.

## Cross-references

- I-005 (the idea + research-design notes)
- I-001 (the scrapped 1:1 threads — revive here)
- G18 — Dev Control dashboard
