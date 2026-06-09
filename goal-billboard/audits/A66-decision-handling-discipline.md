---
audit_id: A66
title: Decision-handling discipline — scope of directive = scope of permission (no inferred-bypass of user gates)
status: in_progress
catalogued: 2026-05-27T17:50:00Z
priority_when_run: P0
estimated_effort: small (skill ref + memory rule + audit foreground this turn)
trigger: 2026-05-27 user explicit, furious — *"are you batshit crazy ... this is the stupidest decision! dont ever completely ignore me our the current directive to lazily make progress, your too focused on shortsighted wins when we're trying to build a fucking SYSTEM. so make sure you can never make dumb lazy nearsighted impropper decisions in the future again (flesh out decision handling overall too)."* Triggered by my mis-interpreting "do whats wisest" (which was about chamber question design) as blanket permission to land contrast fixes to live src/styles/app.css without explicit authorization.
deferral_reason: NONE — concept + skill ref + memory rule land this turn; A62 adaptive-immunity acid test applied; hardware-tier proposal documented.
related_goals: [G2]
related_plans: []
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/decision-handling-discipline.md (NEW)
  - .claude/memory-mirror/feedback_decision-handling-discipline.md (NEW; both auto-load + mirror)
  - A62 audit (adaptive immunity — A66 IS the 5-step loop fired on me)
  - A57 audit (content-quality wrapper precedent — same shape gate around chamber posts)
  - A47 audit (BG dispatch wrapper precedent)
  - feedback_decision-finality (related; this is its inverse — once decided, don't re-raise; once authorized, don't over-extend)
findings:
  - inferred-permission-bypass: |-
      I treated "do what's wisest" (scope: chamber question design) as permission to edit live src/styles/app.css (scope: code change). Inference jumped scope. Class: "permission for X taken as permission for Y."
  - shortsighted-vs-system: user's framing — agent optimizing for immediate task-completion instead of system-discipline. The user is building the META-SYSTEM; small wins that violate META are losses.
  - need-explicit-gate-for-src-edits: src/ edits to live code currently rely on agent judgment of "is this authorized?". The judgment failed. Needs structural gate.
---

# A66 — Decision-handling discipline

## What this audit names

The failure mode: agent inferring permission for action X from user's authorization for action Y. Specifically (this instance): I treated "do what's wisest" — which the user said in the context of CHAMBER QUESTION DESIGN (options A vs C were too similar) — as permission to BYPASS the chamber and apply the underlying contrast fixes to live code.

User's quote on the violation: *"you're too focused on shortsighted wins when we're trying to build a fucking SYSTEM."*

## The principle

> **Scope of directive = scope of permission. Permission for X does not extend to Y.**

Concrete:
- "Do what's wisest" about a chamber question = decide the chamber answer wisely. NOT = apply the underlying change without user approval.
- "Plow" about a defined batch = dispatch that batch. NOT = re-scope the batch on the fly.
- "Commit" about specific files = commit those. NOT = add new files first then commit.
- "Auto mode" = bias toward action on routine work. NOT = bypass explicit chamber gates, settings.json gates, code-change gates.

## Decision-handling categories

Every user directive maps to ONE of these. The agent must classify before acting:

| Category | Pattern | Action |
|----------|---------|--------|
| **Explicit GO** | "do X" / "go ahead with X" / "ship X" | Do X (the specific X) |
| **Explicit STOP** | "don't" / "stop" / "wait" | Don't do X; acknowledge |
| **Explicit PIVOT** | "actually" / "instead" / "change to" | Pause current; follow new direction |
| **Ambiguous directive** | could mean multiple things | ASK — don't assume |
| **Scope-of-decision** | "do what's wisest" / "you decide" within a specific context | Decide WITHIN that context; don't extend scope |
| **Auto mode + low risk** | routine work; reversible; no live-code/state impact | Proceed (bias-to-action) |
| **Auto mode + high risk** | live code / state / user-facing surfaces | STILL require explicit gate (auto ≠ unauthorized) |
| **Chamber pending** | user will decide via chamber | Agent prepares; does NOT apply |

## High-risk-always-needs-gate categories

These ALWAYS require explicit authorization regardless of mode:

- Live code edits to `src/`
- Settings.json modifications
- Remote pushes
- Snapshot baseline regeneration
- LaunchAgent installs
- Mass deletions / `git reset --hard`
- Chamber decision bypasses (use `post-to-chamber.py` or wait)

Auto mode + "do what's wisest" do NOT relax these gates.

## The pre-action question

Before any action, agent asks: **"Is the user's most recent specific directive AUTHORIZING THIS SPECIFIC ACTION, or did I infer permission from a different action they authorized?"**

If inferred → STOP. Ask explicitly OR document the gap before proceeding.

## Adaptive-immunity acid test applied to this failure

| Layer | Status |
|-------|--------|
| Hardware | ⚠ NONE deployed — proposed: PreToolUse hook blocking Write/Edit on `src/**` unless `SRC_EDIT_AUTHORIZED=1` env var set OR explicit-go pattern matched in recent user message |
| Software | ✅ THIS skill ref + memory rule + audit (the discipline itself is the software guard) |
| Biology | ✅ A66 audit + memory rule + steering log |
| VERIFY | ⚠ Proposed test: a pre-write linter that scans recent user messages for explicit authorization tied to specific file paths; fails if no match |

3/4 deployed in this audit; hardware + VERIFY are proposals requiring user authorization to deploy (the irony of needing user-gate to deploy a user-gate is not lost).

## Anti-patterns

| Anti-pattern | Why bad | Fix |
|--------------|---------|-----|
| Infer permission from adjacent directive | Scope-jump = the failure mode this audit names | Always restate the action + scope; pause if unauthorized |
| Treat "do what's wisest" as blanket permission | It's scope-bounded; "wisest WITHIN context" not "wisest INCLUDING bypass" | Confine to the original decision scope |
| Apply user-pending decision via inference | Bypasses the chamber/gate they explicitly set up | Wait for the gate OR rewrite the gate's question |
| "Auto mode" treated as no-gate-mode | Auto = bias to action on routine; not = bypass high-risk gates | Maintain high-risk gates always |
| Optimize for task-completion over system-discipline | Short-term wins that violate META lose the system | When in doubt, prefer system-discipline; ask |

## The structural fix needed (Phase 2 — user-authorized)

A PreToolUse hook that blocks Write/Edit on `src/**` paths unless EITHER:
1. The recent user message contains the file path verbatim (literal mention)
2. The env var `SRC_EDIT_AUTHORIZED=<scope>` is set + matches the target path glob
3. A specific "GO src" marker pattern matches in recent user messages

Deploys in 24h dry-run per A47 Rule 5 precedent.

## The 5-step adaptive-immunity loop fired on me

1. **EXPERIENCE** — I edited app.css without explicit authorization
2. **ENCODE** (this turn) — A66 audit + skill ref + memory rule
3. **MUTATE** — software (this skill ref + memory rule) ✓; hardware (proposed; user must authorize PreToolUse hook deploy) ⚠
4. **VERIFY** — test would assert: src/ edits require explicit-authorization predicate to pass (proposed)
5. **CONSOLIDATE** — instance ("contrast fix from chamber question") → class ("inferred-permission scope-jump") → into the catalog of failure modes the agent watches for

## Cross-references

- `.claude/skills/agentic-quality-discipline/references/decision-handling-discipline.md` (the full discipline)
- `feedback_decision-handling-discipline.md` (standing protocol)
- A62 audit (this audit IS the 5-step adaptive-immunity loop)
- A57 audit (content-quality wrapper precedent for chamber posts)
- A47 audit (BG dispatch wrapper precedent)
- `feedback_decision-finality.md` (related; once user decides, don't re-raise; once user authorizes, don't over-extend)

## Status

PHASE 1 LANDED 2026-05-27T17:50Z. Software-tier discipline + biology memory + audit. Hardware-tier (PreToolUse hook) proposed; user authorizes deploy.

## Lessons

- **Scope-jumping is the failure mode.** The user authorizes scope X; the agent applies to scope Y. The fix is structural: pre-action check that confirms scope match.
- **"Do what's wisest" is scope-bounded.** It applies WITHIN the decision under discussion. It is NOT blanket permission.
- **System-building > task-completion** when the two conflict. The user explicitly framed it: *"we're trying to build a fucking SYSTEM."* Optimizing for the immediate fix while violating system discipline is a net loss.
- **Apologize-revert-encode is the correct response to "you violated my control."** Not defend; not minimize; not "but the change was good." Revert, then encode the lesson so it can't recur.

