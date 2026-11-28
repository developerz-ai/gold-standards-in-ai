# developerz-ai/gold-standards-in-ai — docs

> Index — every section directory under `docs/` has a row here; `bin/check-docs` fails on one that is missing.

| Where | What lives there |
|---|---|
| [🧠 Philosophy](00-philosophy.md) | AI-first development and the principles behind this knowledge base. |
| [✍️ Writing for agents](writing-for-agents/README.md) | Context, tooling, behavioral rules, and verification for coding agents — incl. [prompts for weak models](writing-for-agents/prompts-for-weak-models.md). |
| [🏗️ Architecture](architecture/README.md) | Monorepo, SRP, testing, data shape, API contracts, security, [resource lifetimes](architecture/resource-lifetimes.md), [memory & leaks](architecture/memory-and-leaks.md), [query shape](architecture/query-shape.md). |
| [🧰 Stack](stack/README.md) | One doc per layer — SolidJS, Bun + Hono, Rust, databases, mobile. |
| [🛠️ Developer experience](developer-experience/README.md) | `bin/` scripts, dev VPS, the inner loop, CI/CD for agents, [cheap CI](developer-experience/cheap-cicd.md), smart builds, [dependency updates](developer-experience/dependency-updates.md). |
| [🤖 AI agents](ai-agents/README.md) | Building agents — the loop, orchestration, hive mind, tools, context budget, evals, [harness techniques](ai-agents/harness-techniques.md). |
| [☸️ Infrastructure](infrastructure/README.md) | k3s + GitOps, servers and DNS, SSO, secrets, observability. |
| [🎨 Frontend craft](frontend-craft/README.md) | i18n, theming, accessibility, design taste, motion, SEO, assets. |
| [🔄 Workflow](workflow/README.md) | Idea → plan → issues → PRs → deploy; shipping doctrine, [project from zero](workflow/project-from-zero.md), [bug sweeps](workflow/bug-sweep.md), [the `.dz/` contract](workflow/dz-contract.md), [opportunistic improvement](workflow/opportunistic-improvement.md), [the data-risk ladder](workflow/data-risk-ladder.md). |

The brain itself is the repo root's [`CLAUDE.md`](../CLAUDE.md); the
commands are `bin/{setup,dev,check,stop}`.
