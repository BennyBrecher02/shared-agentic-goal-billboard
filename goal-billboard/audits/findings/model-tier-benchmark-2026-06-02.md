---
title: "Model-tier benchmark (Haiku vs Opus on OUR data) — verified, not asserted"
date: 2026-06-02
type: findings
audit_kind: empirical benchmark (3 tasks × Haiku/Opus, ground-truth scored)
trigger: "user 2026-06-02 — 'we need genuine benchmarking on our data to verifiably prove improvement or degradation; opus 4.8 feels fine so far.' (skeptical of the research doc's 1-Opus+N-Haiku claim)"
---

# Model-tier benchmark — Haiku vs Opus on our repo (2026-06-02)

Three tasks from THIS repo, each run at `model:haiku` AND `model:opus` (Agent subagents), scored against a
ground truth I computed independently. The user was right to demand this — it surfaced a real nuance the
research only hypothesized.

| Task | Ground truth | Opus | Haiku | Result |
|---|---|---|---|---|
| **B1 — pure mechanical** (count `.sh` under scripts/ + list scripts/*.sh) | 188; 72 basenames | 188 + full list ✓ | 188 + full list ✓ (identical) | **TIE — both perfect** |
| **B2 — light classification** (6 files → {writer/reader/guard/test/tool}) | (my key) | all reasonable ✓ | **identical to Opus** ✓ | **TIE** |
| **B3 — find-the-needle audit** (which of 4 files have a hardcoded `/Users/<user>/` path) | `project-config.json` = **1 line (L7)**; staged-plist = 4; 2 clean | **exactly 1 hit (L7) ✓ + 4 ✓ + 2 clean ✓ — PRECISE** | found L7 ✓ but **+5 FALSE POSITIVES** (claimed hits on L23/32/35 = a doc-string + port numbers with ZERO `/Users/` content) | **Haiku DEGRADED — cried wolf** |

## VERDICT (verified on our data)
- **Haiku == Opus on pure-mechanical gathering** (enumerate / grep / count / list / light-classify). Proven.
  Downgrading these legs is SAFE — identical output, 80% cheaper.
- **Haiku DEGRADES on the audit/judgment leg** — it fabricated hardcoded-path "hits" on lines that are
  literally port numbers. This is exactly the research's hypothesized risk #1 ("Haiku-misses/over-flags the
  bug"), now CONFIRMED with our data. On a real portability/security/scrutiny fan-out, a Haiku worker would
  inject false findings the synthesizer then has to disprove.

## DECISION (refines the model-routing research; the user's call)
**The "1-Opus-synthesizer + N-Haiku-workers" pattern is safe ONLY when the Haiku workers GATHER, never JUDGE.**
- ✅ Haiku for: enumerate, grep, extract, count, reformat, light-classify (bulk mechanical fan-out).
- ❌ Haiku for: audits, find-the-issue, scrutiny, correctness/security detection, synthesis — keep Opus.
- Since **most of our fan-out IS judgment** (audits, scrutiny, code review, synthesis), the realistic cost
  savings are modest and the false-positive risk is real → **recommendation: STAY Opus-by-default (the user's
  "feels fine" is data-justified for our workload); reach for explicit Haiku ONLY on clearly-mechanical bulk
  gathering (e.g. the 188-file enumerate), never on a detection/audit leg.** A74 (OMIT→inherit) stays the default.

This is the empirical backing for `research/external/claude-3-tier-model-routing-2026.md`'s risk gate.
Models used this session: all 28 prior subagent transcripts = `claude-opus-4-8` (audited); these benchmark
agents = explicit haiku/sonnet/opus (the only sanctioned use of the explicit `model` param — benchmarking).

## SONNET ADDENDUM (2026-06-02 — completing the benchmark; the user caught that I'd tested only the extremes)
The first run tested only Haiku (cheapest) vs Opus (frontier). Sonnet — the balanced middle WITH adaptive
thinking + near-Opus SWE-bench + 1M context — is the tier most relevant to cost-saving on JUDGMENT work, and
it was untested. Re-ran the discriminating legs at Sonnet:

| Leg | Opus | **Sonnet** | Haiku |
|---|---|---|---|
| **B3** audit-needle (find hardcoded `/Users/` path; GT = 1 hit on project-config L7) | precise: 1 hit ✓ | **precise: 1 hit ✓** | over-flagged: 6 (3 fabricated on port-number lines) ✗ |
| **B4** careful-read (which set-line has `-e`; GT = only sync-memory) | perfect ✓ | **perfect ✓** | (not run; B3 is decisive) |

**REVISED 3-TIER VERDICT:**
- **Sonnet MATCHES Opus on the judgment legs (B3 precise, B4 perfect) — at −40% cost + ~2× speed.** It — not
  Haiku — is the genuine cost-saver for our judgment-heavy fan-out.
- **Haiku** matches only on pure-mechanical gathering (B1/B2); it DEGRADES (false positives) on audit/judgment.
- So the better pattern than "1-Opus + N-Haiku" is **"Sonnet workers + an inherited-Opus synthesizer"** —
  Opus-level precision on judgment legs at −40%, with Haiku reserved for truly-mechanical bulk (enumerate/grep).
- The user's "Opus 4.8 feels fine" holds — and Sonnet would *likely* feel just as fine on fan-out at −40%.
  **Next step before changing the default: a real-task A/B (a genuine audit/scrutiny leg at Sonnet vs Opus),
  not just these probes.** A74 (OMIT→inherit-Opus) stays the default until such a trial earns the switch.
