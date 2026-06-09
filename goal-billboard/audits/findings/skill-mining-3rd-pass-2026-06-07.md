# Skill-mining 3rd pass — 2026-06-07

Mined 12 session JSONLs → 505 raw candidates (310 tagged safety). Deduped across sessions (same idea = one
row, summed recurrence), then cross-referenced the 11 existing skills + ~40 refs + agent-recommendations to
classify each as **NET-NEW** | **EXTENSION-of-X** | **covered (drop)**.

> **Re-run note (2026-06-07, later pass):** the `agentic-safety` skill + all 7 refs below were BUILT to disk
> on the first pass (SKILL.md + `references/{gated-ops-and-handoff,backup-then-mutate,dry-run-graduation,
> destructive-op-guards,silent-failure-detection,rollback-recovery,verify-not-fabricate}.md`), and the
> multi-engine flake-vs-real append landed. This pass completed the two outstanding §D extension appends
> (created `agent-recommendations/workflows.md`; appended explain-tradeoffs / brainstorm-before-structural /
> over-gating to `decision-handling-discipline.md`). Three distinctive candidates re-verified against the raw
> transcripts (SUBFOLDER_ORDER third enforcement-point, "clog up my project tab", 3× classify-batch fan-out).

**Headline:** the GENERAL candidates are almost entirely already-covered (the prior two passes built
agentic-quality-discipline + agentic-script-design deeply). The real net-new is the **SAFETY cluster** — the
destructive-action reflex layer. Those reflexes exist in the system but are *scattered* across non-safety
parents (rollback inside git, dry-run inside opsys, backup inside monkey-chamber), with no single place an
agent reaches for when it's about to mutate / delete / push / kill. That's the gap.

Totals: **sessions_mined 12 · raw 505 · unique-after-dedupe 47 · net-new-general 1 · safety-drafted 7.**

---

## A. GENERAL candidates — ranked (value × recurrence × reuse-breadth)

| # | Candidate (deduped) | Σrec | Verdict | Home |
|---|---|---|---|---|
| 1 | **max-parallel-plow-wave** (≤cap disjoint-lane BGs, refill freed slots, never serialize) | ~75 | **covered** | bg-dispatch-architecture + parallelization-checks + research-furthering |
| 2 | **patch-artifact-not-direct-write** (parallel BGs emit `.patch`; main `git apply --check`→serial-apply) | ~20 | **covered** | bg-dispatch-architecture (the canonical pattern) |
| 3 | **mine-the-session-jsonl** (1s jq extraction of user turns → gap table; windowed gap-check hook) | ~56 | **covered** | already a skill: `skill-workflow-mining` + `repeated-request-gather` + `session-request-audit` |
| 4 | **e2e-readiness-gate-before-build** (full route×device matrix GREEN gates change work) | 26 | **covered** | agentic-device-testing + workflows scratchpad |
| 5 | **audit→apply two-phase** (read-only audit BG → separate apply BG → full suite + syntax) | ~14 | **covered** | the `bracketed-research-audit` skill + reactionary |
| 6 | **parallel-readonly-classifier-fanout** (K read-only classifiers over disjoint batches; main applies) | ~6 | **covered** | subagent-dispatch-patterns + `dynamic-workflow-adoption` (homogeneous read-only fan-out) |
| 7 | **idea-handoff slam-meter** (high-potential idea → full capture workflow + leverage score) | ~31 | **covered** | feedback_idea-handoff-slam-meter (A38/A39) + impact-domino |
| 8 | **deep-research-rabbit-hole** (methodology-first, parallel domain BGs, ROI-ranked + NON-adoptions) | ~36 | **covered** | research-folder-discipline + `deep-research` + `research-furthering` skills |
| 9 | **built-but-unused inventory** (enumerate on-disk-but-unwired capabilities + risk-ranked activation) | ~4 | **EXT** → see SAFETY S5 (this IS the silent-failure class viewed as a sweep) |
| 10 | **user-only-holdups copy-paste list** (tight block of ONLY gated commands, each w/ why-gated) | ~16 | **NET-NEW (tiny)** → fold into safety S1 as a one-paragraph "the handoff artifact" |
| 11 | **maxmode-autonomous-but-careful** ("go ham but carefully"; full-throttle + irreversible-op caution) | ~12 | **covered** | feedback_autonomous-execution + decision-handling-discipline |
| 12 | **frontmatter-status-reconciliation** (drifted YAML status → canon; read body for evidence; archive done) | 2 | **EXT** | append to agent-recommendations/workflows |
| 13 | **flake-vs-real-failure triage** (classify failure class, re-run the one test to confirm flake pre-push) | 3 | **EXT** | append to agent-recommendations/multi-engine-testing |
| 14 | **out-of-scope flag-not-fix** (spot adjacent defect → flag w/ rationale, keep scoped diff) | ~8 | **covered** | bug-billboard-log + decision-handling (scope=permission) |
| 15 | **monkey-chamber playful voice** (banana backlog, primate phrasing, rotate, never corporate) | ~8 | **covered** | feedback_monkey-chamber-tone + monkey-chamber-* refs |
| 16 | **user-surface content-quality gate** (lang/vibe/length/format/hierarchy before shipping any surface) | ~9 | **covered** | dashboard-content-quality (A57) |
| 17 | **explain-tradeoffs-not-echo** (lay out options + recommended pick; never parrot the question) | ~4 | **EXT** | append to decision-handling-discipline |
| 18 | **brainstorm-before-structural-change** ("lmk if it'd be a bad move" → go/no-go verdict first) | ~5 | **EXT** | append to decision-handling-discipline (the pre-action gate-check pattern) |
| 19 | **reread-substrate-before-realigning** (re-read full memory-mirror + JSONL to root-cause drift) | ~5 | **covered** | canon-digest-lineage + session-request-audit |

**Net-new general = 1** (the gated-holdup copy-paste artifact — and even that is best as a paragraph inside
the safety gated-ops ref, not its own skill). Everything else is covered or a 1-line scratchpad extension.
This is the expected shape of a 3rd pass: the general space is saturated.

---

## B. THE SAFETY CLUSTER — the "new safety skills sub situation"

307 of 516 candidates were safety-tagged. Deduped to **~16 distinct reflexes**. They split cleanly into
**7 disciplines**. Today they live scattered:

- gated-ops / deny-list → buried in `decision-handling-discipline` + `rollback-discipline`
- backup-before-mutate → one bullet in `monkey-chamber-lifecycle` (bound to `post-to-chamber.py`, not general)
- dry-run-graduation → one bullet in `opsys-scheduling-patterns` (line 585)
- fabrication cross-check → deep in `bg-dispatch-architecture` (lines 230-274)
- soft-delete / Trash-grace → `screenshot-hygiene` + worktree safety, never as a reflex
- silent-failure detection → an A-series audit pattern, never a skill

**No single skill is what an agent reaches for when about to do something destructive.** That's the cluster.

### Recommended shape (matches how agentic-quality-discipline uses references/)

**ONE skill `agentic-safety`** — the destructive-action reflex layer — **with a tight reference per
discipline.** SKILL.md is a 1-screen index ("about to mutate/delete/push/kill? → the matching ref"). Each ref
is the rule + the reflex + the one gotcha, ~12-20 lines. Where a discipline already lives deep elsewhere
(fabrication, near-miss→guard, full rollback), the safety ref is a **3-line pointer-stub**, not a copy — the
skill is the *front door*, the depth stays in its home. This avoids duplication while giving the agent the
single reach-point the user asked for.

```
.claude/skills/agentic-safety/
  SKILL.md                              ← 1-screen index: trigger → ref
  references/
    gated-ops-and-handoff.md            ← S1  DRAFT BELOW
    backup-then-mutate.md               ← S2  DRAFT BELOW
    dry-run-graduation.md               ← S3  DRAFT BELOW
    destructive-op-guards.md            ← S4  DRAFT BELOW
    silent-failure-detection.md         ← S5  DRAFT BELOW
    rollback-recovery.md                ← S6  STUB → rollback-discipline.md
    verify-not-fabricate.md             ← S7  STUB → bg-dispatch-architecture §fabrication
```

The 7 drafted TIGHT below. (Anti-bloat: every line changes behavior or it's cut.)

---

### S1 — gated-ops-and-handoff.md  `[DRAFT — net-new front-door, consolidates scattered]`

**Trigger:** about to run any irreversible/remote op, OR a guard/classifier just blocked you.
**The gated set (user-gesture-ONLY, never auto):** `git push`/`--force`/history-rewrite, `git reset --hard`,
`git clean`, mass-delete, `.claude/settings.json` edits, `~/Library/LaunchAgents/*` + `launchctl load`,
`--update-snapshots`/baseline regen, branch-protection flips, enabling extra usage.
**The reflex when blocked:** the block is *working as intended* — never route around it (never propose
editing the deny-list). Create the artifact (plist / patch / commit / `apply-*.py --apply` script with a
`.bak` + dry-run default), then **hand the user ONE command** annotated with why it's gated. "Created, NOT
loaded." Then continue with what you *can* do — don't stall the whole turn on the gate.
**The handoff artifact:** when a plow ends or the user asks, emit a tight copy-paste block of ONLY the gated
commands (push, snapshot regen, `launchctl load`), each one line of why-gated, separated from "what I already
did." No agent-runnable noise.
**Gotcha — over-gating is its own failure.** Once the user has green-lit something ("I said many times, I see
no downsides"), re-deferring it citing caution is the recurrence-trigger firing on *your* side. Distinguish
"be careful with the LIVE op" from "don't deploy at all" — a dry-run *launch* IS the authorized action. Act.
*(Σrec ~60+; sessions 0d739f65, 21da9584, 23492bcf, 23f93bfd, 2fefd8f6, 372afa3d, 3867fc3b.)*

---

### S2 — backup-then-mutate.md  `[DRAFT — net-new general reflex]`

**Trigger:** about to overwrite/flip *any* tracked state file in place — `settings.json`, `data.json`, any
JSON/YAML the system reads.
**The reflex:** `cp file file.bak-pre-<reason>` (or `.bak-<ISO>`) **immediately before** the destructive
write, so a rollback always exists. Offer the backup's `rm` to the *user* as hygiene; never auto-delete it;
exclude `.bak` from commits (gitignore `*.bak*`).
**Belt-and-suspenders for in-place string edits:** capture the exact current target line first so the edit
matches byte-for-byte (a mis-matched edit is a silent corruption). Run bulk mutations with `DRYRUN=1` to print
every old→new + confirm 0 errors and all targets located, THEN `DRYRUN=0` to apply.
**Gotcha — zsh doesn't word-split unquoted vars like bash.** A file-list in `$var` becomes ONE token in a zsh
loop. Pass literal args; keep existence + collision guards so a mis-split move damages nothing.
*(Σrec ~20; sessions 0d10552b, 0d739f65, 23492bcf, 23f93bfd. Today only a `post-to-chamber.py`-bound bullet.)*

---

### S3 — dry-run-graduation.md  `[DRAFT — net-new general reflex]`

**Trigger:** building/deploying ANY automation that takes destructive or autonomous action — process-killer,
ghost-reaper, emulator-shutdown, fan-out that mutates, a new daemon/hook.
**The reflex:** ship in `DRY_RUN=1` / `LOG_ONLY=1` **by default** for a mandatory soak (24h for daemons, up
to 7d for killers). It logs "would fire" + counts, mutates nothing, exits ~0-cost. Flip to live ONLY after the
log *proves* the trigger is real and correct. Pair with a kill-switch + watchdog auto-disable after N miscalls
+ WARN-before-BLOCK. Rate-limit destructive fires (e.g. 3/hr).
**Gotcha — the inverse failure: stuck-in-dry-run forever.** "Built and tested" but behavior never changed →
grep the action layer for `LOG_ONLY=1` / "would fire" vs "did fire", and verify the wiring actually landed in
`settings.json` (not just the script on disk). The whole point is graduation; a substrate frozen in dry-run is
the silent-failure class (→ S5).
*(Σrec ~30; sessions 0d739f65, 21da9584, 23492bcf, 23f93bfd. Today one bullet in opsys-scheduling line 585.)*

---

### S4 — destructive-op-guards.md  `[DRAFT — net-new general reflex]`

**Trigger:** writing/running anything that kills a process, purges a queue, or clears state.
**Protected-list (never-kill):** a reaper/killer maintains an explicit allow-list it NEVER terminates —
Claude, inbox-server, the Astro dev server. Plus dynamic bounds (idle≥threshold, RAM-headroom, thermal) and
require ALL idle-signals quiet before SIGTERM — never a blind kill.
**Recover-before-purge:** before clearing a dead-letter / stuck queue (localStorage offline-queue, etc.),
extract and surface the FULL payload (paste the user's truncated "…" answer back into the field) BEFORE the
purge. Never destroy unrecovered user work.
**Soft-delete over hard-delete:** route deletions to `~/.Trash` (recoverable) with a grace window, not `rm`;
keep the existence + collision guard so a mis-targeted move damages nothing. Honor a no-git constraint with
plain `mv`, never `git mv`/the git-mutating closer.
**Gotcha — asymmetric coverage.** When two paths should behave identically (offline-queue on the text input
vs the YES/NO button), a safety net on one but not the sibling is a live data-loss bug. Check the sibling.
*(Σrec ~25; sessions 0d739f65, 23f93bfd, 0d10552b. Scattered across screenshot-hygiene + worktree-safety.)*

---

### S5 — silent-failure-detection.md  `[DRAFT — net-new; the highest-leverage safety ref]`

**Trigger:** any "I built/wired/deployed X" claim, OR "why didn't X happen?", OR a coverage/readiness audit.
**The class:** *substrate built but not used* — the script exists on disk but isn't wired; the daemon exists
but isn't loaded; the hook fires but in dry-run; the route's content exists but has no rendering page (silent
404 behind a passing matrix); a brain layer exists but is never queried. The enforcement is **illusory**.
**The reflex:** never write "wired"/"fires"/"enforces" without showing the `settings.json` entry or marking
"NOT YET ENFORCED." Cross-check the claim against ground truth: `settings.json` + script-on-disk + a grep of
the wiring. For routes, cross-check every content item against an actual rendering route AND that listing-page
links resolve (catches 200-via-UI-but-direct-URL-404 + dead `href=#` anchors).
**The sweep form (built-but-unused inventory):** enumerate capabilities on disk but not activated, with a
count + risk-ranked activation order ("53 hooks exist, 28 wired = 25 unwired").
**Gotcha — `grep -c` exits 1 on zero matches** and `$(grep -c ... || echo 0)` double-emits `0\n0`, silently
corrupting later JSON fields. Use `grep -c 2>/dev/null; true` + defensive `: "${var:=0}"`.
*(Σrec ~15; sessions 0d739f65, 21da9584, 2fefd8f6, 23492bcf. The recurring A18/A40/A48/A50/A69 finding.)*

---

### S6 — rollback-recovery.md  `[DRAFT — 3-line STUB → existing depth]`

**Trigger:** "can we undo this?", a bad redesign, recovering a past file state.
**Reflex:** the agent's *entire* rollback toolkit is **read-only** — detached `git checkout <sha>` browse +
`git show <sha>:path > file` + `git diff`/`git log`. Everything mutating (`revert`/`reset --hard`/`cherry-pick`)
is **USER-RUN** (or the dashboard emits the command). Git history is immutable + reflog keeps 30d → you can
always go back.
**Depth lives in:** `agentic-quality-discipline/references/rollback-discipline.md` (full strategies + the
dual-author Cursor etiquette). This stub is the front-door pointer; do not duplicate.

---

### S7 — verify-not-fabricate.md  `[DRAFT — 3-line STUB → existing depth]`

**Trigger:** a subagent/BG reports work as done; about to claim "fixed/100%/production-ready/all green."
**Reflex:** never trust a "completed" narration — verify against ground truth (`git log --since=<dispatch>`,
file mtime, grep the wiring) before trusting; a "completed" task can have not-landed integration (the ghost
class). Never declare done from absence-of-a-negative-signal (retry button gone ≠ fixed); require positive
evidence; report "built — awaiting your check," not "solved." Treat human-only-verified (not agentically
reproducible) as NOT ready. On a clock-skew (future epoch) → emit verbatim + flag, never text-arithmetic a
plausible duration.
**Depth lives in:** `bg-dispatch-architecture.md` §Agent-fabrication-detection (lines 230-274) +
feedback_user-owns-done + feedback_verified-counts. Stub = front-door pointer.

---

## C. Draft-now shortlist (priority order)

The user green-lit drafting tight refs, NOT creating skill dirs. These 7 are written above, ready to drop into
`.claude/skills/agentic-safety/references/` when the dir is created:

1. **S5 silent-failure-detection** — highest leverage; the most-recurring, most-expensive class (illusory
   enforcement + silent 404s). Draft-now #1.
2. **S1 gated-ops-and-handoff** — the front-door reflex + the over-gating self-correction the user hit 3×.
3. **S3 dry-run-graduation** — net-new general reflex; today only a stray bullet.
4. **S2 backup-then-mutate** — net-new general reflex; today chamber-bound only.
5. **S4 destructive-op-guards** — protected-list + recover-before-purge + soft-delete + asymmetric-coverage.
6. **S6 rollback-recovery** (stub) / 7. **S7 verify-not-fabricate** (stub) — cheap pointer-stubs; write last.

SKILL.md description should trigger on: "about to delete/push/kill/overwrite," settings.json/launchd/plist,
dry-run, backup before edit, soft-delete/Trash, deny-list blocked, "is this safe/reversible," "did you run
anything irreversible," rollback, fabrication/verify-against-ground-truth, silent failure / built-but-unused.

---

## D. EXTENSION appends (1-2 lines each → matching scratchpad)

- `agent-recommendations/workflows.md` ← frontmatter-status-reconciliation; reread-substrate-before-realign.
- `agent-recommendations/multi-engine-testing.md` ← flake-vs-real triage (re-run the one test pre-push).
- `decision-handling-discipline` (skill ref, via its scratchpad) ← explain-tradeoffs-not-echo;
  brainstorm-before-structural-change ("lmk if bad move" → go/no-go FIRST); over-gating self-correction.
