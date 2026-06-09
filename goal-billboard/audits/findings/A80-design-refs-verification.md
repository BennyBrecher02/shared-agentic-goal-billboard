---
title: A80 — Adversarial verification of the 12 design-system vocab refs
created: 2026-05-29T05:52:00Z
ns_context: A80-design-refs-verification
trigger_source: READ-ONLY adversarial QA pass over the design-system/*.md refs drafted overnight
mode: read-only (no fixes applied — flags for the morning)
schema_version: 1
---

# A80 — Design-system refs verification

Scope: the 12 new refs + README under
`.claude/skills/agentic-webdesign/references/design-system/`
(skipped the pre-existing `light-section-pattern.md`). Each ref was checked,
adversarially, against the REAL `src/styles/app.css` (1558 lines) plus the
components/pages it cites (`Hero.astro`, `ScrollHint.astro`, `CtaSplit.astro`,
`reveal.ts`, `highway.astro`, `index.astro`, etc.).

Method: full read of `app.css`; targeted `grep`/`sed` confirmation of every
load-bearing value, line citation, frequency census, and class-usage count;
independent WCAG contrast recomputation (Python, sRGB, sanity-checked against
black/white = 21.0).

---

## Adversarial verification (read-only)

### `typography-roles.md` — ⚠ minor

- ✅ **Display ramp values EXACT** (app.css:137-142). All six roles' size /
  weight / line-height / tracking verified character-for-character:
  xl `clamp(64px,9vw,128px)`/700/0.96/-0.025em · lg `clamp(42px,5.2vw,76px)`/700/1.04/-0.022em ·
  md `clamp(34px,3.8vw,54px)`/600/1.08/-0.018em · sm `clamp(32px,3.4vw,48px)`/600/1.1/-0.014em ·
  headline-lg `clamp(26px,2.4vw,36px)`/500/1.25/-0.005em · quote `clamp(28px,2.6vw,40px)`/300/1.4/-0.005em.
- ✅ **Mobile step-down block EXACT** (app.css:147-154), incl. xl line-height 0.96→1.03.
- ✅ **Outfit declarations** at app.css:23 (`--font-sans`) and :68 (`--font`); `font-feature-settings: "lnum" 1, "ss01" 1` at :111. All correct.
- ✅ **Body-prose role sizes EXACT**: `.card-row .cell p` 14px/1.65 (:650), `.process-list .step p` 15px/1.6 (:716), `.faq-item .a` 15px/1.65 (:574), `.hero-headline-sub` clamp(17px,1.4vw,20px)/1.55 weight 300 (:426-429), `.lede-copy` 17px/1.6/54ch (:250).
- ⚠ **The "101 uses" eyebrow count is an overcount as stated.** The ref's
  Sources block says "eyebrow 101" (and the prose calls it "101 uses — more
  than every display role combined"). Reality:
  - raw `grep -roh eyebrow src/` = **102** (this is the number the ref rounded from)
  - but that raw count includes ~36 occurrences inside `app.css` itself, many of
    which are the **`--eyebrow-padding-inline` token / `.hero-top` comments**, not class usages.
  - true **markup** usages (`.astro` `class=` attributes) = **66**.
  So "101 uses" conflates the token name + CSS comments with actual class
  applications. The *relative* claim (eyebrow is the most-used role) still holds —
  but the literal number should be ~66 (markup) not 101. Same overcount is
  repeated in `accent-bar-motif.md` ("101 uses").
- ⚠ **Other usage counts are method-sensitive but directionally right.** Raw
  grep gives t-display-md 31, t-display-xl/lg 9 each, t-quote 6, sec-foot 41,
  sec-counter 32, lede-copy 23 — these match the ref. Markup-only is slightly
  lower (e.g. t-display-md 29, lede-copy 22) because each has ~2 CSS def lines.
  Not load-bearing; flag only that the counts are "grep including CSS defs," not pure usages.

### `letter-spacing-ladder.md` — ✅ clean

- ✅ **Frequency census EXACT.** `grep -oh "letter-spacing: [^;]*" | sort | uniq -c`
  reproduces the ref's census line-for-line: `0.22em ×30 · 0.18em ×14 ·
  -0.005em ×7 · -0.01em ×5 · -0.015em ×4 · 0.04em ×3 · -0.02em ×3 ·
  -0.025em ×3 · 0.24em ×2 · 0.16em ×2 · 0.06em ×2 · 0.05em ×2 · 0.2em ×1 ·
  0.12em ×1 · 0.02em ×1 · -0.04em ×1 · -0.022em ×1 · -0.018em ×1 · -0.014em ×1`.
  Every count is exact.
- ✅ Two-arm model, the `0.22em`-default / `0.18em`-button / `0.24em`-peak rungs,
  the `-0.04em` wordmark outlier (app.css:1035), and the `0.16em` ≤380px tightening
  (app.css:168) all verified against source.
- ✅ The negative display gradient maps correctly to the ramp roles.
- (Best-grounded ref in the set — the census is reproducible byte-for-byte.)

### `color-mortise.md` — ⚠ discrepancies (one material: a WCAG claim is wrong)

- ✅ **Palette hexes EXACT** (`@theme` :14-24 + `:root` :30-41): primary #2dc856,
  primary-deep #29b54e, teal #1a2f3a, teal-deep #0c1c25, plaza #eef1f5,
  surface #ffffff, sage #5c7a6a, on-surface-variant #505c65, ink-on-primary #062114,
  outline-variant #dde3eb, focus-ring `0 0 0 0.25rem rgba(45,200,86,0.28)`.
- ✅ **The four bg-* classes** (:180-183) and their fg pairings — correct.
- ✅ **`data-tone` is real** (41 occurrences); `index.astro:78` is indeed
  `bg-teal-deep ... data-tone="dark"` as cited.
- ✅ **Per-page scrim overrides** all verified: `.prod-hero .bg-tint` 96° gradient
  (:1188-1192, with the `0.92→0.74→0.32→transparent` stops post-CB-1), object-position
  30% (:1128); `.highway-hero .bg-tint` dark band to 65% @ 0.70 (:1239-1243) + the
  object-position cascade 50%→42%→30%→50% (:1155-1166); `.services-hero .bg-tint`
  to 60% @ 0.65 (:1283-1287). `extraClass` hooks confirmed at highway.astro:48,
  fleets.astro:33, services.astro:42, products.astro:147.
- ✅ **The `.sec-foot 0.4→0.6` contrast story is EXACT.** Recomputed: rgba(255,255,255,0.4)
  composited on `--teal` = **3.52:1** (matches the app.css:259-266 comment to 2 d.p.);
  lifted 0.6 on `--teal-deep` = **6.89:1** (ref "~6.9:1" ✓), 0.6 on `--teal` = **6.01:1**
  (ref "~6.6:1" — close, slightly overstated).
- ⚠ **MATERIAL ERROR — the green-on-white contrast claim is wrong and gives a false WCAG pass.**
  The "Contrast verification" table asserts:
  > `--primary` (#2dc856) on `#fff` → ~4.8:1 (AA large)
  Recomputed (standard WCAG sRGB): **#2dc856 on #fff = 2.21:1.** That FAILS
  AA-large (needs 3.0) and AA-normal (needs 4.5). The ref is off by ~2× and then
  uses the bad number to conclude "All four pairings meet WCAG." Green-on-white
  text genuinely does *not* clear AA at body size — which is why the real design
  only uses green-on-white for large/non-critical accents (eyebrows, kickers),
  never body copy. **Recommend: correct to "~2.2:1 — fails AA; reserved for large
  accent text / non-text UI only" so the ref stops implying green body text on
  white is safe.**
- ⚠ **Other hand-estimated ratios are imprecise (none as bad as the above).**
  Recomputed vs claimed: on-surface #1a2f3a on #fff = **13.89:1** (claimed ~17:1,
  overstated but still AAA); #fff on teal-deep = **17.38:1** (claimed ~15:1,
  understated); primary on teal-deep = **7.87:1** (claimed ~5.6:1, understated —
  actually better); ink-on-primary #062114 on green = **7.70:1** (claimed ~9:1,
  overstated but still ≥7 AAA). All except the green-on-white land on the same
  pass/fail verdict the ref gives; only the magnitudes drift. The green-on-white
  one flips the verdict and is the one to fix.

### `accent-bar-motif.md` — ✅ clean (carries the shared eyebrow-count nit)

- ✅ Every instance citation verified against source: `.eyebrow::before` 28×1px
  (:161), `.no-rule` (:162), ≤380px → 20px (:169); `.arr` 18×1px hover→28px
  (:308-316), `.hero-link .arr` 22→36px (:350-352); `.btn-text::after` 2px width
  0→100% (:337-341); `.route-card .go` border-bottom (:1009-1018); `.spec-card .stage`
  border-top `--primary` + green wash (:773-775) with P1-07a comment (:755-766);
  `.pain p.sol` border-top rgba(45,200,86,0.3) (:701); em-dash bullets (:738, :1305);
  `.live` (:441-448); FAQ `+` crossbars (:569-571); footer wordmark -0.04em (:1035);
  `.rule-soft` (:1072). All EXACT.
- ⚠ Inherits the **"eyebrow 101 uses"** overcount (see typography-roles entry). Same fix.

### `staged-reveal-choreography.md` — ⚠ minor (one self-contradiction on easing)

- ✅ **Reveal grammar EXACT** (:281-298): opacity 0→1, translateY(28px)→0,
  `transition: opacity 900ms var(--ease-emp), transform 900ms var(--ease-emp)`,
  delays d1-d5 = 100-500ms, the triple-`!important` reduced-motion reset.
- ✅ **Easing token values EXACT**: `--ease-std: cubic-bezier(0.4,0,0.2,1)`,
  `--ease-emp: cubic-bezier(0.2,0,0,1)` (:65-66).
- ✅ **reveal.ts verified EXACT**: `rootMargin: '0px 0px -8% 0px', threshold: 0.05`,
  `io.unobserve(entry.target)`, and the reduced-motion short-circuit
  `els.forEach((el) => el.classList.add('in-view'))`. Matches the ref's snippets.
- ✅ **CtaSplit.astro EXACT**: `class="copy reveal"` (step 0) + `class="shot reveal d2"` (+200ms).
- ✅ **ScrollHint comet EXACT**: gradient `linear-gradient(180deg, transparent 0%, var(--primary,#2dc856) 100%)`, `scroll-hint-fall 1600ms linear infinite`, delays 400ms/800ms, `animation: none` under reduce (ScrollHint.astro:59-86; ref says 59-87 — off by 1, harmless).
- ✅ **reduced-motion coverage EXACT**: "6 hits across 4 files" confirmed
  (ScrollHint.astro, reveal.ts, blog.astro, app.css).
- ✅ **"1558-line stylesheet" correct** (`wc -l` = 1558; the Read tool's "1559"
  counts a trailing line — the ref's 1558 is right).
- ⚠ **Self-contradiction on the easing count.** The ref's own body correctly
  documents `prod-modal-rise` as `cubic-bezier(0.16, 1, 0.3, 1)` ("overshoot-free
  expo-out", line ~188) — i.e. a THIRD distinct curve. But its **Bans** section
  states: *"A third easing. `--ease-std` … and `--ease-emp` … are the two curves.
  Don't introduce a third `cubic-bezier`."* The code actually contains **four
  distinct cubic-beziers** in app.css:
  - :65 `(0.4,0,0.2,1)` `--ease-std`
  - :66 `(0.2,0,0,1)` `--ease-emp`
  - :1087 `(0.4,0,0.2,1)` (fp-nav inline dup of std — same curve)
  - :1345 `(0.25,0,0,1)` on `.prod-card-img` transform — **a 3rd curve, undocumented**
  - :1434 `(0.16,1,0.3,1)` on `prod-modal-rise` — **a 4th curve, cited in-body but denied by the ban**
  So "only two easings" is **not literally true**. The defensible framing is
  "two *tokenized/reusable* easings (`--ease-std`, `--ease-emp`); the two modal/
  card one-off curves are scoped literals." Recommend rewording the ban from
  "there are only two curves" to "don't add a third *token* / reusable easing"
  so it stops contradicting the code (and this ref's own modal citation).
- minor: delay-class usage counts ("~37 / ~34 / ~41 / ~18 / 1 / 0") are prefixed
  "~" and are plausible-but-unverified-exact; treated as illustrative, fine.

### `named-restraint.md` — ⚠ discrepancies (the "2 easings" closed-set claim is overstated)

- ✅ **@keyframes = 4 total** EXACT: pulse (:445), prod-modal-in (:1421),
  prod-modal-rise (:1443) in app.css + scroll-hint-fall in ScrollHint = 4.
- ✅ **border-radius = 3, none decorative** EXACT: `50%` (:442 live dot),
  `0` (:939 square inputs), `50%` (:1086 fp-nav dot).
- ✅ **No `prefers-color-scheme` anywhere** (count 0) — refusal #9 holds.
- ✅ **One typeface** — only Outfit + system fallbacks; no `@font-face`, no second webfont import.
- ✅ **Three greens** verified: #2dc856, #29b54e, #7DFA9F (the last at :1289 + :1404 only).
- ✅ **Hero.astro autoplay** EXACT: line 110 `<video class="bg-video" autoplay muted
  loop playsinline preload="auto">` + the `visibilitychange`/`video.play()`
  keep-alive at ~188-197 (ref says 185-199 — close).
- ⚠ **"2 easings" closed-set claim is overstated** (same root issue as the
  staged-reveal contradiction). The verification table row says
  `grep "cubic-bezier" app.css` → "only `--ease-std` + `--ease-emp` (+ the fp-nav
  inline dup of std)." That grep actually returns **5 lines / 4 distinct curves**
  (see staged-reveal entry: the `(0.25,0,0,1)` prod-card-img curve and the
  `(0.16,1,0.3,1)` modal-rise curve are NOT the two tokens and are NOT the
  fp-nav dup). So refusal #5 "No third easing curve" and the closed-set headline
  "2 easings" are literally false as written. Same fix: scope the claim to
  "2 reusable easing *tokens*," and acknowledge the 2 scoped one-off modal/card curves.
  (This propagates to `README.md` line 41 and `premium-feel-formula.md` Term 2,
  which both repeat "2 easings.")
- ✅ The `border-radius (3)`, `@keyframes (4)`, `prefers-reduced-motion (6/4 files)`
  closed-set numbers are all exactly right — only the easing number is wrong.

### `section-padding-rhythm.md` — ✅ clean

- ✅ **`.section-inner` padding EXACT** (:229): `clamp(30px,4.2vh,58px)
  clamp(24px,4vw,64px) clamp(20px,2.6vh,32px)`. `.flush-top` `clamp(34px,4.6vh,64px)` (:233).
- ✅ **Margin-zeroing reset** at :119-120 — correct, and the "section owns the
  rhythm, not paragraphs" thesis matches the code.
- ✅ Every special-band citation verified: hero-body :384, foot :1027, stat :461,
  app-half :584, cta-copy :1049, sec-foot :269; `.lede` rhythm :240/:242/:244/:250;
  `.mini-stats` 40px :523; mobile overrides :452-456 / :605-610; fullPage min-height
  auto :208-210. All EXACT.
- ✅ The `vh`-vertical / `vw`-horizontal split is an accurate reading of the file.

### `section-archetypes.md` — ✅ clean

- ✅ **All chassis + archetype line ranges verified** against source: `.section`
  :190-219, `.section-inner` :224-233, `.lede` :237-256, `.sec-foot` :267-275;
  hero :361-456, stats-band :459-483, card-row :637-658, spec-row/card :745-786,
  surface-card :660-691, models :723-742, compliance-row :859-869, connectors/brands
  :1497-1543, pain-grid :694-705, process-list :708-720, tier-compare :1293-1309,
  faq-wrap :541-578, cta-split :1046-1069, quote-section :613-634, form-wrap :878-1022,
  app-split :581-595, coverage :513-538, portal-grid :486-510. Every range correct.
- ✅ **The quoted /highway §04 template matches the real file nearly verbatim**
  (highway.astro Hardware section ~127-152): `bg-surface`, `data-tone="light"`,
  eyebrow `— DC fast hardware`, `t-display-md text-ink`, `spec-row reveal d2`,
  `sec-foot dark` with `04 / 07`. Confirmed.
- ⚠ trivial: ".lede used 25 times" — markup grep = **24** (off by 1). "sec-foot on
  essentially every section" — markup = 37, consistent. Not load-bearing.

### `offset-grid-rivers.md` — ✅ clean

- ✅ **Every grid ratio + line citation EXACT**: `.lede` 1fr:1.3fr (:239),
  `.coverage` 1fr:1.4fr (:514), `.faq-wrap` 1fr:1.4fr (:542), `.portal-grid`
  1.1fr:1fr (:487), `.cta-split` 1.2fr:1fr (:1046), `.form-wrap` 1.2fr:1fr (:878),
  `.foot-grid` 1.4fr:repeat(3,1fr) (:1030).
- ✅ `--container: 1280px` (:48), `.container` gutter `clamp(24px,4vw,64px)` (:186),
  `.is-container` (:232), `--eyebrow-padding-inline` (:63), `.lede` gap
  `clamp(36px,4.5vw,80px)` (:240), `min-width:0` on `.hero-top > *` (:413) and
  `.spec-card` (:750), the 991px single-col collapse lines (:255/517/545/490/1047/879),
  `.prod-grid` gap `clamp(20px,1.6vw,28px)` (:1312). All verified.
- ✅ The "asymmetric never-50/50" thesis and the `minmax(0,Nfr)` overflow rationale
  are accurate readings of the code.

### `section-as-stage-set.md` — ✅ clean

- ✅ Hero stage layers verified: backdrop :362-364 (z:0), scrim `.bg-tint` :365-370
  (z:1), `.hero-top`/`.hero-rail-info` z:3 (:405/:436), `isolation: isolate` (:361),
  `justify-content: flex-end` (:383), `.is-centred` (:386), `.lede align-items:end` (:241).
- ✅ Per-hero scrim override citations (:1188-1290) — correct.
- ✅ Mini-stages EXACT: `.spec-card .stage` :767-778 (the literally-named `.stage`),
  `.prod-card-stage` :1326-1356, L3 night variant :1364-1377.
- ✅ `"A section = one full-viewport idea."` is a real comment at app.css:189.
- No discrepancies. (Doctrine ref — fewer hard numbers, and the ones it cites check out.)

### `premium-micro-details.md` — ⚠ trivial (one imprecise grep command; prose number correct)

- ✅ **Every hover-wash line EXACT**: stats :466 (0.06), portal-card :501 (0.06),
  surface-card :667 (0.06), app-half :588 (0.04), state-active:hover :520 (0.45),
  model.featured :729 (0.04), tier-pane :1295 (0.04), chip.featured :686 (0.08).
- ✅ **Card lifts EXACT**: route-card `0 10px 24px -12px rgba(26,47,58,0.18)` (:1002,
  negative-spread reading correct); prod-card `0 24px 56px rgba(12,28,37,0.10), 0 4px
  12px rgba(12,28,37,0.04)` (:1324) + border-color rgba(45,200,86,0.35); prod-card-img
  `scale(1.04)` (:1355). L3 dark `0 28px 60px rgba(0,0,0,0.45)` (:1360-1363).
- ✅ **Primary glow EXACT**: resting `0 12px 28px rgba(45,200,86,0.22)…` + hover `…0.32…` (:320/:324).
- ✅ `#7DFA9F` is exactly 2 occurrences (:1289, :1404) — claim holds.
- ✅ ScrollHint comet anatomy, focus-ring (:43), `::selection` (:121), eyebrow
  hairline, spec-card seam, modal-close rotate (:1449), socials inversion (:973) — all verified.
- ⚠ trivial: the prose "**26 times**" for `rgba(45,200,86)` is **correct** (21 with
  the space variant `rgba(45, 200, 86` + 5 no-space `rgba(45,200,86` = 26). BUT the
  cited command in Sources — ``grep -c "rgba(45, 200, 86" app.css → 26`` — actually
  returns **21**, because it omits the no-space instances. Number right; command as
  written is reproducibly off. (Same "26" repeated in premium-feel-formula — also fine as a number.)

### `premium-feel-formula.md` — ⚠ minor (inherits "2 easings")

- ✅ Synthesis is internally consistent and its concrete citations (type ramp
  :137-153, tokens :30-43, focus-ring :43, selection :121, reveal :281-298,
  spacing :224-275, primary glow :318-326, washes, spec seam :773-774, product
  studio :1326-1356) all resolve to real, correct lines.
- ✅ "26 times" green count — correct number (see above).
- ⚠ Term 2 repeats the **"2 easings"** closed-set figure (line 60) — same
  overstatement as named-restraint (4 distinct cubic-beziers exist; 2 are tokens).
  Fix once at the source (named-restraint) and propagate to this ref + README.

### `README.md` — ⚠ minor (inherits "2 easings")

- ✅ Lists 13 refs accurately; the one-line summaries match each ref's content;
  the "8-axis rubric → ref" mapping is sane.
- ⚠ Line 41 repeats the **"2 easings"** closed-set claim — same fix as above.
- nit: header says "12 sibling refs … codified overnight" / "(13 refs)" /
  "All 12 are status: draft" in different spots — internally consistent if you
  read it as "12 new + 1 pre-existing active (light-section-pattern) = 13 in the
  family." Not an error, just worth a sentence to disambiguate.

---

## Cross-ref contradiction summary

1. **"2 easings" (the one real cross-cutting falsehood).** `named-restraint.md`
   (refusal #5 + closed-set headline), `staged-reveal-choreography.md` (Bans),
   `premium-feel-formula.md` (Term 2), and `README.md` (line 41) all assert exactly
   two easing curves. Code has **four distinct cubic-beziers** (2 tokens +
   `(0.25,0,0,1)` on prod-card-img + `(0.16,1,0.3,1)` on prod-modal-rise). The
   staged-reveal ref even **cites the modal curve in its own body** while its Bans
   deny a third exists. Single fix: reframe as "2 reusable easing *tokens*; 2 scoped
   one-off curves on the modal/card" everywhere it appears.

2. **"eyebrow 101 uses"** appears in `typography-roles.md` AND `accent-bar-motif.md`.
   Overcount: true markup usages ≈ 66; the 101/102 figure folds in the
   `--eyebrow-padding-inline` token + CSS comments. Relative claim (most-used role)
   is fine; literal number isn't.

3. **No value-vs-value contradictions between refs.** Where two refs cite the same
   thing (display tracking, reveal duration, the four bg tokens, the green hexes,
   the spec-card seam, the `.section-inner` padding), they agree with each other AND
   with the code. No the reference design value was found mislabeled as Evium's — every ref
   is careful to frame the reference design as "source pattern" and cite Evium's own values
   (and the px→em, fixed-rem→clamp, 250ms→100ms, 2.75s→900ms divergences are all real).

---

## Overall confidence verdict

**HIGH confidence in the refs as a whole — safe for the user to rely on after
three small fixes.** This is unusually well-grounded documentation: the letter-
spacing census reproduces byte-for-byte, every structural line-number citation I
spot-checked (dozens across all 12 refs) resolved to the correct rule, the
reveal/ScrollHint/CtaSplit/Hero code snippets are verbatim, and the
@keyframes/border-radius/reduced-motion/green-hex closed-set counts are exact.

Three things to fix in the morning, in priority order:

1. **[MATERIAL] `color-mortise.md` green-on-white contrast** — "~4.8:1 (AA large)"
   is actually **2.21:1 (fails AA)**. This is the only error that flips a
   pass/fail verdict and could mislead someone into shipping green body text on
   white. Fix the number and the "all four pairings meet WCAG" conclusion.

2. **[CLASS] "2 easings"** — overstated in 4 places (named-restraint, staged-reveal,
   premium-feel-formula, README). Reframe as "2 reusable tokens + 2 scoped one-offs."
   Resolves a self-contradiction inside staged-reveal.

3. **[MINOR] "eyebrow 101 uses"** — overcount in 2 refs; true markup ≈ 66. Also
   tighten the `rgba(45, 200, 86`/26 grep command in premium-micro-details Sources
   (the *number* 26 is right; the *command* as written returns 21).

Everything else (the structural citations, the values, the cross-ref agreement)
is clean. No fabricated Evium-facts found beyond the contrast estimate and the
easing-set overstatement; both are over-claims of degree, not invented mechanics.

— A80, read-only pass, 2026-05-29T05:52Z. No files modified except this one.

BG-COMPLETE-SENTINEL
