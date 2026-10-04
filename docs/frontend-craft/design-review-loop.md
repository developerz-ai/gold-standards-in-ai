# 🔍 Design Review Loop — critique, audit, harden, polish

**Rule: no UI ships on the builder's word.** Render it, screenshot it, score it with a rubric, fix in one batch, re-screenshot. The builder judges its own work too kindly, so the final verdict comes from a **fresh-context reviewer**. Deterministic tells run as a lint guard, not as a matter of taste.

Related: taste rules → [design-taste.md](design-taste.md) · token/system spec → [design-md.md](design-md.md) · agent eyes (Playwright MCP) → [README](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless) · copy review → [ux-copy.md](ux-copy.md) · responsive/native/perf → [adaptive-ui.md](adaptive-ui.md) · comp fidelity → [reference-image-design.md](reference-image-design.md) · lazy/truncated output → [output-completeness.md](../writing-for-agents/output-completeness.md).

## 🔁 The loop
```
build fully ──▶ capture (desktop + mobile, one batch) ──▶ validate captures
      ▲                                                         │
      │                                                         ▼
  fix ALL findings in one batch ◀── critique + audit + detector (isolated)
      │
      └──▶ recapture same viewports ──▶ fresh reviewer: verdict ──▶ ship | fix | rebuild | recapture
                                                    └──▶ ship ──▶ documenter updates DESIGN.md
```
| Rule | Why |
|---|---|
| Build the whole surface **before** the first screenshot | Per-tweak screenshots burn tokens and never converge |
| Capture **all target viewports in one round**: web `1440` + `390` wide; add the user's real viewport if known | The width that breaks is the one the user sees first |
| **Build thread inspects once or twice, then hands off** to a fresh reviewer | Open-ended self-QA gets worse results at higher cost than a fresh reviewer |
| Fix **every** finding of a round in one batch, then recapture | Fixing one issue per round never finishes |
| Stop the moment a round resolves nothing | Thrashing means the approach is wrong, not the details |
| After round 2, the reviewer's list is the only work list | Don't start your own new hunt for issues |

### 📸 Capture validity. An invalid capture proves nothing.
- Settle or disable entrance animations first. Otherwise content hidden by animation timing looks missing and gets "fixed" into a regression.
- Full-page shots from document top. Fresh tab per assessment; never reuse a stale one. Wait for fonts: `await page.evaluate(() => document.fonts.ready)`.
- Open every file once before sending it on: no blank/black regions, no half-loaded state, no wrong section behind the filename.
- Never judge fidelity from one full-page thumbnail. Crop regions and compare side by side with the comp/reference.
- Say what produced the evidence (emulated viewport, synthesized touch, real device) and what stayed untested.

## 🧭 Pass order: who owns what
| Pass | Question | Output | Edits code? |
|---|---|---|---|
| **Critique** | Is this good design for *this* product? | Heuristic score /40, P0–P3 issues, persona red flags | ❌ report only |
| **Audit** | Is it technically sound? | Dimension score /20, P0–P3 issues with file:line | ❌ report only |
| **Harden** | Does it survive real data, errors, and locales? | Fixed states + edge cases | ✅ |
| **Directional** | Too timid / too loud / too cluttered? | One targeted refinement | ✅ |
| **Polish** | Is the last 10% consistent and finished? | Aligned, clean, verified diff | ✅ |
| **Detector** | Any mechanical tells? | Rule hits (exit 2 on findings) | ❌ guard |

Order: critique + audit (in parallel, isolated) → harden → directional (if needed) → polish (always last). Keep polish as refinement. If the concept itself is wrong, say so and redesign; don't hide a redesign inside polish.

## 🎯 Critique: heuristics and scoring
**Start with the specificity verdict**, before looking at any detector output: *could an unrelated product use this UI unchanged?* If yes, it fails. Name the category-interchangeable choices. Anti-slop specifics → [design-taste.md](design-taste.md).

### Nielsen 10, each scored 0–4 (honest: 4 = genuinely excellent; most real UIs land 20–32/40)
Anchors for every heuristic: **0** absent/broken · **1** rare · **2** partial, major gaps · **3** good, minor gaps · **4** excellent everywhere.

| # | Heuristic | Check for |
|---|---|---|
| 1 | Visibility of system status | Loading indicators, action confirmation, progress, active nav, inline validation |
| 2 | Match real world | User's vocabulary, no jargon, natural order, recognizable icons |
| 3 | User control & freedom | Undo, cancel, Esc, back, clear filters, exit multi-step flows |
| 4 | Consistency & standards | Same term/action/visual = same meaning everywhere; platform conventions |
| 5 | Error prevention | Confirm destructive ops, constrained inputs, smart defaults, autosave |
| 6 | Recognition > recall | Visible options, labeled icons, recents, autocomplete |
| 7 | Flexibility & efficiency | Shortcuts, bulk actions, power paths that stay hidden from novices |
| 8 | Aesthetic & minimalist | Only necessary info, clear hierarchy, purposeful emphasis |
| 9 | Error recovery | Plain language, names the problem + fix, near the source, keeps input |
| 10 | Help & docs | Contextual, task-focused, reachable without leaving context |

- Landing pages / portfolios: #7 and #10 may be `n/a` with a one-line reason. Renormalize: max = 4 × scored (e.g. `24/32`). Never print `/40` over a partial set.
- Bands by %: **≥90 Excellent** (ship) · **≥70 Good** · **≥50 Acceptable** · **≥30 Poor** · **<30 Critical** (redesign).

### Visual craft checks (score what the render shows, not what the code intends)
| Axis | Pass when |
|---|---|
| Hierarchy | One primary element, 2–3 secondary, rest muted. Primary action obvious in 5 s |
| Type scale | Clear size/weight steps (≥1.25× between roles); body measure 65–75ch; display ≤ ~6rem |
| Rhythm | Tight within groups, generous between; more space above a heading than below it |
| Alignment | Shared grid; baselines/titles/CTAs aligned across sibling cards; optical fixes where needed |
| Contrast | Body ≥4.5:1, large ≥3:1, focus ring visible; no gray text on colored surfaces |
| Consistency | One icon family/stroke, one radius scale, one accent, tokens only |
| Emotional register | Tone matches product/audience; reassurance at high-stakes moments; strong ending (peak-end) |
| Coverage | Every requirement in the brief is present and findable within seconds |

### Cognitive load
Load types: **intrinsic** (the task; structure it) · **extraneous** (bad design; eliminate) · **germane** (learning; support it).
8 checks (0–1 fails OK · 2–3 moderate · 4+ critical): single focus · chunks ≤4 · related items grouped · obvious hierarchy · one decision at a time · **≤4 visible options per decision point** · no remembering across screens · progressive disclosure.
- Count options per decision: ≤4 fine · 5–7 group or disclose · 8+ overloaded. Actions: 1 primary + 1–2 secondary, rest in a menu. Top nav ≤5.
- Name violations by type: wall of options · memory bridge (step 1 info needed at step 3) · hidden location · jargon barrier · flat noise floor · inconsistent pattern · multi-task demand · context switch.

### Personas (pick 2–3, walk the primary task, report what **broke**, not a profile)
| Persona | Probes |
|---|---|
| Power user | Shortcuts, Esc on modals, bulk actions, skippable onboarding, core task < 60 s |
| First-timer | Unlabeled icons, jargon, no success confirmation, next step unclear in 5 s |
| A11y-dependent | Keyboard-only flow, focus visible, alt text, color-only meaning, announced state, time limits |
| Stress tester | 0 / 1000 items, emoji/RTL/pasted-spreadsheet input, refresh mid-flow, two tabs, double submit |
| Distracted mobile | Thumb zone, 44px targets, state kept on app switch, slow network, typing minimized |

| Interface | Personas |
|---|---|
| Landing / marketing | First-timer, stress tester, mobile |
| Dashboard / data / admin | Power user, a11y |
| Checkout / e-commerce | Mobile, stress tester, first-timer |
| Onboarding | First-timer, mobile |
| Forms / wizards | First-timer, a11y, mobile |

Add 1–2 project personas from `PRODUCT.md` users, **only** when real audience data exists.

### Severity (shared by every pass)
| Tag | Meaning | Action |
|---|---|---|
| **P0** | Blocks task completion / data loss | Fix now |
| **P1** | Significant difficulty, WCAG AA fail | Fix before release |
| **P2** | Annoyance, workaround exists | Next pass |
| **P3** | Polish, no real user impact | If time permits |

Tiebreak: *would a user contact support about it?* If yes, it's ≥P1. Too many P3s is noise. Report the top 3–5.

### Close the critique (the report is not the finish)
1. **Report in the reply first**, in full. Persisting it to a file is bookkeeping, not delivery.
2. Persist a snapshot: `docs/design/critiques/<surface>-<date>.md` with score, max, `n/a` list, P0/P1 counts. Print the trend (`24 → 28 → 32 /40`); mixed maxima → show each denominator.
3. **End with 2–4 targeted questions**, each tied to findings, each with 2–3 concrete options: priority (which issue category first) · intent (was this tone deliberate?) · scope (top 3 / all / critical only) · constraints (anything off-limits). Skip only with < 3 priority issues, and then print `Questions skipped: <reason>`.
4. After answers: ordered action list mapping each issue to a pass (harden, directional, polish…), ending with polish.
- Accepted "won't fix" findings go in `docs/design/critiques/ignore.md`; later critiques drop them.
- Polish reads the latest snapshot as its backlog and marks it closed when every priority issue is resolved. A snapshot whose target changed since is stale: re-critique.

### 📋 Critique prompt (paste to a fresh subagent)
```text
You are a design director reviewing a finished UI. You did not build it. Edit nothing.
Inputs: request=<original ask>; target=<file or URL>; screenshots=<desktop.png, mobile.png>;
design system=<DESIGN.md path or "none">; product=<PRODUCT.md path>.
1. Specificity verdict first: could an unrelated product use this UI unchanged? Name the interchangeable choices.
2. Score Nielsen's 10 heuristics 0-4 in a table (n/a allowed for #7/#10 on marketing pages; renormalize the max).
3. Cognitive load: list failed checks; flag any decision point with >4 visible options.
4. Walk the primary task as 2-3 fitting personas; list the exact elements that failed each.
5. 2-3 strengths (specific), then 3-5 priority issues: [P0-P3] what / why it hurts users / concrete fix / which pass fixes it.
6. 2-3 provocative questions ("What would a confident version look like?").
Be direct. Name elements ("the Save button"), never "some elements". No "consider exploring".
```

## 🔧 Technical audit: 5 dimensions, each 0–4, total /20
Web only; native → [Native deltas](#-native-ios--android-deltas).

| Dimension | Check for | 0 → 4 |
|---|---|---|
| **Accessibility** ([floor](accessibility.md)) | Contrast <4.5:1 (AAA 7:1 where required), missing labels/roles/states, no focus ring, keyboard traps, div-buttons, skipped heading levels, missing alt, unlabeled inputs, missing required indicators. Reduced motion: flag both "no alternative" **and** a global `0.01ms` kill that deletes useful state feedback | Fails WCAG A → AA fully met |
| **Performance** | Layout thrash (read/write in loops), animating layout properties, unbounded blur/shadow, no lazy images, `will-change` left on, dead deps, needless re-renders; LCP <2.5 s, INP <200 ms, CLS <0.1 | Unoptimized → lean |
| **Responsive** | Fixed widths, touch targets <44px, horizontal scroll, breaks at 200% zoom/text scaling, mouse-only drag handlers, no `touch-action` on pointer-drag surfaces, missing breakpoints | Desktop-only → fluid, gestures work under touch |
| **Theming** | Hard-coded colors, broken/low-contrast dark mode, wrong token types, values that don't update on theme switch → [theming-dark-mode.md](theming-dark-mode.md) | No tokens → full system |
| **Integrity** | Detector hits verified in context, design-system drift, placeholder/decorative content posing as real, structure interchangeable with an unrelated product | Systemic drift → coherent |

Bands: **18–20** Excellent · **14–17** Good · **10–13** Acceptable · **6–9** Poor · **0–5** Critical.
Lead with the **integrity verdict** (pass/fail: is this a coherent, product-specific system?). Each finding: `[P?] name · file:line · category · user impact · standard violated (WCAG x.x.x) · fix · pass that fixes it`. Also report **systemic patterns** ("hard-coded colors in 15+ components") and **what's working**. Verify every finding; unverified findings are noise.

## 🧱 Harden: design for real data, not demo data
Rule: **a UI that only works with perfect data is not ready for production.**

- [ ] **Empty**: no items, no results, no notifications. Show what it is plus a next action, never blank
- [ ] **Loading**: initial, pagination, refresh. Skeleton shaped like the layout; say *what* is loading; time estimate for long operations
- [ ] **Error**: offline, timeout, 400 (inline field errors), 401 (to login), 403 (explain permission), 404, 429 (rate limit), 500 (generic + support). Retry button. Keep user input
- [ ] **One widget fails ≠ whole page fails.** Isolate error boundaries
- [ ] **Long text**: 100+ char names, long titles. `min-width: 0` on flex/grid children; `overflow-wrap: anywhere`; clamp/ellipsis only with the full text reachable
- [ ] **Short/missing**: empty string, 1 char, null avatar, missing image (reserve the aspect ratio)
- [ ] **Big numbers**: millions/billions, negative, 0, `tabular-nums` in tables
- [ ] **Many items**: 1000+ rows (virtualize/paginate), 50+ options (search); debounce search input (~300ms)
- [ ] **i18n**: +30–40% text budget (German), RTL via logical props (`margin-inline-start`), CJK, emoji, `Intl` for dates/numbers/currency, real plural rules → [i18n.md](i18n.md), [dates-money-timezones.md](dates-money-timezones.md)
- [ ] **No fixed widths on text containers.** `px-4` not `w-24` on buttons
- [ ] **Offline / slow 3G**: throttled network, optimistic updates with rollback → [pwa-offline.md](pwa-offline.md)
- [ ] **Concurrency**: disable submit while pending, double-click 10× → one request, race-safe
- [ ] **Permissions**: can't view / read-only / can't edit. Say why
- [ ] **Validation**: constraints in markup + hint text (`maxlength`, `pattern`, `aria-describedby`); server validates everything anyway
- [ ] **Interrupted gestures**: second pointer, `pointercancel`, `lostpointercapture`, release outside, window `blur` → drag state cleared, next drag works. One behavioral regression test per gesture fix
- [ ] **Degradation**: core path works without JS where feasible; feature detection, not browser detection; forced-colors / high-contrast mode keeps meaning
- [ ] **Cleanup**: listeners, timers, subscriptions, in-flight requests aborted on unmount
- [ ] **Zoom 200% + 16px min input font** (iOS zooms focused inputs <16px)

Seed data for screenshots: `Wolfeschlegelsteinhausenbergerdorff`, `مرحبا بالعالم`, `東京都渋谷区`, `👩🏽‍💻🚀`, `9,876,543,210.00`, `""`.

## 🎚️ Directional passes (one named target, scope is sovereign)
Touch only the named target. Reuse the system's own vocabulary; no new colors, fonts, radii unasked. Each ends by handing off to polish.

| Pass | Diagnose | Move |
|---|---|---|
| **Bolder / quieter** | Target opts out of the system's strongest moves / everything shouts | Levers + skeleton test → [motion-and-delight.md](motion-and-delight.md) |
| **Distill** | Competing actions, 5–7 colors, everything visible at once | ONE primary goal; 1–2 colors + neutrals; one family, 3–4 sizes, 2–3 weights; inline over modal; cut copy in half. Mystery ≠ minimalism; never remove info users decide with. Log what was removed and where it moved |
| **Typeset / colorize / layout** | Weak hierarchy, gray-on-everything, monotonous rhythm | Two isolated assessments: (1) judgment, each question answered with a selector or computed value; (2) mechanical scan. State the system (roles, strategy, spatial thesis) **before** editing. "Yes" without evidence isn't verification |
| **Copy / onboarding** | Unclear labels, dead empty states | → [ux-copy.md](ux-copy.md) |
| **Adapt / optimize** | Breaks on a device class, jank | → [adaptive-ui.md](adaptive-ui.md) |

Variant exploration of one element → [reference-image-design.md](reference-image-design.md).

## ✨ Polish: the last 10%
Triage order: broken tasks/data loss/misleading state/inaccessible paths → missing states → flow/hierarchy/responsive drift → visual/motion inconsistency → code cleanup. **Don't perfect one corner while the rest stays below the bar.**

Classify each drift before fixing it: **missing token** (promote to a token, only if genuinely reused) · **one-off** (swap in the shared component) · **conceptual mismatch** (align with neighboring flows) · **local defect** (just fix it). Fix at the narrowest correct level.

- [ ] Flow matches neighbors: terminology, disclosure, save behavior (optimistic vs pessimistic), routing. Arrival, transition, empty and recovery paths connect
- [ ] Every control has default, hover, focus-visible, active, disabled, loading, error, and success states
- [ ] Spacing on the scale; optical alignment (icon+text, play glyphs, button text) fixed by 1–2px where the math looks off
- [ ] Sibling cards: titles, prices, feature lists, and CTAs share a baseline; CTAs pinned to the bottom
- [ ] Same-role type identical everywhere; `text-wrap: balance` on headings, `pretty` on body; no orphans
- [ ] One icon family, one stroke weight, optically sized
- [ ] Semantic color tokens only; contrast re-checked in **every** state and theme
- [ ] Images: explicit aspect ratio (no CLS), responsive sources, meaningful alt
- [ ] Browser surfaces themed: `::selection`, caret, scrollbars, focus ring, underline offset, tabular numerals
- [ ] Motion: interruptible, transform/opacity only, one deliberate moment. Never add animation just to make polish visible
- [ ] Copy: consistent terms/casing, controls say what they do, errors state problem + recovery. Ask before changing claims
- [ ] Active nav item marked; no `href="#"` dead links; custom 404; skip link
- [ ] Walk the path with mouse, keyboard and touch at mobile, intermediate and wide widths
- [ ] Console clean, no debug output, dead code, unused imports, orphaned styles, or `z-index: 9999`
- [ ] Final `git diff` read: remove accidental churn and temp artifacts

## ♻️ Redesigning an existing UI without breaking it
First classify: **preserve** (modernize, keep the brand) · **overhaul** (new visual world, keep content + IA) · **greenfield** (the brand itself changes). Ambiguous → ask once.

| Rule | Detail |
|---|---|
| **Scan → diagnose → fix** | Identify framework, styling method, versions (e.g. Tailwind v3 vs v4), and the current dial reading *before* editing |
| **Keep the stack** | No framework/styling-lib migration. Check the dependency file before adding any import |
| **Improve in place, never rewrite** | Small, reviewable diffs; run tests/click the flow after each |
| **Baseline first** | Screenshot every key route/state; record SEO baseline (ranking pages, titles, structured data, OG). SEO migration is the #1 redesign risk → [seo.md](seo.md) |
| **Respect the incumbent system** | Extend the existing DESIGN.md/tokens. Flag pre-existing drift; don't fix it unasked |

**Never change silently:** URL slugs · primary nav labels · form field names and order (analytics + autofill) · test IDs, a11y names, analytics hooks · logo/wordmark · legal, consent, cookie copy.

Decision: IA, content, SEO sound → **targeted evolution** (fix priority below; most of the value at a fraction of the risk). Structural debt (broken IA, no system, broken mobile) → full redesign with strict content preservation.

Fix priority (biggest impact, lowest risk first): **1** font/type → **2** palette cleanup (one accent, one gray family, no pure `#000`) → **3** hover/active/focus states → **4** layout + spacing (max-width container, grid, `min-height: 100dvh`) → **5** swap generic components → **6** loading/empty/error states → **7** final type scale + spacing.

Checklist of what agents usually forget: legal links, back navigation from every page, custom 404, form validation, skip link, favicon, meta/OG tags → [seo.md](seo.md), real (non-Lorem, non-"John Doe", non-`99.99%`) content.

## 👥 Separate-reviewer pattern
Rule: **the builder never grades its own work.** A reviewer that inherits the builder's transcript also inherits its framing, its optimism, and its blind spots. General planner/worker/reviewer → [orchestration.md](../ai-agents/orchestration.md).

| Pattern | How |
|---|---|
| **Isolated dual assessment** | 2 parallel subagents: **A** = design critique (LLM judgment), **B** = detector + browser evidence. Neither sees the other's output. Merge after both finish: agreements, what B caught that A missed, B's false positives. Don't rerun the detector in the parent |
| **Why isolate A from B** | Deterministic output anchors judgment. A must reach the specificity verdict unprimed |
| **Degraded mode is loud** | No subagent tool → run sequentially; the report's first line says `DEGRADED: single-context (<reason>)`. A silent degraded review counts as failed |
| **Fresh finish reviewer** | Spawned with **no forked history**, edits nothing, has no browser. Input packet: request, confirmed answers, artifact paths, screenshot paths, direction contract, reference/comp, detector findings, taste rules |
| **Inventory before reading the contract** | Reviewer lists the reference's salient elements in its own words first; anchoring on the builder's summary inherits what it dropped |
| **Reading allowance** | Screenshots + contract first, sample primary files, stop reading around the midpoint of its turn budget and write. A run cut off before its sections exist returns nothing |
| **Fixed disposition vocabulary** | First line is exactly one of: `recapture` (evidence invalid) · `rebuild` (fidelity failed wholesale; skip the fix batch, re-derive, full re-review) · `fix` (ordered material fixes) · `ship`. Derived, never felt: calibrate against the reference, not the visible effort |
| **Verdict pass** | After fixes + recapture, the **same** reviewer scores each listed fix `resolved / partial / unresolved` from the new screenshots, plus ≤ 3 regressions the batch introduced. No new hunt. Narration of a fix is not evidence |
| **Scope-honest reporting** | "All 3 fixes resolved" ≠ "no issues remain". Open findings are never announced as a pass |
| **User evidence wins** | User's screenshot contradicts a `ship` → spawn a **new** full review with their evidence. Never patch inline and self-certify |
| **Stop on non-convergence** | Keep going while each round resolves findings; the same findings come back → stop, show the table → [stall rule](../ai-agents/agent-work-limits.md) |
| **Documenter after ship** | Fresh pass records the built world in DESIGN.md → [design-md.md](design-md.md#-new-world-final-designmd-comes-from-the-build) |

**Asset provenance.** Every shipped raster records its origin: the exact generation prompt (embedded in file metadata or a sidecar) or the source/license of a sourced image. A raster a fix batch abandons is deleted in the same batch.

### 📋 Finish-reviewer prompt
```text
You are the finish reviewer. Fresh eyes; you did not build this. Edit nothing, render nothing.
Inputs: <request>, <confirmed answers>, <artifact paths>, <screenshots: desktop.png, mobile.png>,
<direction contract>, <reference/comp or "none">, <DESIGN.md>, <detector findings>, <taste rules path>.
Read screenshots and reference first; list the reference's salient elements in your own words
before reading the contract. Stop reading by mid-budget and write.
0 Evidence: every required capture exists and shows what its name claims. If not -> recapture only.
1 Coverage: every requirement of the request present and findable.
2 Fidelity: per element: match / adaptation (cite the answer or constraint that forced it) / missing /
  contradicted / added. Uncited deviation = defect. Mandatory rows: TYPE (lettering character),
  MATERIAL (flat CSS or faked bevels where real material was promised = contradicted),
  GROUND (page field value and temperature vs reference; hunt drift toward cream or blue-slate).
3 Ceiling: native devices of the chosen world left unused.
4 Contract: each block of the direction contract kept? Memory test on the first viewport.
5 Truth: no invented claims or fake metrics; demo data labeled; placeholders marked.
6 Floor: hold screenshots against the taste bans; each banned element is a material fix.
First line: "disposition: recapture|rebuild|fix|ship". Then sections: persistence, fidelity,
ceiling, material_fixes (ordered, fidelity before craft, max 8; an asset fix says "produce: <x>"),
keep (one line: what must not be diluted). No praise. Do not run a second detector pass.
```

## 🤖 Deterministic anti-pattern detection (lint/CI guard)
Rule: **mechanical tells get caught by a machine on every edit, not by review.** Two tiers: a **fast tier** the agent runs on the files it just touched (unambiguous only: broken images, overflow, contrast, gradient text, glow, system drift) and a **deep pass** in `bin/check`/CI over every UI file in the diff. Exit `0` = clean, `2` = findings. Wiring → [guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md), [ai-first-cicd.md](../developer-experience/ai-first-cicd.md).

As of 2026-09, the leading open-source detector ships ~60 rules. Distilled:

**Static: source scan (CSS/markup/copy) is enough**
| Group | Rules |
|---|---|
| Surfaces | Thick one-side accent border on a card (side-tab) · accent border on a rounded card · hairline border + wide diffuse shadow · zero-offset colored glow shadow · radial halo/spotlight wash on dark · repeating-gradient stripes · hairline grid-line background · reflexive cream/beige page · purple/violet gradient or cyan-on-dark palette · gradient text (`background-clip: text`) |
| Structure | Nested cards · small rounded icon tile stacked above a heading · kicker/eyebrow label above a heading · hero eyebrow pill chip · numbered section labels (01/02/03) · large inline SVG built from primitive shapes · many-vertex organic `clip-path` |
| Type | Overused default font as the whole identity · italic serif display hero · letter-spacing crushed tighter than about −0.04em · tracking >0.05em on body · all-caps body · `text-align: justify` without hyphens · line-height <1.3 · skipped heading level · `font-size` off the declared ramp |
| Motion | Bounce/elastic/overshoot easing · decorative pulsing dot · fake blinking cursor · auto-scrolling marquee · image scale/rotate on hover · transitions on `width/height/padding/margin` |
| Copy | Marketing buzzwords (streamline, empower, supercharge, seamless, world-class, cutting-edge) · "X. No Y." aphoristic cadence ×3+ · "…is theater" framing · ≥8 em-dashes in body copy (advisory) |
| System drift | Font / color / radius / font-size not declared in DESIGN.md → [design-md.md](design-md.md) |
| Assets | `<img>` with empty/missing/placeholder `src` · raster buried under near-opaque wash or ~0 opacity |

**Rendered: needs a headless browser (computed styles + layout)**
Low contrast (<4.5:1 / 3:1) · gray text on colored bg · text overflowing its container / horizontal scroll · text occluded by an overlapping element · line length >~80ch · cramped padding · body text touching the viewport edge · heading closer to previous block than to its own content · flat type hierarchy (<1.25× steps) · monotonous spacing (one value everywhere) · content invisible at rest (failed reveal animation) · uncaught script error on load · body text <12px, functional UI text <11px · full-sentence hero at display size eating the fold · one column far past the fold beside a short sibling · cards flush against a scroller edge · positioned popover clipped by an `overflow: hidden` parent · same text repeated 3+ times in one card.

### Starter static guard (cheap, no dependencies but `rg`)
```bash
#!/usr/bin/env bash
# bin/ui-lint: fail on mechanical UI tells. Exit 2 on findings.
set -uo pipefail
G=(--glob '*.{css,scss,html,tsx,jsx,vue,svelte,astro}' --glob '!**/node_modules/**')
hits=0
check() { local name=$1; shift
  if rg -n -i "${G[@]}" "$@" src/; then echo "^^ $name"; hits=1; fi; }
check gradient-text      -e 'background-clip:\s*text' -e '\bbg-clip-text\b'
check layout-transition  -e 'transition(-property)?:[^;]*\b(width|height|padding|margin|top|left)\b'
check bounce-easing      -e 'cubic-bezier\([^)]*(-0?\.[0-9]|1\.[0-9]*[1-9])' -e '\b(bounce|elastic)\b[^;]*;'
check side-tab           -e 'border-(left|right|inline-start|inline-end):\s*([3-9]|[1-9][0-9])px' -e '\bborder-[lr]-[4-8]\b'
check glow-shadow        -e 'box-shadow:\s*0(px)?\s+0(px)?\s+[1-9][0-9]*px[^;]*(#|rgb|hsl|oklch)'
check pure-black-white   -e '(color|background(-color)?):\s*(#000|#000000|#fff|#ffffff)\b'
check justified-text     -e 'text-align:\s*justify'
check z-index-9999       -e 'z-index:\s*9{3,}'
check vh-not-dvh         -e 'height:\s*100vh'
check scroll-listener    -e "addEventListener\(\s*['\"]scroll['\"]"
check dead-link          -e 'href="#"'
check empty-img          -e '<img[^>]*src=(""|\{\s*""\s*\})'
check placeholder-copy   -e 'lorem ipsum' -e '\bJohn Doe\b' -e '\bAcme\b'
check buzzwords          -e '\b(seamless(ly)?|supercharge|empower|elevate|unleash|next-gen|world-class|cutting-edge)\b'
exit $(( hits * 2 ))
```
Rendered rules: run axe + custom `getComputedStyle` checks in the same Playwright job that captures screenshots. **New rule → fixture first**: ≥ 4 should-flag cases and ≥ 5 false-positive shapes, failing test before the rule.

### Triage every hit
| Verdict | Action |
|---|---|
| Real defect | Fix it. **Never** add an ignore to push a blocked write through |
| Confident false positive (fixture, deliberate demo, subject-appropriate motion like a literal bouncing ball, user-confirmed choice) | Narrowest ignore, reason written as `"<who decided>: <evidence>"`; write "user confirmed" only if they did. Disclose it |
| Unsure | Leave it standing, ask the human once |

Ignore ladder, narrowest first: **rule + value** (e.g. one font) → **one rule in one file** → whole file (fixtures, generated, deliberate slop demos) → rule project-wide. The last two silence rules that don't exist yet: human sign-off only. Keep ignores in one reviewable config; inline disable comments only for files that leave the repo standalone.

## 📱 Native (iOS / Android) deltas
Audit from source (SwiftUI/UIKit/Compose/RN/Flutter). HTML/CSS detectors **don't apply**; the reviewer's taste check is the only slop gate, and the reviewer packet says so. Same /20 scoring; dimensions become: **A11y** (VoiceOver/TalkBack labels, Dynamic Type / `sp`, Reduce Motion) · **Touch** (44 pt / 48 dp) · **Theming** (semantic system colors / Material roles + Dynamic Color) · **Conformance** (critical: back gestures alive, safe areas and insets, platform nav, native controls) · **Adaptivity** (size classes, split view, foldables) · **Perf** (launch, list recycling, main-thread gestures, recomposition). Platform rules → [adaptive-ui.md](adaptive-ui.md).

Slop test: *would a fluent user of the platform trust every screen, or does it read as a ported website?*

Capture from simulator/emulator, **never a browser**:
```bash
xcrun simctl io booted screenshot phone.png
xcrun simctl ui booted appearance dark
adb exec-out screencap -p > phone.png
adb shell cmd uimode night yes
adb shell settings put system font_scale 1.3   # restore 1.0 after
```
Simulators give breadth. Gestures, refresh rate, and performance need real hardware; say which one produced the evidence.

---
Sources: [pbakaus/impeccable](https://github.com/pbakaus/impeccable) · [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill)
