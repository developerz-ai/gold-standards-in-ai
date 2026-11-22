# 🏗️ Architecture

How we structure code so it's fast to build, easy for an agent to navigate, and safe to change.

| File | Topic |
|---|---|
| [monorepo.md](monorepo.md) | 📦 Monorepo — each part in the best language for the job |
| [tech-stack.md](tech-stack.md) | 🧰 The default stack and why each piece |
| [solid-srp.md](solid-srp.md) | 🧱 SOLID / SRP, small files, custom errors, thin layers |
| [testing.md](testing.md) | 🧪 Unit + integration testing as a first-class citizen |
| [data-and-scale.md](data-and-scale.md) | 📈 Shape now / capacity later · bounded sweeps · migrations vs backfills · dev engine ≠ prod engine |
| [api-contracts.md](api-contracts.md) | 🔌 HTTP contract — one error registry, status codes, keyset pages, Idempotency-Key, retries, versioning |
| [app-security.md](app-security.md) | 🛡️ App security — authz in the data layer (cross-tenant → 404), sessions/CSRF/CSP, R2 uploads, rate limits, log redaction, `VITE_` env |
| [abstractions-and-growth.md](abstractions-and-growth.md) | 🪜 Pre-MVP → really big · declare once, project everywhere · why a guard is not an abstraction |
| [resource-lifetimes.md](resource-lifetimes.md) | ♻️ RAII for services — `using` / `asyncDispose`, the creator releases on every exit, sweeper as backstop, bounded drain |
| [memory-and-leaks.md](memory-and-leaks.md) | 🫗 Unit-of-work vs process lifetime, cgroup caps, OOMKill evidence, shallow health checks hide leaks |
| [query-shape.md](query-shape.md) | 🔎 Query shape and N+1 — one query per page, not per row |

## The one-paragraph version
A product is a **monorepo** of `apps/*` (user-facing) and `packages/*` (shared domain + infra). Default runtime is **Bun + TypeScript**; reach for **Rust** on hot paths and **Swift/Kotlin** for native mobile. Every module has **one responsibility** (SRP), files stay **≤ 800 lines** (functions ≤ 120), errors are **custom types**, and everything is **type-safe** (TypeScript + Zod). Tests ship with every module — they're how the agent verifies itself.

Two things scale with the product rather than being decided once: **[capacity](data-and-scale.md)** (shape now, capacity later) and **[abstraction](abstractions-and-growth.md)** (duplicate freely at pre-MVP; declare-once-project-everywhere at 5+ surfaces). Both are staged, and both are wrong if you skip a rung in either direction.
