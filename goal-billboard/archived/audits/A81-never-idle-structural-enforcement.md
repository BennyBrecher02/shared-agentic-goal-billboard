---
audit_id: A81
title: Never-idle structural enforcement — why it kept failing, and the actual fix (Workflow, not daemon)
status: completed
catalogued: 2026-05-29
trigger: 2026-05-29 — user (frustrated, repeated): *"wheres the bg research tasks i asked you to enforce and audit and enforce and audit a million times already… why are we always [not] juggling at peak capacity always swapping for a research when an agent resource slot becomes available? flesh this out"*
serves: throughput / autonomy enforcement
---

# A81 — Never-idle structural enforcement

## The recurring failure (named by the user, repeatedly)
"Always swap a research task into an agent slot the moment it frees" keeps being a **discipline I fail**, despite being designed three times: A63 (idle-capacity-furthering), A65 (autonomic research daemon), A80 (the never-idle protocol). I kept *hand-launching* research per turn — imperfectly, and the user kept having to ask. That's the textbook "discipline is fragile" failure.

## The root cause I'd missed
A65 specced the fix as a **bash daemon** (Daemon #1). But **a bash daemon cannot spawn Claude agents** — only the orchestrator (via the Agent tool) or a **native Workflow** can. So the A65 design *could never actually execute the spawning*; it stayed a doc, and the work fell back to my per-turn discipline (which failed). The design named the right properties (autonomous, runs-regardless-of-agent-state) but the wrong mechanism.

## The actual fix — a Workflow, not a daemon
The native dynamic **Workflow** primitive CAN do what the daemon couldn't: spawn agents in a loop and **auto-cap concurrency, refilling each slot the instant one frees**. `parallel(queue.map(c => () => agent(c.prompt)))` *is* "juggle at peak capacity, swap research into the freed slot" — enforced by the primitive, not by my willpower.

**Built + running:** `.claude/workflows/research-furthering.js`
- Generic: fans out a queue of read-only research candidates, concurrency-capped, each writing a findings artifact.
- Default queue = **systems research** (the agentic OS — per the 2026-05-29 pivot off Evium): A65-daemon-feasibility · hook-skip-impl · Heart-rearm · reactionary-health · stats-correlation-insights · dashboard-wave2 · scheduler-G1-health · memory-skill-health · CoT-ledger-health · testing-toolbelt-G9. Override via `args.candidates`.
- First run: `wf_d573c578` (2026-05-29).

## The standing enforcement (how it stops recurring)
- **When there's idle agent capacity + a research queue → run `research-furthering`** (don't hand-juggle). This is the binding mechanism, not a discipline.
- **The 7–8 band (user 2026-05-29):** ceiling **8**, floor **7** — never let in-flight drop below 7 without starting more research. Encoded in `dispatch-bg.sh`. `research-furthering` keeps a batch's slots full (`parallel()` auto-refills as slots free); **CHAIN a fresh batch the instant one completes** — that's how the floor holds *across* batches, not just within one. (Read-only research is light/IO-bound, so 7–8 is the steady-state target; a brief handoff overlap above 8 between batches is fine.)
- Reconciles A65: the daemon *idea* (autonomous, runs-regardless) realized by the *right mechanism* (Workflow's loop+concurrency, which can actually spawn agents).
- The make-work guard still applies: the queue holds GENUINE candidates; when it empties, the workflow ends (it doesn't pad).
- Heart × Autonomic pairing (A65) updated: the "daemon" for research-furthering is this Workflow; the other A65 daemons (memory-consolidator, etc.) need the same mechanism-recheck (does each need a Workflow, a hook, or is bash enough?) — flagged for the a65-daemon-feasibility research candidate (running now).

## Cross-references
`feedback_idle-capacity-furthering` (A63) · `feedback_autonomic-system` (A65) · A80 (never-idle protocol) · `.claude/workflows/research-furthering.js` · the systems-research outputs under `context/markdowns/research/systems/`.
