# 🖼️ Reference-Image-First Design — generate the look, then build it

**Rule: when visual quality matters and image gen is available, the first artifact is a picture, not code.** Generate section comps → pick one direction → measure it into tokens + `DESIGN.md` → build to match → diff build against comp. The image is the design source. Code is the translation layer.

Related: image-gen API/plumbing → [media-generation.md](../ai-agents/media-generation.md) · identity file → [design-md.md](design-md.md) · screenshot/critique loop → [design-review-loop.md](design-review-loop.md) · banned defaults → [design-taste.md](design-taste.md) · picking a direction → [design-directions.md](design-directions.md) · brand marks → [brand-identity.md](brand-identity.md) · motion the comp only implies → [motion-and-delight.md](motion-and-delight.md) · real copy → [ux-copy.md](ux-copy.md) · viewports/devices → [adaptive-ui.md](adaptive-ui.md) · asset weight → [assets-optimization.md](assets-optimization.md).

## ❓ Why image first
| Without a comp | With a comp |
|---|---|
| Agent designs *in code* → converges on the average UI (centered hero, 3 equal cards, purple glow) | Image model explores composition freely; code only translates |
| "Looks good" judged from memory | Build is **diffed against a target** — misses become numbers |
| Taste debated in prose | Taste decided by pointing at a picture |
| Direction drifts per session | Approved comp + `DESIGN.md` pin it across sessions |

- Code-first is fine when: bug fix, structural task, precise design system already exists, extending an established surface.
- Image-first is the default when: new app/landing/redesign, "make it beautiful/premium/modern", anything briefed in visual words.
- Models systematically believe their HTML/CSS recreation of an image succeeded when it didn't → **measure, never self-certify.**

## 🔁 The pipeline
```
brief ─▶ 3 divergent directions (1 hero comp each) ─▶ human picks / delegates
   ─▶ per-section comps in the chosen world ─▶ detail close-ups where unclear
   ─▶ analyze → tokens + DESIGN.md ─▶ region spec (what's code vs raster)
   ─▶ produce real assets (plates) ─▶ build hero at comp size ─▶ diff ─▶ fix
   ─▶ remaining sections ─▶ motion ─▶ responsive ─▶ review loop
```
| Phase | Exit gate |
|---|---|
| Directions | Human approved one comp (or explicitly delegated — record why) |
| Section comps | One readable image per section; all read as one brand |
| Analysis | Every color hex'd, type measured, spacing cadence written down |
| Plates | Every raster region exists as a clean asset ≥1.5× its box |
| Hero | Build of first viewport diffed at comp's pixel size; no missing/contradicted regions |
| Finish | [design-review-loop.md](design-review-loop.md) verdict = ship |

- **No page code before the plates exist.** A page written first draws its imagery in CSS gradients and clip-paths — the quiet deletion of the approved design.
- Comp-led builds are frontier-model work (hold a measured layout across many attempts). Small/fast model → go code-led, put the ambition in a written first-viewport contract instead.

## 🎲 Diverge first: three directions, then choose
**Three is the number.** One comp invites rubber-stamping; three surfaces the composition worth building.

| Situation | What the 3 comps vary |
|---|---|
| New app, no identity | Whole world: palette strategy, type voice, material, composition |
| Established identity, new surface | **Structure only**: topology, sequence, density, hierarchy, focal point. Palette/type/components locked |
| One direction committed | The uncertainty an image can resolve: layout topology, density, hero scale |

- Derive directions from the **audience's world** (artifacts, places, rituals, publications, interfaces they read daily) — not a style catalog. List 7, drop near-duplicates, need ≥3 material families.
- Name the category's default page and its predictable opposite → both are the rut, excluded.
- Equal fidelity per option. A half-finished comp next to a polished one rigs the vote.
- Established identity → pass a **screenshot of an existing page as a reference image**. Prose paraphrase of a design system drifts; pixel references don't. State what carries over (chrome, palette, type, component character) and what must not (that page's content).
- Show all three together, ask: what carries forward, what feels false, approve / combine / revise / reject. **Then stop and wait.** No code before approval.
- Record the approval with the comp (e.g. a sidecar `comp-01.json` with the exact prompt + `"approved": true`) so it survives sessions.
- Unchosen comps stay in the repo as the spent hand — useful later as critique references ("what the image dared that the build didn't").

## 🧩 One image per section, never one tall board
| Rule | Why |
|---|---|
| **1 section = 1 image**, landscape (16:9 / 16:10 / 21:9 for hero) | Tiny text in a full-page board is unreadable → unextractable |
| Landing page, no count given → 6 sections; full site → 8 | Default high; under-generation is the common failure |
| Unclear detail → **generate a fresh close-up**, never crop/zoom the old image | Crops destroy spacing, type scale, margins |
| Complex section → primary image + 1–2 extraction renders (pricing cards, nav, testimonial) | Detail views exist to be measured |
| Label outputs `Section 3 of 8: Pricing` | Forces the full set; no stopping at the hero |
| Regenerate, don't settle | Text too small, nav fake, too crowded, website-in-a-phone → redo |

**Continuity across frames (lock before image 2):** same palette + accent logic, type family + scale, CTA family, radius language, image treatment (grade, framing, material), icon mood, copy voice.
**Variation allowed:** composition anchor, background mode, section size/density, where the one "second-read" moment sits.

Default section packs:
| Pack | Sections |
|---|---|
| 4 | hero · features · proof · CTA |
| 8 | hero · trust · features · product showcase · use cases · testimonials · pricing · CTA |
| 12 | + problem/solution · workflow · metrics/integrations · FAQ · footer |

## ✍️ Prompt anatomy — UI comps
**Lead with structure, not atmosphere.** Atmosphere-first prompts return a poster ("the fish market", not "the fish market's website"). Self-check: could it hang as a poster, or read as a photo with text on it? Not a comp → regenerate with the layout stated more literally. Inverse failure: world rendered, subject missing — point at the subject before accepting.

| Slot | Say |
|---|---|
| Surface + viewport | "Website section comp, 1440×900 desktop, landscape" / "iOS app screen, 390×844, portrait" |
| Visitor's job | One line: what they must understand/do. Readable back from the image with no caption |
| Regions in order + scale | "Top: slim nav, logo left, 4 links, one ghost button. Hero: headline in lower-left 40%, 2 lines…" — say "no navigation" if none |
| Focal moment | Exactly one dominant move; everything else holds still |
| Type | Character + weight + case + tracking + line count ("refined grotesk, medium weight, tight tracking, 2-line headline") |
| Palette | Named roles + hex: ground, surface, ink, muted, one accent |
| Imagery | Stance + treatment: "duotone product macro, palette-locked", "graded editorial photo, full-bleed with bottom scrim" |
| Copy | **Real product name + real short copy**, verbatim in quotes |
| Density/space | "Generous section padding, airy, 3/10 density" |
| Forbid | Invented claims (prices, customers, stats), fake brands, clichés, the category default — **not** the subject's own medium |

Always forbid (unless briefed): purple→blue AI gradients · glowing edges / orbs / blobs · glass stacked without reason · gradient headline text · cards-in-cards · pill/badge/micro-label spam · pseudo-system labels ("00 orchestration layer") · 3 identical KPI columns · unreadable logo tickers · "Acme / Nexus / NovaCore" · "elevate / seamless / unleash / next-gen" · scroll-down chevrons · 4+ line hero headlines.

### Web specifics
- **Composition anchor**: left-text/right-image is the most overused AI hero — allowed, never the reflex. Rotate: bottom-left over image · centered low · stacked center · off-grid editorial · image-as-canvas · inverted split · mini minimalist.
- **Hero scale**, pick one per site: Giant statement · Mid editorial · Mini minimalist (tiny logo, short line, thin CTA, mostly space).
- **Background mode** per section, vary it: solid + inline asset · paper/grid texture · full-bleed image + tonal overlay · side image 60/40 · duotone · radial vignette + product crop · micro-noise gradient · color-blocked diptych.
- Variety check across the set: same anchor ≤2 sections in a row; same background mode ≤3 in a row; non-minimal briefs get ≥1 full-bleed and ≥1 mini section. Minimal/swiss briefs: suspend — restraint is the design.
- Hero headline 1–3 lines, 5–10 words; one primary CTA; first viewport must still breathe on a small laptop (1280×720).
- Pick one **concept spine** (archive/dossier, precision instrument, journey, stage, living system, artifact) and exactly one **second-read moment** (oversized numeral, single material switch, side-rail note, asymmetric bleed).
- Motion is implied, not drawn: name 2 motion energies ("pinned narrative", "staggered float-up") so the builder knows → [motion-and-delight.md](motion-and-delight.md).

### Mobile specifics
- **Portrait at device size.** A phone screen comped landscape misstates the composition.
- Pick platform first: iOS-native · Android-native · cross-platform neutral. Don't mix idioms.
- Draw the system: status bar, safe areas, home indicator, tab bar (2–5), sheet docking zone. Nothing critical in unsafe regions.
- One screen per image; a flow = a believable sequence (onboarding → auth → home; browse → detail → cart). Ask: what action leads from screen N to N+1?
- Lock an **app design bible** before screen 2: device frame + scale, palette, type rhythm, radius, icon style, nav model, card/list behavior, shadow language.
- First screen: one focal point, 1–3 line statement, one CTA, no chips/stats. Not a website hero in a phone frame.
- Type never small: if it feels small, split the content into another screen.
- Device frame: consistent, evenly padded, never touching canvas edges, content stays the hero.
- Custom-feeling icons, not default line-icon library vibes.

## 📋 Paste-ready prompt templates
Web section comp:
```text
High-fidelity website section design comp, desktop 1440x900, landscape, flat UI render (not a photo of a screen).
Product: "<Real Name>" — <one-line what it does>. Visitor's job here: <understand X / start trial>.
Section <N> of <total>: <Hero>.
Layout, top to bottom: <slim nav: wordmark left, 3 text links, ghost button right>. <Headline "<exact 5-9 words>" in <refined grotesk, medium, tight tracking>, 2 lines, anchored lower-left over <full-bleed graded photo of <subject's real object>>, bottom scrim>. <Subline "<exact 12-18 words>">. <One primary button "<Start free>">.
Focal moment: <the photo crop>. Everything else quiet.
Palette: ground <#F4F1EA>, ink <#1C1B19>, muted <#6B665E>, accent <#C2410C> used only on the CTA.
Density 3/10, generous whitespace. Motion implied: <slow parallax drift>.
Same brand world as previous sections: <radius 6px, hairline dividers, duotone imagery>.
Do NOT include: invented stats, prices, customer logos, fake brand names, purple/blue gradients, glowing orbs, glassmorphism, gradient text, pill badges, cards inside cards, scroll chevrons.
All text crisp and legible.
```
Mobile screen:
```text
High-fidelity iOS app screen, portrait 390x844, shown in a clean minimal iPhone frame with even margins; the UI is the hero.
App: "<Real Name>" — <what it does>. Screen <2> of <5> in flow <onboarding → auth → home>: <Home>.
Regions: status bar; large title "<Today>"; <photo-led card strip, fixed 4:5 crops>; <list of 3 entries with real text>; tab bar with 4 labeled tabs, custom icons, consistent stroke.
Type: <soft humanist sans>, body >= 16pt equivalent, clear title/body/label contrast.
Palette: <warm neutral ground #EFE9E1, ink #211F1C, one accent #2F6B4F>. Subtle paper grain.
Respect safe areas and home indicator. No nested cards, no chip rows, no fake charts, no tiny labels.
```
Detail/extraction close-up:
```text
Same design system as the attached reference (palette, type, radius, image treatment unchanged).
Render ONLY the <pricing cards> at 2x scale, straight-on, large legible text, visible padding and borders,
primary vs secondary button states side by side. No new elements, no redesign.
```

## 🔬 Analyze the image → tokens + DESIGN.md
Treat the approved comp as a spec. Analysis is structured, not a vibe summary.

| Extract | How |
|---|---|
| Text | Copy every readable string verbatim — headline, sub, CTAs, nav, labels. Unreadable → close-up render |
| Palette | Sample pixels per region; name + hex + role; note grade/tint on imagery |
| Type | Measure cap height, x-height, weight, width class, tracking, line count, line-height; match faces by shape, not by guess |
| Spacing | Headline→sub, text→button, card gaps, section top/bottom, gutters; find the base unit and the cadence |
| Components | Button size/radius/fill vs outline, card structure, dividers, borders, shadow color + blur, input style |
| Layout | Grid columns, max-width, alignment axes, region boxes in % of frame |
| Motifs | Repeated moves that define the language (hairlines, crop ratios, numerals) |
| Unknowns | List them — then resolve, don't paper over |

- Write findings into `DESIGN.md` via the section skeleton in [design-md.md](design-md.md#-the-section-skeleton); tokens go to `tokens.css` in the same change.
- Only promote values that recur (3+ uses, same intent) to tokens; one-offs stay local.
- Ambiguity resolution order: visible design language → layout/spacing logic → component family → mood/polish → new close-up → fresh section render → **only then** the simplest faithful option. Never jump to generic defaults.
- Agent prompt:
```text
Open comps/approved/*.png. For each section: list every readable string verbatim; sample hex per region
(ground, surface, ink, muted, accent, borders); measure headline cap height, weight, tracking, line count;
measure spacing between headline/sub/CTA/section edges in px at comp scale; name the grid.
Output: tokens.css + DESIGN.md per docs/frontend-craft/design-md.md. Mark any value you estimated
rather than measured with (est). List unclear regions needing a close-up render. Do not write components yet.
```

## 🗺️ Region spec: what is code, what is raster
**The medium follows what the pixels are, not what feels buildable.**

| Region kind | Ships as |
|---|---|
| Text, controls, nav, chrome, dividers | Semantic HTML/CSS — never rasterized |
| Diagrams with countable parts, flat shape systems, anything that moves/scales/responds | Code (SVG ≤ icon size, or data-driven chart) |
| Photos, figures, product objects, illustrations with shading/perspective | **Raster plate** |
| Named textures (paper, cloth, grain, brushed metal) | Raster — clean patch, tiled |

- Every visible thing in the comp gets a region (callouts, notes, tables included). Unnamed ink can never be flagged missing.
- No region larger than ~¼ of the frame for text/controls — that's a column; split it.
- **Nothing on the page the comp doesn't show**: no extra borders, rules, kickers, nav items.
- Allowed concessions only: closest obtainable font · icon glyphs · real defects in the comp (typos).
- Store region boxes as % of the comp; emit them as CSS custom properties (`--r-hero-img-x/y/w/h`) and bind to your own semantic elements.

## 🎨 Produce real assets (plates)
**A crop of the comp is a reference, never a shipping pixel.** Shipped crops = blurry site with baked-in UI text.

| Rule | Detail |
|---|---|
| Regenerate each raster region from its crop | Same subject, composition, palette, lighting, material — UI text and page chrome removed |
| Resolution | ≥1.5× the region's rendered box (then optimize → [assets-optimization.md](assets-optimization.md)) |
| Background | Isolated figure/object → transparent PNG (native alpha, check on light + dark, no halos, holes clear). Photo/texture → opaque |
| Don't redesign | No new objects, no restyle. CSS draws radius, shadow, card transform — strip those from the raster |
| Two misses on a region | Keep the better one, flag for human review, name the drift |
| Provenance | Embed the exact generation prompt in the file metadata or a sidecar; sourced/stock images record their origin |
| Parallelize | One asset-producer subagent per batch of regions, working from the spec — never from its own re-inventory |

### Generated vs real imagery
| Use generated | Use real (photo/screenshot) |
|---|---|
| Illustration, texture, abstract material, atmosphere | Your actual product UI (screenshot the real app) |
| Demonstration content in greenfield work — **labeled synthetic** | Team, customers, testimonials, locations — anything presented as real |
| Hero objects when no shoot exists yet (list for replacement) | The subject's physical object when a decisive real photo exists — one great photo beats five mediocre |
| Icons/marks in a locked style → [brand-identity.md](brand-identity.md) | Logos of partners/customers (never invent) |

- Truth binds **claims**, not demonstrations: prices, customers, benchmarks, capabilities stay uninventable; illustrative data may be authored at full fidelity if labeled.
- Verify stock URLs resolve; never ship hotlinks that break.
- Hand the user a "replace with real material" list at finish.

## 🏗️ Image → code translation rules
- **Build the hero first, at the comp's exact dimensions.** Plates placed at their boxes before any text. Then the semantic layer.
- **Copy the comp's words verbatim** in the hero pass. Rewording is a separate, stated decision afterwards → [ux-copy.md](ux-copy.md).
- Size type from **measured** cap height, not "looks like 48px". Set in the matched face.
- Match spacing logic, not approximate density. Don't compress generous comp spacing into default tight spacing.
- **Anti-drift**: no swapping distinctive sections for generic rows, no flattening type hierarchy, no re-adding boxes the comp removed, no redrawing topology while keeping the palette (that's a second art direction).
- Comp is a north star for **semantic, responsive, accessible** code. Never rasterize UI text or controls. States, focus, a11y, i18n remain your job.
- After hero passes: sections inherit the same corner language, line weights, palette. Uncovered regions inherit the system.
- Then responsive: first viewport must hold at 1280–1600, not only at comp width. Fluid columns, no fixed-pixel grid → [adaptive-ui.md](adaptive-ui.md).

## 📏 Visual comparison loop
Numbers, not impressions. Capture rules (settle animation, fonts ready, full-page from top, open every file) → [design-review-loop.md](design-review-loop.md#-capture-validity-an-invalid-capture-proves-nothing).

```bash
# capture build at comp size, then diff (any headless browser + ImageMagick)
npx playwright screenshot --viewport-size=1440,900 http://localhost:5173 build-hero.png
magick compare -metric RMSE comps/approved/hero.png build-hero.png diff-hero.png
magick comps/approved/hero.png build-hero.png +append side-by-side.png
```
| Per region, classify | Fix |
|---|---|
| **match** | — |
| **drift** (size/position off) | Edit numbers: "cap height 78px build vs 103px comp" *is* the edit |
| **missing** | Produce the plate / add the element |
| **contradicted** | Re-derive structure from the spec box |
| **added** (ink the comp doesn't have) | Delete it |

- Whole-frame pixel score is a sanity check (ballpark: ~70%+ overall with zero missing/contradicted regions), **per-region crops are the verdict**. Never judge fidelity from one full-page thumbnail.
- Open the listed region crops *before* editing. Repeated attempts on the same blocker without new information = stop, re-derive.
- Max 2 inspection rounds in the build thread; then a **fresh-context reviewer** gets comp + captures + spec and issues ship / fix / rebuild / recapture.

## 🎛️ Live in-browser variants
For tuning an existing element (not a new world): generate N variants in the running page, cycle them in place, accept one, bake it into source.

| Rule | Detail |
|---|---|
| Identity lock first | One sentence of what's on screen: real hex, loaded fonts, topology, surface treatment, voice. Every variant must pass side-by-side as the same brand |
| Default vs departure | Default (~90%) varies *within* identity. Departure (new fonts/hues/family) only on explicit "redesign / something completely different" |
| 3 variants, 3 different axes | Hierarchy · layout topology · type system (existing faces) · color strategy (existing tokens) · density · structural decomposition. Three "tighter" variants = failure |
| Direction words map to axes | bolder (scale/saturation/structure) · quieter (color/ornament/spacing) · distill (noise/redundant content/nesting) · colorize (hue families) · layout (arrangement, not spacing tweaks) |
| Whole-element replacement | Each variant is a complete element (HTML + scoped CSS), not a CSS patch; first visible, rest hidden |
| Knobs, not regenerations | 0 params for leaf elements, 0–1 small card, ~2 section, 2–4 hero. Knob = CSS var or data attribute ("a bit tighter" without a regen) |
| Accept = carbonize | Move rules into the owning stylesheet with real selectors, bake knob values, strip wrappers/markers, delete preview CSS |
| Never touch prod | Local dev server only; never weaken CSP to make it work |

Minimal pattern without any tool: render variants as sibling components behind a `?variant=1..3` query param, screenshot all three in one batch, pick, delete the losers in the same commit.

## 🚫 Pitfalls
| Pitfall | Fix |
|---|---|
| Garbled/unreadable generated text | Quote exact short strings; fewer words per image; close-up render; code sets the real text anyway |
| Impossible layouts (overlaps no grid can make, text over busy image) | Reject at comp review; ask "which grid produces this?"; re-prompt with explicit columns + scrim |
| Over-polished mockup (perfect photo lighting, fake depth, glass everywhere) | Judge it as the shipped screen; if the build can't reach it without CSS theatrics, simplify the comp |
| Poster, not interface | Lead prompt with regions; name visitor's job |
| Subject deleted by exclusion list | Exclusions bind claims, not the medium the subject lives in |
| One tall full-page board | One image per section; regenerate |
| Cropped old image as "detail" | Fresh close-up render |
| Frames drift into different brands | Lock palette/type/radius/treatment before image 2; pass image 1 as reference |
| Plates shipped as comp crops | Regenerate at ≥1.5×, text removed |
| Painted material rebuilt in CSS (gradients, many-vertex `clip-path`) | Reclassify as raster plate |
| Code "improves" the comp into a template | Anti-drift rules; diff per region |
| Invented stats/logos/testimonials in comp copied to prod | Forbid in prompt; replace list at finish |
| Landscape comp for a mobile-first screen | Portrait at device viewport |
| Self-certified fidelity | Diff + fresh reviewer; builder never grades its own match |

## ✅ Checklist
- [ ] 3 divergent directions shown; one approved (recorded) or delegated (reason stated)
- [ ] One readable image per section; close-ups for anything unclear; frames read as one brand
- [ ] Every string, hex, type size, spacing value extracted; estimates marked
- [ ] `DESIGN.md` + `tokens.css` written from the analysis
- [ ] Every region classified code vs raster; plates produced ≥1.5×, with provenance
- [ ] Hero built at comp size, diffed per region, no missing/contradicted
- [ ] Responsive at 390 / 1280–1600; review-loop verdict = ship
- [ ] Synthetic content labeled; replace-with-real list handed over

As of 2026-09: current image models render short UI text reliably but still garble dense paragraphs and small labels — budget for close-up re-renders and never trust comp body copy.

Sources: [taste-skill](https://github.com/Leonxlnx/taste-skill) (`imagegen-frontend-web`, `imagegen-frontend-mobile`, `image-to-code`, `stitch-design-taste`) · [impeccable](https://github.com/pbakaus/impeccable) (`visualize`, `new-work`, `generate`, `live`, `extract`, asset-producer agent)
