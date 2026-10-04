# ♿ Accessibility

**WCAG 2.2 AA is the floor** for every web and native surface. Accessible = works by keyboard, screen reader, zoom and reduced motion — **behaviour**, not just contrast. Native element first, ARIA last. The gate catches the mechanical part; a manual pass covers the rest.

Scattered rules live with their topic — this doc links, doesn't repeat:

| Need | Go to |
|---|---|
| Global reduced-motion guard, `:focus-visible` ring, visually-hidden mixin | [css-scss-craft.md → Accessibility](css-scss-craft.md#accessibility) |
| Designing the reduced-motion path (not just killing it) | [motion-and-delight.md](motion-and-delight.md#-reduced-motion--fewer-and-gentler-not-none) |
| Contrast ratios, roles that swap by theme | [brand-identity.md](brand-identity.md#-color-identity) · [theming-dark-mode.md](theming-dark-mode.md#accessibility) · [design-taste.md](design-taste.md) |
| Touch target sizes, hover-only traps, zoom/reflow, focus order vs visual order | [adaptive-ui.md](adaptive-ui.md#-touch-targets-and-thumb-zones) |
| Accessible names, error wording, announcing errors | [ux-copy.md](ux-copy.md#-errors) |
| A11y persona walk, audit score, native deltas | [design-review-loop.md](design-review-loop.md#-technical-audit-5-dimensions-each-04-total-20) |
| Semantic HTML for crawlers + screen readers | [seo.md](seo.md) |
| Native apps (SwiftUI / Compose) | [../stack/mobile.md](../stack/mobile.md) |

## 🎯 WCAG 2.2 — what 2.2 added at A/AA
As of 2026-10: WCAG 2.2 is the current W3C Recommendation; meeting 2.2 AA also meets 2.1 AA (what most law still cites). 4.1.1 Parsing is removed.

| SC | Level | Rule for us |
|---|---|---|
| 2.4.11 Focus Not Obscured (Minimum) | AA | Focused element never fully hidden by sticky header/footer, cookie bar, chat bubble. `scroll-padding-block` = sticky heights |
| 2.5.7 Dragging Movements | AA | Every drag (reorder, slider, map pan, kanban) has a single-pointer alternative: buttons, menu "Move to…", click-to-place |
| 2.5.8 Target Size (Minimum) | AA | ≥ 24×24 CSS px or 24px spacing. Our default is bigger → [adaptive-ui.md](adaptive-ui.md#-touch-targets-and-thumb-zones) |
| 3.2.6 Consistent Help | A | Help/contact link in the same place on every page |
| 3.3.7 Redundant Entry | A | Never ask twice in one flow: prefill or "same as billing" checkbox |
| 3.3.8 Accessible Authentication (Minimum) | AA | No memory/puzzle test to log in. Allow paste + password managers (`autocomplete="current-password"`, `one-time-code`), offer passkeys/magic link |

Visible focus itself is 2.4.7 (AA), not 2.4.11.

**3.3.8 vs [captcha.md](captcha.md):** object-recognition challenges (hCaptcha image picks) pass AA via the exception; text/math puzzles do not. Prefer invisible/passive mode, never block paste on the password or OTP field. AAA 3.3.9 drops the object-recognition exception.

## 🧱 Native first
| Want | Use | Not |
|---|---|---|
| Action | `<button type="button">` | `<div onClick>` + `role="button"` + `tabindex` + key handlers |
| Navigation | `<a href>` | button that calls `navigate()` |
| Modal | `<dialog>` + `showModal()` | div overlay + hand-rolled trap |
| Disclosure | `<details>/<summary>` or `button[aria-expanded]` | toggled div |
| Popover / menu shell | `popover` attribute + `popovertarget` | absolutely-positioned div with click-outside code |
| Form field | `<label for>` + input | placeholder as label |
| Landmarks | `<header> <nav> <main> <footer>`, one `<h1>`, no skipped levels | `<div class="nav">` |

- Lint it: Biome's `a11y` rule group at `error` (alt text, button `type`, click-without-key handlers, invalid ARIA) runs in `bin/check` ([linting-ci.md](../developer-experience/linting-ci.md)).
- Icon-only control → `aria-label` = outcome ([ux-copy.md](ux-copy.md#-microcopy-by-function)); decorative SVG → `aria-hidden="true"`.
- Color never the only signal: error = icon + text + color.

## 🔐 Dialogs: contain focus, return focus
`showModal()` makes the page behind inert, moves focus in, closes on Esc. Return focus explicitly — cheap and deterministic.
```tsx
import { createEffect, createUniqueId, type JSX } from "solid-js";

export function Modal(props: { open: boolean; onClose: () => void; title: string; children: JSX.Element }) {
  let el!: HTMLDialogElement;
  let opener: HTMLElement | null = null;
  const titleId = createUniqueId();
  createEffect(() => {
    if (props.open && !el.open) { opener = document.activeElement as HTMLElement; el.showModal(); }
    else if (!props.open && el.open) el.close();
  });
  return (
    <dialog ref={el} aria-labelledby={titleId}
      onClose={() => { props.onClose(); opener?.focus(); }}>
      <h2 id={titleId}>{props.title}</h2>
      {props.children}
      <button type="button" onClick={() => el.close()}>Close</button>
    </dialog>
  );
}
```
- Initial focus: `autofocus` on the first field, or on the **safe** action for destructive confirms ([ux-copy.md](ux-copy.md#-confirmations--destructive-actions)).
- Menus, popovers, comboboxes: Esc closes **and** focus returns to the trigger.
- Route change in the SPA: move focus to the new `<h1>` (`tabindex="-1"`) and update `document.title`.

## 📢 Live regions for async status
Region must be **in the DOM before** the text changes — mount it empty, then set text.
```tsx
const [status, setStatus] = createSignal("");
// <p role="status" class="sr-only">{status()}</p>   polite: saves, "3 results", "Copied"
// <p role="alert">{error()}</p>                      assertive: blocking errors only
await save(); setStatus("Draft saved");
```
- `role="status"` (polite) for routine; `role="alert"` only for errors that block. Toasts announce too.
- Loading: `aria-busy="true"` on the region being replaced; announce completion, not every tick.

## 📝 Form errors
```tsx
<label for="email">Email</label>
<input id="email" name="email" type="email" autocomplete="email" required
  aria-invalid={!!err()} aria-describedby={err() ? "email-err email-hint" : "email-hint"} />
<p id="email-hint">We send the receipt here.</p>
<Show when={err()}><p id="email-err">{err()}</p></Show>
```
- On submit with errors: focus the first invalid field (or an error summary listing links to each field).
- `aria-invalid` only after the user leaves the field or submits, never on first keystroke.
- `autocomplete` tokens on every personal field — feeds 3.3.7 and 3.3.8. Wording → [ux-copy.md](ux-copy.md#-forms).

## ⌨️ Keyboard paths
| Pattern | Keys |
|---|---|
| Everything interactive | Tab / Shift+Tab reach it; visible focus ring; DOM order = visual order |
| Button / link | Enter (both), Space (button) |
| Dialog, menu, popover | Esc closes, focus returns |
| Tabs, radio group, listbox, menu | Roving tabindex: one Tab stop, arrows move inside |
| Combobox | Arrows move `aria-activedescendant`, Enter selects, Esc clears/closes |
| Long page with repeated nav | "Skip to main content" link first in DOM |

- Custom composite widgets (combobox, tree, grid) → follow the WAI-ARIA Authoring Practices pattern exactly or use a headless library that does (Kobalte for Solid). Never invent key bindings.
- No keyboard trap outside a modal; no `tabindex` > 0.

## 🌀 Reduced motion
Opt in to movement under `prefers-reduced-motion: no-preference`; keep state feedback when reduced → [motion-and-delight.md](motion-and-delight.md#-reduced-motion--fewer-and-gentler-not-none). Autoplay/carousel > 5s needs pause (2.2.2).

## 📱 Web ↔ SwiftUI ↔ Compose
| Need | Web | SwiftUI | Jetpack Compose |
|---|---|---|---|
| Name | `<label>` / `aria-label` | `.accessibilityLabel("Delete draft")` | `contentDescription = "Delete draft"` |
| Extra description | `aria-describedby` | `.accessibilityHint(…)` | `semantics { stateDescription = … }` (state) |
| Role | native element / `role` | `.accessibilityAddTraits(.isButton)` | `semantics { role = Role.Button }` |
| Heading | `<h2>` | `.accessibilityAddTraits(.isHeader)` | `semantics { heading() }` |
| Hide decorative | `aria-hidden="true"` | `.accessibilityHidden(true)` | `contentDescription = null` / `clearAndSetSemantics {}` |
| Group as one | wrapping element / list item | `.accessibilityElement(children: .combine)` | `semantics(mergeDescendants = true) {}` |
| Announce change | `role="status"` | `AccessibilityNotification.Announcement("Saved").post()` | `semantics { liveRegion = LiveRegionMode.Polite }` |
| Text scaling | `rem`, 200% zoom | Dynamic Type: text styles (`.body`), `@ScaledMetric` | `sp` units, test font scale 1.3–2.0 |
| Reduced motion | `prefers-reduced-motion` | `@Environment(\.accessibilityReduceMotion)` | system "Remove animations" (animator scale 0) |
| Min target | 24 px AA, 44 px ours | 44×44 pt | 48×48 dp |

Prefer system controls (`Button`, `Toggle`, `Switch`) — they come labelled and traited. Screen readers: VoiceOver (iOS/macOS), TalkBack (Android). Font-scale capture commands → [design-review-loop.md](design-review-loop.md#-native-ios--android-deltas).

## 🧪 Testing
| Layer | What | Catches |
|---|---|---|
| Lint | Biome `a11y` group in `bin/check` | missing alt/label, div-buttons, bad ARIA |
| Locators | Playwright `getByRole` / `getByLabel` only — no CSS selectors for controls | a control with no accessible name or role **fails the test** |
| axe in CI | `@axe-core/playwright` on every key route, both themes | contrast, names, ARIA validity, landmarks |
| Manual | keyboard-only + screen reader on key journeys | focus order, announcements, trap/return, meaning |

```ts
import AxeBuilder from "@axe-core/playwright";
test("checkout is accessible", async ({ page }) => {
  await page.goto("/checkout");
  await page.getByLabel("Email").fill("ada@example.com");
  await page.getByRole("button", { name: "Pay now" }).click();
  const r = await new AxeBuilder({ page }).withTags(["wcag2a", "wcag2aa", "wcag21aa", "wcag22aa"]).analyze();
  expect(r.violations).toEqual([]);
});
```
- **Clean axe ≠ accessible.** As of 2026-10, automated rules reliably test well under half of WCAG success criteria (Deque's own audit data: ~57% of issues by *volume*, inflated by contrast). Never report "accessible" from axe alone.
- Run axe after interactions (dialog open, error state shown), not just on load. Same Playwright job as screenshots → [design-review-loop.md](design-review-loop.md#-deterministic-anti-pattern-detection-lintci-guard) · committed journeys → [testing.md → E2E](../architecture/testing.md#-e2e--committed-journeys-not-ad-hoc-clicking).
- Manual pass per release on key journeys (sign-up, login, core task, checkout): unplug the mouse; then VoiceOver/NVDA (web), VoiceOver (iOS), TalkBack (Android). The agent can drive the keyboard pass via [Playwright MCP](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless) and read the a11y tree; screen-reader listening stays human.

## ☑️ Checklist
- [ ] Native elements; Biome `a11y` + axe clean in `bin/check`/CI.
- [ ] Every control reachable and operable by keyboard; focus visible and never obscured.
- [ ] Dialogs/menus: Esc closes, focus returns to trigger.
- [ ] Async status in a pre-mounted live region; errors linked via `aria-describedby` + `aria-invalid`.
- [ ] Every drag has a click alternative; targets ≥ 24px (ours 44px).
- [ ] Login: paste allowed, password managers work, no puzzle.
- [ ] 200% zoom / font scale 2.0 without loss; reduced-motion path designed.
- [ ] Manual keyboard + screen-reader pass on key journeys, recorded in the PR.
