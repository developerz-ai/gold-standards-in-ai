# 🏗️ Architecture

How we structure code so it's fast to build, easy for an agent to navigate, and safe to change.

| File | Topic |
|---|---|
| [monorepo.md](monorepo.md) | 📦 Monorepo — each part in the best language for the job |
| [tech-stack.md](tech-stack.md) | 🧰 The default stack and why each piece |
| [solid-srp.md](solid-srp.md) | 🧱 SOLID / SRP, small files, custom errors, thin layers |
| [testing.md](testing.md) | 🧪 Unit + integration testing as a first-class citizen |
| [data-and-scale.md](data-and-scale.md) | 📈 Shape now / capacity later · bounded sweeps · migrations vs backfills · dev engine ≠ prod engine |
| [abstractions-and-growth.md](abstractions-and-growth.md) | 🪜 Pre-MVP → really big · declare once, project everywhere · why a guard is not an abstraction |

## The one-paragraph version
A product is a **monorepo** of `apps/*` (user-facing) and `packages/*` (shared domain + infra). Default runtime is **Bun + TypeScript**; reach for **Rust** on hot paths and **Swift/Kotlin** for native mobile. Every module has **one responsibility** (SRP), files stay **≤500 LOC**, errors are **custom types**, and everything is **type-safe** (TypeScript + Zod). Tests ship with every module — they're how the agent verifies itself.

Two things scale with the product rather than being decided once: **[capacity](data-and-scale.md)** (shape now, capacity later) and **[abstraction](abstractions-and-growth.md)** (duplicate freely at pre-MVP; declare-once-project-everywhere at 5+ surfaces). Both are staged, and both are wrong if you skip a rung in either direction.
