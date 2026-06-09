---
idea_id: I-015
title: Longitudinal "prove we're actually getting better" improvement-verification metric
lifecycle: captured
serves_northern_star: G4   # skill-system maintenance; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: med
related: A28 (skill-health audit — point-in-time) · scripts/run-skill-usage-count.sh
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-015 — Longitudinal improvement-verification metric (a trend line, not a snapshot)

**What it is.** A metric that demonstrates the system genuinely improves **over time** — distinct from the
point-in-time skill-health audit. Snapshot the skill-metrics JSON per session-bundle and chart
skill-count / ref-depth / use-coverage over time, so "are we trending better" becomes an observable line, not
a feeling.

**Why it matters.** The skill-health audit (A28) + `scripts/run-skill-usage-count.sh` answer *"what's the
state now"*; they do NOT answer *"are we trending better."* The user asked the trend question directly
(23f93bfd, 2026-05-26): *"the longer we develop the better our skills have gotten right? but how can we
confirm that? we need an audit and new metrics and stat tracking."* No longitudinal/trend metric exists on
disk today — skill-metrics is current-state only, so his actual question has no measurement.

**Concrete first step.** Snapshot the skill-metrics JSON into the per-session-bundle capture (G11), then add
a simple trend chart (skill-count / ref-depth / use-coverage over session-bundles) to the dashboard so the
improvement curve is visible across time.

**Leverage: MED.** Awaiting greenlight (surface-only by contract).
