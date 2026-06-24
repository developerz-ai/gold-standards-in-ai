# 💅 CSS3 & SCSS Craft

A working reference for modern styling: token-driven theming, SCSS that generates CSS for you, and the CSS3 features (grid, container queries, `:has()`, `color-mix()`, `light-dark()`) worth reaching for in 2025+. Every snippet is copy-pasteable. Motion always ships with a reduced-motion guard.

## Design tokens & CSS custom properties
One semantic layer drives the whole UI. Store color **channels** (not full colors) so you can vary alpha without redefining the color. Tokens for color, space, motion, and elevation all live as custom properties on `:root`.

```css
:root {
  /* color channels — space-separated RGB so alpha is free */
  --c-bg: 14 16 22;
  --c-surface: 22 26 36;
  --c-text: 232 236 244;
  --c-muted: 148 158 178;
  --c-accent: 99 102 241;

  /* semantic aliases built from channels */
  --bg: rgb(var(--c-bg));
  --surface: rgb(var(--c-surface));
  --text: rgb(var(--c-text));
  --muted: rgb(var(--c-muted));
  --accent: rgb(var(--c-accent));
  --accent-soft: rgb(var(--c-accent) / 0.12); /* alpha from the same channel */

  /* spacing scale (4px base) */
  --space-1: 0.25rem; --space-2: 0.5rem; --space-3: 0.75rem;
  --space-4: 1rem;    --space-6: 1.5rem; --space-8: 2rem;

  /* motion tokens — durations + named easings */
  --dur-fast: 120ms;
  --dur-base: 220ms;
  --dur-slow: 400ms;
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);

  --radius: 0.625rem;
}
```

Separating channel from color is the trick: `rgb(var(--c-accent) / 0.12)` gives a tinted variant with no extra token.

### `light-dark()` — one declaration, both themes
Set `color-scheme`, then let `light-dark()` pick per scheme. No media query, no class toggle for the simple cases.

```css
:root { color-scheme: light dark; }

.card {
  background: light-dark(#ffffff, rgb(var(--c-surface)));
  color:      light-dark(#1a1f2b, rgb(var(--c-text)));
  border:     1px solid light-dark(#e6e8ee, rgb(255 255 255 / 0.08));
}
```

For a manual toggle and the full theming strategy, see [theming-dark-mode.md](theming-dark-mode.md).

## SCSS power features
SCSS earns its place by *generating* CSS — functions compute values, maps + `@each` emit variables and utilities, mixins package patterns. Prefer this over hand-writing repetitive rules.

### Functions
```scss
@use "sass:math";
@use "sass:color";

// px → rem against a 16px root
@function rem($px, $base: 16) {
  @return math.div($px, $base) * 1rem;
}

// fluid value: clamp(min, preferred, max), preferred scales with viewport
@function fluid($min, $max, $vw-min: 360, $vw-max: 1280) {
  $slope: math.div($max - $min, $vw-max - $vw-min);
  $intercept: $min - $slope * $vw-min;
  @return clamp(
    #{$min}px,
    #{$intercept}px + #{$slope * 100}vw,
    #{$max}px
  );
}

// readable text color for a given background (luminance threshold)
@function contrast($bg, $light: #fff, $dark: #111) {
  @return if(color.channel($bg, "lightness", $space: hsl) > 55%, $dark, $light);
}
```

### Maps + `@each` → emit CSS variables
Author the scale once; let SCSS write every custom property.

```scss
$space: (1: 0.25rem, 2: 0.5rem, 3: 0.75rem, 4: 1rem, 6: 1.5rem, 8: 2rem);
$dur:   (fast: 120ms, base: 220ms, slow: 400ms);
$ease:  (out: cubic-bezier(0.16, 1, 0.3, 1), spring: cubic-bezier(0.34, 1.56, 0.64, 1));

:root {
  @each $k, $v in $space { --space-#{$k}: #{$v}; }
  @each $k, $v in $dur   { --dur-#{$k}: #{$v}; }
  @each $k, $v in $ease  { --ease-#{$k}: #{$v}; }
}
```

### Mixins
```scss
@mixin flex-center($gap: 0) {
  display: flex;
  align-items: center;
  justify-content: center;
  @if $gap != 0 { gap: $gap; }
}

@mixin gradient-text($from, $to) {
  background: linear-gradient(135deg, $from, $to);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
}

@mixin line-clamp($lines: 2) {
  display: -webkit-box;
  -webkit-line-clamp: $lines;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

// visually hidden but available to screen readers
@mixin sr-only {
  position: absolute;
  width: 1px; height: 1px;
  padding: 0; margin: -1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
  border: 0;
}

@mixin focus-ring($color: var(--accent), $width: 2px, $offset: 2px) {
  &:focus-visible {
    outline: $width solid $color;
    outline-offset: $offset;
  }
}

// elevation: 0..4 → progressively stronger surface + shadow
$elevations: (
  0: (bg: var(--bg),      shadow: none),
  1: (bg: var(--surface), shadow: 0 1px 2px rgb(0 0 0 / 0.2)),
  2: (bg: var(--surface), shadow: 0 4px 12px rgb(0 0 0 / 0.28)),
  3: (bg: var(--surface), shadow: 0 12px 32px rgb(0 0 0 / 0.34)),
);
@mixin surface($level: 1) {
  $e: map.get($elevations, $level);
  background: map.get($e, bg);
  box-shadow: map.get($e, shadow);
}

@mixin glow-border($color: var(--accent), $size: 1px) {
  border: $size solid rgb(var(--c-accent) / 0.4);
  box-shadow: 0 0 0 $size rgb(var(--c-accent) / 0.15),
              0 0 24px rgb(var(--c-accent) / 0.25);
}

// transition helper — pass any number of properties via @each
@mixin transition($props...) {
  $list: ();
  @each $p in $props {
    $list: append($list, #{$p} var(--dur-base) var(--ease-out), comma);
  }
  transition: $list;
}
```

```scss
@use "sass:map";
.btn   { @include transition(background, transform, box-shadow); }
.title { @include gradient-text(var(--accent), #a78bfa); }
.toast { @include surface(2); }
```

### Breakpoint mixins
One `$breakpoints` map, three intents: up, down, between.

```scss
$breakpoints: (sm: 480px, md: 768px, lg: 1024px, xl: 1280px);

@mixin bp-up($name) {
  @media (min-width: map.get($breakpoints, $name)) { @content; }
}
@mixin bp-down($name) {
  @media (max-width: map.get($breakpoints, $name) - 0.02px) { @content; }
}
@mixin bp-between($from, $to) {
  @media (min-width: map.get($breakpoints, $from))
     and (max-width: map.get($breakpoints, $to) - 0.02px) { @content; }
}

.sidebar {
  width: 100%;
  @include bp-up(lg) { width: 280px; }
}
```

### Placeholders & nesting
Placeholders (`%name`) share rules with zero output until `@extend`ed. Nest with `&__element` / `&.is-state`, and **keep nesting shallow** — 2–3 levels max.

```scss
%card-base {
  border-radius: var(--radius);
  padding: var(--space-4);
  @include surface(1);
}

.panel {
  @extend %card-base;

  &__header { font-weight: 600; }     // .panel__header
  &__body   { color: var(--muted); }  // .panel__body

  &.is-active { @include glow-border; } // .panel.is-active
  &:hover     { transform: translateY(-2px); }
}
```

## Animations & keyframes
Define once, reuse via `animation`. **Every animated component is wrapped by the reduced-motion guard** (see Accessibility).

```css
@keyframes shimmer   { 100% { background-position: 200% 0; } }
@keyframes pulse     { 0%,100% { opacity: 1; } 50% { opacity: 0.4; } }
@keyframes blink     { 0%,100% { opacity: 1; } 50% { opacity: 0; } }
@keyframes dot-pulse { 0%,100% { transform: scale(1); opacity: 0.5; }
                       50%     { transform: scale(1.4); opacity: 1; } }
@keyframes fade-in   { from { opacity: 0; } to { opacity: 1; } }
@keyframes message-in{ from { opacity: 0; transform: translateY(8px); }
                       to   { opacity: 1; transform: translateY(0); } }
@keyframes card-rise { from { opacity: 0; transform: translateY(12px) scale(0.98); }
                       to   { opacity: 1; transform: translateY(0) scale(1); } }
@keyframes spin      { to { transform: rotate(360deg); } }
@keyframes float     { 0%,100% { transform: translateY(0); }
                       50%     { transform: translateY(-6px); } }
```

```css
/* loading skeleton */
.skeleton {
  background: linear-gradient(90deg,
    rgb(255 255 255 / 0.04) 25%,
    rgb(255 255 255 / 0.10) 37%,
    rgb(255 255 255 / 0.04) 63%);
  background-size: 200% 100%;
  animation: shimmer 1.4s infinite linear;
}

/* typing caret — steps() makes it snap, not fade */
.caret { animation: blink 1s steps(1, end) infinite; }

/* "agent is thinking" dots */
.tool-dot { animation: dot-pulse 1.2s ease-in-out infinite; }
.tool-dot:nth-child(2) { animation-delay: 0.15s; }
.tool-dot:nth-child(3) { animation-delay: 0.30s; }

.message  { animation: message-in var(--dur-base) var(--ease-out) both; }
.card     { animation: card-rise var(--dur-slow) var(--ease-spring) both; }
.spinner  { animation: spin 0.7s linear infinite; }
.badge    { animation: float 3s ease-in-out infinite; }
```

## Gradients
```css
/* linear, multi-stop */
.hero { background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 50%, #ec4899 100%); }

/* radial glow centered, and off-center blobs layered over a base color */
.scene {
  background:
    radial-gradient(40% 60% at 20% 0%,  rgb(99 102 241 / 0.30), transparent 70%),
    radial-gradient(50% 50% at 90% 20%, rgb(236 72 153 / 0.22), transparent 70%),
    rgb(var(--c-bg));
}

/* gradient text */
.title {
  background: linear-gradient(135deg, #818cf8, #c084fc);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
}

/* animated gradient — oversize the background, slide its position */
.animated-bg {
  background: linear-gradient(120deg, #6366f1, #8b5cf6, #ec4899, #6366f1);
  background-size: 300% 300%;
  animation: gradient-pan 8s ease infinite;
}
@keyframes gradient-pan {
  0%   { background-position: 0% 50%; }
  50%  { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}
```

A conic gradient (`conic-gradient(from 0deg, ...)`) is the basis for pie/ring charts and rotating gradient borders.

## Modern CSS
The 2025 baseline — reach for these before JS or extra markup.

### Fluid type & spacing with `clamp()` / `min()` / `max()`
```css
:root {
  --step-0: clamp(1rem, 0.9rem + 0.5vw, 1.125rem);   /* body */
  --step-2: clamp(1.5rem, 1.2rem + 1.5vw, 2.25rem);  /* heading */
}
h1 { font-size: var(--step-2); }
.container { width: min(100% - 2rem, 72rem); margin-inline: auto; } /* gutter + cap */
.gap { gap: max(1rem, 2vw); }
```

### CSS Grid
```css
/* 12-column track */
.grid12 { display: grid; grid-template-columns: repeat(12, 1fr); gap: var(--space-4); }

/* responsive card grid, no media queries */
.cards { display: grid; gap: var(--space-4);
         grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); }

/* app shell: sidebar + content, full height */
.app {
  display: grid;
  grid-template:
    "side header"  auto
    "side main"    1fr / var(--side-w, 280px) 1fr;
  min-height: 100dvh;
}
.app > .sidebar { grid-area: side; }
.app > .topbar  { grid-area: header; }
.app > .content { grid-area: main; }
```

`subgrid` (`grid-template-columns: subgrid`) lets a child align to its parent's tracks — perfect for cards whose rows must line up across the grid.

### Container queries
Style by *parent* width, not viewport — components become truly portable.

```css
.card { container-type: inline-size; container-name: card; }

@container card (min-width: 380px) {
  .card__layout { grid-template-columns: 96px 1fr; }
}
```

### `:has()` — the parent selector
```css
.field:has(input:invalid)  { --ring: #ef4444; }
.card:has(img)             { padding-top: 0; }
.form:has(:focus-visible)  { box-shadow: 0 0 0 1px var(--accent); }
```

### Glassmorphism with `backdrop-filter`
```css
.glass {
  background: rgb(255 255 255 / 0.06);
  backdrop-filter: blur(12px) saturate(140%);
  border: 1px solid rgb(255 255 255 / 0.12);
  border-radius: var(--radius);
}
```

### `aspect-ratio`, containment, `color-mix()`
```css
.thumb { aspect-ratio: 16 / 9; object-fit: cover; }

/* isolate layout/paint cost of an off-screen-heavy widget */
.feed-item { contain: layout style; }

/* mix without preprocessor math, at runtime */
.btn:hover { background: color-mix(in srgb, var(--accent) 85%, white); }
.muted-accent { color: color-mix(in srgb, var(--accent), transparent 60%); }
```

## Effects
```css
/* elevation system as utilities */
.e1 { box-shadow: 0 1px 2px rgb(0 0 0 / 0.2); }
.e2 { box-shadow: 0 4px 12px rgb(0 0 0 / 0.28); }
.e3 { box-shadow: 0 12px 32px rgb(0 0 0 / 0.34); }

/* glow */
.glow { box-shadow: 0 0 20px rgb(99 102 241 / 0.45); }

/* gradient border via padding-box / border-box trick */
.gradient-border {
  border: 1px solid transparent;
  background:
    linear-gradient(var(--surface), var(--surface)) padding-box,
    linear-gradient(135deg, #6366f1, #ec4899) border-box;
  border-radius: var(--radius);
}

/* neon text */
.neon {
  color: #fff;
  text-shadow: 0 0 4px #818cf8, 0 0 12px #6366f1, 0 0 28px #6366f1;
}
```

## Accessibility
Non-negotiables. The reduced-motion guard is global and goes near the top of your stylesheet.

```css
/* honor "reduce motion" everywhere at once */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* keyboard focus only — not on mouse click */
:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
:focus:not(:focus-visible) { outline: none; }

/* branded selection */
::selection { background: rgb(var(--c-accent) / 0.35); color: var(--text); }
```

## Cross-browser scrollbars
Firefox uses `scrollbar-*`; WebKit/Blink use `::-webkit-scrollbar`. Cover both, and stop scroll chaining with `overscroll-behavior`.

```scss
@mixin custom-scrollbar($thumb: rgb(255 255 255 / 0.2), $track: transparent) {
  scrollbar-width: thin;                 // Firefox
  scrollbar-color: $thumb $track;        // Firefox: thumb track
  overscroll-behavior: contain;          // no scroll chaining to parent

  &::-webkit-scrollbar { width: 10px; height: 10px; }
  &::-webkit-scrollbar-track { background: $track; }
  &::-webkit-scrollbar-thumb {
    background: $thumb;
    border-radius: 999px;
    border: 2px solid transparent;
    background-clip: content-box;
  }
  &::-webkit-scrollbar-thumb:hover { background: rgb(255 255 255 / 0.32); }
}

.scroll-area { @include custom-scrollbar; overflow-y: auto; }
```

## Component patterns
Where the mixins pay off.

### Button system
```scss
@mixin btn-base {
  @include flex-center(0.5rem);
  @include transition(background, transform, box-shadow);
  @include focus-ring;
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius);
  font-weight: 600;
  cursor: pointer;
  user-select: none;
  &:active { transform: translateY(1px); }
  &:disabled { opacity: 0.5; pointer-events: none; }
}

.btn {
  &--primary {
    @include btn-base;
    background: var(--accent);
    color: #fff;
    &:hover { background: color-mix(in srgb, var(--accent) 88%, white); }
  }
  &--ghost {
    @include btn-base;
    background: transparent;
    color: var(--text);
    border: 1px solid rgb(255 255 255 / 0.12);
    &:hover { background: rgb(255 255 255 / 0.06); }
  }
}
```

### Responsive sidebar via CSS variables
Drive the layout width with a custom property so a collapse toggle is a single var change — grid handles the rest.

```css
.app {
  --side-w: 280px;
  display: grid;
  grid-template-columns: var(--side-w) 1fr;
  transition: grid-template-columns var(--dur-base) var(--ease-out);
}
.app.is-collapsed { --side-w: 64px; }

@media (max-width: 768px) {
  .app { --side-w: 0px; grid-template-columns: 1fr; } /* sidebar overlays instead */
}
```

---
**See also:** [theming-dark-mode.md](theming-dark-mode.md) · [i18n.md](i18n.md) · [../architecture/solid-srp.md](../architecture/solid-srp.md)
