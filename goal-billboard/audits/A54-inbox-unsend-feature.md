---
audit_id: A54
title: Inbox UNSEND feature (pre-send-only; restore text to field)
status: proposed
catalogued: 2026-05-27T14:20:00Z
priority_when_run: P2
estimated_effort: small
trigger: A user posts to the inbox and wants to retract before the agent has read it.
deferral_reason: Reserved future number — proposed but never written as a full audit yet. Stub exists so the audit-catalog daemon (Rule 1 gaps) sees a file at this ID and stops flagging it as a sequential gap.
related_goals: [G2]
serves_northern_star: G2
belongs_to_goal: G18
findings: []
---

# A54 — Inbox UNSEND feature (pre-send-only; restore text to field)

## Status note

This is a **reserved, proposed-only** catalog ID. It was registered (with this title +
`proposed` status) in the catalog registry but never written up as a full audit. This stub
materializes the ID as a file so the audit-catalog maintainer daemon
(`consumer-audit-catalog-scan-daemon3.sh`) — whose Rule 1 (sequential gaps) is **file-based**,
not registry-aware — no longer reports it as a sequential gap.

## Why this audit matters (if promoted)

An inbox UNSEND affordance: **pre-send-only** (before the agent has consumed the message),
restoring the typed text back into the input field so the user can edit or discard it.

## What it would look at

- The pre-send window semantics (what "not yet read" means against the drain hooks).
- Restoring text to the field vs a hard delete; interaction with the poll-gap drain (A68).

## How to trigger

Promote from `proposed` → `available` and write the full scope when the retract-before-read
need recurs. Until then it remains a reserved number.
