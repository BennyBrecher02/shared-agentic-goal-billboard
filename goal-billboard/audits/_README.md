# Audit catalog — naming + partition conventions

## ID partition (multi-orchestrator coordination)

To prevent ID-race conditions when two agents claim a new audit ID at the same time, the audit number space is **partitioned by originator**:

| Prefix | Originator | Range | Current next-ID |
|---|---|---|---|
| `A<N>` | Main orchestrator (M4 / main Anthropic account) | 1–999 | A72 |
| `B<N>` | M2 worker (M2 machine / second Anthropic account) | 1–999 | B1 (none claimed yet) |
| `C<N>` | Future third orchestrator | 1–999 | C1 |
| `H<N>` | Human user (you, ad hoc audits) | 1–999 | H1 |

Each originator claims its own next-ID from its own range. No shared counter; no race.

## Filename format

`<PREFIX><N>-<short-slug>.md`

Examples:
- `A71-inbox-never-lost-mutation.md` (main)
- `B3-design-handoff-batch-1-review.md` (M2, after multi-orchestrator goes live)
- `H1-2026-redesign-color-system-rethink.md` (human ad-hoc)

## Required frontmatter

```yaml
---
audit_id: A<N>  # matches filename prefix
title: <Title>
status: available  # available | in_progress | completed | verified_complete | cancelled
catalogued: <ISO 8601 UTC>
priority_when_run: P0 | P1 | P2 | P3
estimated_effort: small | medium | large
related_goals: [G<N>, ...]
related_plans: [<path>, ...]
serves_northern_star: G<N>  # which goal this serves
findings: [...]
---
```

## Verified-complete gate (per A62 acid test)

An audit may set `status: verified_complete` ONLY if all four adaptive-immunity tethers are deployed AND the VERIFY test passes:

1. Hardware tier — immutable barrier deployed
2. Software tier — executable guard deployed
3. Biology tier — memory rule + skill ref + this audit
4. VERIFY tier — test that fails if the failure can recur

A71 is the first audit to reach all 4 (see `adaptive-immunity-discipline.md` skill ref positive-example section).

## Cross-references

- `feedback_adaptive-immunity-discipline` — the 4-tether discipline
- `plans/multi-orchestrator-new-subscription-plan.md` — multi-orchestrator architecture (why prefix-partition exists)
- `feedback_verified-counts` — current catalog scale verification
