---
audit_id: A52
title: Bespoke lefter promoted to top of every dashboard page
status: proposed
catalogued: 2026-05-27T14:20:00Z
priority_when_run: P2
estimated_effort: small
trigger: A dashboard tops pass surfaces the bespoke-lefter element as worth elevating to the top of every page.
deferral_reason: Reserved future number — proposed but never written as a full audit yet. Stub exists so the audit-catalog daemon (Rule 1 gaps / Rule 4 xref) sees a file at this ID and stops flagging it. Cited as a sibling by A57.
related_goals: [G2]
serves_northern_star: G2
belongs_to_goal: G18
findings: []
---

# A52 — Bespoke lefter promoted to top of every dashboard page

## Status note

This is a **reserved, proposed-only** catalog ID. It was registered (with this title +
`proposed` status) in the catalog registry but never written up as a full audit. This stub
materializes the ID as a file so the audit-catalog maintainer daemon
(`consumer-audit-catalog-scan-daemon3.sh`) — whose Rule 1 (sequential gaps) and Rule 4
(broken intra-catalog `A<N>` xref) are **file-based**, not registry-aware — no longer reports
it as a gap or as a broken reference. A57 cites A52 as a sibling.

## Why this audit matters (if promoted)

The bespoke-lefter element is a candidate for elevation to the top of every dashboard page,
for consistent at-a-glance context. Whether that's worth the vertical real-estate is the
question this audit would answer.

## What it would look at

- Where the bespoke-lefter currently renders, and the cost/benefit of promoting it page-wide.
- Interaction with the dashboard tops pass (A55) and the matrix birdseye tab (A53).

## How to trigger

Promote from `proposed` → `available` and write the full scope when a dashboard tops pass
flags this. Until then it remains a reserved number.
