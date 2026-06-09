---
created: 2026-05-26T19:30:00Z
updated: 2026-05-26T19:30:00Z
audit_id: A15
phase: initial
status: findings-locked
---

# A15 — Research folder utilization (initial findings, 2026-05-26 PM)

## Current state

**Folder contents (`context/markdowns/research/`):**

| File | Sources | Topic | Status |
|---|---|---|---|
| `README.md` | n/a | Schema + index | active (continuously updated) |
| `device-coverage-research-2026-05-24.md` | n/a (origin pre-frontmatter) | Android version share / AVD selection | findings applied |
| `filename-convention-research.md` | 9 | Date-in-filename convention | findings-locked, applied |
| `preview-vs-chrome-research-2026-05-25.md` | n/a (origin pre-frontmatter) | Preview vs Chrome capability matrix | findings drove use-case split |
| `resource-capacity-research-2026-05-25.md` | n/a (origin pre-frontmatter) | M4 24GB capacity for scheduler resources | findings backed resource model |
| `timestamp-logging-bloat-research.md` | 7 | When timestamp logging adds value vs bloat | findings-locked, applied |

**Last-touched:** 2026-05-26 06:47 local. The folder was **created** that morning during the timestamp-logging research consolidation — and has had **zero additions** in the subsequent 13 hours, despite multiple research-shaped investigations during that window.

**Schema conformance:** 3 of 5 substantive files have proper `sources: <N>` + `findings_drove:` frontmatter. The 3 "origin" files (device-coverage / preview-vs-chrome / resource-capacity) were retro-moved from notes and don't fully conform — they cite sources inline in the body but lack the structured frontmatter fields. Cosmetic issue, but worth normalizing.

## Misrouted research-shaped content (candidates for backfill — surface before moving)

Scanning `notes/` and the steering log for content that meets the research bar (multi-source synthesis + derived finding + drove a decision):

### Top 5 misrouted candidates

1. **`notes/agentic-vs-scripted-gray-area.md`** — Multi-source synthesis (user dialogue paraphrased + cost-data reasoning + opsys-fundamentals citations). Derived a finding ("per-step verdict, not global; recurring audit gated by stats") that drove A1 catalog + agentic-vs-scripted memory feedback entry. **Verdict: research-shaped. Candidate for move + a research-folder retrospective version.**

2. **`notes/scheduler-resource-model-v2.md`** — Comparative analysis (4-category resource taxonomy with citations to opsys-scheduling-patterns ref + audited capacity numbers). Drove the scheduler resource model implementation. **Verdict: design + research hybrid. The taxonomy section is research; the v2 design is plan-shaped. Candidate for SPLIT — extract the taxonomy section into research, leave the design here.**

3. **`notes/last-attempt-recommendations.md`** — Investigation of the Evium(last-attempt) reference codebase: what's salvageable, what to discard. Multi-source (the reference codebase + Drive export + current Astro state). Drove decisions in highway §05 + state-pages handoffs. **Verdict: research-shaped scan/inventory. Candidate for move with light reframing as "Evium(last-attempt) salvage audit."**

4. **Today's design-handoff Mode-A discipline derivation** — 5-source research (Anthropic developer docs + Claude design playbook + 3 Slate-Studio/Design.dev/Webflow workflow guides) that produced the "extend-existing-system vs fresh-design" two-mode framework now in `agentic-design-handoff/SKILL.md`. **NOT currently filed anywhere as research.** The methodology lives in the skill ref; the source citations + comparative analysis vanished. **Verdict: research-shaped synthesis that never landed in research/. Reconstruct retrospectively from skill + steering log.**

5. **A11 concurrency-ceiling investigation (2026-05-26 PM)** — Binary search 3↔4↔6↔10 with empirical timing measurements. Findings doc at `goal-billboard/audits/findings/phaseB-concurrency-ceiling-2026-05-26-phase1.md` is in the right place (audit findings), but the *generalizable finding* ("Astro dev server saturates at 3+ concurrent Playwright workers per host") is research-shaped — drove the cluster-readiness plan's per-node shard math. **Verdict: keep findings doc where it is; cross-reference into a new research file `astro-devserver-concurrency-ceiling-research.md` that distills the generalizable rule for future reference.**

### Honorable mentions (lower priority)

- `notes/preview-vs-chrome-research-2026-05-25.md` — wait, already moved (verify no orphan). _Confirmed in research/, no orphan._
- `notes/scheduler-unattended-work-options.md` — survey of 3 ways to run scheduler unattended; could be research but is more proposal-shaped. **Verdict: leave in notes.**
- `notes/hook-and-skill-recommendations.md` — observation list, not synthesis. **Verdict: leave in notes.**

## The cost of underutilization

1. **Insights don't compound across sessions.** A future session asking "what's our take on agentic-vs-scripted?" reads `notes/agentic-vs-scripted-gray-area.md` and finds analysis but no clear "this is settled research." Without the research/ filing, agents re-investigate from scratch.

2. **Sources get lost.** The design-handoff Mode-A derivation cited 5 sources. Those URLs are no longer recoverable from the skill ref alone — and would need to be re-found if challenged.

3. **Skill refs can't be audited against their origin.** A skill says "the convention is X" — but with no `findings_drove:` backlink, you can't trace which research justified it. Drift is invisible.

4. **The reactionary layer can't harvest patterns.** A reactionary skill (`research-synthesis`) that periodically scans research/ for trends has nothing to scan if research is in notes/.

5. **The discipline weakens by default.** Standing-protocols memory says "research → research/" — but if same-session evidence shows it's not happening, the rule's authority erodes.

## Recommendations

Documented in `context/markdowns/plans/research-folder-automation-plan.md`:

- **Phase 1 (today):** Backfill candidates with user approval per item — no blanket-move.
- **Phase 2:** Advisory hook that surfaces "this looks research-shaped; consider research/" when an agent writes to notes/ with research markers (≥2 source citations, "investigate"/"comparative"/"survey" framing).
- **Phase 3:** Extend `skill-cross-link-rebuild.sh` to also link research → skill refs when a research file's `findings_drove:` points at a skill.
- **Phase 4:** `scripts/research-synthesis.py` quarterly aggregation pass.

## Lessons

The folder exists; the schema is documented; the README is current. **What's missing is the trigger discipline** — agents need either (a) automated reminders or (b) stronger habit reinforcement to actually file research-shaped work in research/ rather than the convenient default of notes/. Phase 2's advisory hook closes this gap structurally.
