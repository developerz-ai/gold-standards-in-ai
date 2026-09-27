# 📐 Adaptive UI — responsive, touch, native & fast

**Adapting means redesigning for the new context, not scaling pixels.** Each device class gets its own layout, its own input model and its own density, while the information architecture stays the same everywhere. A phone gets a restructured layout, and a tablet gets something other than a stretched phone UI. Desktop is a separate design too, not the mobile layout with extra padding.

| Need | Go to |
|---|---|
| `clamp()` scales, Grid, container-query syntax, breakpoint mixins | [css-scss-craft.md](css-scss-craft.md) |
| PWA, workers, offline, `dvh` basics | [pwa-offline.md](pwa-offline.md) |
| Image/video/font formats, budgets, optimize-before-commit | [assets-optimization.md](assets-optimization.md) |
| Core Web Vitals targets (LCP/INP/CLS) | [seo.md](seo.md#performance--core-web-vitals) |
| Native audit scoring, simulator capture | [design-review-loop.md](design-review-loop.md#-native-ios--android-deltas) |
| Swift/Kotlin repo shape, stack | [../stack/mobile.md](../stack/mobile.md) |
| Look, voice and motion of each app | [design-directions.md](design-directions.md) · [brand-identity.md](brand-identity.md) · [motion-and-delight.md](motion-and-delight.md) · [ux-copy.md](ux-copy.md) · [reference-image-design.md](reference-image-design.md) |

## 🧭 Ask before adapting
| Question | Changes |
|---|---|
| Device class: phone, tablet, foldable, desktop, TV, print? | layout topology |
| Input: touch, mouse, keyboard, stylus, or a mix? | target size, hover, shortcuts |
| Posture: one thumb on the go, or two hands at a desk? | nav placement, density |
| Connection and CPU: a cheap Android on slow 4G? | bundle size, images, virtualization |
| Platform expectation: iOS, Android or web? | controls, back behavior, type |

Rules that apply everywhere:
- NEVER hide core functionality on small screens. If a feature matters, make it work there.
- NEVER change the information architecture between device classes. Keep the same destinations with a different presentation.
- NEVER use device or user-agent sniffing. Use feature and media queries instead.
- NEVER assume a desktop device is powerful, or that a desktop device has no touch screen.
- ALWAYS support landscape. Lock orientation only when the task demands it, never to hide a layout bug.

## 📏 Breakpoints: let the content decide
- Write mobile-first: base styles are for the narrowest screen, and `min-width` layers add complexity. Desktop-first (`max-width`) makes phones parse styles they will never use.
- Find each breakpoint empirically: start narrow, stretch the window until the design breaks, and add a breakpoint there. Device widths make bad breakpoints.
- About 3 structural breakpoints is usually enough. Handle the steps between them with fluid values (`clamp()`) and intrinsic layouts.
- Declare each component's collapse in the component itself, so no `<768px` behavior is left to "the framework handles it".
- Cap the content width, and let the margins absorb extra space on 4K screens. Line lengths stay readable that way.

### Collapse patterns seen across 70+ production design systems (As of 2026-09)
| Element | Wide → narrow |
|---|---|
| Card/product grid | 4-up → 3 → 2 → 1. **Drop columns, never reflow rows into odd orphans.** |
| Display type | stair-steps down each breakpoint (e.g. 80 → 56 → 48 → 36px) or one `clamp()` |
| Top nav | full links → compact → logo + primary CTA + menu. **The primary CTA stays visible at every width.** |
| Sticky right rail (booking, checkout) | → sticky bottom bar with price and the one CTA |
| Sub-nav / filter pills | wrap row → horizontal scroll rail → `<select>` |
| Docs sidebar | persistent → top accordion |
| Pricing comparison table | columns → horizontal scroll → one card per tier or an accordion |
| Footer link columns | 6 → 3 → 2 → accordion |
| Section padding | 96 → 64 → 48px |
| Multi-search bar | segmented pill → one tappable pill that opens a full-screen search |
| Hero mockup | beside the copy → below the copy, scaled to about 80% |

Put a **Responsive Behavior** section in every app's [DESIGN.md](design-md.md) with four parts: a breakpoints table (name, width, key changes), touch targets, collapsing strategy, and image behavior.

## 📦 Container queries vs media queries
| Use | When |
|---|---|
| `@container` | the component appears in several contexts (card in a sidebar and in main content, a widget in a dashboard) |
| `@media (width >= …)` | page-level topology, such as nav shape, app-shell columns or the page gutter |
| `@media (pointer/hover/…)` | the input model, which a container cannot know |
| Intrinsic (`auto-fit`, `flex-wrap`, `clamp`) | first choice, because it needs no query at all |

```css
.card-host { container: card / inline-size; }
.card { display: grid; gap: var(--space-3); }
@container card (width >= 28rem) {
  .card { grid-template-columns: 8rem 1fr; }
}
/* cqi = 1% of container inline size: type that fits its box */
.card h3 { font-size: clamp(1rem, 0.8rem + 2cqi, 1.5rem); }
```

## 🌊 Fluid type and spacing
```css
:root {
  /* min at ~360px viewport, max at ~1280px */
  --step-0: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  --step-3: clamp(1.75rem, 1.2rem + 2.4vw, 3rem);
  --step-5: clamp(2.25rem, 1rem + 5.5vw, 5rem);
  --space-s: clamp(0.75rem, 0.7rem + 0.25vw, 1rem);
  --space-l: clamp(2rem, 1.4rem + 2.6vw, 4rem);
  --gutter: clamp(1rem, 0.5rem + 2.5vw, 3rem);
}
body { font-size: var(--step-0); }
.prose { max-width: 65ch; }
```
- ALWAYS mix `rem` into the fluid term (`1rem + 2vw`). A pure `vw` value ignores browser zoom and fails WCAG 1.4.4.
- Body text stays at 16px or larger on the web. Body line length stays between 45 and 75 characters (`ch`). The wider the line, the more line height it needs.
- Use fluid display type on marketing surfaces. Dense product UI keeps a fixed role scale so the layout stays predictable.
- Light text on a dark background needs slightly more line height and tracking, and one step more weight.
- Test at 200% zoom and 320px width (WCAG 1.4.10 reflow). Nothing may clip, and the page scrolls only vertically.

## 🧱 Layout primitives: compose, don't one-off
Name layouts by the relationship they control, not by the page they appear on.

| Primitive | Relationship | Replaces |
|---|---|---|
| **Stack** | vertical rhythm between siblings | margin soup |
| **Cluster** | inline items that wrap (tags, buttons) | float and `inline-block` hacks |
| **Sidebar** | fixed side + fluid main, which stack when too narrow | a breakpoint per sidebar |
| **Switcher** | row → column at a threshold, all or nothing | awkward 2+1 wraps |
| **Grid** | equal auto-fit cells | per-breakpoint column counts |
| **Center** | max-width + gutter | `container` classes with magic numbers |
| **Reel** | horizontal scroll with snap | carousels built in JS |

```css
.stack   { display: flex; flex-direction: column; gap: var(--stack-gap, var(--space-s)); }
.cluster { display: flex; flex-wrap: wrap; gap: var(--space-s); align-items: center; }
.sidebar { display: flex; flex-wrap: wrap; gap: var(--space-l); }
.sidebar > :first-child { flex-basis: 16rem; flex-grow: 1; }
.sidebar > :last-child  { flex-basis: 0; flex-grow: 999; min-inline-size: 55%; }
.switcher { display: flex; flex-wrap: wrap; gap: var(--space-s); }
.switcher > * { flex-grow: 1; flex-basis: calc((40rem - 100%) * 999); }
.grid   { display: grid; gap: var(--space-s);
          grid-template-columns: repeat(auto-fit, minmax(min(16rem, 100%), 1fr)); }
.center { box-sizing: content-box; max-inline-size: 72rem; margin-inline: auto;
          padding-inline: var(--gutter); }
.reel   { display: flex; gap: var(--space-s); overflow-x: auto;
          scroll-snap-type: x mandatory; overscroll-behavior-x: contain; }
.reel > * { flex: 0 0 auto; scroll-snap-align: start; }
```
- Use `gap` for spacing between siblings. Use the `min(16rem, 100%)` form inside `minmax` so a column never overflows at 320px.
- DOM order is focus order. If a layout reorders visually (`order`, `grid-area`), check that tab order still makes sense.

## 👆 Touch targets and thumb zones
| Platform | Minimum target | Spacing |
|---|---|---|
| Web, coarse pointer | 44×44 CSS px (WCAG 2.5.5 AAA). AA floor is 24×24 (2.5.8) | ≥ 8px between |
| iOS | 44×44 pt | breathing room |
| Android | 48×48 dp | ≥ 8 dp |
| Primary mobile CTA, form inputs | 48px tall (fintech-grade: 56px inputs) | full width on phones |

A small visible mark can still have a large hit area:
```css
.icon-btn { position: relative; inline-size: 1.5rem; block-size: 1.5rem; }
.icon-btn::after { content: ""; position: absolute; inset: -0.625rem; } /* 44px hit */
```
Thumb zones on a one-handed phone:

| Zone | Put here |
|---|---|
| Bottom third (easy) | primary actions, tab bar, sticky CTA, sheet handles |
| Middle (OK) | content, list rows, secondary actions |
| Top corners (hard) | rare, destructive or navigational items (settings, close, back) |

- Put destructive actions away from the primary action, not next to it.
- Give every tap feedback within 100ms (`:active` state, a highlight, or haptics on native).
- A swipe gesture must never be the only way to do something. Always provide a visible control too.

## 🖱️ Hover, pointer and input modes
Screen size says nothing about input. Touch laptops and tablets with keyboards are common.

```css
/* hover effects only where hover exists and is precise */
@media (hover: hover) and (pointer: fine) {
  .card:hover { translate: 0 -2px; box-shadow: var(--shadow-2); }
}
/* any touch-capable pointer: enlarge controls even on a touch laptop */
@media (any-pointer: coarse) {
  .btn, .input { min-block-size: 44px; }
}
.btn:focus-visible { outline: 2px solid var(--focus); outline-offset: 2px; }
```
- NEVER put functionality behind hover alone (tooltips holding info, hover-revealed row actions, mega menus). Touch users cannot reach it. Show the control, or reveal it on tap or focus.
- Tooltips add to visible content and never replace it.
- Use the right mobile keyboard: `type="email|tel|url|number"`, `inputmode="numeric|decimal|search"`, `enterkeyhint="search|send|next"`, `autocomplete="one-time-code|postal-code|…"`.
- Desktop gets keyboard shortcuts, context menus and Shift or Cmd multi-select. Phones get swipe actions and long-press. Build both on the same underlying commands.
- Custom controls (sliders, drag surfaces, before/after): test the drag *and* a page scroll across the control on real touch devices. A screenshot proves the layout, not the gesture.

Minimal SolidJS signal for when JS needs to know the input mode:
```tsx
import { createSignal, onCleanup } from "solid-js";

export function createMedia(query: string) {
  const mql = window.matchMedia(query);
  const [matches, set] = createSignal(mql.matches);
  const on = (e: MediaQueryListEvent) => set(e.matches);
  mql.addEventListener("change", on);
  onCleanup(() => mql.removeEventListener("change", on));
  return matches;
}
// const coarse = createMedia("(any-pointer: coarse)");
// <Show when={coarse()} fallback={<HoverMenu/>}><SheetMenu/></Show>
```
Prefer CSS whenever it can do the job. Use a JS media signal only when *behavior* changes (sheet vs popover), not styling. On SSR, pass a neutral default and let the value settle after hydration, or the layout shifts.

## 📱 Viewport, safe areas and the keyboard
```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```
```css
.hero      { min-block-size: 100svh; }   /* never 100vh: URL bar jump on mobile */
.app-shell { min-block-size: 100dvh; }
.bottom-bar {
  position: sticky; inset-block-end: 0;
  padding-block-end: max(0.75rem, env(safe-area-inset-bottom));
  padding-inline: max(1rem, env(safe-area-inset-left)) max(1rem, env(safe-area-inset-right));
}
.sheet { overscroll-behavior: contain; }  /* no scroll chaining to the page */
```
- Set `viewport-fit=cover` together with the `env()` padding every time. Setting only one of them clips content behind the notch or the home indicator.
- NEVER set `maximum-scale=1` or `user-scalable=no`. Blocking pinch zoom is an accessibility failure. On iOS, keep inputs at a 16px font size or larger to avoid focus auto-zoom.
- When the on-screen keyboard is open, sticky footers and bottom sheets must not end up behind it. Test every form on real iOS and Android. `interactive-widget=resizes-content` in the viewport meta helps on Chromium (As of 2026-09, not honored everywhere).

## 🧭 Mobile navigation
| Pattern | Use when |
|---|---|
| Bottom tab bar | 3–5 top-level **destinations** (never actions), used often |
| Top bar + back | hierarchical drill-down |
| Menu/drawer | secondary nav only, with primary destinations still visible |
| Full-screen overlay | search, complex filters, multi-step pickers |
| Bottom sheet | contextual actions, short tasks, replacing dropdowns and popovers |
| Sticky bottom CTA | a single primary action on long pages (product, booking, checkout) |
| Segmented control / tabs | switching views within one screen |

Anti-patterns:

| Anti-pattern | Why it fails |
|---|---|
| Hamburger hides every primary destination | Users can't see where to go, so engagement drops |
| Tab bar plus FAB plus top tabs plus hamburger on one screen | Four navigation systems compete for attention |
| More than 5 tabs, or a "More" tab full of core features | The IA was left unfinished |
| Sticky header and footer take more than 20% of the viewport | The content ends up in a letterbox |
| Hover or mega menu ported to touch | Nothing responds to a tap |
| Carousel used as navigation | Content hidden behind swipes gets missed |
| Modal without scroll lock or safe-area padding | The page scrolls behind it, or the close button sits under the notch |
| Custom back behavior that breaks the browser or system back | Breaks the user's muscle memory |

Navigation shape by width: phone uses a bottom bar or compact top bar, tablet uses a rail or compact horizontal nav, and desktop uses a persistent sidebar or full top nav. The destinations are the same at every width.

## 🎚️ Density per device
| Context | Density | Disclosure |
|---|---|---|
| Phone | airy. One primary task per screen, text 16px or larger | progressive (tabs, accordions, `<details>`, sheets) |
| Tablet | medium. 2 columns, master-detail | side panels |
| Desktop | dense where the job needs it: tables, multiple panels | show more at once |
| Operator/pro tools | highest density, fixed role scale, tabular figures | shortcuts over chrome |

- If the text looks small, the design isn't finished. Simplify, cut content, or split the screen before shrinking any type.
- Keep density a single variable (`--density: 1`) and multiply spacing by it. Compact and comfortable modes then cost one line of CSS.
- Mix calm and dense screens across a flow. Don't pack every screen to the maximum.

## 📊 Tables and data on mobile
| Strategy | Best for |
|---|---|
| Priority columns: hide low-priority columns and move them to the row detail | sortable lists with 1–2 key fields |
| Scroll container with a sticky first column | numeric comparison where the grid matters |
| Transform each row into a card (`data-label`) | records read one at a time |
| Accordion per row or per tier | pricing tiers, specs |
| Summary → detail screen | anything with more than 6 fields |

```css
.table-wrap { overflow-x: auto; overscroll-behavior-x: contain; }
.table-wrap th:first-child, .table-wrap td:first-child {
  position: sticky; inset-inline-start: 0; background: var(--surface);
}
td { font-variant-numeric: tabular-nums; }
@container table (width < 36rem) {
  .cards-table tr { display: grid; padding-block: var(--space-s); }
  .cards-table thead { display: none; }
  .cards-table td::before { content: attr(data-label); font-weight: 600; }
}
```
- Charts: use fewer ticks and direct labels instead of a legend, and on touch, tap to see a value instead of hovering. Offer a table view as a fallback.

## 🖼️ Responsive images
```html
<img src="hero-800.avif" alt="…" width="1600" height="900"
     srcset="hero-480.avif 480w, hero-800.avif 800w, hero-1600.avif 1600w"
     sizes="(width >= 64rem) 50vw, 100vw"
     fetchpriority="high" decoding="async">
<picture> <!-- art direction: different crop, not just resolution -->
  <source media="(width >= 48rem)" srcset="wide.avif">
  <img src="tall.avif" alt="…" width="800" height="1000" loading="lazy">
</picture>
```
- The LCP image loads eagerly with `fetchpriority="high"`. Everything below the fold uses `loading="lazy"`. NEVER lazy-load above-the-fold content.
- Always set `width` and `height`, or `aspect-ratio`, so the image reserves its space. That keeps CLS at zero.
- On phones, crop in on the subject instead of letterboxing a wide image. Keep product UI screenshots at their aspect ratio, never cropped.
- `display: none` hides an image but it still downloads. Use `<picture>` or `sizes` to avoid fetching it at all. Formats and budgets: [assets-optimization.md](assets-optimization.md).

## 🍎🤖 Native: iOS HIG vs Material 3 deltas
Our apps are native Swift and Kotlin ([../stack/mobile.md](../stack/mobile.md)). **Translate idioms between the platforms, never transplant them.** The platform's rules govern structure, navigation and interaction. Brand shows through the layers the platform leaves open: tint, color roles, type accents, motion personality and content.

| Concern | iOS (SwiftUI) | Android (Compose, Material 3) |
|---|---|---|
| Top-level nav | Tab bar, 2–5 sections. iPad: sidebar (`NavigationSplitView`) | Navigation bar (3–5) on compact width → rail → drawer on expanded width |
| Hierarchy | `NavigationStack` push. Large title on root screens, collapsing to inline | Top app bar + screen transitions. One FAB for the single primary action |
| Back | Left-edge swipe plus chevron. **Never disable the swipe** | System Back and **predictive back** (`enableOnBackInvokedCallback`). Never hijack it |
| Sheets and dialogs | `.sheet` + `.presentationDetents([.medium, .large])`, swipe to dismiss. Action sheet, alert | `ModalBottomSheet`, Material dialog only for interrupting decisions. Snackbar for transient feedback |
| Type | SF Pro. **Dynamic Type** text styles (Body 17pt, 11pt floor). No fixed pt sizes | Roboto or a brand font themed through the **type scale** (Display…Label). **`sp`**, never px |
| Color | Semantic system colors, one tint, system materials (Liquid Glass as of iOS 26). No hand-rolled glass | Color roles, **Dynamic Color** on Android 12+ with a static fallback, tonal elevation |
| Icons | SF Symbols | Material Symbols |
| Controls | Toggle, segmented control, stepper, system pickers, context menus, swipe actions | Switch, chips, filled, tonal and outlined buttons, Material pickers |
| Haptics | `.sensoryFeedback(.success, trigger:)`. Light for selection, success or warning on outcomes | `LocalHapticFeedback` / `performHapticFeedback(CONFIRM\|REJECT)`. Sparingly |
| Motion | System push, sheet rise, dismiss reverses the entrance. Honor **Reduce Motion** | Container transform, shared-axis, fade-through. Honor **Remove animations** |
| Insets | Safe area: notch, Dynamic Island, home indicator | **Edge-to-edge is enforced** (As of 2026-09, targetSdk 35 and up). Handle status bar, nav bar, cutout and **IME** insets |
| Adaptivity | Size classes (`horizontalSizeClass`). iPad Split View gives a phone-width window | Window size classes (compact, medium, expanded). Foldable posture and hinge. Multi-window |
| Touch | 44×44 pt | 48×48 dp, 8 dp apart |

- **Phone → tablet: restructure, don't stretch.** Use list + detail side by side, multi-column grids, and popovers where the phone used sheets.
- Drive structure from size classes or window size classes, never from device-model checks. Multitasking can hand the app any width.
- Web → native: rebuild the product. Use platform navigation, platform controls, touch-first affordances and scalable type. Slop test: *would a fluent user of the platform trust this, or does it read as a ported website?*
- Haptics confirm outcomes such as a success, a toggle or a snap point. NEVER put haptics on every tap or on scroll.

## ⚡ UI performance
Measure first, fix the biggest bottleneck, and measure again, on a mid-range Android over throttled 4G rather than a desktop on fiber. Targets are in [seo.md](seo.md#performance--core-web-vitals).

| Metric | Main causes | Fixes |
|---|---|---|
| **LCP** | late hero image, render-blocking CSS/JS, client-only render | SSR, preload plus `fetchpriority` on the hero, inline critical CSS, a CDN |
| **CLS** | images without dimensions, late fonts, content injected above existing content | `width`/`height` or `aspect-ratio`, fallback font metrics, reserved slots for embeds and banners |
| **INP** | long tasks, heavy handlers, big re-renders | split work across tasks (`await scheduler.yield?.()`), move work to [workers](pwa-offline.md), debounce, smaller updates |

**Fonts.** Preload only the one or two critical `woff2` files. Subset them. Load only the weights you use. Adjust the fallback's metrics so a font swap causes no reflow:
```css
@font-face {
  font-family: "Brand Fallback"; src: local("Arial");
  size-adjust: 104%; ascent-override: 92%; descent-override: 24%; line-gap-override: 0%;
}
@font-face { font-family: "Brand"; src: url(/f/brand.woff2) format("woff2"); font-display: swap; }
body { font-family: "Brand", "Brand Fallback", system-ui, sans-serif; }
```
Use `font-display: optional` when a flash of the fallback font is worse than never showing the brand font on a slow first visit.

**Layout thrash.** Batch all DOM reads, then all writes. NEVER interleave `offsetHeight` reads with style writes inside a loop. Animate only `transform` and `opacity`, casually. Treat `width`, `height`, `top`, `left` and margins as layout properties, not animation targets. Use `will-change` only on elements that are actually animating, and remove it afterwards.

**Long lists.**
```css
.feed-item { content-visibility: auto; contain-intrinsic-size: auto 120px; } /* skip offscreen render */
```
Past about 1,000 rows, or with heavy row content, virtualize: render only the visible window with a headless virtualizer (`@tanstack/solid-virtual`). Paginate or use cursors on the API side as well, so the client never receives everything at once.

**Bundle.**
```tsx
import { lazy } from "solid-js";
const Charts = lazy(() => import("./routes/Charts"));   // route-level split
```
- Starting budget: about 150–170 KB of gzipped JS on the critical path for mobile. Each chart library, editor or map loads only on its own route.
- Audit third-party scripts: each one needs an owner and a reason, or it gets removed.
- Continuous animation effects (grain, noise, blur) go on fixed `pointer-events: none` layers, never on scrolling containers, because they cause mobile repaint storms.

## 🧪 Device test matrix for screenshot review
Capture every row on each UI change and review it with [design-review-loop.md](design-review-loop.md). **Say what produced the evidence** (emulated viewport, Chromium versus WebKit, simulator, or real hardware). Emulation proves layout only. Gestures, keyboards and performance need a real device.

| Class | Viewport (CSS px) | Extra states |
|---|---|---|
| Small phone | 320×568 | 200% zoom reflow, longest locale (de/fi) |
| Phone | 390×844 | dark, keyboard open on forms, **WebKit** engine |
| Large phone | 430×932 | one-handed reach: primary action in the bottom third? |
| Phone landscape | 844×390 | sticky bars fit, no clipped modals |
| Tablet portrait | 768×1024 | touch plus hover off, rail/master-detail |
| Tablet landscape / small laptop | 1024×768 | touch plus pointer both work |
| Desktop | 1440×900 | hover, focus rings, keyboard-only pass |
| Wide | 1920×1080+ | content capped, no stretched lines |
| Throttled | 390×844, 4× CPU, slow 4G | LCP, CLS, INP, skeletons |

```ts
// playwright.config.ts: one project per row; screenshots land in the review folder
const views = { small: [320, 568], phone: [390, 844], tablet: [768, 1024], desktop: [1440, 900] };
export default { projects: Object.entries(views).map(([name, [width, height]]) => ({
  name, use: { viewport: { width, height }, hasTouch: width < 1024, isMobile: width < 768 } })) };
```
Native: capture from a simulator or emulator, never a browser (commands in [design-review-loop.md](design-review-loop.md#-native-ios--android-deltas)). Minimum set: small iPhone, Pro Max-class iPhone, iPad in 1/3 Split View, compact Android phone, a foldable (folded, unfolded and tabletop postures), and an Android tablet. Run each in dark mode and at the largest text scale (Dynamic Type AX sizes, `font_scale 1.3`–`2.0`).

## ✅ Ship checklist
- [ ] Breakpoints come from the content. Each component declares its own collapse, and the primary CTA is visible at every width.
- [ ] Fluid tokens use `rem`. Body text is 16px or larger, lines stay within 45–75ch, and 200% zoom reflows with no horizontal scroll.
- [ ] Touch targets are 44px or larger (48dp on Android). The primary action sits in the thumb zone, and no functionality depends on hover.
- [ ] `viewport-fit=cover` + `env()` insets. `svh`/`dvh`, never `100vh`. Zoom not blocked.
- [ ] Mobile nav uses 5 or fewer visible destinations. Back works everywhere, and there are no stacked navigation systems.
- [ ] Every table has a mobile strategy, and every image has dimensions and `sizes`.
- [ ] Native: platform navigation, controls, back, type scale, insets and haptics. Size classes drive the structure.
- [ ] LCP, CLS and INP are measured on a throttled mid-range device. Long lists are virtualized, and the JS stays within budget.
- [ ] The device matrix is captured, with the source of each screenshot stated.

---
Sources: [pbakaus/impeccable](https://github.com/pbakaus/impeccable) (adapt, adapt.native, ios, android, optimize, layout, typeset) · [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (taste-skill, imagegen-frontend-mobile) · [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) (Responsive Behavior sections) · layout primitives after [Every Layout](https://every-layout.dev)
