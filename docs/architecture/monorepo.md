# 📦 Monorepo — Best Language Per Part

One repo per product. A monorepo lets each component use the **best language/runtime for its job** while sharing one `CLAUDE.md`, one CI, one set of conventions. The agent sees the whole system at once.

## Layout: `apps/` + `packages/`
```
product/
├── apps/                 # user-facing / deployable units
│   ├── api/              # Hono REST API (Bun)
│   ├── web/              # SolidJS SPA / SolidStart SSR
│   ├── dashboard/        # SolidJS SPA
│   ├── admin/            # SolidJS SPA
│   ├── worker/           # background jobs (BullMQ)
│   └── webhooks/         # Bun + Hono
├── packages/             # shared, single-responsibility
│   ├── db/               # Drizzle schema + migrations (no business logic)
│   ├── domain/           # pure types + constants (no I/O)
│   ├── auth/             # JWT, TOTP
│   ├── core/             # domain logic
│   ├── events/           # event types
│   ├── audit/            # audit logging
│   ├── i18n/             # translation utilities
│   ├── ui/               # shared Solid components
│   └── mcp/              # MCP server layer
├── sdks/                 # public client SDKs (js, go, python…)
├── bin/                  # setup, dev, check, fmt  (see DX scripts)
├── docker/               # compose + per-app Dockerfiles
└── package.json          # workspaces root
```

## Bun workspaces
```json
{
  "name": "@product/root",
  "workspaces": ["apps/*", "packages/*", "sdks/js"],
  "scripts": {
    "dev": "bun run --filter '*' dev",
    "lint": "biome check .",
    "lint:fix": "biome check --write .",
    "typecheck": "bun run --filter '*' typecheck",
    "test": "bun run --filter '*' test",
    "test:integration": "bun run --filter '*' test:integration"
  }
}
```
Run scoped commands without `cd`:
```bash
bun run --filter @product/api dev      # one app
bun run --filter '*' test              # everything
bun run --filter '@product/*' build    # a scope
```

## Mixing languages in one repo
The monorepo is *the* mechanism for "best tool per part":

| Component | Language | Why |
|---|---|---|
| Web UIs | **SolidJS + Bun** | reactive, tiny bundle, fast HMR → [stack/frontend](../stack/frontend-solidjs.md) |
| Standard APIs / workers | **Bun + Hono** | sub-100ms cold start, one language with the frontend → [stack/backend](../stack/backend-bun-hono.md) |
| Hot-path / high-throughput API | **Rust** | speed, safety, predictable latency → [stack/rust](../stack/rust-apis.md) |
| Native mobile | **Swift + Kotlin** | platform-native UX, share the backend → [stack/mobile](../stack/mobile.md) |
| Performance core | **C++** | compiled core called from the runtime |

They coexist behind one set of `bin/` scripts and one CI. A Rust crate, a Bun app, and a Swift package all live under the same root; the agent reads the root `CLAUDE.md` and knows where each lives.

## Workspace-of-repos (org scale)
Above the product level, a dev's machine holds a *workspace* of many repos:
```
workspace/
├── CLAUDE.md          # org context: stack, repo categories, cross-repo rules
├── repos.json         # manifest: name, category, url, clone?
├── <repo-1>/ … <repo-n>/
└── scripts/setup.sh   # bootstrap a new box: clone all clone:true repos
```
Categories (`core`, `infra`, `libraries`, `ai`, `tools`, `docs`) give the agent instant orientation. Every repo gets the same `.claude/` shape so one learned, all navigable. This is also why every dev runs the identical [Linux VPS](../developer-experience/dev-vps.md).

## Package rules
- **Explicit exports.** Each package declares its public API via `exports`.
- **`db` = schema only. `domain` = pure types, no I/O.** Keep the dependency graph acyclic.
- **One reason to change per package.** [→ SOLID/SRP](solid-srp.md)
