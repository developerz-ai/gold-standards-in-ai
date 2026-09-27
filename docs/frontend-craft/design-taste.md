# 🧑‍🎨 Design Taste — Anti-AI-Slop Defaults

Left alone, an agent ships the **statistical average UI**: Inter, purple gradient, centered hero, three identical cards, fade-up on everything. Every app then looks like every other AI app. This file has two parts: a **quality floor** every app must clear, and a **method that gives each app its own look**.

**Scope.** This file covers taste and the defaults to refuse. Related docs:

| Need | Go to |
|---|---|
| Write/read the project's `DESIGN.md` (tokens, components, do/don't) | [design-md.md](design-md.md) |
| Critique → audit → polish → harden loop | [design-review-loop.md](design-review-loop.md) |
| Model truncating or skipping sections | [../writing-for-agents/output-completeness.md](../writing-for-agents/output-completeness.md) |
| Token plumbing, `clamp()`, keyframes, glass | [css-scss-craft.md](css-scss-craft.md) |
| Light/dark tokens + toggle | [theming-dark-mode.md](theming-dark-mode.md) |
| Let the agent see the UI (screenshots) | [README.md → Playwright MCP](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless) |

## 🧭 Order of operations
1. **Read the brief.** Page kind, audience, vibe words, references, existing brand assets, quiet constraints (a11y-first, regulated, kids). Constraints beat taste.
2. **Classify.** Refinement (keep the current look) vs redesign (replace the look, keep the content). Missing `DESIGN.md` ≠ greenfield: if the code already has a coherent look, inherit it.
3. **Pick the register**: brand vs product (see Register below).
4. **Pick a direction** (see Make each app different) and set the **dials**.
5. **State it in one line before code:** `Reading this as: <page kind> for <audience>, <archetype> language, <type pairing>, <palette strategy>, dials V/M/D = 7/5/3.`
6. Build fully committed → check against the floor below → hand off to the [review loop](design-review-loop.md).
7. Ambiguous brief → ask **one** question ("closer to calm-minimal or loud-editorial?"), not a questionnaire.

**The brief wins.** A pinned font, palette or era overrides every default and ban in this file. The bans apply only to choices the brief left open.

## 🎲 Make each app different
Goal: **every app gets a distinct, recognizable look.** No shared house style. The floor stays the same for every app. The direction changes per app.

### Derive, don't default
1. Write **one sentence of product truth**: what it uniquely does, for whom, in what physical scene (desk at 9am? phone on a train? a dim studio?).
2. Name the **category rut**: the page this category always ships, plus its predictable opposite. Both are off-limits.
3. List **5–7 concrete references from the audience's world**: objects, places, publications, rituals, tools, and the graphic traditions they know (transit signage, lab notebooks, record sleeves, trail maps, tax forms, synth panels). Span at least 3 material families. No near-duplicates.
4. Turn the top 2–3 into directions (archetype + palette + type + one signature interaction). Pick one and **commit to it on every element**: nav, buttons, inputs and links all get restyled in the direction's own style.
5. **Self-check:** if someone could guess the look from the category alone ("fintech → navy + Inter"), or from the category plus the thing it avoided, rework it.

### Contrasting archetypes (starting points, not skins)
| Archetype | Palette strategy | Type character | Shape / depth | Motion | Density | Fits |
|---|---|---|---|---|---|---|
| **Soft premium** | Light neutral + one muted accent, diffuse tinted shadows | Wide grotesk display, calm sans body | Generous radius (16–24px), layered surfaces | Slow springs, 500–700ms focal entrance | Airy (2–3) | Consumer, health, lifestyle |
| **Minimal utility** | Near-monochrome, color only for state | One workhorse sans, strong weight steps | Crisp 4–8px radius, 1px dividers, no shadows | Near-static, 150ms feedback | Medium (4–6) | Docs, dev tools, productivity |
| **Swiss / brutalist print** | Paper + ink + one hazard accent | Heavy neo-grotesk, huge scale jumps, tight leading | 0 radius, visible grid lines, `gap:1px` rules | Hard cuts, no easing theatrics | Bimodal | Portfolios, manifestos, data-heavy editorial |
| **Terminal / telemetry** | Dark substrate, one signal accent | Mono for data only, sans for prose | 0 radius, framed compartments | Instant state, blinking caret at most | Packed (8–9) | Infra, monitoring, CLI products |
| **Editorial / magazine** | Committed ground color + ink | Display face with a point of view + readable text face | Asymmetric grid, pull quotes, big images | One authored scroll moment | Airy–medium | Media, long reads, culture |
| **Playful / maximal** | Full palette (3–4 named roles) or drenched | Chunky or quirky display, rounded sans | Pill controls, stickers, overlap, rotation | Bouncy only here, physics on key actions | Medium | Kids, games, community, events |
| **Civic / trust-first** | Restrained, high contrast, semantic color | System or proven UI face, large body | Predictable, low radius | Minimal, reduced by default | Medium (4–5) | Gov, health records, finance ops |
| **Cold luxury** | Silver-grey, chrome, smoke, one saturated pop | Refined sans, wide tracking on small caps only | Sharp or micro-radius, glass as a specific effect | Slow, precise | Airy | Hardware, fashion, high-end |

### Rotate across apps
- **Record the pick** in the project's [`DESIGN.md`](design-md.md): archetype, fonts, palette strategy, hues, dials.
- **Before a new app:** read the last few sibling apps' `DESIGN.md` files, then differ on **at least 2 of**: archetype, type pairing, palette family, light/dark, dial values.
- **Never reuse the same display face or accent hue in two consecutive apps**, even when it "fit" both.
- **Break the first-choice habit:** rank the candidates, then pick with a deterministic seed (e.g. product-name length mod N). Models reliably take option #1 from any list, and this list is no exception.
- Familiar is fine **when the user asks for it**. Then match the craft level of 2–3 named peer products, played straight.

### Palette families to rotate (as of 2026-09)
Pure mono + one saturated pop · Forest green + bone + amber · Black + warm tan (no beige) · Cobalt + one neutral · Terracotta + cool slate · Olive + brick + paper · Ink navy + signal orange · Plum + lime (committed) · Drenched single hue (the surface *is* the color).

## 🎛️ The three dials
Set them once per surface, then derive layout, motion and density from them.

| Dial | 1–3 | 4–7 | 8–10 |
|---|---|---|---|
| **VARIANCE** | Symmetric 12-col, equal padding, centered | Offsets, mixed aspect ratios, left headers over centered data | Asymmetric fr grids (`2fr 1fr 1fr`), big empty zones, overlap |
| **MOTION** | Hover/active only | CSS transitions + a staggered load-in | Scroll-driven sequences, pinned sections, physics |
| **DENSITY** | Gallery: `py-32+` sections | App: `py-16–24` | Cockpit: tight, 1px rules, no card boxes, tabular numerals |

| Surface | V | M | D |
|---|---|---|---|
| Landing (mainstream SaaS) | 7 | 6 | 4 |
| Agency / creative / portfolio | 8–9 | 7–8 | 3 |
| Editorial / blog | 6 | 4 | 3 |
| Product app / dashboard | 3–4 | 3 | 6–8 |
| Civic / regulated | 3 | 2 | 5 |
| Redesign, preserve look | match | +1 | match |

- VARIANCE ≥ 4 → asymmetric layouts **must collapse to a single column below 768px**.
- **Motion claimed = motion shown.** Ship MOTION ≥ 5 only if the motion actually works. Otherwise drop to 3 and ship a clean static page. Half-built scroll animation is worse than none.

## 🏷️ Register — brand vs product
Pick the register from the **surface**, not the company. A dev tool's landing page is brand. A fashion house's docs page is reading.

| Register | Visitor's job | Expression | Rule |
|---|---|---|---|
| **Persuade** (brand) | Decide + act | Design *is* the product; bold strategies allowed | Offer intelligible in one line, primary action visible, prove one thing only this product can |
| **Operate** (product) | Finish a task | Brand lives in precise details | Scanability, consistency, native affordances beat novelty. System/workhorse fonts OK |
| **Read** | Understand | Quiet, comfortable | Measure, wayfinding, hierarchy first |
| **Experience** | Be inside the work | Interface recedes | Artifact leads from the first viewport |

## 🔤 Typography
| Rule | Value |
|---|---|
| Body size floor (web) | `1rem` / 16px |
| Body measure | 65–75ch (45ch minimum) |
| Display max | ~6rem. A hero headline is ≤ 2–3 lines; if it wraps to 4, shrink the font, don't cut copy |
| Display tracking | `-0.02em` to `-0.03em`; never below `-0.04em` |
| Small caps / labels | positive tracking `0.04–0.08em` |
| Italic display with descenders (`g j p q y`) | `line-height ≥ 1.1` + bottom reserve, or descenders get clipped |
| Families | 1 is often enough; 2 maximum; a second family must do a job the first can't |
| Product UI scale | 3–5 sizes, weights 400/500/600, not just 400/700 |
| Light text on dark | +leading, +a touch of tracking, +one weight step |
| Numbers in tables/data | `font-variant-numeric: tabular-nums` |
| Headings | `text-wrap: balance`; paragraphs `text-wrap: pretty` |

- **Choose faces like objects from the subject's world.** Ask: what would this product look like as a physical object?
- **Reflex faces = you stopped looking** (on brand surfaces, as display): Inter-as-display, Fraunces, Instrument Serif/Sans, Playfair Display, Cormorant, Lora, Crimson, Newsreader, Syne, Space Grotesk, Space Mono, IBM Plex, DM Sans/Serif, Outfit, Plus Jakarta Sans, Roboto, Open Sans, Arial. Use one only with a reason no other face satisfies. "Books want a serif" or "tech wants mono" doesn't count as a reason.
- **Rule of rotation beats any allow-list.** A recommended face becomes slop the moment every app uses it. Record the pick, and don't repeat it next app.
- **Serif is not "premium".** "Creative brief → serif display" is the most tested AI tell. Pick a serif only for a real editorial/heritage/publication identity.
- **Emphasis = italic or weight of the same family.** Never drop a random serif word into a sans headline.
- **Monospace only for code, data, measurement.** Using it as a costume for "technical" is a tell.
- **No system display face** (Impact, Arial Black, the platform sans) as a brand's display voice. Source and self-host the right face ([assets](assets-optimization.md)). Load only the weights you use, use `font-display: swap`, and set metric-matched fallbacks.
- Sentence case over Title Case. Avoid tracked all-caps subheaders everywhere.

## 🎨 Color
1. **Pick a strategy before picking colors:** Restrained (neutrals + 1 accent; default for Operate/Read) · Committed (one saturated color owns 30–60% of the surface) · Full palette (3–4 named roles) · Drenched (the surface is the color).
2. **Build roles, not swatches:** canvas, raised surface, text primary/secondary, action, focus, selection, border, success/warning/error/info, data scale.
3. **Author in OKLCH** for new palettes: lightness and chroma move predictably. Lower the chroma near white and black.

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

- **One neutral family, one hue.** Don't mix warm and cool greys. Pure grey is fine if the direction calls for it.
- **No pure `#000` / `#fff`.** Use off-black and off-white.
- **Accent consistency lock:** once chosen, one accent everywhere. No surprise teal badge in the footer.
- **Keep the accent rare.** Spend it on the primary action and state, not on decoration.
- **Text on colored surfaces:** derive secondary text from that hue. Grey text on color looks dead.
- **Shadows tinted** to the surface hue, with an offset + soft blur. A zero-offset colored glow isn't depth, it's decoration.
- **Contrast (WCAG AA):** body 4.5:1 · large text 3:1 · controls/icons/focus 3:1. Check hover, disabled, placeholder, text on images, and both themes. Never let color be the only signal.
- **Light vs dark comes from the use scene** (who, where, what light), not the category. Design dark mode deliberately; don't just invert the light theme ([theming](theming-dark-mode.md)).
- **One theme per page.** Sections don't flip from dark to cream mid-scroll unless that's a single, deliberate device.
- **Calibration: the saturated AI looks.** Legitimate only if the brief asks for them:
  - Purple→blue gradient + centered hero on a dark mesh.
  - Warm cream ground + high-contrast serif display + terracotta/oxblood accent (also: beige + brass + espresso for "premium" products).
  - Near-black + one neon accent + glowing edges.
  - Broadsheet hairlines + italic display serif + tiny tracked mono labels.

## 📐 Layout & space
- **Squint test:** blur the page. The primary element, secondary element and groups should still read in order.
- **Proximity before containers.** Group by spacing first. Add borders/cards only when spacing can't do the job.
- **Rhythm = contrast.** Tight inside groups, generous between them, more space above a heading than below. Don't repeat one gap value until everything weighs the same.
- **Spacing scale on a 4px base** (4, 8, 12, 16, 24, 32, 48, 64, 96…). No one-off values. Use `gap` for sibling rhythm.
- **Cards only when elevation means hierarchy.** Otherwise use `border-top`, dividers or whitespace. **Never nest a card inside a card.**
- **Declare elevation once:** border *or* shadow. A 1px border under a wide soft shadow is a ghost card.
- **Shape lock:** one radius system (all-sharp · 12–16px soft · pill for small controls only), documented and applied everywhere.
- **Break the 3-equal-cards row.** Use asymmetric grids, zigzag (max 2 in a row), featured + rest, horizontal scroll, or plain prose.
- **Bento is a tool, not a default.** Cell count = item count (no blank tiles). Vary the sizes. Give ≥ 2 cells real visual variation (image, tint, pattern). Don't stack six same-shape rows.
- **Layout families per page:** each family (3-col cards, split, full-bleed quote…) at most once. 8 sections → at least 4 families.
- **No split header** (big headline left, tiny orphan paragraph floating right). Stack them unless the right column carries a real visual.
- **Hero:** fits the first viewport. Headline ≤ 2–3 lines, subtext ≤ ~20 words, CTA visible without scrolling, top padding ≤ ~6rem. Max 4 text elements. Logo walls, trust strips and pricing teasers go in the section **below** the hero.
- **Centered hero only for manifesto/launch copy.** Otherwise use a split, left-aligned text with a right-aligned asset, or asymmetric whitespace (when VARIANCE > 4).
- **Nav:** one line on desktop, ≤ 80px tall. The active page is marked.
- **Align across siblings:** CTAs pinned to card bottoms, feature lists starting at the same Y, shared baselines. Nudge optically after you look at the render.
- **Mechanics:** `min-height: 100dvh`, not `100vh`. Grid over flex percentage math. Max-width container (~1200–1440px). Container queries for reused components. DOM/focus order matches visual order. A documented z-index scale, no `9999`.
- **Operate surfaces:** predictability *is* the affordance. Keep asymmetry for Persuade/Experience.

## 🎞️ Motion
**Every animation must answer "what does this communicate?"** Valid answers: feedback, state change, spatial continuity, attention at a meaningful moment. "It looked cool" is not an answer.

| Duration | Use |
|---|---|
| 100–150ms | Press/toggle feedback |
| 150–300ms | Routine state change, hover |
| 300–500ms | Layout change, overlay, view transition |
| 500–800ms | The one authored focal entrance |

- **Easing:** exponential ease-out for arrivals, `cubic-bezier(0.16, 1, 0.3, 1)`. Exits faster than entrances. No `linear` on UI. Bounce/elastic only in a playful world, never by reflex.
- **One authored moment per surface**, product-specific. Not the same fade-and-rise on every section. Stagger only real lists, and cap the total delay.
- **Content visible by default.** Animate *from* a visible state so a failed script never hides the page.
- **Materials:** transform + opacity as the base. Clip-path, mask, bounded blur, shadow and `backdrop-filter` are allowed when they stay smooth. Never animate `width/height/top/left/margin` (use FLIP/transforms). Set `will-change` only during a known animation.
- **Scroll:** IntersectionObserver, CSS scroll-driven animations (`animation-timeline: view()`), or the existing motion lib. **Never a raw `scroll` listener** or scroll position stored in reactive state.
- **Limits:** max one marquee per page. Non-essential loops stop offscreen. Grain/noise only on a `position: fixed; pointer-events: none` layer, never on scrolling containers. No custom cursors. Don't animate an image on hover; animate its container.
- **Reduced motion = fewer + gentler, not none.** Remove spatial movement and keep opacity/color feedback that confirms actions.

```css
.reveal { opacity: 1; }                                   /* visible by default */
@media (prefers-reduced-motion: no-preference) {
  .reveal { animation: rise 600ms cubic-bezier(0.16, 1, 0.3, 1) both;
            animation-timeline: view(); animation-range: entry 0% cover 30%; }
}
@keyframes rise { from { opacity: 0; transform: translateY(12px); } }
```

## ✍️ Copy & content
- **Real draft copy, never lorem ipsum.** Use the product's own nouns and verbs. AI-cute copy is worse than plain copy.
- **Banned filler:** Elevate, Seamless, Unleash, Supercharge, Next-gen, Revolutionize, Game-changer, Empower, Delve, Tapestry, Unlock, "In today's fast-paced world", "Built for the future of…".
- **No emoji in UI** (headings, buttons, alt text) unless the brand is explicitly chat/social-native, and even then used sparingly.
- **No placeholder people/brands:** John Doe, Jane Smith, Acme, Nexus, SmartFlow. Use locale-appropriate names and believable brands.
- **No fake-precise or fake-round stats** (`99.99%`, `10x`, `48k users`). Numbers come from real data or are visibly labeled as sample data. Never invent prices, customers, benchmarks or capabilities.
- **Buttons = verb + object**, ≤ 3 words for primary CTAs, one line at desktop. **One label per intent** per page (not "Get in touch" + "Let's talk" + "Contact us").
- **Errors:** what failed + why (if known) + how to recover. No "Oops!", no blame, no raw codes as the headline. Success messages without exclamation marks.
- **Destructive actions** name the object + consequence. Prefer undo over confirm. The confirm button repeats the verb, never `OK`/`Yes`.
- **Forms:** persistent label above the input, placeholder is an example (never the label), error below the field.
- **Say it once.** No intro repeating the heading, no micro-meta sentence under a heading ("Each of these ships today, not a roadmap promise").
- **One copy register per page.** Don't mix terminal-speak, editorial prose and marketing punch.
- **Quotes:** ≤ 3 lines, attributed with name + role. Real typographic quotes.
- **Dashes:** the em/en dash used as a separator in UI copy is a strong LLM tell. Use periods, commas, colons, hyphenated ranges.

## 🖼️ Icons & imagery
- **One icon family, one stroke width**, set globally. Using Lucide by default *is* the AI default; choose the icon set deliberately.
- **No emoji or unicode glyphs as icons.** No hand-drawn icon paths.
- **No cliché metaphors** (rocket = launch, shield = security, lightbulb = idea).
- **Real imagery or honest placeholders.** Use generated or real photos. Otherwise leave labeled slots (`<!-- TODO: hero product photo 1600×1200 -->`) and list them for the user. A text-only page is unfinished, not minimalist.
- **No div-built fake screenshots** (fake dashboards, task lists, terminals made of styled boxes). Use a real screenshot, a live mini-component, or no preview.
- **No sketchy/doodle SVG illustration**, `feTurbulence` grain scenes, or geometric masks faking organic cutouts. Crisp vector geometry and diagrams are fine.
- **Backgrounds get texture only from the subject's world.** Decorative grid overlays, stripes and crosshair hairlines need a real map/blueprint/canvas underneath.
- **Logo walls:** real SVG logos only, no category labels under them, placed below the hero, legible in both themes.
- **Nothing overlaid on photos** (tags like `PLATE · 02`), no fake photo credits.

## 🧩 States & the parts you didn't draw
- Every interactive element needs hover, `:active` (≈ `scale(0.98)` / 1px press), focus-visible, disabled, loading, error, and empty states.
- **Loading:** skeletons shaped like the final layout, not a lone spinner. **Empty:** say which empty case it is (first use / no results / filtered / no permission) and give the next action.
- **Theme the browser defaults** from the palette. This is the cheapest sign a page was built, not assembled, and the one agents skip most:

```css
::selection { background: color-mix(in oklch, var(--accent) 30%, transparent); color: var(--ink); }
:root { caret-color: var(--accent); accent-color: var(--accent); scrollbar-color: var(--line) transparent; }
:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
a { text-underline-offset: 0.2em; text-decoration-thickness: 1px; }
```

## 🚫 Banned patterns (unless the brief asks)
| Area | Slop tell |
|---|---|
| Page scaffold | Three identical icon + heading + text cards · hero-metric template (big number, small label, stat row) · card in card · modal for a task that needs no interruption |
| Labels | Eyebrow/kicker above every heading (ration hard: ≤ 1 per 3 sections, ideally 0) · section numbers `01 / 02` · `Step 1 / Phase 01` labels · `BETA`/`v0.6` in hero · "Scroll to explore" cues |
| Surface | Gradient text · glassmorphism as decoration · `border-left` color stripe > 1px on cards/alerts · hard offset `4px 4px 0` shadows outside a real neo-brutal world · neon outer glow · pure black bg |
| Decoration | Status dots before every nav item/row · `·`-separated meta strips · decorative city/time/weather strips · `BRAND. MOTION. SPATIAL.` strip under hero · rotated vertical text · marquee #2 |
| Lists/data | Hairline above *and* below every row of a long spec list · progress bars with grey tracks as comparison visuals · 20-row tables on a marketing page |
| Components | Filled + ghost button pair everywhere · sun/moon toggle by reflex · 3-tower pricing that differs only in height · 3-card testimonial carousel with dots · 4-column footer link farm |
| Motion | Same fade-up on every section · bounce by reflex · infinite micro-loops on static info · scroll-jacking without a narrative reason |

## ✅ Pre-ship taste check (60 seconds)
- [ ] Direction line stated. The look can't be guessed from the category alone. It differs from sibling apps on ≥ 2 axes.
- [ ] No reflex face as display. ≤ 2 families. Body ≥ 16px, 65–75ch.
- [ ] One neutral family, one locked accent, OKLCH tokens, no `#000`/`#fff`, AA contrast in both themes.
- [ ] One radius system. No card-in-card. Hero fits the viewport. ≤ 1 eyebrow per 3 sections.
- [ ] Each animation has a stated purpose. Reduced-motion path exists. No scroll listeners.
- [ ] Zero filler words, lorem, emoji, fake stats, duplicate CTA intents.
- [ ] All states + themed selection/caret/focus/scrollbar.
- [ ] Screenshot it ([Playwright MCP](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless)) at mobile + desktop, then run the [review loop](design-review-loop.md).

## 📋 Paste-ready block (CLAUDE.md / DESIGN.md)
```markdown
## Design taste
- Brief wins: pinned fonts/palette/era override everything below.
- Before UI code, state: "Reading this as: <kind> for <audience>, <archetype>, <fonts>, <palette strategy>, V/M/D=<n/n/n>."
- Direction: derive from product truth + audience's world, not the category. If the look is guessable from the category, rework.
- Differ from sibling apps on >=2 of: archetype, type pairing, palette family, light/dark, dials. Record picks in DESIGN.md.
- Register: Persuade=bold allowed; Operate/Read=predictable, brand in details.
- Type: body >=16px, 65-75ch; display tracking -0.02..-0.03em (floor -0.04); <=2 families; tabular-nums for data; no reflex faces (Inter/Fraunces/Instrument/Playfair/Space Grotesk/DM/Outfit...) as display without a reason.
- Emphasis = same-family italic/weight. Mono only for code/data.
- Color: OKLCH tokens; one tinted neutral family; one locked accent; no #000/#fff; no purple-blue gradient, no gradient text, no neon glow; tinted offset shadows; AA contrast both themes.
- Layout: 4px spacing scale; tight groups, generous separation; cards only for real elevation; never card-in-card; one radius system; no 3-equal-card row; bento cells = items; hero fits viewport; eyebrows <=1 per 3 sections; no section numbers.
- Motion: purpose or delete; 150-300ms routine, exits faster; ease-out cubic-bezier(0.16,1,0.3,1); no reflex bounce; one authored moment; content visible by default; transform/opacity/clip-path only; prefers-reduced-motion = fewer + gentler; no scroll listeners; <=1 marquee.
- Copy: real copy, no lorem/Acme/John Doe; no Elevate/Seamless/Unleash/Next-gen; no emoji in UI; no fake stats; verb+object CTAs <=3 words; one label per intent; errors = what + why + fix.
- Icons: one set, one stroke; no emoji/glyph icons; no div-built fake screenshots; real images or labeled TODO slots.
- Theme ::selection, caret, focus-visible, scrollbar, underline offset. Ship hover/active/focus/disabled/loading/empty/error.
```

---
Sources / further reading: [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) · [pbakaus/impeccable](https://github.com/pbakaus/impeccable).
