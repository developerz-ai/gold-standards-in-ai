# 🌗 Theming & Dark Mode

Auto theme that follows the OS, plus an explicit user override — driven by **semantic CSS variables**, Tailwind-friendly. Define tokens once; every component reads tokens, never raw colors. Themes flip without touching a single component.

## Semantic tokens, defined once
Name tokens by **role**, not by value (`--color-bg`, not `--blue-500`). Store colors as **space-separated RGB channels** so alpha composites cleanly:
```css
:root {
  /* light is the default */
  --color-bg:            253 246 240;
  --color-bg-soft:       245 237 230;
  --color-surface:       255 255 255;
  --color-fg:             38  34  31;
  --color-fg-strong:      17  15  13;
  --color-fg-muted:      110 102  94;
  --color-line:          224 216 208;
  --color-accent:         34 122 197;
  --color-accent-strong:  21  92 152;
}

/* follow the OS when the user hasn't chosen */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg:            18  18  20;
    --color-bg-soft:       28  28  32;
    --color-surface:       34  34  39;
    --color-fg:           228 226 222;
    --color-fg-strong:    248 247 245;
    --color-fg-muted:     150 146 140;
    --color-line:          54  54  60;
    --color-accent:        96 170 240;
    --color-accent-strong:130 190 248;
  }
}

/* explicit override beats the media query */
html[data-theme="dark"] {
  --color-bg:            18  18  20;
  --color-bg-soft:       28  28  32;
  --color-surface:       34  34  39;
  --color-fg:           228 226 222;
  --color-fg-strong:    248 247 245;
  --color-fg-muted:     150 146 140;
  --color-line:          54  54  60;
  --color-accent:        96 170 240;
  --color-accent-strong:130 190 248;
}
html[data-theme="light"] {
  --color-bg:            253 246 240;
  --color-bg-soft:       245 237 230;
  --color-surface:       255 255 255;
  --color-fg:             38  34  31;
  --color-fg-strong:      17  15  13;
  --color-fg-muted:      110 102  94;
  --color-line:          224 216 208;
  --color-accent:         34 122 197;
  --color-accent-strong:  21  92 152;
}
```

Channels (not `#rrggbb`) let you do `rgb(var(--color-bg) / 0.8)` for any opacity:
```css
.toolbar {
  background: rgb(var(--color-bg) / 0.8);  /* 80% over a blur */
  color: rgb(var(--color-fg));
  border-bottom: 1px solid rgb(var(--color-line));
}
```

| Token | Role |
|---|---|
| `--color-bg` | page background |
| `--color-bg-soft` | subtle zones, hovers |
| `--color-surface` | cards, sheets, popovers |
| `--color-fg` | body text |
| `--color-fg-strong` | headings, emphasis |
| `--color-fg-muted` | captions, placeholders |
| `--color-line` | borders, dividers |
| `--color-accent` | primary action, links |
| `--color-accent-strong` | hover/active of accent |

## Tailwind mapping
Map tokens to Tailwind colors with `<alpha-value>` so `bg-bg/80`, `text-fg-muted`, `border-line` all work:
```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";

export default {
  darkMode: ["variant", [
    "@media (prefers-color-scheme: dark) { &:not([data-theme=light] *) }",
    "&:is([data-theme=dark] *)",
  ]],
  theme: {
    extend: {
      colors: {
        bg:            "rgb(var(--color-bg) / <alpha-value>)",
        "bg-soft":     "rgb(var(--color-bg-soft) / <alpha-value>)",
        surface:       "rgb(var(--color-surface) / <alpha-value>)",
        fg:            "rgb(var(--color-fg) / <alpha-value>)",
        "fg-strong":   "rgb(var(--color-fg-strong) / <alpha-value>)",
        "fg-muted":    "rgb(var(--color-fg-muted) / <alpha-value>)",
        line:          "rgb(var(--color-line) / <alpha-value>)",
        accent:        "rgb(var(--color-accent) / <alpha-value>)",
        "accent-strong":"rgb(var(--color-accent-strong) / <alpha-value>)",
      },
    },
  },
} satisfies Config;
```
The `darkMode` variant honors **both** `prefers-color-scheme` **and** a `data-theme` override: the media query applies unless `data-theme="light"` is set, and `data-theme="dark"` forces dark regardless of OS. Now `dark:` utilities are rarely needed — most components are just `bg-bg text-fg`.

```tsx
<button className="bg-accent text-bg hover:bg-accent-strong">Save</button>
<aside className="bg-surface/95 border border-line text-fg-muted">…</aside>
```

## Theme toggle (localStorage → OS → persist)
Resolve order: **explicit choice in `localStorage` → OS preference**. Apply early (in a blocking `<head>` script) to avoid a flash of the wrong theme.
```ts
type Theme = "light" | "dark";

const STORAGE_KEY = "theme";
const mql = window.matchMedia("(prefers-color-scheme: dark)");

function osTheme(): Theme {
  return mql.matches ? "dark" : "light";
}

function stored(): Theme | null {
  const v = localStorage.getItem(STORAGE_KEY);
  return v === "light" || v === "dark" ? v : null;
}

/** Reflect a resolved theme onto <html>. */
function applyTheme(theme: Theme) {
  document.documentElement.setAttribute("data-theme", theme);
}

/** Run once on load: explicit choice wins, else follow the OS. */
export function initTheme() {
  applyTheme(stored() ?? osTheme());
}

/** User picks a theme explicitly — persist it. */
export function setTheme(theme: Theme) {
  localStorage.setItem(STORAGE_KEY, theme);
  applyTheme(theme);
}

/** Forget the explicit choice and follow the OS again. */
export function clearTheme() {
  localStorage.removeItem(STORAGE_KEY);
  applyTheme(osTheme());
}

/** Auto-switch when the OS flips — only if the user hasn't chosen. */
mql.addEventListener("change", () => {
  if (!stored()) applyTheme(osTheme());
});
```

Toggle button:
```ts
export function toggleTheme() {
  const current =
    document.documentElement.getAttribute("data-theme") === "dark"
      ? "dark"
      : "light";
  setTheme(current === "dark" ? "light" : "dark");
}
```

## Accessibility
Respect motion preferences and theme `:focus-visible` / `::selection` from the same tokens:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

:focus-visible {
  outline: 2px solid rgb(var(--color-accent));
  outline-offset: 2px;
}

::selection {
  background: rgb(var(--color-accent) / 0.25);
  color: rgb(var(--color-fg-strong));
}
```

## Rules
- **Use semantic tokens everywhere** — components reference `--color-*` / `bg-bg`, never raw hex or `bg-zinc-900`.
- **Define each token once per theme** — light in `:root`, dark in the media query, both mirrored in the `data-theme` overrides.
- **A `data-theme` override beats the media query** — deterministic for screenshots and Playwright runs (`page.emulateMedia` or set `data-theme="dark"` directly).
- **Apply the theme before first paint** to kill the flash; the toggle only persists when the user explicitly chooses.
- Numbers, dates, and money are formatted per locale, not themed → [dates-money-timezones.md](dates-money-timezones.md). Strings go through the translator → [i18n.md](i18n.md).
