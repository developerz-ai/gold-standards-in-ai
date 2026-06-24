# ⚡ Frontend — SolidJS + Bun

SolidJS is the default web UI: React-like ergonomics, fine-grained reactivity, tiny bundles, fast HMR. Build with Bun.

## Pick the right Solid flavor
| App type | Tooling |
|---|---|
| SPA (dashboard, wallet, admin) | Solid + **Vite** + `@solidjs/router` |
| SSR / SEO (marketing, content) | **SolidStart** on **Vinxi** |
| Browser extension | **WXT** (`@wxt-dev/module-solid`) |

Deps: `solid-js`, `@solidjs/router`, `@tanstack/solid-query` (server state), `tailwindcss`.

## App structure (SPA)
```
apps/dashboard/src/
├── main.tsx            # mount
├── App.tsx            # root + router
├── lib/
│   ├── routes.ts      # lazy route table
│   └── query.ts       # TanStack Query client
├── routes/            # one file per route
├── components/        # shared UI
├── i18n.ts            # → ../frontend-craft/i18n.md
└── styles.css
```

## Lazy route table — keep initial JS small
```tsx
// lib/routes.ts
export const ROUTE_TABLE = [
  { path: "/",      component: lazy(() => import("../routes/Onboarding")) },
  { path: "/repos", component: lazy(() => import("../routes/Repos")) },
];

// App.tsx
export function App() {
  return (
    <Router>
      {ROUTE_TABLE.map((r) => <Route path={r.path} component={r.component} />)}
    </Router>
  );
}
```

## SSR with SolidStart
```json
{ "scripts": { "dev": "vinxi dev", "build": "vinxi build", "start": "vinxi start" },
  "dependencies": { "@solidjs/start": "^1", "@solidjs/router": "^0.15", "vinxi": "^0.4", "solid-js": "^1.9" } }
```

## Extension with WXT
```ts
// wxt.config.ts
export default defineConfig({
  srcDir: "src",
  modules: ["@wxt-dev/module-solid"],
  manifest: {
    side_panel: { default_path: "sidepanel/index.html" },
    permissions: ["sidePanel", "storage", "scripting", "activeTab", "tabs"],
  },
  vite: () => ({ build: { target: "esnext", minify: "esbuild" } }),
});
```

## Conventions
- **Server state** via TanStack Query — never hand-roll fetch caches.
- **Styling** via Tailwind + semantic CSS variables → [../frontend-craft/theming-dark-mode.md](../frontend-craft/theming-dark-mode.md).
- **Components are SRP** — one component, one job; lift shared ones to `packages/ui`.
- **Test** with `@solidjs/testing-library` + `bun test` → [../architecture/testing.md](../architecture/testing.md).
- **i18n / dates / money** are cross-cutting → [../frontend-craft/](../frontend-craft/README.md).
