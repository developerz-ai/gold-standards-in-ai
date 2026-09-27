# 🧑‍🎨 Design Taste — Anti-AI-Slop Defaults

Left alone, an agent ships the **statistical average UI**: Inter, purple gradient, centered hero, three identical cards, fade-up on everything. Every app then looks like every other AI app. This file has two parts: a **quality floor** every app must clear, and a **method that gives each app its own look**.

**Scope.** This file covers taste and the defaults to refuse. Deep topics live next door:

| Need | Go to |
|---|---|
| Catalog of directions to pick from | [design-directions.md](design-directions.md) |
| Write/read the project's `DESIGN.md` + `PRODUCT.md` | [design-md.md](design-md.md) |
| Critique → audit → harden → polish loop | [design-review-loop.md](design-review-loop.md) |
| Motion depth, delight, bolder/quieter | [motion-and-delight.md](motion-and-delight.md) |
| Copy, voice, onboarding, empty states | [ux-copy.md](ux-copy.md) |
| Responsive, touch, native, UI perf | [adaptive-ui.md](adaptive-ui.md) |
| Generate a comp image first, then build it; in-browser variants | [reference-image-design.md](reference-image-design.md) |
| Name, logo, brand kit | [brand-identity.md](brand-identity.md) |
| Token plumbing, `clamp()`, keyframes · light/dark | [css-scss-craft.md](css-scss-craft.md) · [theming-dark-mode.md](theming-dark-mode.md) |
| Model truncating or skipping sections | [../writing-for-agents/output-completeness.md](../writing-for-agents/output-completeness.md) |

## 🧭 Order of operations
1. **Read the brief.** Page kind, audience, vibe words, references, existing brand assets, quiet constraints (a11y-first, regulated, kids). Constraints beat taste.
2. **Decide what is already true** (table below). Missing `DESIGN.md` ≠ greenfield.
3. **Pick the register** from the surface (see Register).
4. **Pick a foundation:** official design system or an aesthetic (see Brief → system map).
5. **Pick a direction** (see Make each app different) and set the **dials**.
6. **State it in one line before code:** `Reading this as: <page kind> for <audience>, <archetype> language, <type pairing>, <palette strategy>, dials V/M/D = 7/5/3.` Then write the [direction contract](#-direction-contract-before-code).
7. Build fully committed → check against the floor → hand off to the [review loop](design-review-loop.md).
8. Ambiguous brief → ask **one** question ("closer to calm-minimal or loud-editorial?"), not a questionnaire. Clear brief → don't ask.
9. Out of scope (dense data tables, code editors, realtime collab)? **Say so**, name the right tool, apply taste only to the parts it fits.

**The brief wins.** A pinned font, palette or era overrides every default and ban in this file. The bans apply only to choices the brief left open. A pinned world pins the world, **not its softest rendition**: its full material range stays in play.

### Decide what is already true
| Situation | Do |
|---|---|
| **Established world** (coherent look in code, even without `DESIGN.md`) | Inherit it. Document it. Don't invent a replacement |
| **Refinement** | Preserve identity, behavior, copy, everything out of scope. Ask before changing factual copy |
| **Redesign** | Keep product truth, content, function, constraints. Old look = evidence, not authority. Replace the world; **never split the difference into polish on the discarded look** |
| **Incomplete brand** | Keep confirmed assets/traits, expand the system with the user |
| **No visual authority** | Create a new world (below) |

**Invention budget scales with scope.** Section/component/state inside a surface → inherits it, no concept round. Whole new surface in an established world → keep the system, derive 5–7 *structures* from content + task, present 3. New or replacement world → the full derivation below.

## 🎲 Make each app different
Goal: **every app gets a distinct, recognizable look.** No shared house style. The floor stays the same for every app. The direction changes per app.

### Derive, don't default
1. Write **one sentence of product truth**: the unique mechanism, for whom, in what physical scene (desk at 9am? phone on a train? a dim studio?).
2. Name the **category rut**: the page this category always ships, plus its predictable opposite. Both are off-limits. A literal reading of the product name/metaphor joins the rut (spend ≤ 1 candidate on it).
3. List **7 concrete references from the audience's world**, ordered by resonance: objects, places, rituals, and the graphic traditions they read daily (transit signage, lab notebooks, record sleeves, trail maps, tax forms, synth panels, a documentation standard). **Span ≥ 3 material families.** > 3 in one family = you stopped at the obvious artifact. Near-duplicates count once.
4. Turn the top 2–3 into full directions: world (palette + type + material) + first viewport + signature interaction + honest risk.
5. **Pick with a seed, not by habit.** Models reliably take option #1 from any list. Rank, then choose deterministically (e.g. product-name length mod N) and write the roll down.
6. **Fuse, then raise.** Weigh each runner-up against the pick on exactly 2 axes: audience identification, product clarity. A runner-up that loses still **donates one discipline** the pick lacks (a palette's total commitment, a grid's density courage), written as a named line. Donate ambition, never clothes: a lifted motif is a costume.
7. **Commit on every element**: nav, buttons, inputs, links all rebuilt in the direction's vocabulary. A stock component inside a committed form is a lapse.
8. **Self-check:** if someone could guess the look from the category alone ("fintech → navy + Inter"), or from category + the thing it avoided, rework.

### Present choices honestly
- Show **one committed direction** + ≤ 2 real alternates, same anatomy each (thesis, palette, materials, first viewport, risk). Never a ranked lineup; that invites the safest card.
- **Standing exit:** always offer "the category standard, played straight". It's the user's door, never your recommendation. If taken: ask for 2–3 peer products, match their craft level, no irony.
- **Re-roll** only on factual grounds (the direction can't carry the product's truth), never taste. After 2 user re-rolls, ask what quality is missing.
- **Energy is not the enemy of trust.** "No hype / no gamification" rules out *devices*, not exuberance.
- **The subject is not a license.** Bookish, warm, family, kids → cream + serif + lamplight is the default wearing the subject's clothes. Book cloth, thread, jackets, endpapers span the saturated spectrum. Treat your first palette for such subjects as already spent.
- **Truth binds claims, not demonstrations.** Author demo data at full fidelity, label it synthetic, list what to replace. Refusing a bold direction because demo data doesn't exist yet is timidity dressed as honesty.

### Archetypes & rotation
Starting points, not skins: pick from the catalog of 28 directions (light restrained, light expressive, dark, period), mix at most two, re-derive the accent from product meaning → [design-directions.md](design-directions.md).
- **Record the pick** in [`DESIGN.md`](design-md.md): archetype, fonts, palette strategy, hues, dials, seed.
- **Before a new app:** read the last 2–3 sibling apps' `DESIGN.md`, then differ from each on **≥ 4 of the 12 sibling axes, incl. ≥ 1 of polarity, display class, accent hue family**. Canonical axes table → [design-directions.md](design-directions.md#-make-siblings-differ). On top: **never reuse the display face or accent hue of the previous app.**
- Palette families to rotate (as of 2026-09): mono + one saturated pop · forest + bone + amber · black + warm tan · cobalt + one neutral · terracotta + cool slate · olive + brick + paper · ink navy + signal orange · plum + lime · drenched single hue.

## 📜 Direction contract (before code)
Write it in the surface brief (e.g. `docs/design/surfaces/<route>.md`), ~150 words. Later agents and the reviewer audit the render against it.
```markdown
## Direction contract
THESIS: <the one idea this surface owns>; refuses <the category-default arrangement>.
OWN-WORLD: <palette + component language, recognizable with all content removed>.
STORY: <what the visitor understands, believes, does>.
FIRST VIEWPORT: <what is where, at what scale; where the primary action sits>.
FORM: <chosen direction, its rank on your list, the seed/roll>.
FINISH: unreviewed and undocumented is unfinished; ends with the fresh review, its verdict, DESIGN.md.
```
- A block that reads like a mood → the direction isn't decided yet.
- **Never ship the contract**: not in HTML comments, `data-*`, hidden DOM, JSON-LD, bundles.
- Memory test for FIRST VIEWPORT: if someone left after one screen, what would they describe an hour later? "A vibe" = not committed.

## 🧱 Brief → design system map
**Rule: if the brief reads as a real design system, install the official package.** Don't hand-recreate its CSS, don't import its tokens and override 90% of them. **One system per project.**

| Brief reads as | Reach for (as of 2026-09) |
|---|---|
| Enterprise / Microsoft-adjacent | Fluent UI |
| Material-flavored product | Material Web + M3 tokens |
| Dense B2B analytics | Carbon |
| Shopify app / Atlassian-style / GitHub-style devtool | Polaris / Atlassian DS / Primer (Brand for marketing) |
| UK / US public sector | GOV.UK Frontend / USWDS (expected, sometimes required) |
| Own-the-code components | shadcn/ui or Radix Themes — **never ship the default theme**: restyle radius, color, shadow, type |

Aesthetics (glass, bento, brutalism, editorial, kinetic type, mesh) have no official package: build with web standards and **label approximations honestly** (e.g. "web glass approximation", not "Liquid Glass"; add a `prefers-reduced-transparency` solid fallback).

## 🎛️ The three dials
Set them once per surface, then derive layout, motion and density from them.

| Dial | 1–3 | 4–7 | 8–10 |
|---|---|---|---|
| **VARIANCE** | Symmetric 12-col, equal padding, centered | Offsets, mixed aspect ratios, left headers over centered data | Asymmetric fr grids (`2fr 1fr 1fr`), big empty zones, overlap |
| **MOTION** | Hover/active only | CSS transitions + a staggered load-in | Scroll-driven sequences, pinned sections, physics |
| **DENSITY** | Gallery: `py-32+` sections | App: `py-16–24` | Cockpit: tight, 1px rules, no card boxes, tabular numerals |

| Surface / vibe words | V | M | D |
|---|---|---|---|
| Landing (mainstream SaaS) | 7 | 6 | 4 |
| Agency / creative / "Awwwards", "experimental" | 8–10 | 7–10 | 3 |
| "Minimal, calm, editorial" / blog | 5–6 | 3–4 | 2–3 |
| "Premium consumer" | 7–8 | 5–7 | 3–4 |
| Product app / dashboard | 3–4 | 3 | 6–8 |
| Civic / regulated / a11y-critical | 3 | 2 | 5 |
| Redesign, preserve · overhaul | match · +2 | +1 · +2 | match |

- VARIANCE ≥ 4 → asymmetric layouts **must collapse to a single column below 768px**, declared per section.
- **Motion claimed = motion shown.** Ship MOTION ≥ 5 only if it works. Otherwise drop to 3 and ship a clean static page.

## 🏷️ Register — brand vs product
Pick the register from the **surface**, not the company. A dev tool's landing page is brand. A fashion house's docs page is reading.

| Register | Visitor's job | Rule |
|---|---|---|
| **Persuade** (brand) | Decide + act | Offer intelligible in one line, primary action visible, prove one thing only this product can. Conversion lives *inside* the direction's vocabulary |
| **Operate** (product) | Finish a task | Earned familiarity. Brand lives in precise details |
| **Read** | Understand | Measure, wayfinding, hierarchy first |
| **Experience** | Be inside the work | Artifact leads from the first viewport; interface recedes |

**Operate specifics.** Slop here isn't flatness, it's **strangeness without purpose**: decorated buttons, display fonts on labels, reinvented scrollbars/controls, a modal as first thought.
- One family is often right. **Fixed rem scale**, not fluid; step ratio 1.125–1.2. Tables may run 120ch.
- Restrained color is the floor. Accent = primary action, selection, state only. A second neutral layer for sidebars/toolbars. No full-saturation color on inactive states.
- 150–250ms transitions. **No page-load choreography**: the app loads into a task.
- Overlays escape their container (`<dialog>`, popover API, `position: fixed`), never clipped by `overflow: hidden`.
- Permitted here: system fonts, top bar + side nav, tabs, command palette, real density, sameness screen to screen.

## 🔤 Typography
| Rule | Value |
|---|---|
| Body size floor (web) | `1rem` / 16px |
| Body measure | 65–75ch (45ch minimum). Wider measure → more leading |
| Display max | ~6rem. Hero headline ≤ 2–3 lines: **widen the container, then shrink the font**; never cut copy to fit. A 4-line hero is a font-size error |
| Hero scale vs asset | Plan together. Headline > 6 words next to a big asset → don't start at the top of the scale |
| Display tracking | `-0.02em` to `-0.03em`; never below `-0.04em` |
| Small caps / labels | positive tracking `0.04–0.08em`. Tracked caps for short markers only, never sentences |
| Italic display with descenders (`g j p q y`) | `line-height ≥ 1.1` + bottom reserve, or descenders clip |
| Families | 1 is often enough; 2 maximum; a second family must do a job the first can't |
| Scale | Enumerated ramp: every `font-size` lands on a step. Adding a step is a design decision. Product UI: 3–5 sizes, weights 400/500/600 |
| Light text on dark | +leading, +a touch of tracking, +one weight step |
| Numbers in tables/data | `font-variant-numeric: tabular-nums` |
| Wrapping | Headings `text-wrap: balance`; paragraphs `text-wrap: pretty` |
| Paragraph rhythm | Spacing **or** first-line indent, never both |

- **State the role system before editing:** roles needed, contrast between them, measure, which faces are authoritative. Then stress it: long headings, +30% translation, 200% zoom, missing weight, fallback font.
- **Choose faces like objects from the subject's world.** Ask: what would this product look like as a physical object?
- **Reflex faces = you stopped looking** (brand surfaces, as display): Inter-as-display, Fraunces, Instrument Serif/Sans, Playfair Display, Cormorant, Lora, Crimson, Newsreader, Syne, Space Grotesk, Space Mono, IBM Plex, DM Sans/Serif, Outfit, Plus Jakarta Sans, Roboto, Open Sans, Arial. Use one only with a reason no other face satisfies. "Books want a serif" / "tech wants mono" doesn't count.
- **Rotation beats any allow-list.** A recommended face becomes slop the moment every app uses it.
- **Serif is not "premium".** "Creative brief → serif display" is the most tested AI tell. Serif only for a real editorial/heritage/publication identity.
- **Emphasis = italic or weight of the same family.** Never drop a random serif word into a sans headline.
- **Monospace only for code, data, measurement.** Mono as a "technical" costume is a tell.
- **No system display face** (Impact, Arial Black, the platform sans) as a brand's display voice. Self-host the right face ([assets](assets-optimization.md)), used weights only, `font-display: swap`, metric-matched fallbacks.
- Sentence case over Title Case. No tracked all-caps subheaders everywhere.

## 🎨 Color
1. **Pick a strategy before picking colors:** Restrained (neutrals + 1 accent; default for Operate/Read) · Committed (one saturated color owns 30–60% of the surface) · Full palette (3–4 named roles) · Drenched (the surface is the color). Name the emotional temperature and dosage too.
2. **Build roles, not swatches:** canvas, raised surface, text primary/secondary, action, focus, selection, border, success/warning/error/info, data scale.
3. **Author in OKLCH** for new palettes. Lower the chroma near white and black; don't keep high chroma at extreme lightness for math's sake.

```css
:root {
  --hue: 152;                                   /* from product meaning, not category habit */
  --canvas:   oklch(0.975 0.006 var(--hue));    /* tinted neutral, not #fff */
  --surface:  oklch(0.995 0.004 var(--hue));
  --ink:      oklch(0.22  0.02  var(--hue));    /* off-black, not #000 */
  --ink-2:    oklch(0.45  0.02  var(--hue));
  --line:     oklch(0.90  0.01  var(--hue));
  --accent:   oklch(0.62  0.16  38);            /* one accent, locked page-wide */
  --shadow:   0 1px 2px oklch(0.3 0.03 var(--hue) / 0.08),
              0 8px 24px oklch(0.3 0.03 var(--hue) / 0.06);
}
```

- **One neutral family.** Don't mix warm and cool greys. Tint neutrals only when the hue creates cohesion; pure grey is valid if the world calls for it.
- **No pure `#000` / `#fff`.** Off-black and off-white.
- **Accent lock:** one accent everywhere, spent on the primary action and state. No surprise teal badge in the footer. Default saturation < 80%.
- **Commit at page scale, not in sprinkles.** The strongest color owns a region or role; scattered tiny accents are noise.
- **Roles can swap by size and theme.** An accent that passes as a large fill may fail as small text on the other theme; give that job to another role instead of dropping contrast.
- **Text on colored surfaces:** derive secondary text from that hue. Grey text on color looks dead.
- **Explicit colors over chains of translucent overlays.** Alpha stacks make contrast depend on what's underneath.
- **Shadows tinted** to the surface hue, offset + soft blur, one light source. A zero-offset colored glow isn't depth.
- **Contrast (WCAG AA):** body 4.5:1 · large text 3:1 · controls/icons/focus 3:1. Check hover, disabled, placeholder, text on images, both themes. Simulate color-vision deficiencies. Never color as the only signal (data: add shape, label, pattern).
- **Light vs dark comes from the use scene** (who, where, what light), not the category ([theming](theming-dark-mode.md)). **One theme per page**: no mid-scroll flips unless it's one deliberate device.
- **Calibration: the saturated AI looks.** Legitimate only if the brief asks:
  - Purple→blue gradient + centered hero on a dark mesh.
  - Warm cream ground + high-contrast serif display + terracotta/oxblood accent (also beige + brass + espresso for "premium consumer").
  - Near-black + one neon accent + glowing edges.
  - Broadsheet hairlines + italic display serif + tiny tracked mono labels.

## 📐 Layout & space
- **Squint test:** blur the page. Primary, secondary and groups still read in order.
- **Proximity before containers.** Group by spacing first; borders/cards only when spacing can't.
- **Rhythm = contrast.** Tight inside groups, generous between, more space above a heading than below. Bottom padding often needs a touch more than top, optically.
- **Spacing scale on a 4px base** (4, 8, 12, 16, 24, 32, 48, 64, 96…). No one-offs. `gap` for sibling rhythm.
- **Cards only when elevation means hierarchy.** **Never card-in-card.** 3–5 intentional cards beat 8.
- **Declare elevation once:** border *or* shadow. A 1px border under a wide soft shadow is a ghost card.
- **Shape lock:** one radius system (all-sharp · 12–16px soft · pill for small controls only), applied everywhere. Nested corners are concentric: inner radius = outer radius − padding.
- **Break the 3-equal-cards row.** Asymmetric grid, zigzag (max 2 in a row), featured + rest, horizontal scroll, plain prose.
- **Bento is a tool, not a default.** Cells = items (no blank tiles; `grid-auto-flow: dense`, then verify spans interlock). Vary sizes. ≥ 2 cells with real visual variation.
- **Layout families per page:** each family (3-col cards, split, full-bleed quote…) at most once. 8 sections → ≥ 4 families. But variation isn't a goal: repetition that aids recognition stays.
- **Section content shape (Persuade):** headline ≤ 8 words + sub ≤ 25 words + one asset **or** one CTA. More needs a reason.
- **Long lists (> 5 items) get a different component**, not a longer `<ul>`: grouped chunks, card-per-item, tabs, scroll-snap pills, featured + "view all". No 20-row tables on a marketing page.
- **No split header** (big headline left, tiny orphan paragraph floating right). Stack them unless the right column carries a real visual.
- **Hero:** fits the first viewport. Subtext ≤ ~20 words, CTA visible without scrolling, top padding ≤ ~6rem, **max 4 text elements** (≤ 1 small label, headline, subtext, CTAs). Taglines under CTAs, trust strips, pricing teasers, avatar rows, logo walls → the section **below**.
- **Centered hero only for manifesto/launch copy.** Otherwise split, left text + right asset, or asymmetric whitespace (VARIANCE > 4).
- **Nav:** one line on desktop (condense or collapse at 1024px), ≤ 80px tall, active page marked.
- **Align across siblings:** CTAs pinned to card bottoms, lists start at the same Y, shared baselines. Nudge optically after looking at the render.
- **Pace the scroll:** vary density, scale, image and quiet inside one grammar. A dense passage earns a quiet one. End on a real close, not a link farm.
- **Mechanics:** `min-height: 100dvh`, not `100vh`. Grid over flex-percentage math. Max-width container (~1200–1440px). Container queries for reused components. DOM/focus order = visual order. Documented z-index scale, no `9999`. Responsive depth → [adaptive-ui.md](adaptive-ui.md).
- **Operate surfaces:** predictability *is* the affordance. Keep asymmetry for Persuade/Experience.

## 🎞️ Motion (floor)
Full rules, durations, easing, choreography, reduced motion → [motion-and-delight.md](motion-and-delight.md).
- **Every animation answers "what does this communicate?"** Feedback, state change, spatial continuity, attention at a meaningful moment. "Looks cool" isn't an answer.
- **One authored moment per surface**, product-specific. Not the same fade-up on every section.
- **Content visible by default.** Animate *from* a visible state so a failed script never hides the page.
- transform/opacity base; clip-path, mask, bounded blur allowed when smooth. Never animate `width/height/top/left/margin`. **No raw `scroll` listeners**, no scroll position or pointer position in reactive state.
- ≤ 1 marquee per page. Non-essential loops stop offscreen. Grain only on a fixed `pointer-events: none` layer. No custom cursors. Don't scale an image on hover; give the container the feedback.
- **Reduced motion = fewer + gentler, not none.**

## ✍️ Copy & content (floor)
Full voice, microcopy, errors, onboarding → [ux-copy.md](ux-copy.md).
- **Real draft copy in the product's own nouns and verbs.** No lorem, no Acme/John Doe, no fake-precise or fake-round stats. Never invent prices, customers, benchmarks, capabilities; label sample data.
- **No filler verbs** (Elevate, Seamless, Unleash, Supercharge, Next-gen, Empower, Delve, Unlock). AI-cute copy is worse than plain copy.
- **No emoji in UI** unless the brand is explicitly chat/social-native.
- **Buttons = verb + object**, ≤ 3 words, one line. **One label per intent** per page.
- **Say it once.** No intro repeating the heading, no micro-meta sentence under a heading. One copy register per page.
- **Dashes:** em/en dash as a UI separator is a strong LLM tell. Periods, commas, colons, hyphenated ranges.

## 🖼️ Icons, imagery & material
- **One icon family, one stroke width.** Lucide by default *is* the AI default: choose the set deliberately. No emoji/unicode glyphs as icons, no hand-drawn icon paths, no cliché metaphors (rocket = launch, shield = security).
- **Imagery priority:** image generation if available → real/brand/open-license photos (verify every URL resolves; seeded placeholders beat broken links) → labeled slots (`<!-- TODO: hero product photo 1600×1200 -->`) listed for the user. Search the subject's physical object, not the category. One decisive photo beats five mediocre ones. Image-first workflow → [reference-image-design.md](reference-image-design.md).
- **A text-only page is unfinished, not minimalist.** Even restrained sites need 2–3 real images.
- **Author the assets; never substitute chrome.** Gradients, glass, generic icon tiles, sparklines, progress rings or many-vertex `clip-path` where an authored asset belongs = the gap wearing chrome.
- **Imitation material is the most reliable machine-made tell:** CSS bevels, embossing, fake leather/metal/chalk. Render the material as a real raster, or don't claim it.
- **No div-built fake screenshots** (dashboards, task lists, terminals made of styled boxes). Real screenshot, live mini-component, or nothing.
- **No sketchy/doodle SVG illustration**, `feTurbulence` grain scenes, or geometric masks faking organic cutouts. Crisp vector geometry and diagrams are fine.
- **Backgrounds get texture only from the subject's world.** Grid overlays, stripes and crosshairs need a real map/blueprint/canvas underneath. No text on high-contrast texture.
- **When the direction names a technique** (canvas, WebGL, view transitions), build the technique, not a static imitation.
- **Logo walls:** real SVG logos only, no category labels under them, below the hero, legible in both themes. Invented brand → invented SVG mark, not a plain-text wordmark. Brand system → [brand-identity.md](brand-identity.md).
- **Nothing overlaid on photos** (tags like `PLATE · 02`), no decorative photo credits.

## 🧩 States & the parts you didn't draw
- Every interactive element: hover, `:active` (≈ `scale(0.98)` / 1px press), focus-visible, disabled, loading, error, empty. Button text and form fields pass contrast in every state.
- **Loading:** skeletons shaped like the final layout. **Empty:** say which empty case (first use / no results / filtered / no permission) + the next action.
- **Theme the browser defaults** from the palette. Cheapest sign a page was built, not assembled, and the one agents skip most:

```css
::selection { background: color-mix(in oklch, var(--accent) 30%, transparent); color: var(--ink); }
:root { caret-color: var(--accent); accent-color: var(--accent); scrollbar-color: var(--line) transparent; }
:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
a { text-underline-offset: 0.2em; text-decoration-thickness: 1px; }
```

## 🔀 Variants of one element
Tuning an existing element? Lock identity first (one sentence: real hex, loaded fonts, topology, surface, voice), then 3 variants on **3 different axes** (hierarchy · topology · type system · color strategy · density · decomposition). New fonts/hues only on an explicit "redesign". Workflow → [reference-image-design.md](reference-image-design.md). Bolder/quieter levers → [motion-and-delight.md](motion-and-delight.md).

## 🚫 Banned patterns (unless the brief asks)
| Area | Slop tell |
|---|---|
| Page scaffold | Three identical icon + heading + text cards · hero-metric template (big number, small label, stat row) · card in card · modal for a task that needs no interruption |
| Labels | Eyebrow/kicker above headings (≤ 1 per 3 sections, ideally 0) · section numbers `01 / 02` · `01 / 4` pagination on tiles · `Step 1 / Phase 01` · `BETA`/`v0.6` in hero · "Brand · No. 01" micro-meta · poetic labels ("Field notes", "Quietly trusted by") · "Scroll to explore" |
| Surface | Gradient text · glass as decoration · `border-left` stripe > 1px on cards/alerts · hard offset `4px 4px 0` shadow outside a real neo-brutal world · neon outer glow · pure black bg · imitation material |
| Decoration | Status dots before every row · `·`-separated meta strips · city/time/weather strips · `BRAND. MOTION. SPATIAL.` strip · rotated vertical text · `<br>`-split italic headline by reflex · decorative sparklines/rings |
| Lists/data | Hairline above *and* below every row · progress bars with grey tracks as comparison · fake live counters ("412 of 800") · version footers on marketing pages |
| Components | Filled + ghost pair everywhere · sun/moon toggle by reflex · 3-tower pricing differing only in height · 3-card testimonial carousel with dots · 4-column footer link farm |
| Motion | Same fade-up on every section · bounce by reflex · infinite micro-loops on static info · scroll-jacking without a narrative reason |

## ✅ Pre-ship taste check (60 seconds)
- [ ] Direction line + contract written. Look can't be guessed from the category. Differs from each sibling on ≥ 4 [sibling axes](design-directions.md#-make-siblings-differ).
- [ ] No reflex face as display. ≤ 2 families. Body ≥ 16px, 65–75ch. Hero ≤ 3 lines, ≤ 4 text elements.
- [ ] One neutral family, one locked accent, OKLCH tokens, no `#000`/`#fff`, AA contrast in both themes.
- [ ] One radius system. No card-in-card. Eyebrow count ≤ ceil(sections ÷ 3). ≥ 4 layout families per 8 sections.
- [ ] Each animation has a stated purpose. Reduced-motion path exists. No scroll listeners.
- [ ] Zero filler words, lorem, emoji, fake stats, duplicate CTA intents, em-dash separators.
- [ ] Real imagery or labeled slots. No fake screenshots, no imitation material.
- [ ] All states + themed selection/caret/focus/scrollbar.
- [ ] Screenshot at mobile + desktop ([Playwright MCP](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless)), then run the [review loop](design-review-loop.md).

## 📋 Paste-ready block (CLAUDE.md / DESIGN.md)
```markdown
## Design taste
- Brief wins: pinned fonts/palette/era override everything below (the whole world, not its softest rendition).
- Missing DESIGN.md != greenfield: inherit a coherent look in code. Redesign replaces the world; never polish the discarded look.
- Before UI code, state: "Reading this as: <kind> for <audience>, <archetype>, <fonts>, <palette strategy>, V/M/D=<n/n/n>." Write the 6-line direction contract (THESIS/OWN-WORLD/STORY/FIRST VIEWPORT/FORM/FINISH) in the surface brief, never in shipped code.
- Direction: product truth + 7 references from the audience's world (>=3 material families) + seeded pick. Guessable from the category = rework.
- Differ from each of the last 2-3 sibling apps on >=4 of the 12 axes in design-directions.md (incl. polarity, display class or accent hue family). Never reuse last app's display face or accent hue.
- Real design system brief -> official package, one per project; shadcn/Radix never in default theme.
- Register: Persuade=bold allowed; Operate=earned familiarity, fixed rem scale, 150-250ms, no page-load choreography.
- Type: body >=16px, 65-75ch; tracking -0.02..-0.03em (floor -0.04); <=2 families; enumerated ramp; tabular-nums; no reflex faces as display; emphasis = same-family italic/weight; mono only for code/data.
- Color: strategy first; OKLCH; one tinted neutral family; one locked accent; no #000/#fff; commit at page scale; tinted offset shadows; AA both themes; no purple gradient, gradient text, neon glow.
- Layout: 4px scale; tight groups, generous separation; cards only for real elevation, never nested; one radius system; no 3-equal-card row; bento cells = items; hero fits viewport, <=4 text elements; eyebrows <=1 per 3 sections; lists >5 items get another component.
- Motion: purpose or delete; one authored moment; content visible by default; transform/opacity; no scroll listeners; reduced motion = fewer + gentler.
- Content: real copy, no lorem/Acme/John Doe/fake stats/filler verbs/emoji; verb+object CTAs, one label per intent.
- Imagery: generate or source real images (verify URLs) or labeled TODO slots; no div fake screenshots, no imitation material, no chrome where an asset belongs.
- Theme ::selection, caret, focus-visible, scrollbar, underline offset. Ship hover/active/focus/disabled/loading/empty/error.
```

---
Sources / further reading: [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) · [pbakaus/impeccable](https://github.com/pbakaus/impeccable).
