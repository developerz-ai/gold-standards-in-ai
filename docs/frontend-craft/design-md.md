# 🎨 DESIGN.md — the design brain

**Every app ships a `DESIGN.md` at repo root. Agents read it before touching UI.** It's the app's visual identity in plain markdown: palette with roles, type scale, components with states, depth, do's & don'ts. `CLAUDE.md` says how to *build*; `PRODUCT.md` says *who and why*; `DESIGN.md` says how it should *look and feel*. No file → the agent falls back to its generic default look.

> As of 2026-09: DESIGN.md began as a Google Stitch convention; a public spec (`google-labs-code/design.md`) adds optional YAML token frontmatter + a linter. Any coding agent reads it as plain markdown; no tooling needed.

## 📍 Where it lives

| File | Answers | Written when |
|---|---|---|
| `CLAUDE.md` / `AGENTS.md` | How to build: stack, commands, conventions | Repo init → [claude-md.md](../writing-for-agents/claude-md.md) |
| `PRODUCT.md` | Who, what, why: users, purpose, voice, brand commitments, evidence on hand | Kickoff, **before** any visual work (template below) |
| `DESIGN.md` | Durable look & feel: atmosphere, tokens, components, rules | Seed at kickoff → final from the built world |
| `docs/design/surfaces/<route>.md` | One surface's strategy: register, job, **direction contract**, open decisions | Before building that surface → [design-taste.md](design-taste.md#-direction-contract-before-code) |
| `src/styles/tokens.css` | The same tokens as code | Alongside DESIGN.md → [css-scss-craft.md](css-scss-craft.md#design-tokens--css-custom-properties) |
| `brand/` | Logo, icons, brand kit files | → [brand-identity.md](brand-identity.md) |

- One line in `CLAUDE.md`: `UI work: read DESIGN.md first. Never add a color/font/radius not in it.`
- `PRODUCT.md` = strictly non-visual. `DESIGN.md` = strictly visual. Surface briefs = strategy for one route only. Never duplicate between them; never promote one surface's composition into the global world.
- Monorepo: one per app dir; a root file only for shared brand primitives. An app with a different platform (native vs web) needs its own `PRODUCT.md`: one inherited record can't hold two platforms.

## 🧾 PRODUCT.md — product truth first
Rule: **facts only, no visuals.** Confirmed answers + explicitly marked open decisions. Omit irrelevant sections; never pad with generic prose.
```markdown
# Product
<!-- product-schema: 1 -->
## Platform            web | ios | android | adaptive
## Stack               greenfield only: the user's answer, or "delegated: <choice + why>"
## Users               primary users, their situation, the job
## Product Purpose     what it does, why it exists, what success means
## Positioning         the mechanism a neighbor could not truthfully copy
## Operating Context   workflows, environments, rituals, documents that are factual parts of use
## Capabilities and Constraints   confirmed features, technical limits, terminology, undecided facts
## Brand Commitments   existing name, voice, assets, "what we are not" the user made binding
## Evidence on Hand    real content, data, testimonials, assets (with paths); absences that must not be fabricated
## Product Principles  3-5 durable strategic principles, no visual recipes
## Accessibility & Inclusion   required standard, known user needs, reading level
```
- **Explore the repo before asking.** Ask ≤ 3 focused questions per round, only for material gaps. Confirm inferences; label inferred facts in the file.
- **Never ask for aesthetics here** (colors, fonts, mood, references). A volunteered binding visual constraint is recorded, not expanded.
- **Record "undecided" instead of inventing.** No invented customers, benchmarks, pricing, testimonials.
- Keep the schema comment: later tooling can tell a deliberately short record from an old one.

## 🧬 One app, one identity
Goal: every app we ship looks **different**: its own colors, type, personality. Not "our house style" repainted.
- `DESIGN.md` is **per-app identity**, derived from `PRODUCT.md` + the direction chosen per [design-taste.md](design-taste.md#-make-each-app-different) (catalog → [design-directions.md](design-directions.md)).
- **Deliberately distinct from sibling apps.** Read the last 2–3 projects' `DESIGN.md`; differ from each on ≥ 4 of the 12 [sibling axes](design-directions.md#-make-siblings-differ) (incl. polarity, display class or accent hue family) and write the "Differs from" line in section 1.
- Record the **direction, dials, seed and sibling diff** in section 1 so the next app can rotate away from them.
- Template below is neutral scaffolding. The *content* must be opinionated.

## 🧭 Pick this app's direction (kickoff, ~10 min)
Answer these, write them into section 1 of `DESIGN.md`, then derive tokens.

| Decision | Pick | Example |
|---|---|---|
| Direction | One (+ ≤ 1 guest) from [design-directions.md](design-directions.md) | "#4 Readme-native" |
| 3 mood words | Adjectives a user would feel, not features | "calm, exact, warm" |
| North star | One named metaphor (offer the user 2–3) | "The Lab Notebook" |
| Canvas polarity | From the use scene: light · dark · warm off-white · bands | Warm off-white |
| One accent | One hue + its job; not the last project's hue family | Terracotta `#c2573a`, CTAs only |
| Type pairing | Display + body (+ mono); **not reused from last project** | Serif display + humanist sans |
| Dials | VARIANCE / MOTION / DENSITY | 4 / 3 / 6 |
| Depth model | Flat+hairlines · tonal surface ladder · soft shadows · atmosphere/photo | Hairlines only |
| Shape | Radius grammar: sharp · 6–8px technical · 12px friendly · pill | 8px buttons, 12px cards |
| 2–3 references | Real sites to borrow *one trait* each from | "A's type rhythm, B's section bands" |
| Anti-references | What it must NOT look like | "Purple-gradient AI SaaS" |

### Same template, three very different apps

| | A · Field-notes journal app | B · Ops dashboard | C · Kids' learning game |
|---|---|---|---|
| Direction family | Editorial | Tool/cockpit | Playful consumer |
| Mood | quiet, literary, warm | dense, exact, calm | bouncy, bright, safe |
| Canvas | Paper cream `#f6f1e7`, ink `#22201c` | Graphite `#0f1113`, 4-step surface ladder | White + big color-block bands |
| Accent | Oxblood `#8a2f2a`, links + one CTA | Signal green `#3fb68b`, status + focus only | Tangerine `#ff7a1a` + sky block `#bfe3ff` |
| Type | Serif display 400, -2% tracking · humanist sans body | Grotesk 500 · mono for every number (`tnum`) | Rounded geometric 700 · 18px body |
| Depth | Flat, hairline rules | Tonal ladder, no shadows | Chunky 0 4px 0 offset "sticker" shadow |
| Shape | 2px, nearly sharp | 6px | 20px + pill buttons |
| Signature | Drop caps, marginalia notes | Keyboard-first command bar | Mascot at section joins |

## ❓ Why markdown, not Figma / JSON
- **LLMs read markdown best.** No parser, no plugin, no export step.
- **Prose carries intent tokens can't:** "one filled CTA per band", "shadows only on product imagery". JSON has values, not judgment.
- **Diffable + reviewable** in the same PR as the CSS change. **Portable** across agents and generators.
- Tokens still exact: hex, px, weights inline or in YAML frontmatter. Markdown ≠ vague.

## 🦴 The section skeleton
Two dialects exist in the wild; both work. Pick one per repo, keep headings **exact** ("Colors", not "Color Palette"; tooling parses them).

| # | Classic (9 sections) | Spec (8 canonical + frontmatter) |
|---|---|---|
| 1 | Visual Theme & Atmosphere | Overview |
| 2 | Color Palette & Roles | Colors |
| 3 | Typography Rules | Typography |
| 4 | Component Stylings | Layout |
| 5 | Layout Principles | Elevation & Depth |
| 6 | Depth & Elevation | Shapes |
| 7 | Do's and Don'ts | Components |
| 8 | Responsive Behavior | Do's and Don'ts |
| 9 | Agent Prompt Guide | (responsive → Layout; motion → Overview/Components) |

What each section must hold:

| Section | Must contain | Skip if |
|---|---|---|
| Atmosphere / Overview | North star, 2–3 sentences of mood, dials, **Key Characteristics** (5–8 bullets), motion grammar + signature interaction | never |
| Colors | Name + value + **role** per color, grouped: brand/accent · surface · text · border · semantic; both theme values per role | — |
| Typography | Families + fallbacks, **enumerated ramp**, hierarchy **table** (role/size/weight/line-height/tracking/use) | — |
| Components | Buttons, inputs, cards, nav, chips + **states** (hover/focus/active/disabled/error) | pre-code seed |
| Layout | Base unit, spacing scale, container, grid, section rhythm, collapse strategy | — |
| Depth | Shadow ladder with exact `box-shadow`, or "flat, depth via X" stated explicitly | — |
| Shapes | Radius scale + which component uses which | folded into Components |
| Do's & Don'ts | 5–8 each, specific, exact values; the banned list for this app | never |
| Responsive | Breakpoints table, collapse strategy, touch targets ≥ 44px, type step-down → [adaptive-ui.md](adaptive-ui.md) | — |
| Agent Prompt Guide | Quick color reference + 3–5 paste-ready component prompts | spec dialect |
| Known Gaps | What wasn't observed / still undecided | nothing open |

### Spec-dialect frontmatter rules (as of 2026-09)
- Top-level groups: **only** `colors`, `typography`, `rounded`, `spacing`, `components`. Shadows, motion, breakpoints go in prose (or a sidecar JSON), never invented top-level keys.
- Token refs: `{colors.primary}`. Components may reference primitives; **primitives never reference each other**.
- Component sub-tokens are limited: `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`. Variants = sibling keys (`button-primary`, `button-primary-hover`).
- Keep the project's canonical color format (OKLCH stays OKLCH). **Frontmatter is normative**: prose names the token, never restates a different value.
- Scale keys use the project's own names (`oxblood-deep`), not framework defaults. Omit empty groups; fabricated tokens pollute the spec.

## ✍️ Writing good entries

| ❌ Weak | ✅ Strong | Why |
|---|---|---|
| `blue: #533afd` | **Indigo** (`#533afd`): primary CTA fill + link emphasis; one filled button per band | Name the **role**, not just the value |
| "dark text" | **Warm Ink** (`#26251e`): all body text; never pure black | Descriptive name + where + ban |
| `rounded-lg` | Gently rounded (`12px`): cards and panels; buttons stay `8px` | Words first, exact value in parens |
| "modern, clean feel" | "Calm lab notebook: warm paper, thin rules, one terracotta signal" | Mood in concrete, sensory words |
| "use nice shadows" | Level 1 `0 1px 3px rgba(0,55,112,.08)` cards; Level 2 floating panels only | Exact tokens, bounded use |
| "don't overuse accent" | **The One Voice Rule.** Accent ≤ 10% of any screen. Rarity is the point. | **Named rules**: memorable, citable |
| "headings are bold" | Display 300 weight, -1.4px @ 56px → -0.2px @ 20px; 400+ breaks the voice | Scaling rule + consequence |
| 40 near-duplicate grays | 4–6 neutrals, each with a job | Extract only what's reused (3+ uses) |

- **Group colors by role**, not hue order. **1–3 Named Rules per section**: `**The <Name> Rule.** <short doctrine>`.
- **Decisive where evidence is decisive:** hard language for real invariants, soft for provisional guidance.
- **Every Don't is grounded** in the real system or a confirmed decision, not generic advice.
- **No raw class names** in prose; translate to intent. **Don't invent components** that don't exist; mark undecided values `[to resolve in implementation]`.

### System rules worth writing down
| Rule | Why |
|---|---|
| **Enumerated type ramp**: every `font-size` in code lands on a listed step; adding a step is a design decision | Real codebases drift to dozens of near-identical sizes no reader can tell apart |
| **Kit consumption**: reach for an existing primitive (button, section, bento, badge, empty state) before inventing a class; a new invented pattern gets flagged for the kit | Stops per-page bespoke vocabularies (`.hero-cta`, `.footer-cta`…) |
| **Role swap by theme/size**: name which role takes over where an accent fails contrast (e.g. small accent text on the light theme) | Dark-first token lists lure agents into unreadable light-theme text |
| **Texture/material budget**: which elements may carry brand texture; never text on high-contrast texture | Texture everywhere = noise; texture under text = illegible |
| **Source-of-truth comment** atop the frontmatter: which token file wins, "change both" | Two sources without a winner drift silently |

## 🚀 Bootstrap one: four paths

| Path | When | Steps |
|---|---|---|
| **Scan** (extract from code) | Existing app, no DESIGN.md | 1. grep `--color-` `--font-` `--radius-` `--shadow-` `--space-` `--ease-` `--duration-` in CSS · 2. Tailwind `theme.extend` / theme.ts / tokens.json · 3. read button/card/input/nav components for variants + states · 4. load the running app (headless browser), sample computed styles of `body h1 a button .card` · 5. keep values used 3+ times · 6. ask the qualitative questions below · 7. write |
| **Reference site** | New app, borrowing a feel | Pick 2–3 sites; record *one* trait each. Extract values from DevTools computed styles. **Blend + re-accent**; never clone one brand whole |
| **Generated reference image** | New app, image generation available | Comp → analyze → DESIGN.md → faithful build. Full pipeline → [reference-image-design.md](reference-image-design.md) |
| **Seed** | Pre-code, no references | Direction table above → atmosphere, palette strategy, type character, depth, shape. Mark `<!-- SEED: re-extract once code exists -->`. Frontmatter `name` + `description` only. No Components section |

**Qualitative questions** (two rounds, ≤ 3 each; offer 2–3 options per question): north star metaphor · overview mood + confirmed anti-references · descriptive names for key colors ("Deep Muted Teal-Navy", not `blue-800`) · elevation philosophy (flat / layered / lifted; ambient or structural shadows) · component feel in one phrase ("tactile and confident").

- Existing DESIGN.md → **never silently overwrite**. Show it; choose refresh / merge / replace.
- Agent prompt for Scan:
```text
Read CLAUDE.md, PRODUCT.md, and src/styles/. Extract the visual system into DESIGN.md
at repo root using the section skeleton in docs/frontend-craft/design-md.md.
Only tokens used 3+ times. Every color: descriptive name + value + role.
Typography as a table plus the enumerated ramp. Components with all states.
Ask me for 3 mood words and a north-star metaphor (offer 2-3 options) before
writing the Overview. Do not invent components or tokens.
```

## 🏁 New world: final DESIGN.md comes from the build
**Rule: for a new or replacement world, write the final DESIGN.md at finish, from the shipped code.** A rulebook written before the build gets defended against reality instead of describing it. The seed guides the build; the documenter records what landed.

Documenter (ideally a fresh subagent, after the [review loop](design-review-loop.md) verdict):
- **The build wins** over the plan. Where the direction contract and code diverge, record the code and note the divergence.
- **Never canonize a defect.** A banned element the build carries (eyebrows, hard offset shadows, glyph icons…) goes in a "not canonized" line, never into the system. One shipped violation must not become house style.
- **Don't ban a device the world natively uses**, and don't record a value just to make a finding disappear.
- **"No changes" is a valid result** for ordinary extensions: report the files checked. Report pre-existing drift; don't repair it unasked.
- Output: paths written (or "no changes" + evidence), a 5-line summary (palette, ramp, named rules), defects not canonized.

## 🔄 Keep it in sync with CSS tokens
**One source of truth: `tokens.css` holds values, DESIGN.md mirrors them + adds intent.** Drift = the agent follows the doc and ships the wrong color.

| Rule | How |
|---|---|
| Same names both sides | `--color-accent` ↔ `accent` / **Accent**; semantic names, not `blue-500` |
| Change both in one PR | Token PR without DESIGN.md diff → reviewer blocks |
| New value needs a doc entry first | "Never add a color/font/radius not in DESIGN.md" in CLAUDE.md |
| Dark mode | Document both values per role → [theming-dark-mode.md](theming-dark-mode.md) |
| Lint (spec dialect) | `npx @google/design.md lint DESIGN.md` |
| Preview page | `docs/design/preview.html`: swatches + tonal ramps, type ramp, 5–10 primitives with hover/focus, **light and dark**. Regenerate with DESIGN.md; screenshot it in review |
| Refresh | After a redesign, or when the drift check fails → re-run Scan, compare against the live app, merge |

Cheap drift check (CI or pre-commit):
```bash
norm() { grep -oiE '#[0-9a-f]{6}\b' "$1" | tr 'A-F' 'a-f' | sort -u; }
diff <(norm DESIGN.md) <(norm src/styles/tokens.css) \
  && echo "DESIGN.md in sync" || { echo "hex drift between DESIGN.md and tokens.css"; exit 1; }
```

### Three kinds of "out of date": keep them apart
| Kind | Example | Owner |
|---|---|---|
| **Tool** | Installed skill/linter older than published | Update the tool |
| **Schema** | Retired section, renamed field, file in an old location | Mechanical fix. A deprecated field is treated as **absent** for every decision, not kept "just in case" |
| **Truth** | Code moved on, doc no longer describes it | Re-run Scan. "N commits since last edit" is a hint, never proof: read the doc against current tokens before calling it stale |

**Never repair drift as a side effect of a design task.** Report it; fix it only when asked.

## 🧩 Extracting components and tokens
- **Extract only what's used 3+ times with the same intent.** Premature abstraction is worse than duplication.
- Two things that look alike but serve different intents **stay separate**.
- No design system yet? **Ask where it should live** before creating one.
- Plan: components, tokens (primitive vs semantic), variants, naming that matches the repo, migration path. Then migrate every instance, delete the old code, update DESIGN.md.

## 📊 What great systems share (74 public DESIGN.md files analyzed, 2026-09)
Patterns, not a style to copy. Use them as *quality bars*; pick your own values.

| Pattern | Seen in | What it looks like |
|---|---|---|
| **One accent, used scarcely** | ~40 of 74 state it explicitly | Single chromatic CTA color; everything else neutral. "One filled button per band" |
| **Never pure black / pure white text** | recurring | Ink is warm or navy-tinted (`#26251e`, `#0d253d`) |
| **Negative tracking on display** | ~56 of 74 | ≈ -2% to -4% of font size at 48–80px, easing to 0 at body |
| **Modest display weight** | ~22 explicit | Display at 300–500; "never bold" as the editorial signal |
| **Type = 1 family + mono** | most | One sans (sometimes a serif display) + mono for code and numbers |
| **8px base spacing** | ~45 | 4/8/12/16/24/32/64; sections 64–96px apart |
| **Hierarchical radius grammar** | nearly all | Buttons 6–8px, cards 12px, hero container 16px, pill only for tags. Never mixed on one screen |
| **Restrained depth** | ~23 no-shadow, ~37 stacked | Hairlines/tonal ladder, or faint stacked shadows (4–12% opacity). Never one heavy drop |
| **Dark = surface ladder** | dark-canvas systems | 3–5 surface steps a few % lighter each; hierarchy without shadow |
| **Product UI as hero decoration** | ~51 | Real screenshots/mockups over illustration or stock art |
| **One signature device** | most | Gradient mesh, color-block bands, mascot, keyword highlight, used at one scale only |
| **Featured tier = polarity flip** | pricing pages | Inverted dark card, not an accent-bordered one |
| **Every Don't names a value** | all good ones | "Don't bump display above 300", "Don't add a second accent" |

Note: ~32 of 74 also use uppercase eyebrows above headings. Common ≠ good: our floor rations them hard ([design-taste.md](design-taste.md#-banned-patterns-unless-the-brief-asks)).

## 🧑‍💻 How agents use it
- **Before UI work:** read DESIGN.md fully; quote the tokens you'll use in the plan.
- **One component at a time**; reference tokens by name (`{colors.primary}`, `--radius-md`).
- **New need not covered** → propose a DESIGN.md addition first, then code.
- **Review loop** checks the UI against DESIGN.md → [design-review-loop.md](design-review-loop.md).
- Keep it scannable: ~150–400 lines. Longer → move rationale to `docs/design/`, keep rules here.

## 📄 Paste-ready template
```markdown
---
name: <App Name>
description: <one-line visual thesis>
# Source of truth: src/styles/tokens.css. Change both in one PR.
colors:
  canvas: "#______"
  surface: "#______"
  ink: "#______"
  ink-muted: "#______"
  hairline: "#______"
  accent: "#______"
  accent-press: "#______"
  on-accent: "#______"
  danger: "#______"
  success: "#______"
typography:
  display: { fontFamily: "<Display>, <fallback>", fontSize: 56px, fontWeight: 400, lineHeight: 1.05, letterSpacing: -1.6px }
  heading: { fontFamily: "<Display>, <fallback>", fontSize: 32px, fontWeight: 500, lineHeight: 1.15, letterSpacing: -0.6px }
  body:    { fontFamily: "<Body>, <fallback>", fontSize: 16px, fontWeight: 400, lineHeight: 1.55, letterSpacing: 0 }
  label:   { fontFamily: "<Mono>, monospace", fontSize: 12px, fontWeight: 500, lineHeight: 1.3, letterSpacing: 0.4px }
rounded: { sm: 4px, md: 8px, lg: 12px, pill: 9999px }
spacing: { xs: 4px, sm: 8px, md: 16px, lg: 24px, xl: 32px, section: 96px }
---

# Design System: <App Name>

## 1. Visual Theme & Atmosphere
**North star: "<named metaphor>"** · Direction: <# name (+ guest)> · Dials V/M/D: <n/n/n> · Seed: <…>
**Differs from <sibling>:** <≥ 4 axes, e.g. polarity, display class, accent family, depth>
<2-3 sentences: mood (3 words), density, canvas polarity, what it must never look like.>
**Key Characteristics:**
- <canvas + ink choice>
- <the one accent and its only job>
- <type pairing + display signature>
- <depth model> · <radius grammar>
- <the one signature device> · <motion grammar: easing, durations, signature interaction>

## 2. Color Palette & Roles
### Brand & Accent
- **<Name>** (`#______`): <exact job>; <where it never appears>
### Surface
- **<Name>** (`#______`): page canvas · **<Name>** (`#______`): cards, inputs
### Text
- **<Name>** (`#______`): body · **<Name>** (`#______`): secondary, captions
### Border & Semantic
- **<Name>** (`#______`): 1px hairlines · danger `#______` · success `#______`
**The <Name> Rule.** <one forceful color doctrine>

## 3. Typography Rules
Display: <family> · Body: <family> · Mono: <family>. <one-line character of the pairing>
Ramp (every font-size is one of): 12 · 14 · 16 · 18 · 24 · 32 · 56

| Role | Size | Weight | Line height | Tracking | Use |
|---|---|---|---|---|---|
| Display | 56px | 400 | 1.05 | -1.6px | Hero only |
| Heading | 32px | 500 | 1.15 | -0.6px | Section titles |
| Body | 16px | 400 | 1.55 | 0 | Paragraphs, max 68ch |
| Label | 12px | 500 | 1.3 | +0.4px caps | Short metadata only |

## 4. Component Stylings
- **Button primary:** accent fill, on-accent text, `md` radius, 10px 18px; hover <…>; active <…>; focus 2px accent ring, 2px offset; disabled 40% opacity
- **Button secondary:** <…>
- **Input:** <stroke, bg, radius>; label above, error below in danger; focus <…>
- **Card:** <bg, border, radius, padding, shadow level>
- **Nav:** <height, bg, active state, mobile treatment>
- **Empty / loading / error states:** <skeletons matching layout; empty state with next action>

## 5. Layout Principles
Base unit 8px. Container <max-width>. Grid <cols/gutter>. Section rhythm `section`. <whitespace philosophy>

## 6. Depth & Elevation
| Level | Treatment | Use |
|---|---|---|
| 0 | flat | default |
| 1 | `<box-shadow or hairline>` | cards |
| 2 | `<box-shadow>` | menus, dialogs |

## 7. Do's and Don'ts
- Do <specific rule with value>
- Don't <specific ban with value>

## 8. Responsive Behavior
| Name | Width | Changes |
|---|---|---|
| Mobile | <768px | 1 col, display 56→36px, full-width buttons |
| Tablet | 768-1023px | 2 col |
| Desktop | ≥1024px | full grid |
Touch targets ≥44px. No horizontal scroll at 375px.

## 9. Agent Prompt Guide
- Quick ref: canvas `#______` · ink `#______` · accent `#______` · radius `md`
- "Build <component>: <surface>, <type role>, <accent use>, <radius>, <states>."
- Known gaps: <undecided or unobserved>
```

## 🚫 Pitfalls
- Hex without role → agent sprinkles accent on backgrounds and body text.
- Mood only, no tokens → every page drifts. Tokens only, no mood → correct but soulless.
- Copying a famous brand's file wholesale → your app looks like theirs. Borrow traits, re-accent.
- Documenting one-offs → the system bloats; the agent treats noise as rules.
- Writing the final rulebook before the build → the build gets bent to fit the doc.
- DESIGN.md older than `tokens.css` → stale doc wins; run the drift check.
- Visual rules in `PRODUCT.md` or product facts in `DESIGN.md` → split them.

## 🔗 Related
[design-taste.md](design-taste.md) · [design-directions.md](design-directions.md) · [design-review-loop.md](design-review-loop.md) · [brand-identity.md](brand-identity.md) · [reference-image-design.md](reference-image-design.md) · [css-scss-craft.md](css-scss-craft.md) · [theming-dark-mode.md](theming-dark-mode.md) · [claude-md.md](../writing-for-agents/claude-md.md) · [output-completeness.md](../writing-for-agents/output-completeness.md)

Sources: [voltagent/awesome-design-md](https://github.com/voltagent/awesome-design-md) · [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) · [pbakaus/impeccable](https://github.com/pbakaus/impeccable)
