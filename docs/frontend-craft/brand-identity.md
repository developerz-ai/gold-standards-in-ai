# 🏷️ Brand Identity — name to logo to kit, per app

**Every app gets its own brand: a strategy, a mark, a palette, a type voice, an image style, and a kit of files.** It's written down before the first screen and wired into `DESIGN.md` + `tokens.css`. No brand → the agent ships a generic lightning-bolt icon, Inter, and a purple gradient, and the app looks like every other AI app.

**Scope: the identity layer only.** Direction + banned defaults → [design-taste.md](design-taste.md) · [design-directions.md](design-directions.md). Tokens doc → [design-md.md](design-md.md). Words → [ux-copy.md](ux-copy.md). Motion → [motion-and-delight.md](motion-and-delight.md). From a reference image → [reference-image-design.md](reference-image-design.md). Image gen API → [media-generation.md](../ai-agents/media-generation.md). `<head>` tags → [seo.md](seo.md). Compression → [assets-optimization.md](assets-optimization.md).

## 🧭 Order of operations
Strategy (5 lines) → core metaphor → mark (mono, SVG) → color → type → imagery + icons → kit export → wire into tokens + `DESIGN.md`.

A brand kit is **a visual argument for why the product exists**, not decoration. Every choice answers: what does it represent, what's the metaphor, how does the mark express it, how does it scale, why is it ownable.

## 📝 Brand strategy in 5 lines
Write this first. Everything visual derives from it. Lives in `PRODUCT.md` under `## Brand Commitments`.
```markdown
## Brand Commitments
- Audience: <who, in what scene — "solo accountants, laptop at a kitchen table, April">
- Promise: <the one outcome — "your books close themselves by Friday">
- Personality: <how it behaves — "the calm senior colleague who's seen every audit">
- 3 adjectives: <felt, not features — "exact, warm, unhurried">
- What we're NOT: <2-3 anti-references — "not a bank, not a neon crypto app, not cute">
- Core metaphor: <one object/idea — "the ledger ruler">
```
- Adjectives = what a user **feels**. `fast, AI-powered, modern` = features/filler → reject.
- "Not" line is load-bearing: it's what the review loop checks against.
- Existing name/logo/colors the user made binding → record here; never redesign them silently.
- Unknown facts → mark `[undecided]`. Never invent customers, awards, or claims.

## 🎲 A distinct identity per app
**Rule: if someone can guess the brand from the category alone, rework it.** (`fintech → navy + shield`, `AI → purple glow + sparkle`, `security → padlock`.)

### Derive the metaphor from meaning
| Category | Core ideas | Symbol logic (pick, then abstract) |
|---|---|---|
| Developer tool | building, precision, control | cursor, frame, scaffold, grid, bracket |
| AI assistant | delegation, clarity | orbit, signal, path, node, handoff |
| Security | vigilance, boundary | eye, seal, protected core, perimeter |
| Voice / audio | rhythm, flow | waveform, pulse ring, speech path |
| Compliance / legal | order, trust | seal, stamp, document fold, monogram |
| Productivity | focus, momentum | path, block, check, horizon |
| Robotics / drones | flight, vision, mission | wing, crosshair, route, zone |
| Luxury / editorial | taste, ritual, restraint | monogram, emboss, vessel, mark |
| Games / chance | reward, tension | gem, die face, card corner, trophy |

Pick from the **product's action**, not the category label. Symbols = raw material: reduce, cut, fold — never literal.

### Rotate against sibling apps
Before choosing, read the last 2-3 projects' `DESIGN.md` + `brand/`. **Always change:** accent hue family, type pairing, mark construction method. **Often change:** canvas polarity, visual mode.

### Visual modes (starting points, not skins)
| Mode | Fits | Palette | Mark logic | Mood |
|---|---|---|---|---|
| Dark builder | dev tools, infra, agents | near-black + one cyan/lime/coral | cursor+frame, modular block | precise, sharp |
| Dark operator | sales, growth, automation | black + amber/red | signal, loop, switch | fast, tactical |
| Calm nature | strategy, travel, wellness | deep green + lime + fog | path, horizon, fold | calm, focused |
| Vigilant | security, monitoring | navy/black + alert chip | eye, boundary, core | serious, exact |
| Light editorial | legal, privacy, docs | ivory + deep blue + red/gold | seal, stamp, monogram | refined, institutional |
| Luxury | beauty, hospitality | stone/espresso + serif | monogram, vessel | adult, expensive |
| Voice | speech, chat | indigo + lilac | wave+initial, orb | fluid, intimate |
| Cultural | music, events, creative | bold pop + halftone/print | custom wordmark, attitude icon | memorable, controlled |

Known counter-positioning move (seen in public DESIGN.md files, 2026-09): warm cream + coral + serif display in a category that defaults to cool slate + blue sans. **Find your category's default, then invert one axis deliberately.**

## 🔣 Logo system
**Rule: simple, symbolic, monochrome-first, SVG, legible at 16px.** Looks like it came from research and reduction.

**Parts:** symbol (reduced metaphor, square-fit) · wordmark (from the display face) · horizontal lockup · stacked lockup (if square slots) · mono ink-on-light + white-on-dark · optional construction sheet.

### Concept methods (use one, combine max two)
| Method | How | Example |
|---|---|---|
| Monogram + meaning | initial + metaphor via cut/fold/negative space | `K` + kite frame; `S` + sound path |
| Product action | main verb as abstract shape | build → scaffold; automate → handoff loop |
| Metaphor fusion | two ideas, one reduced form | shield + mountain; moon + waveform |
| Negative space | the empty part carries the idea | hidden arrow, protected center |
| Construction geometry | circles, diagonals, modules on a grid | orbit path, layered cards |

### Rules
- **Monochrome first.** Design in one flat color. If it fails in black, color won't save it.
- **16px test.** Render the symbol at 16×16 and 32×32. Detail vanishing → simplify or ship a separate small-size variant.
- **≤ 3 shapes** in the symbol at small sizes. Strokes ≥ 1.5px at 16px → draw fills, not hairlines.
- **Clear space** = one stated unit (e.g. symbol cap-height) all sides. **Min size**: symbol 16px, lockup ~80px wide.
- **SVG source of truth**, `viewBox` square for the symbol, paths only (no embedded fonts, no raster).
- **Wordmark = outlined paths**, not live text. Letter-spacing tuned by hand; kern the problem pairs.
- **Optical center**, not geometric. Circles overshoot flat edges by ~1-3%.
- **One mark, repeated identically** across UI, icon, OG, docs. No per-surface variants.

### Banned (unless brief demands)
Generic lightning bolt · rocket · lightbulb · padlock-for-security · sparkle/✨ for AI · brain · random animal · fake heraldic crest · gradient-only mark · swoosh/orbit around the name · copied or near-copied famous marks · clip-art · a plain letter in a rounded square with no idea in it.

## 📱 App icon & favicon set
**Rule: one SVG source → generated set. Symbol only, never the wordmark.** Set below current as of 2026-09.

| File | Size | Notes |
|---|---|---|
| `favicon.ico` | 16/32/48 multi-size | legacy + some crawlers |
| `icon.svg` | vector | modern browsers; can embed `prefers-color-scheme` media query |
| `apple-touch-icon.png` | 180×180 | opaque bg, no transparency, ~10% padding |
| `icon-192.png` | 192×192 | PWA manifest |
| `icon-512.png` | 512×512 | PWA manifest, splash |
| `icon-maskable-512.png` | 512×512 | symbol inside the center **80% safe zone**; full-bleed bg |
| `icon-monochrome.svg` | vector | manifest `purpose: "monochrome"` (themed icons); single color, transparent bg |
| Native (if shipping) | iOS 1024×1024 · Android adaptive 108dp fg/bg layers (66dp safe) | platform stores |

```html
<link rel="icon" href="/favicon.ico" sizes="48x48">
<link rel="icon" href="/icon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
<link rel="manifest" href="/manifest.webmanifest">
<meta name="theme-color" content="#1b1a17">
```
- Favicon in a dark browser tab: test both themes. SVG favicon can swap fill via `@media (prefers-color-scheme: dark)` inside the SVG.
- Icon bg = brand primary or canvas color; mark in the contrasting neutral. Not a gradient soup.
- Manifest `icons[]`: list 192, 512, maskable (`"purpose": "maskable"`), monochrome. PWA wiring → [pwa-offline.md](pwa-offline.md).

## 🖼️ OG / social image templates
**Rule: a template, not a one-off. Rendered per page from route data.** Tags → [seo.md](seo.md#open-graph--twitter-social-cards).

| Spec | Value |
|---|---|
| Size | 1200×630 (1.91:1); export ≤ 300 KB |
| Safe area | keep text/mark inside ~1080×510 center; some platforms crop to square |
| Text | page title ≤ 60 chars, ≥ 48px, display face; 1 line of meta max |
| Mark | symbol or lockup, one corner, same position every card |
| Background | brand canvas or one brand image treatment; no stock photo |
| Variants | `default` (home/brand) · `article` (title + section eyebrow) · `product` (screenshot crop + title) |

- Build as an HTML/SVG template rendered to PNG at build or on request; cache by content hash.
- Test with the longest real title; never overlapping the mark. Home card doubles as the brand's "cover panel": mark + one tagline + canvas.

## 🎨 Color identity
**Rule: one dominant palette — base, primary, one accent, neutrals. Accent repeats everywhere; nothing else competes.**

| Role | Count | Job |
|---|---|---|
| Primary (brand hue) | 1 + ramp | mark, primary CTA, focus, links |
| Accent | 0-1 | one signature moment (highlight, badge, chart series 1) |
| Neutrals | 4-6 + ramp | canvas, surfaces, ink, muted, hairline; tinted toward the primary hue or warm/cool by mood |
| Semantic | 3-4 | success, warning, danger, info — respect domain conventions |

- Hue from **meaning + direction**, never category default. Rotate hue family vs siblings → [design-taste.md](design-taste.md#palette-families-to-rotate-as-of-2026-09).
- **Brand color ≠ text color.** A saturated brand hue often fails 4.5:1 as text. Ship a `primary-text` step that passes.
- Never pure `#000`/`#fff` for ink/canvas unless polarity-as-signature (declare it).
- No generic purple-blue glow, no rainbow, no neon unless the mode is explicitly cultural.

### OKLCH ramps
Author in OKLCH: lightness steps predictably, hue stays put. **Reduce chroma near white and black** — max chroma belongs in the middle.
```css
:root {
  /* primary: terracotta, hue 40 */
  --brand-100: oklch(93% 0.035 40);
  --brand-500: oklch(64% 0.14 40);   /* mark, CTA fill */
  --brand-600: oklch(56% 0.14 40);   /* hover / primary-text on light */
  --brand-700: oklch(47% 0.12 40);   /* pressed, text on tint */
  --brand-900: oklch(28% 0.06 40);
  /* neutrals tinted toward brand hue */
  --ink:    oklch(20% 0.012 40);
  --canvas: oklch(98% 0.006 40);
}
```
### Contrast (WCAG AA — verify with a checker, never by eye)
Body text **4.5:1** · large text (≥24px or ≥18.66px bold) **3:1** · controls, icons, focus ring **3:1** · mark vs background **≥3:1**, aim higher.

- Check both themes, hover/disabled states, and text on brand-tinted surfaces.
- Dark theme = its own composition (surface ladder, lighter primary step), not inverted → [theming-dark-mode.md](theming-dark-mode.md).
- Color never the only signal: pair with text, shape, or position.

## 🔤 Typographic identity
**Rule: display + body (+ mono), max 2 families + mono. Open licenses by default.**

| Role | Job | Character choice |
|---|---|---|
| Display | wordmark basis, headlines, OG titles | carries personality: serif, wide grotesk, slab, rounded |
| Body | UI + reading | neutral, great at 14-16px, wide language coverage |
| Mono (optional) | code, numbers, eyebrows | pairs with the body's x-height |

### Licensing
| License | Web | App bundle | Logo from outlines | Default? |
|---|---|---|---|---|
| SIL OFL (most open fonts) | yes, self-host | yes | yes | ✅ prefer |
| Apache 2.0 | yes | yes | yes | ✅ |
| Commercial desktop-only | no — needs web license | no | usually yes | ❌ |
| Commercial web (pageview-metered) | yes, capped | separate license | check EULA | only if brief pins it |
| Proprietary brand faces | no | no | no | ❌ never copy |

- Record family, version, license, source URL in `brand/fonts/LICENSES.md`.
- Brief pins a paid/proprietary face → pick an open **substitute with matching metrics** and write "substitute for X" in `DESIGN.md`.
- Self-host subset `woff2` → [assets-optimization.md](assets-optimization.md#fonts).
- Wordmark: start from the display face, outline, then customize 1-2 glyphs (a cut, a joined pair, a tail) so it's ownable.
- Don't reuse the last project's pairing. Inter/Roboto/system-only for display = the AI default; allowed for body in operate-register product UI.

## 📸 Imagery & illustration style
**Rule: one treatment, art-directed, from the product's world.** Write it as a spec, not a vibe.
```markdown
Imagery: <subject world> · <treatment> · <crop> · <color grade> · <texture>
e.g. "Misty ridgelines and trail markers · duotone brand-900→brand-100 · wide horizon crops,
      subject low third · cool, desaturated · fine grain 4%"
```
- ✅ Duotone/tritone in the brand ramp · halftone/print texture on product-world scenes · dramatic object crops · real screenshots in brand frames · crisp vector diagrams.
- ❌ Stock people/handshakes · robots, glowing brains, circuit boards · busy collages, floating 3D blobs · div-built fake dashboards · doodle SVG, mixed styles.

- Generated imagery: same style string on every prompt + a reference image → consistency. Route via [media-generation](../ai-agents/media-generation.md).
- Illustration: one line weight, one corner style, palette-only fills. Mascot only if it fits personality — then it's a system (poses, sizes, where it appears).
- Texture only from the subject's world ([design-taste.md](design-taste.md#-icons--imagery)).

## ✳️ Iconography style
**Rule: one family, one stroke width, one grid — matching the mark's geometry.**

- Style: outline · filled · duotone — pick one. Grid 24px, 2px padding. Stroke 1.5px or 2px, global.
- Terminals match the logo: sharp mark → square caps; round mark → round caps/joins.
- Source: one open-licensed set, chosen deliberately (not the reflex default) + custom glyphs on the same grid.

- No emoji or unicode as icons. No mixed sets.
- Custom product icons (the 3-5 unique verbs) drawn to match — that's where brand shows in operate-register UI.
- Inline SVG, `currentColor`, ≤ 5 KB each.

## 🗣️ Voice (pointer)
Voice = personality line + 3 adjectives, applied to words. Full rules → [ux-copy.md](ux-copy.md).
- **Tagline**: ≤ 5 words, specific, no buzzwords. ✅ "Nothing random." "On guard." ❌ "Empowering the future of work."
- Brand name + tagline + one URL + one command = all the text a brand board needs.

## 📦 Brand kit deliverables
```text
brand/
  BRAND.md                  # 5-line strategy, metaphor, usage rules, clear space, min sizes, don'ts
  logo/
    symbol.svg              # source of truth, square viewBox
    symbol-mono-dark.svg    # ink on light
    symbol-mono-light.svg   # white on dark
    wordmark.svg            # outlined paths
    lockup-horizontal.svg
    lockup-stacked.svg
    construction.svg        # optional: grid + geometry
  icons/                    # favicon.ico, icon.svg, apple-touch-icon.png,
                            # icon-192.png, icon-512.png, icon-maskable-512.png, icon-monochrome.svg
  social/
    og-default.png          # 1200x630
    og-template.(html|svg)  # per-page renderer source
  color/palette.md          # OKLCH + hex, roles, contrast pairs checked
  fonts/                    # woff2 subsets + LICENSES.md
  imagery/style.md          # treatment spec + 3-5 approved reference images
  board.png                 # optional overview board (below)
```
- [ ] Symbol passes 16px + monochrome + both-theme tests; wordmark outlined; one clear-space rule
- [ ] Icon set generated from `symbol.svg`; maskable safe zone checked; OG fits longest real title
- [ ] Every text pair ≥ AA; fonts licensed + recorded; imagery + icon style written as specs
- [ ] SVGs minified, PNGs optimized ([assets-optimization.md](assets-optimization.md)); tokens + `DESIGN.md` in the same PR

## 🤖 How an agent produces a brand kit
1. **Strategy**: fill the 5 lines from `PRODUCT.md`; ask the user one question if the "not" line is empty.
2. **Metaphor shortlist**: 3 metaphors × method table → 3 mark directions, each one sentence.
3. **Explore** with image generation (raster = sketches only), or skip straight to step 4 for geometric marks.
4. **Draw the SVG by hand** from geometry: circles, rects, paths on a 24- or 48-unit grid. Simple marks come out cleaner coded than traced.
5. **Or vectorize** a chosen raster: trace → delete stray nodes → snap to grid → merge shapes → fix optical balance → re-test at 16px. Tracing output is never final.
6. **Board** (optional): generate one overview board to sanity-check coherence; the board is a review aid, not a deliverable to ship.
7. **Export** the kit, wire tokens, update `DESIGN.md`, screenshot favicon + OG in the running app.

### Prompt: mark exploration (image model)
```text
Logo symbol exploration sheet for "<NAME>", a <category> for <audience>.
Core metaphor: <metaphor>. Method: <monogram+meaning | negative space | ...>.
6 variations in a 3x2 grid, flat single color <ink hex> on <canvas hex>, no text,
no gradients, no shadows, no mockups. Bold simple geometry that survives 16px.
Avoid: lightning bolts, rockets, sparkles, shields, brains, crests, clip-art,
any resemblance to existing well-known logos.
```
### Prompt: SVG mark (coding model)
```text
Write symbol.svg for "<NAME>". viewBox="0 0 48 48". Metaphor: <metaphor>,
built with <method>. Max 3 shapes. Fills only, no strokes under 3 units.
Single fill currentColor. Snap coordinates to integers or .5. Optical-center it.
Then write wordmark notes: display face <font>, which 1-2 glyphs to customize.
Render at 16, 32, 128px and describe what breaks.
```
### Prompt: brand board (image model, review aid)
```text
Premium brand-guidelines board for "<NAME>". 3x3 grid, strong even gutters,
<dark|light> presentation canvas, sparse large type, one idea per panel.
Strategy: audience <..>, personality <..>, metaphor <..>.
Panels: logo cover · construction · app/browser application · tagline "<≤5 words>" ·
color swatches <hexes> · type specimen <display/body> · physical application ·
image direction (<imagery spec>) · UI detail (chips, input, icon row).
Same mark identical in every panel. No lorem ipsum, no tiny fake text,
no stock people, no rainbow, no copied real logos.
```
- Board rhythm: quiet cover → technical construction → functional UI → emotional image → detail. Not every panel loud.
- Reference boards: extract grid, density, type scale, accent logic. **Never** copy their mark, name, slogan, or composition.

## 🔌 Wire the brand into tokens + DESIGN.md
**Rule: `brand/` holds files; `tokens.css` holds values; `DESIGN.md` holds values + intent. Same names on all three.**
```css
/* src/styles/tokens.css — brand primitives → semantic roles */
:root {
  --brand-500: oklch(64% 0.14 40);
  --brand-600: oklch(56% 0.14 40);
  --color-accent: var(--brand-500);
  --color-accent-text: var(--brand-600);
  --color-focus: var(--brand-500);
  --font-display: "Fraunces", Georgia, serif;
  --font-body: "Source Sans 3", system-ui, sans-serif;
  --logo-min: 16px;
}
```
| `DESIGN.md` section | Gets from the brand |
|---|---|
| 1 · Atmosphere | north star = core metaphor; 3 adjectives; "what we're not" as visual anti-refs; mark + signature device in Key Characteristics |
| 2 · Colors | ramps with roles; `The <Name> Rule` for primary use |
| 3 · Typography | families, license note, wordmark basis |
| 7 · Do's/Don'ts | logo misuse: no recolor, no stretch, no effects, clear space, min size |
| 9 · Agent guide | "Logo: `brand/logo/symbol.svg` only. Never redraw, never approximate with an icon." |

`CLAUDE.md` one-liner:
```text
Brand: read brand/BRAND.md + DESIGN.md before UI. Use brand/logo/*.svg only. No colors/fonts outside tokens.css.
```
Token plumbing → [css-scss-craft.md](css-scss-craft.md#design-tokens--css-custom-properties) · sync + drift check → [design-md.md](design-md.md#-keep-it-in-sync-with-css-tokens).

## 🚫 Pitfalls
- Mark picked from the category label → looks like 50 competitors.
- Logo only tested at hero size → mush in the browser tab.
- Brand hue used as text color → fails contrast; ship a `-text` step.
- Agent redraws the logo inline per component → five slightly different marks. Import the file.
- Commercial font shipped without a web license, or proprietary font copied from a reference brand.
- Sibling app's accent/type reused "because it worked" → house style, not identity.

Sources: [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (brandkit skill) · [pbakaus/impeccable](https://github.com/pbakaus/impeccable) (PRODUCT.md, DESIGN.md, colorize/typeset/document references) · [voltagent/awesome-design-md](https://github.com/voltagent/awesome-design-md) · [W3C Web App Manifest](https://www.w3.org/TR/appmanifest/) · [WCAG 2.2](https://www.w3.org/TR/WCAG22/) · [SIL Open Font License](https://openfontlicense.org/)
