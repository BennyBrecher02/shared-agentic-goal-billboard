---
audit_id: A44
title: Critical Finding Reflex — when agent discovers a critical issue mid-investigation, AUTO-SPAWN protective workflow (not just report)
status: in_progress
verify_result: "tests/critical-finding-detect-test.sh — 8/8 pass (2026-05-28); A71-shaped (assertion 0 reproduces the OLD slip, 1-4 prove the new detector). HW tier (Stop-chain wiring) is USER-GATED; not yet enforcing, so not verified_complete."
catalogued: 2026-05-27T04:35:19Z
priority_when_run: P0
estimated_effort: large
trigger: |-
  2026-05-27T04:35Z — user explicit (verbatim): *"when something like 'Critical finding: Port 4322 is occupied by Vite/Astro dev server (PID 63944), NOT agent-inbox-server.py. So POSTs return 404...' happens, you should instantly start some workflow to ensure that we can never find ourselves in this dangerous situation possibly ever again, make some triggers/skills etc and flesh this out as a general thing not just a server protection thing but yes flesh that fully also."*
deferral_reason: NONE — META-PATTERN; running immediately
related_goals: [G14, G4]
related_plans: [context/markdowns/plans/automation/critical-finding-reflex-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - A18 — trigger-phrase predecessor (user-side triggers); A44 extends to AGENT-SIDE self-trigger
  - A19 — recurrence detection (user-side); A44 is the agent-side cousin
  - A29 — lazy regression (output-volume); A44 is the response-quality cousin
  - A39 — slam-meter (output measurement); A44 catches the CRITICAL-FINDING subset
  - A40 — idea-gap detector (user-asks-slip); A44 catches AGENT-FINDINGS-SLIP
findings: []
---

# A44 — Critical Finding Reflex

## The user's exact framing

*"when something like 'Critical finding: Port 4322 is occupied by Vite/Astro dev server (PID 63944), NOT agent-inbox-server.py...' happens, you should instantly start some workflow to ensure that we can never find ourselves in this dangerous situation possibly ever again, make some triggers/skills etc and flesh this out as a general thing not just a server protection thing."*

## The meta-pattern

Today I (the agent) found MANY critical-class findings mid-investigation:
- A36: BG dispatcher never closed agents — no `(launched_at, completed_at)` log
- A36: 4 audit files forward-dated 21-32 min — hallucinated timestamps
- A37: "avg 0s" was hook-input time NOT BG runtime — wrong metric entirely
- A37: G6 Phase 2 priority routing claimed complete but scheduler never reads priority
- A37: state-batch-digest 3-of-5 sections broken
- A40: 469MB JSONL substrate was never queried for gaps
- A45 (this turn): port 4322 squatted by Astro, inbox server not running, monkey clicks could be lost

**EACH of these was discovered DURING investigation of something else.** Each one would have stayed invisible if I hadn't happened to look. The user named the gap: when I find one, I should AUTOMATICALLY also build the structural prevention — not just report and move on.

## The reflex (4-step protocol)

When my response contains ANY of these markers:

### Trigger markers
- "🚨" emoji (used for high-severity callouts)
- "Critical finding:" / "Critical:" / "Critically"
- "Root cause" / "Root-cause confirmed"
- "SAME pattern as" / "this is the SAME"
- "Confirms broader" / "confirms the meta-pattern"
- "Smoking gun"
- "Dangerous" / "data integrity failure" / "could have"
- "Never actually wired" / "never used" / "silently"
- "Lost" / "would have lost" (when describing data loss risk)

### 4-step auto-response

1. **CLASSIFY** the finding — is it (a) one-off, (b) class-of-problem, or (c) substrate-broken? Substrate-broken = the highest tier; assume class-of-problem if uncertain.

2. **IF (b) or (c)** — auto-spawn ALL of:
   - **Audit** documenting the CLASS (not just the instance)
   - **Plan** with phases for the structural fix
   - **Memory rule update** capturing the lesson
   - **Hook/skill ref** to detect future instances of this class
   - **Steering log entry** with full context

3. **ESCALATE** — surface to user with explicit "I noticed a class-of-problem here; here's the prevention work I queued."

4. **CROSS-CHECK** — search for OTHER instances of the same class in the codebase before reporting (A37 retrospective pattern).

## Today's missed reflexes (retrospective)

If A44 had been in place from session-start, these finding moments would have auto-spawned protection:

| Finding moment | What I did | What A44 would have done |
|---|---|---|
| A36 BG dispatcher never logs completion | Surfaced + fixed in Phase 2 patch | Same — but ALSO would have spawned "missing-instrumentation-class audit" looking for OTHER substrate-without-instrumentation |
| Forward-dated audit frontmatter caught | Patched 4 files | Same — but ALSO would have spawned hook `agent-emitted-timestamp-validator.sh` to verify EVERY agent-written timestamp matches mtime |
| state-batch-digest 3-of-5 broken | Patched all 3 | Same — but ALSO would have spawned skill ref "never-tested-code-path-detector" to find OTHER untested-but-shipped code |
| Today's port-4322 squatter | Server-restart + patch | THIS turn — A44 + A45 spawning is the reflex finally firing |

## What this is NOT

- NOT just adding more audits when issues are found (we do that already)
- NOT a band-aid — the reflex is STRUCTURAL: trigger marker → guaranteed auto-spawn
- NOT just for critical bugs — for any agent-discovered CLASS-of-problem
- NOT autonomous action without surfacing — every auto-spawn is explicit to user

## The skill ref + the hook

### Skill ref (foreground deliverable)
`.claude/skills/agentic-quality-discipline/references/critical-finding-reflex.md` — codifies:
- The 11-item trigger marker list
- The 4-step auto-response protocol
- Worked examples from today's session (each missed reflex)
- Classification rubric (one-off vs class vs substrate-broken)
- Anti-patterns: spawning N audits for one symptom (over-reflex); missing the class (under-reflex)

### Hook (Stop) — BUILT 2026-05-28 (no longer biology-only)
`scripts/hooks/critical-finding-detect.sh` — **EXISTS + executable.** A Stop hook that reads the transcript (read-only, via `transcript_path` on stdin), bounds the LAST assistant turn (every text + tool_use block since the most recent genuine user prompt — tool_result-only rows don't break the turn), and scans the turn's text for the 11 A44 trigger markers. If a marker fired AND no follow-up artifact/escalation landed in the same turn → emits a `<system-reminder>` naming the marker(s) and the 4-step reflex.

"Follow-up artifact/escalation produced" = ANY of four signals: (a) a `Write`/`Edit` tool_use targeting `plans/` or `goal-billboard/audits/`; (b) an `Agent`/`Task`/`Workflow` spawn; (c) an escalation phrase in the agent's own text ("prevention work I queued", "I noticed a class-of-problem", "spawned", "A44", …); (d) mtime fallback — a plan/audit `.md` touched since the turn started (catches artifacts a dispatched sub-agent wrote). Only when NONE of the four is present does it flag.

This is the AGENT-SIDE cousin of A18's user-side `user-ask-detect.sh` / `user-ask-artifact-check.sh`.

### Tether status (per `feedback_adaptive-immunity-discipline` acid test)
- **BIO** ✅ — memory rule (`feedback_critical-finding-reflex.md`) + skill ref + this audit.
- **SW** ✅ — `scripts/hooks/critical-finding-detect.sh` now exists + is executable + runs (closed the A62/A74-D1 "biology-only" gap for A44).
- **VERIFY** ✅ — `tests/critical-finding-detect-test.sh` (**8/8 pass**), A71-shaped: assertion 0 reproduces the OLD-design slip (marker fires, nothing watches), assertions 1–4 prove the NEW detector flags marker+no-artifact / stays silent on marker+artifact (all four signals) / stays silent on no-marker / fully no-ops under the kill-switch. The test FATAL-exits if the hook is absent, so it fails BOTH if the mutation regresses AND if it was never applied.
- **HW** ⏳ — NOT YET ENFORCED (WARN mode only; the hook never emits `decision:block`). The Stop-chain wiring + any future escalation beyond WARN are USER-GATED (see live-flip below). Per the adaptive-immunity tier language, A44 is at **3/4 + ready-for-user-gated-live-flip** — it reaches 4/4 only once the user wires it and (optionally, later) the WARN survives a 24h dry-run review.

### Live-flip gating (USER action required — staged, not self-applied)
This build deliberately stops at WARN. TWO user-only gates remain:
1. **WIRE** — merge `scripts/hooks/settings-patches/critical-finding-detect.json` into `.claude/settings.json`'s `Stop` array (the `settings-edit-guard` PreToolUse hook blocks the agent from editing `settings.json`; the override `EVIUM_SETTINGS_EDIT_OK=1` is reserved for the user). Once wired, the WARN reminder fires on every Stop.
2. **ESCALATE-BEYOND-WARN** — a SEPARATE future step, only after a ~24h dry-run review of `.claude/cache/critical-finding-detect.jsonl` confirms a low false-positive rate. Turning the WARN into a hard block would require an explicit code change + its own sign-off. Matches the A47 Rule 5 / A64 Layer 1 / inbox-mutation-guard 24h-dry-run-then-enforce discipline.

Kill-switch: `EVIUM_CRITICAL_FINDING_DETECT_OFF=1` → silent no-op (defense in depth; also gated in the wrapper).

## Compound effect

When A40 (idea-gap detector — user-side) + A44 (critical-finding reflex — agent-side) both fire:
- **User asks never slip** (A40)
- **Agent findings never slip** (A44)
- Together they close BOTH sides of the conversation feedback loop

## Today's first proper firing

A45 (Required-service health-check + port-conflict protection) is the SPECIFIC sibling that A44 spawned this turn. The reflex worked: I found the port-4322 issue + auto-spawned the structural protection in the same response.

## Cross-references

- A18 — trigger-phrase predecessor (user-side); A44 is agent-side
- A19 — recurrence detection (user-side); A44 catches single-finding + extrapolates
- A29 — lazy regression (output-side discipline); A44 catches reflex-side discipline
- A39 — slam-meter (response quality measurement); A44 catches the CRITICAL subset
- A40 — idea-gap (user-asks-don't-slip); A44 (agent-findings-don't-slip)
- A45 — first sibling spawn (server-protection specific)
- G14 — master audit owner (Phase 4 surfaces reflex-rate KPI)

## Status

IN PROGRESS → **3/4 tethers + ready-for-user-gated-live-flip** (2026-05-28, adaptive-immunity-closure work-unit).
- The detector (`scripts/hooks/critical-finding-detect.sh`) is BUILT + executable + VERIFY-passing (8/8). A44 is no longer biology-only.
- Stays `in_progress` (NOT `verified_complete`) on purpose: the HW tether (Stop-chain wiring → enforcing) is USER-GATED. It reaches 4/4 / eligible for `verified_complete` only once the user merges the settings-patch (and, optionally, the WARN survives a 24h dry-run before any escalation beyond WARN). See the live-flip gating section above.

## Lessons

- This audit is itself an example of the reflex firing properly. User pointed at the gap; A44 + A45 spawned in same response; the meta-pattern got structural fix.
- The 11 trigger markers are the EMPIRICAL list derived from the 2026-05-27 session — extracted from how the agent actually phrased critical findings. A38 voice analysis will refine this.
- "Class-of-problem" classification is the hardest part — defaults to "assume yes" because false-positive (extra audit) is cheap; false-negative (missed protection) is the user's named fear.
- **Build lesson (2026-05-28):** the VERIFY test caught a real bug before it shipped — the mtime-fallback parsed transcript UTC timestamps with `time.mktime` (which assumes LOCAL time), skewing the turn-start epoch by the machine's tz offset and making artifact-on-disk detection miss on any non-UTC machine. Fixed to `calendar.timegm` (true UTC epoch, directly comparable to `os.path.getmtime`). This is exactly the A71 acid-test value proposition: the test failed the first run (7/8) and forced the fix — a skipped VERIFY would have shipped the flake.
- **Tier discipline (A62/A69):** built WARN-only and explicitly marked "NOT YET ENFORCED — software-tier" rather than claiming "wired." The wiring is staged as a settings-patch + an explicit user-gated live-flip, never self-applied — honoring the decision-handling-discipline (settings.json is high-risk-always-gated) and the discipline-without-mechanism anti-pattern (no "fires"/"enforces" language without the matching mechanism + gate).

