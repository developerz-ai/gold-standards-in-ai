# 📲 PWA, Responsive & Offline-First

Build web apps that install, adapt to any screen, stay smooth under load, and keep working with no network — **where it makes sense**. Not every app needs all four; reach for each when the use case calls for it.

## Progressive Web App (PWA)
A PWA is a web app that installs to the home screen and runs full-screen, backed by a **service worker** + a **web app manifest**.

```json
// manifest.webmanifest
{
  "name": "Product",
  "short_name": "Product",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#0f1115",
  "theme_color": "#0f1115",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any maskable" }
  ]
}
```
```html
<link rel="manifest" href="/manifest.webmanifest">
<meta name="theme-color" content="#0f1115">
```
- Generate the SW with **Vite PWA / Workbox** rather than hand-rolling cache logic.
- Provide an **install prompt** (capture `beforeinstallprompt`, show your own button).
- `theme_color` should track the [theme](theming-dark-mode.md).

## Responsive — mobile-first, fluid
Design for the smallest screen first, scale up. Prefer **intrinsic** responsiveness (the layout adapts itself) over a pile of breakpoints.

| Tool | Use |
|---|---|
| `clamp()` / `min()` / `max()` | fluid type & spacing without breakpoints |
| CSS Grid `auto-fit` + `minmax()` | columns that reflow to fit |
| **Container queries** | components that adapt to *their* space, not the viewport |
| `dvh`/`svh` units | correct mobile viewport height (no URL-bar jump) |
| `prefers-reduced-motion` | respect motion settings |

```css
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr)); gap: 1rem; }
@container card (max-width: 28rem) { .card { flex-direction: column; } }
```
Full toolbox + tokens: [css-scss-craft.md](css-scss-craft.md). Test layouts visually with [Playwright MCP](README.md#-let-the-agent-see-the-ui--playwright-mcp-headless) at multiple viewport sizes.

## Web Workers — keep the main thread free
Offload heavy work (parsing, crypto, image processing, big diffs, search indexing) to a **Web Worker** so the UI never janks.

```ts
// heavy.worker.ts
self.onmessage = (e) => { self.postMessage(crunch(e.data)); };

// caller — or wrap with Comlink for an async-function feel
const worker = new Worker(new URL("./heavy.worker.ts", import.meta.url), { type: "module" });
worker.postMessage(payload);
worker.onmessage = (e) => setResult(e.data);
```
- **Rule:** anything that could block paint for >50ms goes in a worker.
- **Comlink** turns a worker into ergonomic `await`-able calls.
- The PWA **service worker** is a different thing (network/cache proxy) from a **web/dedicated worker** (compute) — use both for their jobs.

## Offline-first (where possible)
Treat the network as an enhancement, not a requirement — for apps people use on the move or on flaky connections.

- **Cache strategies** (Workbox): app shell = *cache-first*; API GETs = *stale-while-revalidate*; mutations = *network-first* with a queue.
- **Local store:** IndexedDB (via `idb`) for structured data; the Cache API for assets/responses.
- **Optimistic UI:** apply the change locally, queue the write, reconcile on reconnect.
- **Background Sync:** register a sync so queued mutations flush when connectivity returns (fall back to a retry-on-reconnect loop where unsupported).
- **Conflict policy:** decide last-write-wins vs. merge per entity *before* you ship sync.
- **Signal state:** show online/offline + "pending changes" so the user trusts it.

```ts
// queue a mutation while offline, replay on reconnect
await db.add("outbox", { url, method, body, ts: Date.now() });
addEventListener("online", flushOutbox);
```

## When to use what
| Need | Reach for |
|---|---|
| Installable, app-like, push | PWA (manifest + service worker) |
| Works on any screen | mobile-first + fluid CSS + container queries |
| Heavy compute without jank | Web Worker (+ Comlink) |
| Usable on flaky/no network | offline-first (Workbox + IndexedDB + sync) |

Don't gold-plate: a simple internal dashboard rarely needs offline sync. Match the effort to how users actually run the app.
