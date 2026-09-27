# 🎭 Design Directions — a catalog to rotate from

**28 ready-to-use visual directions. Each app picks one. Sibling apps pick different ones.** A direction is a recipe (palette strategy, type posture, shape, depth, motion, one signature move), not a skin. Pick one, re-derive the tokens from the product, and record it in the app's [`DESIGN.md`](design-md.md).

| Need | Go to |
|---|---|
| Quality floor, dials, register, banned defaults | [design-taste.md](design-taste.md) (this file feeds its step 4, "Pick a direction") |
| Write the pick into the project | [design-md.md → Pick this app's direction](design-md.md#-pick-this-apps-direction-kickoff-10-min) |
| Motion per direction, in depth | [motion-and-delight.md](motion-and-delight.md) |
| Voice that matches the look | [ux-copy.md](ux-copy.md) |
| Logo, name, brand assets | [brand-identity.md](brand-identity.md) |
| Generate reference images for the pick | [reference-image-design.md](reference-image-design.md) |
| Density, breakpoints, input modes | [adaptive-ui.md](adaptive-ui.md) |

## 🧭 How to use
1. **Map the product** to 2–3 candidates (table below). Read the brief first: a pinned font/palette/era wins over this file.
2. **Check siblings.** Read the last 2–3 apps' `DESIGN.md`. Drop candidates that fail the [sibling axes](#-make-siblings-differ).
3. **Pick with a seed**, not by taste. Models always take option #1 ([design-taste → Derive](design-taste.md#derive-dont-default)).
4. **Re-derive, don't paste.** Example hex values are starting points. Take the hue from the product's world, keep the *structure* (lightness steps, chroma level, roles). Author final tokens in OKLCH.
5. **Record** direction number, what you changed, and which axes differ from siblings (block at the bottom).
6. Build → clear the [floor](design-taste.md#-pre-ship-taste-check-60-seconds) → [review loop](design-review-loop.md).

**Fonts:** every named face is free (SIL OFL, mostly on Google Fonts) as of 2026-09. Verify the license before shipping. "for X" = free stand-in for a proprietary brand face. Display faces are unique per direction on purpose. Body faces may repeat.

## 🗺️ Product → candidates
| Product / audience | Candidates |
|---|---|
| Payments, banking, B2B finance | 1 · 10 · 24 |
| Enterprise, infra, procurement-heavy B2B | 2 · 17 · 7 |
| Dev tool, CLI, API, SDK | 17 · 18 · 4 · 9 · 21 |
| AI assistant, productivity, writing | 3 · 14 · 23 |
| Collaboration, whiteboard, creative tool | 6 · 8 · 22 |
| Publication, newsletter, long reads | 5 · 23 · 22 |
| Marketplace, travel, food, UGC | 13 · 11 · 12 |
| DTC commerce, sports, fashion | 16 · 24 · 27 |
| Consumer health, wellness, lifestyle | 14 · 12 · 10 |
| Single flagship product, hardware | 25 · 26 · 14 |
| Monitoring, ops, security, logistics | 19 · 17 · 20 |
| Analytics, databases, trading | 20 · 1 · 19 |
| Multi-product suite, course catalog | 7 · 6 |
| Portfolio, studio, manifesto | 15 · 26 · 23 |
| Kids, community, events | 8 · 22 · 6 |
| Luxury, automotive, aerospace | 26 · 27 · 25 |
| Anniversary, nostalgia, games | 28 · 8 |
| Civic, regulated, a11y-first | 2 · 5 (plus the civic archetype in [design-taste](design-taste.md#archetypes--rotation)) |

Entry format: `canvas` · mood words · seen in (public sites analyzed, as of 2026-09; analysis only, never a target).

## ☀️ Light & restrained

### 1 · Precision fintech
`light` · exact, calm, trustworthy · seen in: Stripe, Coinbase
- **Palette** (restrained): bg `#f7f8fb` · surface `#fdfdfe` · ink navy `#0e2239` · muted `#52617a` · accent cobalt `#2451e6` `oklch(0.51 0.23 265)`. One filled CTA per band.
- **Type**: Hanken Grotesk 300 display (for Söhne) + Hanken 400 body. `tnum` on every amount. Display `-0.03em`.
- **Shape · depth**: pill buttons, 8px cards. Stacked navy-tinted shadows: `0 1px 3px rgb(14 34 57/.08), 0 8px 24px rgb(14 34 57/.06)`.
- **Motion**: calm 200ms ease-out. Numbers animate only when the data is real.
- **Signature**: atmospheric gradient band across the top third (warm hues, never purple→blue). Featured tier = navy polarity flip.
- **Fit**: ✅ payments, API infra, B2B finance · ❌ kids, lifestyle, retail

### 2 · Engineered corporate
`light` · rigorous, sober, precise · seen in: IBM, NVIDIA, BMW
- **Palette** (restrained): bg `#f7f8f9` · surface-lift `#eef0f2` · ink `#16181b` · muted `#5a616b` · accent machine green `#2e7d12` `oklch(0.52 0.16 139)`. Accent at most twice per viewport.
- **Type**: Red Hat Display 300 at 56–76px vs Red Hat Text 400 body with `+0.01em`. Light display is the voice. Display `-0.01em`.
- **Shape · depth**: 0–2px radius everywhere. Inputs square with a bottom rule. No shadows: 1px hairlines + surface change.
- **Motion**: productive 110–240ms, no entrances.
- **Signature**: 12px accent square pinned to one card corner.
- **Fit**: ✅ enterprise, infra, B2B procurement, regulated · ❌ consumer delight, kids

### 3 · Warm workspace
`light` · calm, capable, human · seen in: Cursor, Zapier, Intercom, Lovable
- **Palette** (restrained): bg warm paper `#f4f4ef` · surface `#fdfdfb` · ink `#26251f` · muted `#6b685e` · accent vermilion `#c2410c` `oklch(0.55 0.17 38)`. Every neutral carries the same warm hue.
- **Type**: Onest 450 display (for Camera/Saans) + Onest 400 body, JetBrains Mono for code. Display 400–500, never bold. `-0.025em`.
- **Shape · depth**: 8px buttons, 12px cards. Hairline-only: white card on paper, no shadow.
- **Motion**: quiet 150–200ms. Life lives *inside* product mockups, not the chrome.
- **Signature**: real product screenshots as the hero. Washed pastel status pills (bg/text pairs) for in-product states only.
- **Fit**: ✅ dev tools, productivity, AI assistants, docs · ❌ gaming, luxury

### 4 · Readme-native
`light` · honest, minimal, nerdy · seen in: Ollama, OpenCode
- **Palette** (mono): one continuous sheet `#fbfbfa` · snippet surface `#f2f1ef` · ink `#1f1d1c` · muted `#6e6a67` · accent = ink. Links may take one blue.
- **Type**: variant A = Commit Mono for everything (for Berkeley Mono). Variant B = Nunito 600 display + `system-ui` body. Tracking 0.
- **Shape · depth**: A = 4px; B = full pill. Zero shadows. One inverted dark block for the single product demo.
- **Motion**: none beyond copy-to-clipboard confirmation and a caret.
- **Signature**: the install command *is* the hero (copyable pill under the headline). `[+]` / `[-]` ASCII bullets. One line-drawn mascot.
- **Fit**: ✅ CLIs, OSS libraries, local-first tools · ❌ non-technical audiences. Mono-everywhere is valid only here.

### 5 · Print magazine
`light` · literate, authoritative, timeless · seen in: Wired
- **Palette** (duet): paper `#fbfbf9` · ink `#121212` · muted `#5c5c5c` · rule `#dcdcd8` · accent = inline link blue `#0a6aa8` only.
- **Type**: Bodoni Moda display (tall, high contrast) + Source Serif 4 body at 18–20px; meta in Source Serif small caps. Justified only for a real publication identity. Display `-0.02em`.
- **Shape · depth**: 0px everywhere, square buttons. Flat. 1px rules between story rows; one 2px ink border for the subscribe moment.
- **Motion**: none. Image fade-in only.
- **Signature**: masthead wordmark band. Lead story → 2-up → bylined rows.
- **Fit**: ✅ publications, newsletters, archives · ❌ dashboards, SaaS. Skip italic display + tracked mono labels (the AI broadsheet tell).

## 🎨 Light & expressive

### 6 · Pastel block studio
`light` · playful, generous, confident · seen in: Figma, Miro, Notion
- **Palette** (monochrome chrome + blocks): bg `#fbfbfb` · ink `#141414` · muted `#5f5f5f` · blocks lime `#dff38a` · lilac `#d9ccff` · mint `#bfeedd` · coral `#ff9a7a` · one deep navy `#1d2350`. CTA = ink pill.
- **Type**: Mona Sans variable at fine weights (330/450/540) for everything. One family that flexes. Display `-0.02em`.
- **Shape · depth**: pill buttons, 24px blocks. Blocks replace shadows. A rare real shadow reads as an event.
- **Motion**: sticky-note collage drifts in once, slightly off-axis. Cards snap, never float.
- **Signature**: full-width pastel blocks, each owning one chapter of the story.
- **Fit**: ✅ collaboration, creative tools, education · ❌ finance ops, civic

### 7 · Category spectrum
`dark or light` · systematic, navigable, bold · seen in: HashiCorp, Webflow, MiniMax
- **Palette** (full palette as code): chrome dark `#0d0e10` / ink `#eef0f2` / muted `#9aa0a8`, or light `#f6f6f4` / `#17181a` / `#5b5e63`. Each product or category owns one hue as a full card fill: violet `#7b42f6` · amber `#ffcf25` · cyan `#14c6cb` · coral `#f25c54` · green `#3fbf6f`. Buttons stay ink.
- **Type**: Familjen Grotesk 600 display / 400 body. Eyebrow-style category labels at most once per section. Display `-0.02em`.
- **Shape · depth**: 8px buttons, 12px cards. Surface ladder for depth. Hue cards sit at the same z-plane: color = meaning, not elevation.
- **Motion**: hover tints within the card's own hue.
- **Signature**: color is navigation. A user can tell which product a section covers from the corner of their eye.
- **Fit**: ✅ multi-product suites, modules, course catalogs · ❌ single-product apps (nothing to encode)

### 8 · Claymation toybox
`light` · warm, tactile, cheeky · seen in: Clay
- **Palette** (full palette): bg peach-white `#fff8f1` · ink navy `#151a2d` · muted `#5b6072` · card fills pink `#ff6fa8` · teal `#0f766e` · lavender `#c9b8ff` · peach `#ffc59e` · ochre `#e0a526`. CTA = ink.
- **Type**: Gabarito 600 display at 64–72px (for Plain) + Nunito Sans body. Rounded letterforms carry warmth, never above 600. `-0.03em`.
- **Shape · depth**: 12px buttons, 16px cards, 24px feature cards. Color contrast instead of shadow.
- **Motion**: springy. The one family where overshoot is on-brand. Mascot idles only while in view.
- **Signature**: 3D clay-render illustrations and characters at section joins. Real product UI inside colored cards.
- **Fit**: ✅ B2B tools with personality, onboarding-heavy consumer apps, community · ❌ medical, legal

### 9 · Sketchbook engineering
`light` · witty, candid, technical · seen in: PostHog
- **Palette** (restrained): bg olive-cream `#eeefe9` · surface `#fdfdf8` · ink `#23251d` · muted `#5f6156` · accent marigold `#f7a501` `oklch(0.78 0.17 73)` as a pill **fill with ink text** (fails as text on bg).
- **Type**: Rubik 400–800, one family. Hierarchy from weight steps, not color. Display `-0.02em`.
- **Shape · depth**: 6px cards, 8px containers, pill chips. Flat hairlines. Dark code island inside a white card.
- **Motion**: static mascots, micro-feedback only.
- **Signature**: hand-drawn mascot marginalia. Tinted full-panel callouts for tip/warn/info (no thick left stripe).
- **Fit**: ✅ dev analytics, OSS with humor, internal tools · ❌ luxury, enterprise procurement

### 10 · Chunky friendly money
`light` · upbeat, plain-spoken, safe · seen in: Wise
- **Palette** (committed): bg sage `#e9ece6` · surface `#fbfcfa` · ink `#0f110c` · muted `#55594f` · accent lime `#9fe870` `oklch(0.86 0.17 135)` as a **fill with ink text** only.
- **Type**: Bricolage Grotesque 800 display at 64–120px + Be Vietnam Pro 400/600 body. Heavy display vs plain body is the story. `-0.03em`.
- **Shape · depth**: 24px on cards *and* buttons. White card on sage is the elevation. No shadows.
- **Motion**: live figures roll on input change; 200ms press spring.
- **Signature**: an interactive calculator/converter card as the hero.
- **Fit**: ✅ consumer fintech, budgeting, pricing tools · ❌ dense analytics

### 11 · Flagship retail
`light` · welcoming, familiar, crafted · seen in: Starbucks
- **Palette** (one hue, four roles): bg warm neutral `#f2f0eb` · surface `#fdfcfa` · ink `#1e2a26` · muted `#5f625e` · hue ladder deep `#1e3932` / brand `#006241` / action `#00754a` / tint `#d4e9e2` · gold `#cba258` for loyalty ceremony only.
- **Type**: Manrope 700 display (`-0.01em`) + Manrope body. Young Serif only for the loyalty/ceremony moment: a second family with one job.
- **Shape · depth**: 50px pill buttons, 12px cards. Whisper dual shadow `0 0 .5px rgb(0 0 0/.14), 0 1px 1px rgb(0 0 0/.24)`.
- **Motion**: press `scale(0.95)`. Floating action lifts on scroll stop.
- **Signature**: one floating circular order/primary action, bottom-right. Dark hue bands bookend the page.
- **Fit**: ✅ retail, food, loyalty, hospitality · ❌ dev tools, B2B dashboards

### 12 · Orbit institutional
`light` · warm, assured, connected · seen in: Mastercard
- **Palette** (restrained): bg putty `#f3f0ee` · surface `#fbfaf9` · ink `#141413` · muted `#62605c` · accent magenta `#a8326e` `oklch(0.51 0.16 353)` for 1px arcs, dots, and focus. CTA = ink.
- **Type**: Albert Sans 500 display (for Mark) + Albert Sans 450 body. The soft in-between weight is the point. `-0.02em`.
- **Shape · depth**: extreme radius: 40px heroes, pill cards, circular image masks. Cushion shadow `0 24px 48px rgb(20 20 19/.08)`.
- **Motion**: circle-mask reveals. One slow arc drift that stops offscreen.
- **Signature**: circular portraits linked by thin orbit arcs, each with a docked "satellite" arrow button. Ghost watermark headline behind.
- **Fit**: ✅ networks, foundations, service catalogs, institutions going warm · ❌ dense tools

### 13 · Photo-first marketplace
`light` · generous, inviting, browsable · seen in: Airbnb, Pinterest
- **Palette** (restrained): bg `#fcfcfc` · photo backdrop `#f4f4f2` · ink `#222222` · muted `#6a6a6a` · accent coral-red `#d42f4c` `oklch(0.57 0.20 17)` on the search orb, save state, one CTA.
- **Type**: Figtree 600 at modest sizes (22–32px display, for Circular/Cereal) + Figtree 400. The photos carry the weight. `-0.01em`.
- **Shape · depth**: 14–16px cards, pill search. One shadow tier on hover: `0 0 0 1px rgb(0 0 0/.02), 0 2px 6px rgb(0 0 0/.04), 0 4px 8px rgb(0 0 0/.1)`.
- **Motion**: swipe carousels inside cards. The save "pop" is the one bounce.
- **Signature**: pill search bar split by hairlines, ending in an accent orb. A grid that preserves each image's aspect.
- **Fit**: ✅ marketplaces, travel, recipes, real estate, UGC · ❌ products with no imagery

### 14 · Soft structural premium
`light` · serene, tactile, expensive · seen in: ElevenLabs, taste-skill "soft structuralism"
- **Palette** (restrained): bg silver `#f3f4f5` · surface `#fbfbfc` · ink `#17191c` · muted `#5f646b` · accent deep teal `#1f6f6b` `oklch(0.49 0.08 190)`. Optional pastel atmosphere orbs on a fixed layer.
- **Type**: Archivo at width 115–125, weight 500, for display + Archivo 400 normal width for body. One family, the width axis does the contrast. `-0.03em`.
- **Shape · depth**: 24–32px squircles, concentric (inner radius = outer − padding). Diffuse tinted shadows `0 1px 2px oklch(0.3 0.02 250/.06), 0 20px 40px oklch(0.3 0.02 250/.08)` + inner top highlight.
- **Motion**: slow springs 500–700ms, `cubic-bezier(0.32,0.72,0,1)`. Floating island nav. Blur only on fixed/sticky layers.
- **Signature**: double-bezel card (tinted outer shell, inset inner core). Trailing icon nested in its own circle inside the button.
- **Fit**: ✅ consumer health, wellness, hardware, portfolios · ❌ dense data, civic

### 15 · Swiss industrial print
`light` · blunt, structural, graphic · seen in: taste-skill brutalist print mode
- **Palette** (mono + hazard): paper `#f4f4f0` · surface `#eae8e3` · ink `#111111` · muted `#5a5a55` · accent hazard red `#d71818` `oklch(0.56 0.22 28)`, the only color: thick rules, strike-throughs, vital data.
- **Type**: Schibsted Grotesk 900 uppercase at `clamp(4rem,10vw,12rem)`, leading 0.88, tracking `-0.04em` (the floor). Fragment Mono uppercase 11–13px `+0.08em` for meta.
- **Shape · depth**: 0 radius. Visible grid: `display:grid; gap:1px` on an ink background. No shadows, no gradients.
- **Motion**: hard cuts, instant state.
- **Signature**: oversized numerals or letters bleeding off the viewport edge.
- **Fit**: ✅ portfolios, architecture, manifestos, data editorial · ❌ kids, health, onboarding flows

### 16 · Athletic campaign
`light` · kinetic, loud, physical · seen in: Nike, Vodafone
- **Palette** (neutral chrome): bg `#fcfcfc` · product swatch `#f5f5f5` · ink `#111111` · muted `#707072` · no chrome accent. Sale/semantic red `#d30005` is the only hue.
- **Type**: Anton uppercase display, leading 0.9, burned into photography (for Futura condensed) + Jost 400/500 body.
- **Shape · depth**: pill on every control; 0 radius on product cards. No shadows. Photography is the only depth.
- **Motion**: hover a color swatch → the product image swaps. Campaign video in the hero.
- **Signature**: giant compressed uppercase headline over full-bleed campaign imagery. Product photo *is* the card, on a flat gray swatch.
- **Fit**: ✅ DTC commerce, sports, events, fashion · ❌ SaaS, docs

## 🌑 Dark & polarity

### 17 · Midnight dev-tool
`dark` · precise, fast, nocturnal · seen in: Linear, Raycast, Composio
- **Palette** (restrained): bg `#08090a` · ladder `#0f1011 → #141516 → #191a1c → #1f2023` · ink `#f2f3f4` · muted `#8a8f98` · hairline `#23252a` · accent `#5560d4` `oklch(0.54 0.18 275)` on primary + focus only. Rotate the hue per app.
- **Type**: Geist 500 display at 64–80px + Geist 400 body + Geist Mono for code. `-0.03em`.
- **Shape · depth**: 8px buttons, 12px cards. Surface ladder + hairline + a 1px top-edge highlight `inset 0 1px 0 rgb(255 255 255/.04)`. No drop shadows, no glow.
- **Motion**: 120–200ms. The command palette opens from `scale(.98)`. Keyboard first.
- **Signature**: the real product screenshot as the hero. Keycap glyphs next to actions.
- **Fit**: ✅ dev tools, issue trackers, infra · ❌ lifestyle, older audiences

### 18 · Ember terminal
`dark` · warm, focused, unhurried · seen in: Warp
- **Palette** (no chroma): bg warm brown-black `#2b2622` · surface `#342e29` · ink `#f7f5f0` · muted `#aaa197` · hairline `#4a423b` · primary button = the ink color. No accent.
- **Type**: Host Grotesk 400 display at 64px + Host Grotesk body + Red Hat Mono for code. Quiet hero. `-0.025em`.
- **Shape · depth**: 3–4px radius. Surface contrast + hairline only.
- **Motion**: caret blink and typed output in the hero terminal only.
- **Signature**: warm-brown dark instead of blue-black. Every neutral shares that warmth.
- **Fit**: ✅ terminals, writing tools, focus modes, night use · ❌ bright consumer

### 19 · Tactical telemetry
`dark` · dense, mechanical, alert · seen in: taste-skill telemetry mode
- **Palette** (mono + signal): bg `#0c0d0c` · surface `#141514` · ink phosphor `#e6e6e3` · muted `#8b8d88` · accent hazard red `#ff2a2a` `oklch(0.64 0.24 28)`. Optional single green readout `#4af626`, one element only.
- **Type**: Barlow Condensed 700 uppercase headers + Azeret Mono 11–14px uppercase `+0.06em` for all data. `tnum` everywhere.
- **Shape · depth**: 0 radius. Framed 1px compartments. An optional scanline sits on a fixed, `pointer-events:none` layer.
- **Motion**: instant. Value ticks on update. Caret blink at most.
- **Signature**: `[ BRACKETED ]` labels and crosshairs at grid intersections.
- **Fit**: ✅ monitoring, ops, security, logistics control · ❌ non-technical marketing, kids

### 20 · Voltage signal
`dark` · energetic, bold, data-proud · seen in: ClickHouse, Binance
- **Palette** (committed): bg `#0b0c0e` · surface `#1a1b1e` · ink `#f4f4f2` · muted `#9a9ca3` · accent electric yellow `#f2f25a` `oklch(0.93 0.17 109)` as fill with ink text, and for huge numerals. Up/down green/red as text color only.
- **Type**: Epilogue 800 display + Epilogue 400 body; Martian Mono for stats and code. `-0.03em`.
- **Shape · depth**: 6–8px radius. Surface steps. A full-bleed accent band stands in for elevation.
- **Motion**: snappy. Counters only from live data.
- **Signature**: giant accent-colored numerals. One full-bleed accent CTA band per page.
- **Fit**: ✅ databases, trading, analytics, performance · ❌ calm wellness, heritage

### 21 · Redline console
`dark + light` · irreverent, sharp, dev-savvy · seen in: Sentry
- **Palette** (committed): bg violet midnight `#1b1330` · surface `#251b40` · ink `#f6f3ff` · muted `#a9a0c4` · accent lime `#c2ef4e` `oklch(0.89 0.19 124)`. Light polarity for pricing and dense reference pages.
- **Type**: Big Shoulders Display 800 (chunky, near-condensed) + Karla body. Buttons uppercase `+0.02em`.
- **Shape · depth**: 6px buttons. A halo in the canvas color vignettes the CTA. Light pages use soft shadows.
- **Motion**: stickers float in at section joins, then stay still.
- **Signature**: a lime highlight chip wrapping one keyword per headline, like syntax highlighting. Sticker mascots.
- **Fit**: ✅ observability, dev tools with attitude, gaming-adjacent communities · ❌ finance, health

### 22 · Tabloid rave
`dark` · loud, current, gleeful · seen in: The Verge
- **Palette** (full palette): bg `#131313` · surface `#2d2d2d` · ink `#f5f5f5` · muted `#a3a3a3` · hazard-tape accents acid mint `#3cffd0` + ultraviolet `#5200ff` · story tiles yellow/pink/orange at full saturation.
- **Type**: Unbounded 800 display up to ~6rem + Work Sans body. Timestamps in body face, uppercase, `tnum`.
- **Shape · depth**: 20–40px tiles. Color is elevation. 1px hazard outlines. Active tab = `inset 0 -1px 0` underline.
- **Motion**: tiles invert on hover. Live timestamps tick.
- **Signature**: a vertical timeline of saturated rounded tiles on a dashed rule, timestamps on the rail.
- **Fit**: ✅ media, music, culture, events, feeds · ❌ enterprise, regulated

### 23 · Noir editorial
`dark` · literary, confident, premium · seen in: Resend
- **Palette** (no accent): bg `#0a0a0b` · surface `#111114` · deep well `#07070a` · ink `#f5f5f7` · muted `#9b9ba3` · hairlines white at 6% / 14%. One low-opacity radial wash per section, hue rotating, never two per section.
- **Type**: Gloock 400 display at 72–96px (for Domaine) + Hanken Grotesk body. Serif only if the brand has a real literary voice. `-0.02em`.
- **Shape · depth**: 8px buttons, 12px cards. Translucent hairlines, no shadows. A single light card inset on black is the top elevation.
- **Motion**: the wash fades in with its section. The code window types once.
- **Signature**: the serif headline is the brightest thing on screen. CTA = small off-white rectangle.
- **Fit**: ✅ premium APIs, email/writing tools, studios · ❌ kids, dense ops. No neon edge glow.

### 24 · Polarity chapters
`alternating` · cinematic, bold, mainstream · seen in: PlayStation, Revolut, Uber
- **Palette** (bands): ink band `#0c0c0d` · paper band `#fafaf9` · brand band crimson `#b3122f` `oklch(0.49 0.19 20)` (swap per app) · muted on paper `#5e5e62` · muted on ink `#a0a0a6`.
- **Type**: Urbanist 300 display (light weight at big sizes) + Urbanist 400 body at 18px. `-0.02em`.
- **Shape · depth**: pill CTAs, 8px cards. The band change *is* the divider. Cards lift only on press.
- **Motion**: one reveal per band. Key art may drift a little.
- **Signature**: the page as chapters of full-bleed dark/light/brand bands, one editorial moment each. Featured tier = polarity flip.
- **Fit**: ✅ consumer platforms, entertainment, mobility, telecom · ❌ docs, dashboards (one theme per surface there)

### 25 · Product museum
`light ↔ dark tiles` · reverent, quiet, premium · seen in: Apple, Tesla
- **Palette** (restrained): bg `#fbfbfd` · parchment `#f5f5f7` · dark tile `#0d0d0f` · ink `#1d1d1f` · muted `#6e6e73` · accent `#0066cc` `oklch(0.52 0.18 256)` on every interactive element. Nothing else has hue.
- **Type**: Sofia Sans 600 display (for SF Pro) + Sofia Sans 400 body. `-0.02em`.
- **Shape · depth**: tiny pill CTAs, 18px utility cards. Exactly one shadow, on product renders: `3px 5px 30px rgb(0 0 0/.22)`. Frosted sticky sub-nav.
- **Motion**: product reveal tied to scroll (only if it's smooth). 300ms nav.
- **Signature**: viewport-tall alternating tiles, each a centered headline + one-line tagline + two small links + one render. Centered is right here (launch register).
- **Fit**: ✅ hardware, single flagship products · ❌ anything without great renders

### 26 · Cinematic void
`dark` · austere, awed, rare · seen in: Bugatti, SpaceX, Runway
- **Palette** (none): bg `#070707` · surface `#141414` · ink `#f2f2f2` · muted `#8c8c8c` · at most a pale ice link `#c3d9f3`. Photography supplies all color.
- **Type**: Lexend Giga uppercase 400, `+0.08em` (breaks the negative-tracking default on purpose; only for short uppercase lines) + Lexend 300 body. Variant "film reel": one tight grotesk in sentence case, `-0.03em`.
- **Shape · depth**: 1px ghost-outline pills, never filled. Zero shadows. Grade the image darker instead of adding a scrim.
- **Motion**: slow 800–1200ms crossfades between media. Nothing else moves.
- **Signature**: full-viewport media + one tracked line + one ghost pill. 120px section rhythm.
- **Fit**: ✅ luxury, aerospace, film/creative AI, architecture · ❌ forms, density, operate surfaces

### 27 · Motorsport compressed
`dark` · aggressive, engineered, fast · seen in: Lamborghini, Ferrari, BMW M
- **Palette** (committed): bg `#0b0b0b` · ladder `#181818 → #202020` · ink `#f5f5f5` · muted `#9a9a9a` · one racing accent, e.g. gold `#ffc000` `oklch(0.84 0.17 85)` filled with ink text. Or a racing red, or a brand tricolor used only as a stripe.
- **Type**: Saira Extra Condensed 800 uppercase display, leading 0.92 + Saira 300–400 body. Button labels uppercase `+0.08em`.
- **Shape · depth**: 0 radius. Darkness ladder instead of shadows. Video hero.
- **Motion**: hard `clip-path` wipes, not fades. Fast.
- **Signature**: a 4px brand-color stripe divider. One geometric motif taken from the product (hexagon, angle, chevron).
- **Fit**: ✅ automotive, sports teams, gaming hardware, performance gear · ❌ wellness, text-heavy

## 📼 Period

### 28 · Retro-web revival
`light` · nostalgic, cheeky, collectible · seen in: Dell 1996, Nintendo 2001 (archived)
- **1996 catalog**: page framed by an 8px ink border · ribbon cards tinted sage `#c9d8b6` / salmon `#f4b6a0` / periwinkle `#b8c0e8` · link `#0000ee` · red CTA `#cc0000` · system Arial Black/Times stack (period-accurate; the brief justifies system faces here) · 1px hard rules, GIF-style "NEW!" sticker bursts.
- **2001 molded plastic**: periwinkle `#9aa6e0` bevel plates (light top edge, indigo `#2a3160` bottom line) · chamfered 45° corners · carbon `#1a1f3d` command slab with halftone texture · warm orange `#ff7a00` = action only · outlined chunky display with a hard drop shadow.
- **Rules**: only when the brief asks for nostalgia. Modern floor still applies: AA contrast, focus rings, 16px body, responsive.
- **Fit**: ✅ anniversaries, games, retro products, parody · ❌ as a default for anything

## 🧩 Signature move library
**One signature per app, used at one scale.** Swap a direction's move for another here, then re-derive it from the product (the shape, the hue, the content it carries).

| Move | What it is | Suits |
|---|---|---|
| Atmospheric band | Gradient/horizon stripe across the hero top or page foot | 1, 14, 23 |
| Polarity-flip featured tier | Recommended plan inverts to the dark surface, not an accent border | 1, 3, 24 |
| Keyword highlight chip | One headline word wrapped in accent, like syntax highlighting | 21, 9 |
| Corner square | Small accent square pinned to a card corner | 2, 15 |
| Color-coded categories | Each product/module owns a hue as a card fill | 7, 6 |
| Orbit arcs + satellite CTA | Circular images joined by 1px arcs, docked arrow button | 12 |
| Ghost watermark | Heading-scale text in canvas+2% behind content | 12, 25 |
| Brand stripe | 4px multi-color divider at key moments | 27, 16 |
| Floating circular action | One persistent primary action, bottom-right | 11, 13 |
| Command-as-hero | Install command / live input as the first thing on the page | 4, 10, 17 |
| Giant sign-off wordmark | Footer wordmark at display-xxl, tinted near canvas | 22, 6, 20 |
| Double-bezel card | Card nested in a tinted shell with a concentric radius | 14 |
| Timeline feed | Tiles on a vertical rule with timestamps | 22, 17 |
| Logo-geometry echo | Logo shape scaled up to architecture (slashes, chevrons) → [brand-identity.md](brand-identity.md) | any |

## 🔀 Mix two directions
**One base, one guest.** The base owns type, radius grammar, depth model, density, and the signature. The guest donates **at most two disciplines**: palette strategy, motion character, imagery approach, density courage. Donate ambition, never clothes: a lifted motif is a costume ([design-taste → Derive](design-taste.md#derive-dont-default)).

| Base | + Guest donates | Result |
|---|---|---|
| 17 Midnight dev-tool | 9's imagery approach (hand-drawn marginalia from *your* product's world) | A dev tool with humor, still keyboard-first |
| 5 Print magazine | 20's committed palette (numerals in one voltage hue) | Data journalism with punch |
| 1 Precision fintech | 24's band rhythm | A consumer bank landing page with gravity |
| 3 Warm workspace | 6's palette commitment (one pastel block per page) | A friendlier productivity app |
| 13 Photo-first marketplace | 18's warm-dark polarity | A night-mode food or events marketplace |

- **Never mix** two depth models (e.g. tinted shadow stack + surface ladder), two radius grammars, or two signature moves.
- **Clashes to avoid:** 15 + 14 (0 radius vs squircles) · 19 + 8 (instant vs springy) · 26 + 7 (no color vs color-as-code).
- Write the mix as a named line in `DESIGN.md`: `Base #17, guest #9 (imagery approach)`.

## 🧬 Make siblings differ
**A new app must differ from each of the last 2–3 sibling apps on ≥ 4 of these axes, including at least one of: polarity, display class, accent hue family.** This is on top of the design-taste rule (never reuse a display face or accent hue in consecutive apps).

| # | Axis | Values to rotate |
|---|---|---|
| 1 | Canvas polarity | light · dark · alternating bands |
| 2 | Neutral temperature | cool · pure · warm (one family per app) |
| 3 | Palette strategy | restrained · committed · full palette / category code · drenched · none |
| 4 | Accent hue family | 12 families, 30° apart. Siblings sit ≥ 60° apart |
| 5 | Display class | grotesk · wide · condensed · rounded · serif · mono |
| 6 | Weight posture | light display (300) · regular (400–500) · heavy (800+) |
| 7 | Case + tracking | sentence tight · uppercase compressed · uppercase wide |
| 8 | Radius grammar | 0 · 2–4 · 8–12 · 24+ · pill-everything |
| 9 | Depth model | hairline · surface ladder · tinted shadow stack · color block · photographic · bevel |
| 10 | Motion character | instant · calm · springy · cinematic · hard cuts |
| 11 | Imagery | product screenshots · photography · illustration/mascot · type-only |
| 12 | Signature move | from the library above; never the sibling's |

- Same direction twice across siblings → allowed only if you change ≥ 4 axes. At that point it's effectively a different direction, so pick one.
- Directions next to each other in this file differ less than you'd think (e.g. 17 vs 18). Check the axes, not the numbers.

## 🚫 Don't clone a brand
**Every direction is a recipe distilled from many sites. Shipping any single brand's look is forbidden.**
- **NEVER** use: a brand's proprietary font (name or scraped files), logo or mark, exact palette triplet (canvas + ink + accent), signature device at the same scale, or copy.
- **Screenshot test:** if a viewer can name the source brand, rework. Shift the accent ≥ 30° in hue, change canvas temperature, or swap the display class.
- `seen in` = where the pattern was observed (public DESIGN.md analyses, as of 2026-09). It is not a target.
- Brief says "make it like <brand>" → match the **craft level** and one or two traits, then re-accent (per [design-md → Reference site](design-md.md#-bootstrap-one-four-paths)).

## 📋 Record the pick (paste into DESIGN.md §1)
```markdown
**Direction:** #9 Sketchbook engineering (base) + guest #20 (committed palette)
**Re-derived:** accent hue 73° → 152° (product = greenhouse sensors); display Rubik → Bricolage Grotesque
**Differs from <sibling-app> on:** polarity (light vs dark), display class (rounded vs grotesk), accent family (152° vs 275°), depth (hairline vs surface ladder)
**Signature (one, one scale):** mascot marginalia at section joins
**Dials V/M/D:** 5/4/6
```

---
Sources: [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) (74 public DESIGN.md analyses, read in full) · [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (soft, minimalist, brutalist, taste, gpt-taste variants) · [pbakaus/impeccable](https://github.com/pbakaus/impeccable).
