---
idea_id: I-005
title: Zoomed-out / birds-eye board view (where the goal-constellation threads become meaningful)
lifecycle: captured
created: 2026-05-31
related: I-001 (board SVG strings/threads — built then scrapped at 1:1 scale; this is their real home)
belongs_to_goal: G18
---

# I-005 — Zoomed-out board view (the real home for the red threads)

## The idea

A zoomed-out / birds-eye mode for the board where the whole goal-constellation structure is visible at once — cards shrink to dots/nodes, goals become hubs, and the **connections between them are the point**. At that scale, the red-thread yarn (I-001) showing card→goal (and goal→goal) links would be MEANINGFUL — it'd reveal the shape of the work: which goals are dense, which are isolated, how research/plans cluster.

## Why now (the origin)

2026-05-31: the literal red threads (I-001) were built at the current **1:1 card scale**, and the user's verdict was decisive — *"a cute concept but it plays out as a terribly messy useless feature… at our current scale it is meaningless clutter slop."* Correct: at 1:1, ~23 strings draped over full-size cards is noise. The threads were scrapped (the logical `belongs_to_goal` connections kept). The user's insight: *"maybe add a new research idea for a zoomed-out version of the board and maybe then we can use the red threads."* The threads weren't wrong — they were at the wrong **zoom level**.

## What to research / design (when picked up)

- A zoom control (or a separate "constellation map" view) that collapses cards → nodes and goals → hubs, so the whole ~449-item / 18-goal structure fits one screen.
- At that scale, draw the connection lines (the scrapped I-001 yarn, reborn) — card→goal, plus the (currently thin) goal→goal cross-links.
- Interaction: click a hub to zoom into that constellation (back to the 1:1 board, filtered to that goal).
- Tradeoffs: SVG/canvas node-graph vs a CSS-grid minimap; performance at ~449 nodes; how it coexists with the by-goal / Flow / Rabbit-holes views.

## Status

Captured / parked until prioritized. The `belongs_to_goal` data (now wired) is the substrate it needs — so it's buildable when wanted.
