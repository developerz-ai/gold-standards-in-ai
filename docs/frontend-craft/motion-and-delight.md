# ✨ Motion & Delight

**Motion carries meaning or it gets deleted.** Every animation is feedback, orientation, continuity or personality. Each app picks its own motion signature, so two apps built from these standards never move the same way.

| Need | Go to |
|---|---|
| Duration table basics, one-authored-moment rule, the three dials | [design-taste.md](design-taste.md) |
| Motion tokens, keyframe library, global reduced-motion guard | [css-scss-craft.md](css-scss-craft.md) |
| Picking the visual direction the motion must match | [design-directions.md](design-directions.md) |
| Words on success/empty/error states | [ux-copy.md](ux-copy.md) |
| Touch vs pointer, input-adaptive behavior | [adaptive-ui.md](adaptive-ui.md) |
| Brand voice that delight must express | [brand-identity.md](brand-identity.md) |
| Recording the motion signature | [design-md.md](design-md.md) |

## 🎯 Four jobs of motion
| Job | Question it answers | Example | Budget |
|---|---|---|---|
| **Feedback** | "Did it hear me?" | Press scale, toggle knob, copy ✓ | 80–150ms, always on |
| **Orientation** | "Where did that come from / go?" | Drawer slides from its edge, toast from its corner | 200–350ms |
| **Continuity** | "Is this the same thing?" | List row morphs into detail view | 250–450ms |
| **Personality** | "Whose product is this?" | The one signature entrance, a completion moment | 500–900ms, rare |

- No job → no animation. "Looks cool" is not a job.
- Write the **motion thesis** before code: focal moment · continuity points · feedback points · perf budget.
- Focal moment comes from the product's own mechanism (a ledger balancing, a route drawing, a note folding). Generic fade-and-rise is not a thesis.

## ✅ Animate vs ❌ never
| Animate | Never animate |
|---|---|
| State changes the user caused | Static info that nobody touched |
| Elements entering/leaving the flow | Every section on scroll with the same fade-up |
| Spatial relationships (origin → destination) | Text people are trying to read (no typewriter on body copy) |
| Progress that is real | Fake progress, staged delays before success |
| Reorder, filter, expand/collapse | `width/height/top/left/margin` directly (use FLIP/transform) |
| One authored moment per surface | Infinite loops on non-live data |
| Live status (real-time dot, recording pulse) | Images on hover (animate the container) |
| Hover on pointer devices | Page load choreography on Operate surfaces (dashboards, forms) |

- **Content visible by default.** Animate *from* a visible state; a failed script never hides the page.
- **Interruptible.** A second click mid-animation reverses from the current value. CSS transitions do this for free; keyframes don't.
- **Frequency kills.** An action done 100×/day gets ≤150ms or no motion. A once-per-onboarding moment may take 800ms.

## ⏱️ Duration — scale with distance, size and frequency
| Situation | Duration |
|---|---|
| Press/active, checkbox tick, color hover | 80–150ms |
| Tooltip, dropdown, small popover | 120–200ms in · 80–120ms out |
| Toast, drawer, modal | 200–300ms in · 150–200ms out |
| Page/view transition, shared element | 250–450ms |
| Full-screen takeover, focal entrance | 500–900ms |
| Loop (pulse, shimmer) | 1.2–2s per cycle |

- **Exit ≈ 60–75% of entrance.** Leaving things should get out of the way.
- **Bigger travel → longer**, but not linearly. 2× distance ≈ 1.3× time.
- **Mobile ~20% shorter** than desktop for the same element: shorter travel, impatient thumbs.
- Feedback >200ms reads as latency, not polish.

## 📈 Easing
| Curve | Value | Use |
|---|---|---|
| ease-out-quart | `cubic-bezier(0.25, 1, 0.5, 1)` | Default arrival, calm |
| ease-out-expo | `cubic-bezier(0.16, 1, 0.3, 1)` | Confident arrival, fast start, long settle |
| ease-out-swift | `cubic-bezier(0.32, 0.72, 0, 1)` | Sheet/drawer, native-feeling |
| ease-in-quart | `cubic-bezier(0.5, 0, 0.75, 0)` | Exits only (accelerate away) |
| ease-in-out-cubic | `cubic-bezier(0.65, 0, 0.35, 1)` | Things moving *across* the screen, both ends visible |
| back-out (overshoot) | `cubic-bezier(0.34, 1.56, 0.64, 1)` | Playful worlds only: badges, checkmarks |
| `steps(n)` | `steps(1, end)` | Caret blink, sprite, terminal/brutalist hard cuts |
| `linear` | `linear` | Spinners, progress bars, scroll-linked, marquees. Never UI state |

- **Arrivals decelerate. Exits accelerate. Cross-screen moves ease both ends.**
- Default `ease` / `ease-in-out` keywords read as generic. Pick a named token.
- One project = 2–3 easing tokens max. More curves = no signature.

## 🌀 Spring vs bezier
| Use a spring when | Use a bezier when |
|---|---|
| Motion is interruptible by gesture (drag, swipe, fling) | Fixed-duration state change |
| Velocity must carry over (throw a card) | You need exact choreography timings |
| Physical metaphor is the brand (playful, soft premium) | Minimal utility, terminal, civic |

- **Spring params:** stiffness = snap, damping = settle, mass = weight. `stiffness 100 / damping 20` = weighty premium. `stiffness 400 / damping 30` = snappy UI. Damping ratio < 0.7 → visible bounce; keep bounce for playful worlds.
- **CSS springs without JS:** `linear()` approximates any spring curve (baseline in all engines as of 2026-09). Generate points with a spring→`linear()` tool; commit as a token.

```css
:root {
  /* approximate gentle spring, slight overshoot; tune per brand */
  --ease-spring-soft: linear(0, 0.25 8%, 0.6 18%, 0.9 30%, 1.03 42%, 1.01 60%, 1);
  --dur-spring: 520ms; /* springs need the tail; don't cut to 200ms */
}
.sheet[open] { transition: translate var(--dur-spring) var(--ease-spring-soft); }
```

## 🎼 Choreography & stagger
- **Stagger only real lists** (results, cards in a grid, menu items). Never whole page sections.
- Per-item delay **30–60ms**. Cap staggered items at ~6–8; the rest arrive with the last. **Total sequence ≤ 400ms.**
- **Order = reading order = importance.** Headline → supporting → action. Action never arrives last on a busy screen.
- **Overlap, don't queue.** Next element starts at ~40–60% of the previous one.
- **Exits don't stagger.** Everything leaves together, fast.
- **One direction of travel per surface.** Mixed up/left/scale entrances read as noise.
- Parent/child: container arrives first (or not at all), children settle inside it.

```css
.results > li {
  animation: rise 360ms var(--ease-out) both;
  animation-delay: calc(min(var(--i), 7) * 45ms); /* cap at 8 items */
}
@keyframes rise { from { opacity: 0; translate: 0 8px; } }
```
```tsx
<For each={items()}>{(it, i) => <li style={{ "--i": i() }}>{it.name}</li>}</For>
```

## 🧭 Modern CSS primitives (as of 2026-09)
| Primitive | What it replaces | Support |
|---|---|---|
| `@starting-style` + `transition-behavior: allow-discrete` | JS mount timers for enter/exit of `display:none`, `<dialog>`, popover | All engines |
| Same-document View Transitions (`document.startViewTransition`) | FLIP libraries for state morphs | All engines |
| Cross-document View Transitions (`@view-transition { navigation: auto; }`) | SPA-only page morphs | Chromium, Safari; not Firefox |
| Scroll-driven (`animation-timeline: scroll()` / `view()`) | Scroll listeners, IntersectionObserver reveals | Chromium, Safari; Firefox behind flag |
| `@property` typed custom props | JS tweening of gradients/angles | All engines |
| `interpolate-size: allow-keywords` | Measured `height: auto` hacks | Chromium only → fallback `grid-template-rows: 0fr → 1fr` |
| `linear()` | JS spring runtimes for simple cases | All engines |

**Enter/exit for popovers & dialogs, no JS:**
```css
.pop {
  opacity: 1; scale: 1;
  transition: opacity 160ms var(--ease-out), scale 160ms var(--ease-out),
              display 160ms allow-discrete, overlay 160ms allow-discrete;
  @starting-style { opacity: 0; scale: 0.96; }
}
.pop:not(:popover-open) { opacity: 0; scale: 0.98; transition-duration: 100ms; }
```

**Shared-element morph (list row → detail):**
```css
.row-thumb   { view-transition-name: var(--vt); }  /* unique per item */
.detail-hero { view-transition-name: var(--vt); }
::view-transition-group(*) { animation-duration: 320ms; animation-timing-function: var(--ease-out); }
```
```ts
const go = (update: () => void) =>
  document.startViewTransition ? document.startViewTransition(update) : update();
// SolidJS: go(() => setSelected(id)) — Solid updates DOM synchronously inside the callback
```

**Scroll-linked, progressively enhanced:**
```css
@supports (animation-timeline: view()) {
  @media (prefers-reduced-motion: no-preference) {
    .chapter-img { animation: settle linear both; animation-timeline: view(); animation-range: entry 0% cover 40%; }
  }
}
@keyframes settle { from { scale: 0.92; opacity: 0.4; } }
.read-progress { transform-origin: 0 50%; animation: grow linear both; animation-timeline: scroll(root); }
@keyframes grow { from { scale: 0 1; } }
```
- Scroll-driven only when scroll position *means* something (progress, narrative chapter). Never a raw `scroll` listener, never scroll position in reactive state.
- Scroll-hijack (pinned, horizontal pan) only on Persuade/Experience surfaces with a narrative. Pin at `top top`, not halfway.

## 🔘 Micro-interaction catalog
| Element | Default treatment | Notes |
|---|---|---|
| Button press | `scale: 0.97` or `translate: 0 1px`, 80–100ms | Pick one per product; apply everywhere |
| Button hover | Color/background shift 150ms | Lift/translate only if the direction allows it |
| Toggle | Knob slides with ease-out 150ms, track color crossfades | Knob may squash-stretch in playful worlds |
| Checkbox | Check path draws (`stroke-dashoffset`) 150–200ms | Overshoot only if playful |
| Copy to clipboard | Icon swaps to ✓ + label "Copied", 1.5–2s, then reverts | Announce via `aria-live="polite"` |
| Input focus | Border/ring color 120ms, no size change | Never shift layout on focus |
| Validation error | Message fades in under field; optional 1 small shake (≤ 4px, 250ms) | Shake never on every keystroke |
| Success (routine save) | Inline ✓ or "Saved" fade, no toast | Certainty, not celebration |
| Tab / segmented control | Indicator slides between tabs (shared element) | Great VT use-case |
| Accordion | Grid-rows 0fr→1fr, chevron rotates 180° | 200–300ms |
| Toast | Enter from its anchor edge, exit faster, pause timer on hover/focus | Stack with slight offset |
| Drag | Lift (shadow + scale 1.02), others FLIP out of the way | Spring on drop |
| Number change | Tabular numerals; roll/crossfade digits ≤ 300ms | Never animate money on every tick |

**Copy button (SolidJS):**
```tsx
function CopyButton(props: { text: string }) {
  const [done, setDone] = createSignal(false);
  let t: number | undefined;
  const copy = async () => {
    await navigator.clipboard.writeText(props.text);
    setDone(true); clearTimeout(t); t = window.setTimeout(() => setDone(false), 1800);
  };
  onCleanup(() => clearTimeout(t));
  return (
    <button class="copy" data-done={done()} onClick={copy}>
      <span aria-live="polite">{done() ? "Copied" : "Copy"}</span>
    </button>
  );
}
```
```css
.copy[data-done="true"] { color: var(--success); }
.copy span { transition: opacity 120ms var(--ease-out); }
```

## ⏳ Loading, skeletons, optimistic UI
| Wait | Show |
|---|---|
| < 100ms | Nothing. Instant |
| 100ms–1s | Nothing for first ~300ms, then subtle inline indicator (button spinner, dimmed row) |
| 1–10s | Skeleton matching final layout; or streamed content as it arrives |
| > 10s | Real progress (steps or %), time estimate, cancel. Let user leave and come back |

- **Delay the indicator ~300ms**, then keep it ≥ 400ms once shown. No flash-of-spinner.
- **Skeleton = the real layout** in grey: same heights, same grid. Zero CLS on swap. Shimmer is optional; a static skeleton is calmer (Minimal utility, Civic).
- **Progress never lies.** No fake 0→90% crawl. Truthful steps ("Uploading · Checking · Done") beat a spinner.
- **Optimistic by default** for reversible, high-success actions (like, toggle, reorder, rename). Pessimistic for money, deletes without undo, anything irreversible.
- **Undo > confirm.** Remove the row instantly with a 5s "Undo" toast instead of a confirm dialog.

```tsx
const [liked, setLiked] = createSignal(props.initial);
const toggle = async () => {
  const prev = liked();
  setLiked(!prev);                       // instant UI, INP stays low
  try { await api.like(props.id, !prev); }
  catch { setLiked(prev); toast.error("Couldn't save. Try again."); } // rollback visibly
};
```

## 🎉 Delight — restraint rules
**Delight = product character revealed at a moment that earned it.** Not a whimsy layer.

| Moment | Earned response | Never |
|---|---|---|
| Routine save/click | Certain, quiet feedback | Confetti, sound, toast spam |
| Real milestone (first project shipped, inbox zero, streak) | Expanded moment, 1–2s, skippable | Blocking modal on every repeat |
| Empty / first use | Clear next action first, then voice (illustration, one line of product-specific copy) | Cute art with no CTA |
| Waiting | Truthful progress + useful context | Rotating jokes that hide slowness |
| Error / recovery | Problem + fix first; warmth optional | Jokes near money, privacy, lost work |
| Discovery (shortcuts, easter eggs) | Rewards curiosity, reveals real utility | Hiding required features |

- **Intensity ∝ consequence ÷ frequency.** Celebrate the rare and meaningful; make the frequent feel certain.
- **Survives the 100th use.** If it would annoy on repeat, show it once (persist a flag) or not at all.
- **Specific enough that a neighboring product couldn't reuse it.** Confetti is generic; the ledger lines snapping to balance is yours.
- Sound only with opt-in, respects mute. Haptics only on touch, only for confirmations.
- Easter eggs: discoverable, harmless, never in the critical path, never on regulated/civic surfaces.
- Delight never delays the task. Confetti renders *after* the success state is already visible.

## 🎚️ Bolder & quieter — turning the dial
Scope is sovereign: dial the **named target only**. Reuse the system's own vocabulary; add no new colors, fonts or primitives unasked.

| Lever | Bolder (timid UI) | Quieter (loud UI) |
|---|---|---|
| Hierarchy | One element at full strength; quiet its neighbors | Keep 1–2 anchors, recede the rest |
| Type | Display face at full scale, bigger jumps | Weights down one step (900→600, 700→500), smaller jumps |
| Color | Commit the accent to a large surface once | Saturation to 70–85%, neutrals dominate, accent ≤ 10% |
| Motion distance | 24–48px travel, clip-path/mask reveals | 8–16px travel or opacity only |
| Motion curves | Expo/spring, one authored focal sequence | ease-out-quart, no overshoot, no bounce |
| Motion count | Add *one* signature move, not ten | Remove decorative loops; keep feedback |
| Effects | Section becomes a pace change (density/rhythm peak) | Drop glows, stacked shadows, blur, gradients |
| Space | Tighter where energy needed, bigger contrast | More air, even rhythm, align rogue elements |

- **Bolder ≠ more effects.** More effects make it flatter. Commit one decisive move, quiet everything around it.
- **Quieter ≠ generic.** Precision replaces volume. If it lost its point of view, you went too far.
- **Skeleton test:** strip the copy. Does structure alone still say what matters? If not, boldness lives only in font size.

## 🚀 Overdrive — ambitious showcase effects
Push past what users expect from a web page. **Context decides:** particles on a portfolio = impressive; on a settings page = embarrassing. A settings page with instant optimistic saves and morphing states *is* overdrive.

| Surface | Overdrive looks like |
|---|---|
| Marketing / portfolio | Shader background, cinematic cross-page transition, scroll narrative, kinetic type |
| Functional UI | Dialog morphing from its trigger, drag with spring physics, streaming validation |
| Performance-critical | 100k-row list at 60fps (virtualized), search that never flickers, main thread never blocks |
| Data-heavy | Canvas/WebGL charts, animated transitions between data states, force layouts that settle |

- **Propose 2–3 directions first** (technique, ambition level, perf cost, support), get a pick, then build.
- **Iterate visually** in a real browser; ambitious effects never work first try.
- **Progressive enhancement non-negotiable:** `@supports`, feature-detect WebGPU → WebGL2 → CSS fallback that still looks good.
- Lazy-init heavy runtimes (WebGL, WASM) near viewport. Pause offscreen. Test on a mid-range phone.
- **One extraordinary moment per surface.** Competing wows cancel out.
- Never use spectacle to cover weak fundamentals. Fix hierarchy/type/spacing first.
- **Removal test:** take it away. If nobody misses it, delete it.

## ⚡ Performance
| Rule | Why |
|---|---|
| Animate `transform`, `opacity` (+ `translate/scale/rotate` props) by default | Compositor-only: no layout, no paint |
| `clip-path`, `filter`, `backdrop-filter`, shadow: bounded regions only | Repaint cost scales with area |
| `will-change` added just before, removed after | Permanent layers eat GPU memory |
| Batch DOM reads then writes (FLIP: First, Last, Invert, Play) | Interleaving forces sync layout |
| Pause loops offscreen (`IntersectionObserver` → `animation-play-state: paused`) and on `visibilitychange` | Battery, CPU |
| Continuous values (pointer, scroll, drag) outside reactive state | Per-frame re-renders collapse on mobile |
| Grain/noise on a `position: fixed; pointer-events: none` layer | Never on scrolling containers |
| One animation engine per component tree | Two libs fight over frames |

- **Frame budget 16.7ms (60fps)**; ~8ms on 120Hz. Below ~50fps on a mid-range device → simplify.
- **INP < 200ms** (as of 2026-09 Core Web Vital). Visual response must start in the next frame after input; do the expensive work after paint, never before the press state renders.
- Don't `await` network before showing feedback. Feedback first, then fetch.

## ♿ Reduced motion — fewer and gentler, not none
- **Opt in, don't only opt out.** Write movement inside `@media (prefers-reduced-motion: no-preference)`. The global kill-switch in [css-scss-craft.md](css-scss-craft.md) is the safety net, not the design.
- Replace spatial travel with crossfade. Keep color/opacity feedback that confirms actions.
- Must go static: parallax, scroll-hijack, auto-playing loops, magnetic/tilt, zooms, large slides, vestibular triggers.
- Autoplaying anything > 5s needs a pause control regardless of preference.
- Offer an in-app "Reduce motion" setting that sets `data-motion="reduce"` on `<html>`, same rules.

```css
.drawer { opacity: 0; transition: opacity 160ms linear; }
.drawer[open] { opacity: 1; }
@media (prefers-reduced-motion: no-preference) {
  :root:not([data-motion="reduce"]) .drawer { translate: 100% 0; transition: translate 280ms var(--ease-out), opacity 200ms; }
  :root:not([data-motion="reduce"]) .drawer[open] { translate: 0 0; }
}
```

## 🎭 Motion personality per direction
Pick the row matching the app's direction ([design-directions.md](design-directions.md)), then **change at least one variable** from the last sibling app. Record it in `DESIGN.md` as the **motion signature**: 2 easing tokens · duration scale · press behavior · one signature move.

| Direction | Curves | Durations | Press / hover | Entrances | Signature move ideas |
|---|---|---|---|---|---|
| **Soft premium** | Soft spring (`linear()`), expo-out | Slow: 300 / 500 / 700ms | Scale 0.98, soft shadow grow | Blur-to-sharp + small rise | Card expands into sheet (shared element) |
| **Minimal utility** | quart-out only | Fast: 100 / 150 / 250ms | Background shift, no transform | None; content just there | Instant optimistic updates, sliding tab indicator |
| **Swiss / brutalist** | `steps()`, `linear` | 0ms cuts or 120ms | Invert colors, hard | Hard cut, clip-path wipe | Grid lines drawing, type slamming to baseline |
| **Terminal / telemetry** | `steps()`, linear | Instant state, 80ms | Outline/invert | Line-by-line print for logs only | Blinking caret, live counters ticking in tabular nums |
| **Editorial / magazine** | expo-out, in-out for page turns | Medium; one long 800ms | Underline draws | Mask reveal on the lead image | One scroll-driven chapter moment |
| **Playful / maximal** | back-out overshoot, bouncy spring | Medium, snappy | Squash (scale 0.94 → 1.04 → 1) | Pop + rotate a few degrees | Sticker drag, celebratory completion |
| **Civic / trust-first** | quart-out | Fast, minimal | Color only | None | Clear step progress; reduced motion by default |
| **Cold luxury** | Long ease-in-out, slow expo | Slow, precise: 600–900ms | Opacity/color only, no scale | Slow crossfade, light sweep | Product reveal under a moving light gradient (`@property`) |

- Same product family, different apps → vary the **signature move** and **curve family** at minimum.
- Real systems commit: some brands use color-only hovers and no transform ever; others make `scale(0.95)` press the one system-wide micro-interaction. Either is a signature; mixing both is none.

## 🚫 Anti-patterns
- Same fade-up on every section. Stagger on non-lists.
- Bounce/elastic by reflex outside a playful world.
- Infinite pulse/float on static cards. Second marquee on a page.
- `transition: all` (animates things you didn't intend, incl. layout).
- Scroll listener, scroll position in state, `setInterval` animations.
- Spinner flashing for 50ms. Fake progress. Delayed success for a flourish.
- Confetti for routine actions. Unskippable celebration. Sound without opt-in.
- Custom cursors, hover image trails on Operate surfaces.
- "Motion claimed, not shown": half-built scroll sequences that break. Ship static instead.

## ☑️ Checklist
- [ ] Motion thesis written: focal moment, continuity, feedback, budget.
- [ ] Every animation has one of the four jobs.
- [ ] 2–3 easing tokens, a duration scale, exits faster than entrances.
- [ ] Interruptible; repeated use still feels good.
- [ ] Compositor-only by default; 60fps on a mid-range phone; INP < 200ms.
- [ ] Reduced-motion path is designed, not just killed.
- [ ] Delight intensity matches consequence and frequency.
- [ ] Motion signature recorded in `DESIGN.md` and differs from the last sibling app.

---
Sources: [pbakaus/impeccable](https://github.com/pbakaus/impeccable) (animate, delight, overdrive, bolder, quieter) · [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (taste-skill, v1, gpt-tasteskill, soft-skill) · [voltagent/awesome-design-md](https://github.com/voltagent/awesome-design-md) · MDN: View Transitions, Scroll-driven animations, `@starting-style`, `linear()`, `prefers-reduced-motion`.
