---
audit_id: A80
title: The never-idle protocol — always-find-safe-work via research-furthering
status: in_progress
catalogued: 2026-05-29
trigger: 2026-05-29 — user — *"how can there be nothing at all for you to do instead of idling? what about the rule that you should always be parallelizing background research... flesh this out for an audit then plow safely... can i make you find safe work for you to do?"*
serves: throughput / autonomy discipline (and biases the well toward G2)
---

# A80 — The never-idle protocol

## The gap (a recurrence — the 3rd idle/parallelize correction this session-family)
When the concrete-build queue thinned, I defaulted to **STOP** ("nothing safe to do"). That violated the standing `idle-capacity research-furthering` rule + G15. The recurrence chain this session: parallelization-blowup → "why wait if I'm gone" → "how can there be nothing to do." Same root cause each time: **I treat idle as a valid default. It is not.**

## The distinction I conflated (the core insight)
- **make-work** = redundant / low-value busy-work to *look* busy or *burn* budget. ANTI-PATTERN (user: "use the budget is NOT a goal").
- **research-furthering** = genuine investigation on REAL open questions that advance the NS / the system / verification. **The sanctioned, inexhaustible idle-fill.**

My error: I correctly avoided make-work, then *wrongly used "avoid make-work" to justify idle.* Avoiding make-work ≠ stopping. It means **finding real work.** The standing rule says fill idle capacity with research-furthering — not stop.

## The always-available safe-work well (it never runs dry)
When the obvious queue thins, real work always exists in these categories, ranked by default priority:
1. **NS-serving research** *(highest — bias here)* — deep-dives on real G2 defects/needs (structured data, a11y, perf, content). Keeps the well from drifting into pure infra.
2. **Adversarial verification of recent outputs** — QA what was just built before the user relies on it (catch errors early).
3. **Real open-question rabbit-holes (G15)** — genuine architecture/system uncertainty.
4. **Codebase exploration for undiscovered issues** — real catalog audits (pick by genuine suspected value, not run-the-catalog-for-its-own-sake).
5. **Cross-system analysis** — run + interpret existing instrumentation (correlation engine, stats) for real insight.
6. **Learning consolidation** — distill run-groups into LESSONS / skill-ref updates.

## The decision-procedure (per idle-detection)
```
IF in-flight < cap  AND  budget < ceiling  AND  no pivot signal:
    pick the highest-priority item from the well that is GENUINELY valuable
       (make-work guard: "would this artifact actually matter?" — if no, go DOWN the list, never idle)
    launch it (prefer NS-serving) · log it · keep the board current
```
**Idle is never the default. "Nothing to do" is a near-impossible state** — reaching for it means I'm wrong; go to the well.

## The boundary (never-idle ≠ reckless)
make-work guard (real research only) · budget gate (stand down >80% burn) · pivot-respect (a user redirect overrides the well) · NS-bias (don't let the well become infra-only while G2 sits) · reserved-list still gated (push/settings/src/launchd/deletes).

## Structural enforcement — answering "can you make me?"
**Yes — and it shouldn't rely on my judgment (which failed 3×).** Proposed (gated — needs your nod):
1. **Reinforce the memory rule** — fold "idle is never the default; go to the well" into `feedback_idle-capacity-furthering` as binding.
2. **The autonomic research-furthering daemon (A65)** — on idle-capacity-detected (heartbeat tier), auto-surface/auto-launch a well item within the 7-gate safety contract. The real structural fix: makes never-idle *automatic*, not discretionary.
3. **A heartbeat idle-detector** — when in-flight drops + budget OK, fire a "go to the well" signal.

## This audit's plow (the framework, demonstrated)
Launched 4 genuine research BGs (read-only, real questions, durable artifacts, 2 NS/system-serving):
- design-ref adversarial verification (well #2)
- structured-data / JSON-LD plan (well #1 — serves G2)
- hook system-event-skip proposal (real noise fix)
- malformed-`data.json` culprit pin (board #11, real bug)

## Cross-references
`feedback_idle-capacity-furthering` · `feedback_autonomic-system` (A65 daemon) · G15 (rabbit-holes infra) · the 2026-05-29 session writeup (Lessons 1+2) · `findings-leaderboard.md`.
