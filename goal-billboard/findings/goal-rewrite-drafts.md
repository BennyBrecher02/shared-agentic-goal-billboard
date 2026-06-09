---
title: Goal-rewrite drafts — proposed crisper definitions for G4 / G8 / G14 / G15
domain: goal-billboard / goal-data-quality
status: DRAFT — awaiting user sign-off (goal files NOT edited)
created: 2026-05-31T13:05Z
serves_northern_star: G2
belongs_to_goal: G18
source: context/markdowns/research/systems/goal-wiring-analysis.md §1 (vague goals)
applies_after: the goal-wiring linkage pass (belongs_to_goal stamped; G3→G18; G12/G13→paused)
---

# Goal-rewrite drafts (PROPOSALS — not applied)

The goal-wiring linkage pass is done: `belongs_to_goal` is stamped across the corpus and
the board now coheres by-goal (orphan 90% → ~20%; 15 goals carry members). That surfaced
the **second** half of the analysis — four goals whose *definitions* are vague enough that
future artifacts still won't know where to route. Per the user's standing rule that goal
definitions are the user's substrate, **these are drafts only; no goal file was edited.**
Sign off (or amend) each and I'll apply.

Each block gives: the defect (why it's vague), the proposed crisper scope line, the proposed
"done," and the exact frontmatter/body deltas I'd make.

---

## G4 — Skill system + reactionary infrastructure maintenance

**Defect (split identity).** G4 is explicitly "maintenance-mode — never done"; its exit
criteria are health *metrics* (memory ≤30, no ref >2000 lines), not completion. Because
nothing can ever "attach and close" under it, it silently became the dumping ground for all
skill/memory/hook/organic-OS research — and the linkage pass just confirmed that: **G4 now
holds the single largest constellation (~96 members).** That number is *correct* given the
approved routing, but it's a symptom: a maintenance goal shouldn't be where one-shot research
and discrete build-outs land.

**Proposed split.** Keep G4 as a standing/maintenance goal, but separate its two identities:

1. **The standing health contract** — the invariants (memory ≤30 entries; no skill ref
   >2000 lines; cross-links valid; skill descriptions match content). These are *health
   metrics* and belong on the **dashboard health panel**, not as goal "exit criteria." A
   maintenance goal is never "done"; it's *in-spec or out-of-spec*.
2. **Rolling scoped skill-system initiatives** — discrete, closeable build-outs (e.g. "A28
   per-skill refresh wave," "event-bus rollout," "skill-health-metrics dashboard"). Each of
   these should be able to spin off as its **own scoped goal** (or live under G17 when it's
   throughput-shaped) so it has a real "done."

**Proposed scope line (one sentence to stamp into the goal body):**
> *G4 owns the skill / reactionary / hook / organic-OS substrate and its health invariants;
> discrete build-outs spin off as their own scoped goal or live under G17. G4 is NOT the
> by-goal home for one-shot research — research attaches to the goal it serves.*

**Proposed "done" reframe.** Replace the "exit criteria = health metrics" framing with:
"G4 has no terminal `achieved` state; it is `in-spec` while all health invariants hold and
`needs-attention` when any is breached. Discrete initiatives under it close individually."

**Frontmatter/body delta (on sign-off):** rewrite the `## Exit criteria` section per above;
add the scope line to `## Notes`; leave `serves_northern_star`, `belongs_to_goal` links, and
history untouched.

---

## G8 — Northern Star / Guiding Light deep integration

**Defect (META + self-referential, and it caused the homogenization).** The goal is itself
META ("it serves all NS work indirectly"); its `serves_northern_star: G2` is self-referential;
and it is **the goal that created the migration** (`migrate-add-ns-field.py`) that stamped
`serves_northern_star: G2` onto 320 artifacts — the exact mass-homogenization that collapsed
the by-goal axis in the first place. Scope ("NS context propagates through plans, audits, BG
prompts, hooks, scheduler, token attribution, every panel") is broad-but-real, but "done" is
fuzzy because Phases 4–5 are open-ended.

**Proposed tightening — pin "done" to the 5 concrete exit-criteria already listed:**
1. Every plan/audit/change record carries `serves_northern_star:` ✅ (Phase 1 landed).
2. Every BG dispatch prepends NS context (Phase 3 wrapper — adopt consistently).
3. `state-batch-digest` fires "NS AT RISK" when stale (Phase 2 — landed; confirm firing).
4. Stats tab shows per-NS token attribution (Phase 4 — open).
5. Every dashboard panel shows the "clears path for: G<NS>" badge (Phase 5 — open).

When 4 + 5 land and 1–3 are verified-in-production, G8 → `achieved`.

**Proposed explicit non-goal (the anti-recurrence guard — stamp into the body):**
> *G8 is NOT the by-goal grouping mechanism. `serves_northern_star` is the SPINE / Mission
> field (one NS at a time); the parent-goal field is `belongs_to_goal` and is separate (see
> goal-wiring-analysis §0). No future NS migration may write `serves_northern_star` as a
> stand-in for a parent goal.*

This single sentence prevents the next migration from re-homogenizing the corpus and undoing
this whole pass.

**Frontmatter/body delta (on sign-off):** tighten `## Exit criteria` to the 5 bullets with
explicit landed/open status; add the non-goal paragraph to `## Notes`.

---

## G14 — End-of-session realignment + master system audit

**Defect (overlaps G11; leans on pipeline-stage jargon; aspirational counts).** Scope overlaps
G11 (session bundle) heavily; the definition leans on "Stage 3 of the post-shutdown pipeline,"
which only parses if you already hold G10 + G11 in your head; and "13 post-analysis hooks" is
an aspirational number, not a scoped deliverable.

**Proposed re-anchor against G11 (the capture-vs-act seam) — one boundary line for both bodies:**
> *G11 = CAPTURE the session journey (within-session bundle + analyzers). G14 = ACT on it
> across sessions (cross-session recurrence detection + a realignment digest that pre-seeds the
> next session).*

**Proposed "done" reframe (drop the literal "13 hooks"):**
- The realignment digest is generated automatically at session end **and** feeds the resume
  briefing (so the next session opens with concrete priorities, never cold).
- The cross-session recurrence module is live and produces a real alert on ≥3 consecutive
  bundles.
- Replace "13 post-analysis hooks" with "the post-analysis hooks that cascade *structural*
  action" (count is an outcome, not a target).

**Frontmatter/body delta (on sign-off):** add the boundary line to `## Why it exists` (and the
mirror line into G11's body); rewrite `## Exit criteria` per the three bullets; strike the "13
post-analysis hooks" literal wherever it appears.

> Note (no merge): per analysis §2, G11 and G14 stay **separate** — capture vs act is a real
> seam. Optionally tag G10 / G11 / G14 with a shared `initiative: graceful-lifecycle` so the
> board can *band* them as a super-cluster without collapsing their identities. (Tag, not
> merge — flagged for the user, not applied here.)

---

## G15 — Rabbit-holes deep-research infrastructure

**Defect (methodology/vehicle goal with ambiguous membership).** G15 is a *vehicle* goal — it
owns the rabbit-holes/ subfolder + discipline + catalog. But it legitimately *produces* research
across many topical goals, so its own membership is ambiguous: does an RH about local-AI belong
to G15 (the vehicle) or G12/G13 (the topic)? The linkage pass already enacted the resolution
below — RH content was routed to its *topical* goal (RH-014→G17, RH-017→G12, token-cheap-out→G9,
top-starred-github→G4, RH-015→G19) — so this rewrite is mostly *ratifying* what the pass assumed.

**Proposed scope (own the FRAMEWORK, not the content):**
> *G15 owns the rabbit-hole **framework + the rabbit-holes/ folder discipline** — the
> methodology-first ordering, the synthesis→spawn pattern, the catalog, and the research-method
> refs. Individual rabbit-hole **content attaches to its topical goal, not to G15** (RH-014
> throughput → G17; RH-017 local-AI → G12; token-cheap-out → G9; top-starred-GitHub → G4;
> RH-015 blog → G19; reference-design → G2). What lands under G15 is the framework itself + the
> shared `research/methodology/` corpus.*

**Proposed "done":** framework shipped (methodology ref + folder discipline + catalog) **and**
≥1 rabbit hole has demonstrably spawned a downstream audit/plan/goal (RH-015→G19 already
satisfies this). Then G15 → `achieved` for v1, with an annual-refresh cadence as the standing
follow-on.

**What the pass already did under this rule:** `research/methodology/*` (5 docs) → G15; every
RH cluster → its topical goal. So G15 currently carries ~6 framework/methodology members and the
RH *content* lives under G17/G12/G9/G4/G19/G2 — exactly the boundary this rewrite proposes to
make explicit.

**Frontmatter/body delta (on sign-off):** add the scope paragraph to `## Why it exists`; restate
`## Exit criteria` per the two-part "done"; no link changes needed (the pass already routed).

---

## Cross-cutting items flagged but NOT applied (need a user call)

- **G6 — live-agent messaging:** the analysis couldn't confirm G6's status (it surfaced as a
  grouping key but its definition was thin). G6 **does** have a file (`active/G6-live-agent-messaging.md`)
  and the pass routed 2 messaging plans to it (`live-agent-message-queue-plan`,
  `inbox-never-lost-mutation-plan`). Confirm G6 is still a live goal (vs folding into G17) — if
  it folds, add `superseded_by:` and re-home its 2 members.
- **G12 + G13 merge:** the pass moved both to `paused/` (decision 5). The §2 *merge* question
  (one "Local-AI mass-wielding (single→cluster)" goal with phases, mirroring G17 absorbing
  G7+G16) is still open — paused-but-separate is the current state.
- **`initiative: graceful-lifecycle` band** for G10/G11/G14 — optional, not applied.
