# 🔍 Design Review Loop — critique, audit, polish, harden

**Rule: no UI ships on the builder's word.** Render it, screenshot it, score it with a rubric, fix in one batch, re-screenshot. The builder judges its own work too kindly, so the final verdict comes from a **fresh-context reviewer**. Deterministic tells run as a lint guard, not as a matter of taste.

Related: taste rules → [design-taste.md](design-taste.md) · token/system spec → [design-md.md](design-md.md) · agent eyes (Playwright MCP) → [README](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless) · lazy/truncated output → [output-completeness.md](../writing-for-agents/output-completeness.md).

## 🔁 The loop
```
build fully ──▶ capture (desktop + mobile, one batch) ──▶ validate captures
      ▲                                                         │
      │                                                         ▼
  fix ALL findings in one batch ◀── critique + audit + detector (isolated)
      │
      └──▶ recapture same viewports ──▶ fresh reviewer: verdict ──▶ ship | fix | rebuild | recapture
```
| Rule | Why |
|---|---|
| Build the whole surface **before** the first screenshot | Per-tweak screenshots burn tokens and never converge |
| Capture **all target viewports in one round**: web `1440` + `390` wide; add the user's real viewport if known | The width that breaks is the one the user sees first |
| **Max 2 inspection rounds** in the build thread, then hand off | Open-ended self-QA gets worse results at higher cost than a fresh reviewer |
| Fix **every** finding of a round in one batch, then recapture | Fixing one issue per round never finishes |
| Stop the moment a round resolves nothing | Thrashing means the approach is wrong, not the details |
| After round 2, the reviewer's list is the only work list | Don't start your own new hunt for issues |

### 📸 Capture validity. An invalid capture proves nothing.
- Settle or disable entrance animations first. Otherwise content hidden by animation timing looks missing and gets "fixed" into a regression.
- Full-page shots from document top. Fresh tab per assessment; never reuse a stale one.
- Open every file once before sending it on: no blank/black regions, no half-loaded state, no wrong section behind the filename.
- Never judge fidelity from one full-page thumbnail. Crop regions and compare side by side with the comp/reference.
- Say what produced the evidence (emulated viewport, synthesized touch, real device) and what stayed untested.
- Wait for fonts before capturing: `await page.evaluate(() => document.fonts.ready)`.

## 🧭 Pass order: who owns what
| Pass | Question | Output | Edits code? |
|---|---|---|---|
| **Critique** | Is this good design for *this* product? | Heuristic score /40, P0–P3 issues, persona red flags | ❌ report only |
| **Audit** | Is it technically sound? | Dimension score /20, P0–P3 issues with file:line | ❌ report only |
| **Harden** | Does it survive real data, errors, and locales? | Fixed states + edge cases | ✅ |
| **Polish** | Is the last 10% consistent and finished? | Aligned, clean, verified diff | ✅ |
| **Detector** | Any mechanical tells? | Rule hits (exit 2 on findings) | ❌ guard |

Order: critique + audit (in parallel, isolated) → harden → polish (always last). Keep polish as refinement. If the concept itself is wrong, say so and redesign; don't hide a redesign inside polish.

## 🎯 Critique: heuristics and scoring
**Start with the specificity verdict**, before looking at any detector output: *could an unrelated product use this UI unchanged?* If yes, it fails. Name the category-interchangeable choices. Anti-slop specifics → [design-taste.md](design-taste.md).

### Nielsen 10, each scored 0–4 (honest: 4 = genuinely excellent; most real UIs land 20–32/40)
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

- Landing pages / portfolios: #7 and #10 may be `n/a`. Renormalize: the max becomes 4 × scored (e.g. `24/32`). Never print `/40` over a partial set.
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

### Cognitive load: 8 checks (0–1 fails OK · 2–3 moderate · 4+ critical)
Single focus · chunks ≤4 · related items grouped · obvious hierarchy · one decision at a time · **≤4 visible options per decision point** · no remembering across screens · progressive disclosure.

### Personas (pick 2–3, walk the primary task, report what **broke**, not a profile)
| Persona | Probes | Pick for |
|---|---|---|
| Power user | Shortcuts, Esc on modals, bulk actions, skippable onboarding | Dashboards, data tools |
| First-timer | Unlabeled icons, jargon, no success confirmation, unclear next step | Onboarding, forms, landing |
| A11y-dependent | Keyboard-only flow, focus visible, alt text, color-only meaning, announced state | Every UI |
| Stress tester | 0 / 1000 items, emoji/RTL input, refresh mid-flow, double submit | Checkout, forms |
| Distracted mobile | Thumb zone, 44px targets, state kept on app switch, slow network | Mobile, e-commerce |

### Severity (shared by every pass)
| Tag | Meaning | Action |
|---|---|---|
| **P0** | Blocks task completion / data loss | Fix now |
| **P1** | Significant difficulty, WCAG AA fail | Fix before release |
| **P2** | Annoyance, workaround exists | Next pass |
| **P3** | Polish, no real user impact | If time permits |

Tiebreak: *would a user contact support about it?* If yes, it's ≥P1. Too many P3s is noise. Report the top 3–5.

### 📋 Critique prompt (paste to a fresh subagent)
```text
You are a design director reviewing a finished UI. You did not build it. Edit nothing.
Inputs: request=<original ask>; target=<file or URL>; screenshots=<desktop.png, mobile.png>;
design system=<DESIGN.md path or "none">.
1. Specificity verdict first: could an unrelated product use this UI unchanged? Name the interchangeable choices.
2. Score Nielsen's 10 heuristics 0-4 in a table (n/a allowed for #7/#10 on marketing pages; renormalize the max).
3. Cognitive load: list failed checks; flag any decision point with >4 visible options.
4. Walk the primary task as 2-3 fitting personas; list the exact elements that failed each.
5. 2-3 strengths (specific), then 3-5 priority issues: [P0-P3] what / why it hurts users / concrete fix.
Be direct. Name elements ("the Save button"), never "some elements". No "consider exploring".
```

## 🔧 Technical audit: 5 dimensions, each 0–4, total /20
| Dimension | Check for | 0 → 4 |
|---|---|---|
| **Accessibility** | Contrast <4.5:1, missing labels/roles/states, no focus ring, keyboard traps, div-buttons, skipped heading levels, missing alt, unlabeled inputs, motion without a `prefers-reduced-motion` alternative | Fails WCAG A → AA fully met |
| **Performance** | Layout thrash (read/write in loops), animating layout properties, unbounded blur/shadow, no lazy images, `will-change` left on, dead deps, needless re-renders; LCP <2.5 s, INP <200 ms, CLS <0.1 | Unoptimized → lean |
| **Responsive** | Fixed widths, touch targets <44px, horizontal scroll, breaks at 200% zoom, mouse-only drag handlers, missing breakpoints | Desktop-only → fluid everywhere |
| **Theming** | Hard-coded colors, broken/low-contrast dark mode, wrong token types, values that don't update on theme switch → [theming-dark-mode.md](theming-dark-mode.md) | No tokens → full system |
| **Integrity** | Detector hits verified in context, design-system drift, placeholder/decorative content posing as real | Systemic drift → coherent |

Bands: **18–20** Excellent · **14–17** Good · **10–13** Acceptable · **6–9** Poor · **0–5** Critical.
Each finding: `[P?] name · file:line · category · user impact · standard violated (WCAG x.x.x) · fix`. Also report **systemic patterns** ("hard-coded colors in 15+ components") and **what's working**. Verify every finding; unverified findings are noise.

## 🧱 Harden: design for real data, not demo data
Rule: **a UI that only works with perfect data is not ready for production.**

- [ ] **Empty**: no items, no results, no notifications. Show what it is plus a next action, never blank
- [ ] **Loading**: initial, pagination, refresh. Skeleton shaped like the layout; say *what* is loading
- [ ] **Error**: offline, timeout, 400 (inline field errors), 401 (to login), 403 (explain permission), 404, 429 (rate limit), 500 (generic + support). Retry button. Keep user input
- [ ] **One widget fails ≠ whole page fails.** Isolate error boundaries
- [ ] **Long text**: 100+ char names, long titles. `min-width: 0` on flex/grid children; `overflow-wrap: anywhere`; clamp/ellipsis only with the full text reachable
- [ ] **Short/missing**: empty string, 1 char, null avatar, missing image (reserve the aspect ratio)
- [ ] **Big numbers**: millions/billions, negative, 0, `tabular-nums` in tables
- [ ] **Many items**: 1000+ rows (virtualize/paginate), 50+ options (search)
- [ ] **i18n**: +30–40% text budget (German), RTL via logical props (`margin-inline-start`), CJK, emoji, `Intl` for dates/numbers/currency, real plural rules → [i18n.md](i18n.md), [dates-money-timezones.md](dates-money-timezones.md)
- [ ] **No fixed widths on text containers.** `px-4` not `w-24` on buttons
- [ ] **Offline / slow 3G**: throttled network, optimistic updates with rollback → [pwa-offline.md](pwa-offline.md)
- [ ] **Concurrency**: disable submit while pending, double-click 10× → one request, race-safe
- [ ] **Permissions**: can't view / read-only / can't edit. Say why
- [ ] **Interrupted gestures**: second pointer, `pointercancel`, `lostpointercapture`, release outside, window `blur` → drag state cleared, next drag works
- [ ] **Cleanup**: listeners, timers, subscriptions, in-flight requests aborted on unmount
- [ ] **Zoom 200% + 16px min input font** (iOS zooms focused inputs <16px)

Seed data for screenshots: `Wolfeschlegelsteinhausenbergerdorff`, `مرحبا بالعالم`, `東京都渋谷区`, `👩🏽‍💻🚀`, `9,876,543,210.00`, `""`.

## ✨ Polish: the last 10%
Triage order: broken tasks/data loss/inaccessible paths → missing states → flow/hierarchy/responsive drift → visual/motion inconsistency → code cleanup. **Don't perfect one corner while the rest stays below the bar.**

Classify each drift before fixing it: **missing token** (promote to a token) · **one-off** (swap in the shared component) · **conceptual mismatch** (align with neighbouring flows) · **local defect** (just fix it). Fix at the narrowest correct level.

- [ ] Every control has default, hover, focus-visible, active, disabled, loading, error, and success states
- [ ] Spacing on the scale; optical alignment (icon+text, play glyphs, button text) fixed by 1–2px where the math looks off
- [ ] Sibling cards: titles, prices, feature lists, and CTAs share a baseline; CTAs pinned to the bottom
- [ ] Same-role type identical everywhere; `text-wrap: balance` on headings, `pretty` on body; no orphans
- [ ] One icon family, one stroke weight, optically sized
- [ ] Semantic color tokens only; contrast re-checked in **every** state and theme
- [ ] Images: explicit aspect ratio (no CLS), responsive sources, meaningful alt
- [ ] Browser surfaces themed: `::selection`, caret, scrollbars, focus ring, underline offset, tabular numerals
- [ ] Motion: interruptible, transform/opacity only, one deliberate moment rather than an entrance on every section; reduced-motion alternative keeps state changes visible
- [ ] Copy: consistent terms/casing, sentence case, controls say what they do, errors state the problem + recovery, no `Oops!`, no `!` on success
- [ ] Active nav item marked; no `href="#"` dead links; custom 404; skip link
- [ ] Console clean, no debug output, dead code, unused imports, orphaned styles, or `z-index: 9999`
- [ ] Final `git diff` read: remove accidental churn and temp artifacts

## ♻️ Redesigning an existing UI without breaking it
| Rule | Detail |
|---|---|
| **Scan → diagnose → fix** | Identify framework, styling method, and versions (e.g. Tailwind v3 vs v4) *before* editing |
| **Keep the stack** | No framework/styling-lib migration. Check the dependency file before adding any import |
| **Improve in place, never rewrite** | Small, reviewable diffs; run tests/click the flow after each |
| **Preserve behavior** | Routes, handlers, form names, test IDs, a11y names, analytics hooks all stay |
| **Baseline first** | Screenshot every key route/state *before* touching anything; diff after |
| **Respect the incumbent system** | Extend the existing DESIGN.md/tokens. Flag pre-existing drift; don't fix it unasked |

Fix priority (biggest impact, lowest risk first): **1** font/type → **2** palette cleanup (one accent, one gray family, no pure `#000`) → **3** hover/active/focus states → **4** layout + spacing (max-width container, grid, `min-height: 100dvh` not `100vh`) → **5** swap generic components → **6** loading/empty/error states → **7** final type scale + spacing.

Checklist of what agents usually forget: legal links, back navigation from every page, custom 404, form validation, skip link, favicon, meta/OG tags → [seo.md](seo.md), real (non-Lorem, non-"John Doe", non-`99.99%`) content.

## 👥 Separate-reviewer pattern
Rule: **the builder never grades its own work.** A reviewer that inherits the builder's transcript also inherits its framing, its optimism, and its blind spots. General planner/worker/reviewer → [orchestration.md](../ai-agents/orchestration.md).

| Pattern | How |
|---|---|
| **Isolated dual assessment** | Spawn 2 parallel subagents: **A** = design critique (LLM judgment), **B** = detector + browser evidence. Neither sees the other's output. Merge only after both finish, noting where they agree, what B caught that A missed, and B's false positives |
| **Why isolate A from B** | Deterministic output anchors judgment. A must reach the specificity verdict unprimed |
| **Degraded mode is loud** | No subagent tool available → run sequentially, and the report's first line says `DEGRADED: single-context (<reason>)`. A silent degraded review counts as failed |
| **Fresh finish reviewer** | Spawned with **no forked history**, edits nothing, has no browser. Gets an input packet: original request, confirmed answers, artifact paths, screenshot paths, design contract, detector findings, craft-floor rules |
| **Fixed disposition vocabulary** | First line is exactly one of: `recapture` (evidence invalid) · `rebuild` (fidelity failed wholesale) · `fix` (ordered material fixes) · `ship` |
| **Verdict pass** | After fixes + recapture, the **same** reviewer scores each listed fix `resolved / partial / unresolved`. It does not start a new hunt |
| **Scope-honest reporting** | "All 3 fixes resolved" ≠ "no issues remain". Open findings are never announced as a pass |
| **User evidence wins** | User's screenshot contradicts a `ship` → spawn a **new** full review with their evidence. Never patch inline and self-certify |
| **Budget** | 2 fix rounds unattended; then show the table and let the human decide |

### 📋 Finish-reviewer prompt
```text
You are the finish reviewer. Fresh eyes; you did not build this. Edit nothing, render nothing.
Inputs: <request>, <confirmed answers>, <artifact paths>, <screenshots: desktop.png, mobile.png>,
<DESIGN.md>, <detector findings>, <taste rules path>.
Checks in order:
0 Evidence: every required capture exists and shows what its name claims. If not -> recapture.
1 Coverage: every requirement of the request present and findable.
2 Fidelity: vs reference/comp, per region: match / adaptation / missing / contradicted / added.
3 Floor: hold screenshots against the taste rules; each banned element is a material fix.
4 Truth: no invented claims or fake metrics; placeholders marked as placeholders.
First line: "disposition: recapture|rebuild|fix|ship". Then an ordered list of material fixes
(what, where, why, concrete change). Do not run a second detector pass.
```

## 🤖 Deterministic anti-pattern detection (lint/CI guard)
Rule: **mechanical tells get caught by a machine on every edit, not by review.** Run tiered: an **immediate tier** (unambiguous: broken images, overflow, contrast, gradient text) as a post-edit hook, and the **full set** in CI/pre-merge. Exit `0` = clean, `2` = findings. Wiring → [guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md), [ai-first-cicd.md](../developer-experience/ai-first-cicd.md).

As of 2026-09, the leading open-source detector ships ~60 rules. Distilled:

**Static: source scan (CSS/markup/copy) is enough**
| Group | Rules |
|---|---|
| Surfaces | Thick one-side accent border on a card (side-tab) · accent border on a rounded card · hairline border + wide diffuse shadow · zero-offset colored glow shadow · radial halo/spotlight wash on dark · repeating-gradient stripes · hairline grid-line background · reflexive cream/beige page · purple/violet gradient or cyan-on-dark palette · gradient text (`background-clip: text`) |
| Structure | Nested cards · small rounded icon tile stacked above a heading · kicker/eyebrow label above a heading · hero eyebrow pill chip · numbered section labels (01/02/03) · large inline SVG built from primitive shapes · many-vertex organic `clip-path` |
| Type | Overused default font as the whole identity · italic serif display hero · letter-spacing crushed tighter than about −0.04em · tracking >0.05em on body · all-caps body · `text-align: justify` without hyphens · line-height <1.3 · skipped heading level |
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
check justified-text     -e 'text-align:\s*justify'
check z-index-9999       -e 'z-index:\s*9{3,}'
check vh-not-dvh         -e 'height:\s*100vh'
check dead-link          -e 'href="#"'
check empty-img          -e '<img[^>]*src=(""|\{\s*""\s*\})'
check placeholder-copy   -e 'lorem ipsum' -e '\bJohn Doe\b' -e '\bAcme\b'
check buzzwords          -e '\b(seamless(ly)?|supercharge|empower|elevate|unleash|next-gen|world-class|cutting-edge)\b'
exit $(( hits * 2 ))
```
Rendered rules: run axe + custom `getComputedStyle` checks in the same Playwright job that captures screenshots.

### Triage every hit
| Verdict | Action |
|---|---|
| Real defect | Fix it. **Never** add an ignore to push a blocked write through |
| Confident false positive (fixture, deliberate demo, subject-appropriate motion) | Narrowest ignore (rule + value, or rule + file), with a written reason; disclose it |
| Unsure | Leave it standing, ask the human once |

Ignoring a whole file or rule silences rules that don't exist yet. That needs human sign-off. Keep ignores in one reviewable config, not scattered inline comments.

## 📱 Native (iOS / Android) deltas
Audit from source (SwiftUI/UIKit/Compose/RN/Flutter). HTML/CSS detectors **don't apply**; the reviewer's taste check is the only slop gate. Same /20 scoring, but dimensions change:

| Dimension | iOS | Android |
|---|---|---|
| A11y | VoiceOver labels/traits, Dynamic Type (no fixed pt), Reduce Motion | TalkBack labels, `sp` not `px`, Remove animations |
| Touch | ≥44×44 pt | ≥48×48 dp, 8 dp apart |
| Theming | Semantic system colors, one tint, system materials | Material color roles, Dynamic Color + static fallback, tonal elevation |
| **Conformance** (critical) | Edge-swipe back alive, safe area (notch, Dynamic Island, home indicator), tab bar 2–5 sections, SF Symbols, native controls | Predictive Back never hijacked, edge-to-edge insets incl. IME, nav bar→rail by width, Material components, one FAB |
| Adaptivity | Size classes, iPad Split View, orientation | Window size classes, multi-window, foldable posture |
| Perf | Launch time, list recycling, main-thread work in gestures, full-size image decode for thumbnails | Same + recomposition counts |

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
