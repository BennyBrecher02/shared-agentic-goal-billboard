---
audit_id: A9
title: Deny-list audit (load-bearing vs friction-only)
status: in_progress
catalogued: 2026-05-26T11:30:00Z
last_run: 2026-05-26T19:46:00Z
priority_when_run: P1
estimated_effort: medium
trigger: After A10 (scheduler workaround audit) surfaces specific code that exists because of deny-list pressure OR when an agent reports being blocked by 3+ different denies in one session OR when adding new automated workflows that need specific permissions
deferral_reason: Live state matters — need usage data from current scheduler + future scheduler work to know which denies are friction vs which are saving us. A.3 just landed; A.4-A.5 will produce evidence.
related_goals: [G1, G4]
related_plans: []
related_refs:
  - .claude/settings.json
  - .claude/skills/agentic-quality-discipline/SKILL.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G1
findings:
  - context/markdowns/goal-billboard/audits/findings/deny-list-audit-2026-05-26-phase1.md
  - context/markdowns/goal-billboard/audits/findings/A9-deny-list-2026-05-26.md
---
# A9 — Deny-list audit

## Why this audit matters

The deny list in `.claude/settings.json` is the project's user-safety surface. It blocks destructive operations agents could trigger by accident. **But not every deny earns its keep** — some block things that never come up; some block things the scheduler / hooks legitimately need + we've been building workarounds.

This audit separates **load-bearing denies** (catastrophic-risk-prevention; KEEP) from **friction denies** (block useful work + lead to workarounds; LOOSEN with care).

It's distinct from A10 (which catalogs the workarounds we built). A9 is "should the deny exist?"; A10 is "given that deny X exists, what code did we write to dodge it?"

## What it would look at

For each entry in `settings.json` deny list:

1. **Category:** catastrophic (data loss possible if not denied) / dangerous (recoverable but bad) / friction (annoying but not unsafe)
2. **Trigger frequency:** how often has it fired since 2026-05-24? (estimate from session traces / pattern matches)
3. **Workaround cost:** what code or process exists to dodge this deny? (link to A10 findings if available)
4. **Side-agent impact:** does this deny also protect against side-agent (bro-WT-style) accidents? If yes, weight higher
5. **Specific allowlist alternative:** could the deny be kept blanket BUT a tightly-scoped allowlist override the specific safe case?

## Expected outputs

- `audits/findings/deny-list-audit-{date}-phase1.md` with per-deny verdict (Phase 1)
- Additional findings docs per phase (e.g., `-phase3.md` after Phase 3 applies changes)
- Three buckets:
  - **KEEP unchanged** — catastrophic / load-bearing
  - **KEEP + add specific allow** — blanket deny stays, narrow allow added for known-safe paths
  - **REMOVE** — never useful; was over-cautious from the start (likely empty)
- For each KEEP+ALLOW: the exact JSON line to add, with reasoning

## SAFETY PROTOCOL (insane levels)

**This audit touches the user-safety surface. Treat every change as load-bearing.**

### Phase 1 — Read-only discovery (no settings.json changes)
1. Enumerate every deny entry
2. For each, write rationale + workaround cost + risk-if-removed
3. Produce findings doc as MARKDOWN, not code change
4. **Stop here. Surface findings to user.**

### Phase 2 — User decision per entry (sequential)
1. User reviews findings doc
2. User explicitly approves each proposed change ("yes, add this specific allow")
3. **Never bundle changes** — each entry is its own decision
4. **Never imply approval from silence** — explicit "yes" required

### Phase 3 — Apply one entry at a time
For each approved entry:
1. **Backup** `settings.json` first: `cp .claude/settings.json .claude/settings.json.backup-pre-A9-step-N`
2. Apply ONE change via Edit tool
3. Validate JSON: `jq . .claude/settings.json > /dev/null`
4. Run scheduler tick to verify nothing broke
5. Run a tool call that would have hit the previous deny — confirm the new behavior
6. Commit (separate commit per change for atomic revertibility): `git commit -m "settings: <description of 1 allow change>"`
7. Wait. Observe for 1+ sessions. Then move to next entry.

### Non-negotiables (NEVER remove these denies regardless)
- `Bash(rm -rf*)`, `Bash(rm -r*)`, `Bash(rm -f*)`, `Bash(rm /*)`, `Bash(rm ~/*)`, `Bash(rm ../*)`
- `Bash(sudo*)`
- `Bash(git push*)` (user explicitly pushes themselves)
- `Bash(git worktree remove .)`, `Bash(git worktree remove ..)`, `Bash(git worktree remove ../EviumOverhaul)`, `Bash(git worktree remove /*)`, `Bash(git worktree remove ~/*)`
- `Bash(npm publish*)`, `Bash(curl -X DELETE*)`
- Any allow that would be SCOPED-LESS (e.g., `Bash(git worktree remove ../EviumOverhaul-*)` without the `change-` or `calibration-` qualifier — too broad)

### Roll-back protocol
If any approved change has unexpected consequences:
1. `cp .claude/settings.json.backup-pre-A9-step-N .claude/settings.json`
2. `git revert` the commit
3. Update findings doc with the regression
4. Re-evaluate

## How to trigger

Whichever fires first:
- A10 produces a list of scheduler workarounds that would benefit from specific allow additions
- An agent reports being blocked by 3+ different denies in one session
- A new automated workflow needs specific operations not currently allowed
- User says "let's audit the deny list"

## What this does NOT do

- Doesn't remove ANY catastrophic-path denies
- Doesn't change the blanket `*--force*` deny — only adds narrow exceptions for specific safe paths
- Doesn't change anything without per-entry user approval
- Doesn't bundle multiple changes into one settings.json edit

## Notes

The settings-edit-guard hook (`scripts/hooks/settings-edit-guard.sh`) currently blocks Write|Edit on settings.json. For A9 implementation, the user has to unset `EVIUM_SETTINGS_EDIT_OK=0` or edit settings.json manually. **This is a feature, not a bug** — the audit's safety relies on settings.json edits being deliberate.
