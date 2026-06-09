---
audit_id: A7
title: Style-token audit (hardcoded values vs design tokens)
status: available
catalogued: 2026-05-26T10:30:00Z
priority_when_run: P1
estimated_effort: medium
trigger: Before any major design refactor OR when 3+ new pages land in src/pages/ OR before client handoff OR when a typography/color regression is reported
deferral_reason: |-
  Phase 5 site overhaul (G2) is queued behind G1 scheduler. Style tokens are stable (brand colors locked: green #2DC856, dark teal #1A2F3A / #0C1C25; Outfit font). Drift hasn't been observed yet but accumulates silently across many edits.
related_goals: [G2]
related_plans: []
related_refs:
  - .claude/skills/agentic-page-scrutiny/SKILL.md
  - .claude/skills/agentic-quality-discipline/SKILL.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
serves_guiding_light: G1
---
# A7 — Style-token audit

## Why this audit matters

Design tokens (`--primary`, `--bg-dark`, `--ease-std`, the Outfit font stack) are the single source of truth for the site's brand. Hardcoded values that should reference these tokens are silent design debt — they don't fail any test, but they:
- Drift the brand (a hex color *almost* matches `--primary` but not quite)
- Break dark-mode / theme-swap if added later
- Make refactors expensive (find-and-replace across N files instead of editing one token)

Brand colors and typography are locked. Stylesheet hygiene is the open variable.

## What it would look at

**Color values:**
- Search `src/` for hex colors (`#[0-9a-fA-F]{3,8}`) that match or near-match brand tokens
- Search for `rgb(...)`, `rgba(...)`, `hsl(...)` literals that should reference vars
- Detect "almost" matches: `#2DC857` (off-by-1 from `--primary`)

**Typography:**
- Search for `font-family:` declarations that hardcode Outfit instead of using the body font inheritance or `var(--font-display)`
- Search for `font-size:` with literal pixel/rem values that should be in the type-scale tokens

**Easing / motion:**
- Search for `transition: ... <bezier curve>` literals that should reference `--ease-std`, `--ease-in-out`, etc.
- Search for `transition-duration` values that should reference `--dur-fast`, `--dur-base`

**Spacing / radius:**
- Search for spacing values that don't fit the spacing scale (e.g., 13px when scale is 4/8/12/16/24/32)
- Border-radius literals that should reference `--radius-sm`, `-md`, `-lg`

**Z-index:**
- Search for `z-index:` literal values; should reference a z-scale variable

## Expected outputs

- `notes/style-token-audit-{date}.md` with per-file findings
- A categorized table: "drop-in fix" (hex → var) vs "design decision needed" (off-by-1 — keep or normalize?)
- A patch (or several PRs / changes through the scheduler) that converts the drop-in fixes
- An updated section in `src/styles/app.css` if new tokens need to be defined (e.g., a missing `--ease-bounce` that's appearing repeatedly)
- Recommendations for tokens we've outgrown (e.g., 4 places use `--primary-light` which doesn't exist — should it?)

## How to trigger

Any of these:
- Before any major design refactor
- 3+ new pages added to `src/pages/`
- Before client handoff
- A typography/color regression is reported
- User says "audit the styles"

## What NOT to catch

- Intentional one-offs (e.g., a CSS-art element using exact pixel positioning — that's not a token violation)
- Third-party component overrides where the values are dictated by the library
- Legacy components marked LOCKED (`HeaderBatteryLefter.astro` per project memory)

## Possible downsides (low, mostly nuance)

- **First-run noise.** A site that hasn't had this audit before will surface many findings. Expect to discard maybe 30-40% as intentional.
- **Token-design feedback loop.** Sometimes the right fix is "add a missing token," not "use the existing one." The audit produces both kinds of findings.
- **CSS detection edge cases.** `getComputedStyle` results, shadow tokens, `color-mix()`, and dynamic CSS all evade simple regex. Coverage is "static `src/`-tree" not "everything that ends up in the DOM."

## Implementation notes (when run)

- Use `rg` or `grep -E` with patterns rooted at `src/`
- Skip `node_modules`, `dist`, `.astro` (already gitignored)
- Cross-reference findings against the canonical token list (extract from `src/styles/app.css`)
- For the patching step: prefer scheduler changes (one per file or one per component group) over a giant single PR

