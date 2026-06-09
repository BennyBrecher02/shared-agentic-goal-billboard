# Ideas lane — the durable backing store for the idea machinery

> **The gap this closes:** the system has heavy idea-lifecycle machinery
> (`idea-conflict-check`, `impact-domino-analysis`, the monkey chamber, the
> never-miss-an-idea reflex) but no *home* for an idea between "captured" and
> "built." Ideas lived only in prose, chamber posts, and rabbit-holes — so they
> could "grow invisibly beyond what was intended." This directory is where every
> idea is **surfaced and tracked** so nothing slips.

## Routing — ideas vs pins vs goals vs bugs (read this if unsure where something goes)

An **idea** = a user-facing concept **awaiting greenlight** (surface-only, lives here). It is NOT a **pin** (agent-converged-but-paused WORK → `pins/`), NOT a **goal** (multi-session initiative → `active/`), NOT a **bug** (→ `bug-billboard/`), NOT an **ephemeral session task** (→ the Task tool). **Canonical router table:** `goal-billboard/pins/_index.md` → "What this lane is NOT." This boundary is the structural guard against ideas↔pins drift — the two share a cloned parser (`parse_ideas.py` ← `parse_pins.py`), so the routing rule is what keeps them from becoming one accidental system.

## THE SAFETY CONTRACT (non-negotiable — read this first)

This lane is **SURFACE-AND-TRACK ONLY**. It is a *visibility* substrate, never an
*execution* one.

- An idea file here does **NOT** schedule, dispatch, or auto-start any build.
- Nothing in the dashboard, the scheduler, or any hook reads this directory and
  acts on it. The Board view *renders* these files; it never *runs* them.
- **The user is the explicit gate.** An idea moves from `captured` →
  `greenlit` only by a deliberate user action (the user editing `lifecycle:` to
  `greenlit`, or saying so). Graduation to an actual build (a plan, a
  multi-change item, a goal) is **always a human decision**, never automated.
- The Board surfaces two visible states so the user *sees* the line:
  **"captured · awaiting your review"** vs **"greenlit by you"**. Everything in
  between (conflict-check, impact-domino) is *advisory scoring*, not a trigger.

This mirrors the system's established firewall: the dev-handoff queue has **zero
apply buttons** by design; the chamber surfaces decisions but never decides; the
git-rollback page copies commands but never executes them. The ideas lane is the
same discipline applied to ideas — **the board renders state; the user owns
graduation.**

## File shape — `I-NNN-slug.md`

```yaml
---
idea_id: I-001
title: One-line idea name
lifecycle: captured        # captured | greenlit | building | built | dropped
serves_northern_star: G2   # which goal/NS this would further (optional)
belongs_to_goal: G4        # the PARENT goal this belongs under (the by-goal grouping key; optional, ≠ serves_northern_star)
impact_score: 7.5          # from impact-domino-analysis (advisory; optional)
conflict_checked: 2026-05-31  # date idea-conflict-check last ran (advisory; optional)
source: chamber            # chamber | rabbit-hole | user-prompt | research | session
source_ref: context/markdowns/research/systems/foo.md   # back-link to the journal lineage that spawned it (optional)
dor:                       # Definition-of-Ready — the gate the idea must clear BEFORE promotion (see below; advisory, never auto-greenlights)
  scoped: true             # done_looks_like is written + the touch-surface is bounded
  domino_checked: 2026-05-31  # date impact-domino-analysis ran (the "is it worth it?" leverage map)
  no_conflict: 2026-05-31  # date idea-conflict-check ran clean vs live system + stockpile
  leverage_scored: 7.5     # the impact/effort score (mirrors impact_score; the "how big is the win" number)
created: 2026-05-31T00:00Z
updated: 2026-05-31T00:00Z
---

# I-001 — One-line idea name

One paragraph: what the idea is, why it might matter, what it would touch.
The first body line becomes the card summary on the Board.
```

### The `lifecycle:` ladder (what each state MEANS)

| State       | Meaning                                          | Who sets it |
|-------------|--------------------------------------------------|-------------|
| `captured`  | logged, awaiting the user's review               | any agent / the user |
| `greenlit`  | **the user has explicitly approved building it** | **USER ONLY** |
| `building`  | a build is actively underway (post-greenlight)   | set when work starts |
| `built`     | shipped                                           | set on completion |
| `dropped`   | the user decided not to pursue it                 | the user |

`captured` is the safe default. **No automation may set `greenlit`** — that is
the structural enforcement of the safety contract above.

## Definition-of-Ready (DoR) — the gate at the `captured → greenlit` line

The DoR is the **checklist an idea must clear before it is *ready to promote*** into
a proposal or a goal (the G1 gate in `ARTIFACT-ROUTING.md` §2). It does **not** decide
promotion — the user still owns greenlight (the safety contract above). The DoR just
makes "is this idea ready for that decision?" **inspectable at a glance**, so the user
can greenlight fast instead of re-deriving the homework, and so a half-baked idea
doesn't silently stall. It is the readiness analogue of the user-owned Definition-of-Done.

**The four criteria (the `dor:` front-matter block):**

| Criterion | Field | What "ready" means | Filled by |
|---|---|---|---|
| **scoped** | `scoped: true` | `done_looks_like` is written and the touch-surface is bounded (you know what it touches and when to stop). | the capturing agent / a triage pass |
| **domino-checked** | `domino_checked: <date>` | `impact-domino-analysis` has run — the downstream dominoes + the "is it worth it?" leverage map exist. | the impact-domino tick |
| **no-conflict** | `no_conflict: <date>` | `idea-conflict-check` ran clean — the idea doesn't fight the live system or duplicate a stockpiled-but-unbuilt idea. | the conflict-check gate |
| **leverage-scored** | `leverage_scored: <num>` | an impact/effort score exists (mirrors `impact_score`) — the size of the win is quantified, not vibed. | the impact-domino tick |

**Rules:**
- The block is **advisory and optional**. A missing/partial `dor:` means *"not yet ready
  to promote"* — it is **never** a trigger and **never** auto-sets `greenlit`. The Board may
  surface readiness (e.g. "3/4 DoR met") but acts on nothing.
- The triage passes (`impact-domino-analysis`, `idea-conflict-check`) **fill the DoR in**;
  the user **reads** it to greenlight. Same firewall as everywhere else: scoring is
  advisory, the graduation decision is the user's.
- DoR sits at the **`captured → greenlit`** transition; the **Definition-of-Done** (a
  user-only check) sits at **`active goal → achieved`**. Two gates, both user-owned at the
  decision, the DoR machine-*assistable* on the readiness homework.

## How it's parsed

`scripts/dashboard/parse_ideas.py` reads `I-*.md` front-matter (a clone of
`parse_pins.py`) → an ideas block → `model["board"]` via
`parse_unified_board.py`. Read-only; writes nothing. The Board's "Ideas lane"
and the "Loose threads"/"stockpile" saved views surface these.
