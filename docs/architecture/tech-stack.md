# 🧰 The Default Stack

Defaults, not dogma. Start here; deviate with a reason. The point of having a default is that an agent (and a human) can scaffold a new project without re-litigating every choice.

## At a glance
| Layer | Default | Notes |
|---|---|---|
| Runtime | **Bun** (1.3+) | fast, built-in test runner, single-binary compile |
| Language | **TypeScript** (strict) | no `any` |
| Frontend | **SolidJS** | + Vite (SPA) / SolidStart+Vinxi (SSR) / WXT (extension) |
| Server state | **TanStack Solid Query** | |
| Styling | **Tailwind + CSS variables** | semantic tokens, dark mode → [frontend-craft](../frontend-craft/theming-dark-mode.md) |
| Backend | **Hono** | lightweight, sub-100ms cold start |
| Hot-path API | **Rust** (axum, tokio, sqlx) | when speed/latency matters → [rust-apis](../stack/rust-apis.md) |
| ORM | **Drizzle** | type-safe, schema-first |
| DB driver | **postgres.js** | not node-postgres |
| Database | **PostgreSQL** → **YugabyteDB** at scale | distributed, Postgres-wire compatible |
| Cache / queue store | **Dragonfly** | Redis-compatible, faster |
| Jobs | **BullMQ** (TS) · **Wurk/Sidekiq** (Rails) | |
| Validation | **Zod** | runtime schema for all external data |
| Object storage | **Cloudflare R2** | S3-compatible, no egress fees |
| Lint/format | **Biome** | one Rust binary, replaces ESLint+Prettier |
| CI | **GitHub Actions on Blacksmith** | fast runners → [linting-ci](../developer-experience/linting-ci.md) |
| Hosting | **OVH & Hetzner** (VPS / dedicated) | value hardware, self-hosted → [servers](../infrastructure/servers-and-dns.md) |
| Image registry | **GHCR** (public) · **DOCR / DigitalOcean** (private) | tag `latest` + `<sha>` |
| Deploy (big) | **k3s + ArgoCD** | GitOps → [infrastructure](../infrastructure/kubernetes-gitops.md) |
| AI | **Vercel AI SDK** + **OpenRouter** / **z.ai** / **Anthropic** | → [ai-agents](../ai-agents/README.md) |
| Mobile | **Swift** (iOS) + **Kotlin** (Android) | native, shared backend → [mobile](../stack/mobile.md) |

## Why these
- **Bun + TypeScript everywhere** means the frontend and backend share one language, one toolchain, one mental model — the agent context-switches less.
- **SolidJS** gives React-like ergonomics with a fraction of the runtime; tiny bundles, fast loops.
- **Drizzle + Postgres** keeps the database type-safe end to end; YugabyteDB is the drop-in path to horizontal scale because it speaks Postgres.
- **Dragonfly** is a faster, more memory-efficient Redis you can run on one box where you'd have needed a cluster.
- **Rust** isn't the default — it's the scalpel for the 5% of the system that's latency- or throughput-critical.
- **Biome** is ~10–100× faster than ESLint+Prettier, so the lint gate never slows the [inner loop](../00-philosophy.md).

## When to reach past the default
| Need | Reach for |
|---|---|
| Sub-millisecond API latency, huge throughput | Rust |
| Distributed SQL, multi-region | YugabyteDB |
| Native mobile UX | Swift + Kotlin |
| Mature web framework w/ batteries | Rails (e.g. internal tools), still in the monorepo |
| Big multi-service prod deploy | k3s + ArgoCD; otherwise a single VPS + Docker Compose |

Each piece has its own doc under [stack/](../stack/README.md). The monorepo ([monorepo.md](monorepo.md)) is what lets you mix them freely.
