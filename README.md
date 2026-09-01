# 🏆 Gold Standards in AI

> **How we build software fast, with AI agents, at developerz-ai.**
> A knowledge base — not a codebase. Markdown only. Read it, copy the patterns, ship.

This repo is a **knowledge base for coding agents** (and the humans who run them). It distills how we build production systems end to end — the stack, the architecture, the infra, and most importantly *how to set up a project so an AI agent can do excellent work from the first prompt*.

Point any agent at this repo before it starts a new project. The more an agent knows about how we work — conventions, tooling, the exact commands, where things go — the better and faster it builds. 🚀

Think of it as **the judgment of a senior engineer with 30 years of experience who also tracks the latest tech** — distilled into docs. An agent that reads it inherits that judgment by default: it reaches for the right tool, the modern pattern, and the boring-correct choice without being told each time. 🧓⚡

> 🎯 **Who this is for:** AI-first orgs where **agents write ~100% of the code** and humans review, steer, and operate. Every standard here optimizes for that reality — make the repo so legible and so well-tooled that an agent ships production-quality work autonomously, and a human verifies it at the PR. If humans still write most of your code, much of this still helps, but the rules are tuned for full autonomy.

## 🧭 Start here

| If you want to… | Read |
|---|---|
| Understand the whole philosophy | [docs/00-philosophy.md](docs/00-philosophy.md) |
| Make a repo great for AI agents | [docs/writing-for-agents/](docs/writing-for-agents/README.md) |
| Pick the stack & structure | [docs/architecture/](docs/architecture/README.md) · [docs/stack/](docs/stack/README.md) |
| Grow a project from pre-MVP to really big | [abstractions & growth](docs/architecture/abstractions-and-growth.md) · [shape now, capacity later](docs/architecture/data-and-scale.md) |
| Get fast feedback loops (DX) | [docs/developer-experience/](docs/developer-experience/README.md) · [inner loop](docs/developer-experience/inner-loop.md) |
| Set up CI/CD for agents | [ai-first-cicd.md](docs/developer-experience/ai-first-cicd.md) |
| Build AI agents | [docs/ai-agents/](docs/ai-agents/README.md) · [freedom & access](docs/ai-agents/agent-work-limits.md) |
| Deploy & operate | [docs/infrastructure/](docs/infrastructure/README.md) |
| Polish the frontend | [docs/frontend-craft/](docs/frontend-craft/README.md) |
| Move fast on GitHub | [docs/workflow/](docs/workflow/README.md) |

## 📚 The full map

### 0. 🧠 Philosophy
- [00-philosophy.md](docs/00-philosophy.md) — AI-first development. **DX is the agent's runtime** (only AI writes the code). Freedom + access, lazy by default, proactive by default.

### 1. ✍️ Writing for agents (the most important section)
Distilled from years of running Claude Code daily. Make every repo a place an agent thrives.
- [writing-for-agents/README.md](docs/writing-for-agents/README.md) — the index
- [claude-md.md](docs/writing-for-agents/claude-md.md) — 🗂️ `CLAUDE.md`, the project brain
- [skills-commands-agents.md](docs/writing-for-agents/skills-commands-agents.md) — 🧩 skills, slash commands & subagents
- [hooks-and-permissions.md](docs/writing-for-agents/hooks-and-permissions.md) — 🔒 quality gates & trust
- [memory-and-mcp.md](docs/writing-for-agents/memory-and-mcp.md) — 🧠 memory & 🔌 MCP servers
- [compressed-config.md](docs/writing-for-agents/compressed-config.md) — 🗜️ write config that costs fewer tokens
- [behavioral-rules.md](docs/writing-for-agents/behavioral-rules.md) — 🎯 steer *how* the agent codes
- [planning-and-docs.md](docs/writing-for-agents/planning-and-docs.md) — 📐 planning, docs & the initial idea
- [workflow-commands.md](docs/writing-for-agents/workflow-commands.md) — 🛠️ `/planx` + `/feature` — the plan-slice → agent → PR seam, and why **no git worktrees**
- [mcp-json.md](docs/writing-for-agents/mcp-json.md) — 🔌 `.mcp.json` — the repo's MCP servers, committed and secret-free
- [guards-and-gotchas.md](docs/writing-for-agents/guards-and-gotchas.md) — 🛡️ **make the machine careful** — lint guards, doctor checks, preventive rules vs runbook

### 2. 🏗️ Architecture
- [architecture/README.md](docs/architecture/README.md) — the index
- [monorepo.md](docs/architecture/monorepo.md) — 📦 monorepo so **each part uses the best language for the job**
- [tech-stack.md](docs/architecture/tech-stack.md) — 🧰 the default stack & why
- [solid-srp.md](docs/architecture/solid-srp.md) — 🧱 SOLID / SRP, small files, custom errors
- [testing.md](docs/architecture/testing.md) — 🧪 unit + integration testing as a first-class citizen
- [data-and-scale.md](docs/architecture/data-and-scale.md) — 📈 **shape now, capacity later** · bounded sweeps · migrations vs backfills · dev engine ≠ prod engine
- [abstractions-and-growth.md](docs/architecture/abstractions-and-growth.md) — 🪜 **pre-MVP → really big** · declare once, project everywhere · **a guard is the tax paid for not deleting the bypass** · why the rule of three doesn't fire when an AI writes the code

### 3. 🧰 Stack
- [stack/README.md](docs/stack/README.md) — the index
- [frontend-solidjs.md](docs/stack/frontend-solidjs.md) — ⚡ SolidJS + Bun + Vite/WXT
- [backend-bun-hono.md](docs/stack/backend-bun-hono.md) — 🦊 Bun + Hono + Drizzle
- [rust-apis.md](docs/stack/rust-apis.md) — 🦀 Rust for hot-path / high-throughput APIs
- [databases.md](docs/stack/databases.md) — 🐘 Postgres → 🌍 YugabyteDB · 🐉 Dragonfly · ☁️ R2
- [mobile.md](docs/stack/mobile.md) — 📱 native Swift (iOS) + Kotlin (Android)

### 4. 🛠️ Developer experience
- [developer-experience/README.md](docs/developer-experience/README.md) — the index
- [dx-scripts.md](docs/developer-experience/dx-scripts.md) — 🏃 `bin/setup`, `bin/dev`, `bin/check`
- [dev-vps.md](docs/developer-experience/dev-vps.md) — 🐧 every dev on a Linux VPS with Claude Code (no OS wars)
- [codegraph.md](docs/developer-experience/codegraph.md) — 🕸️ CodeGraph: a knowledge graph of your code for agents
- [inner-loop.md](docs/developer-experience/inner-loop.md) — ⏱️ scoped checks vs the full gate — the loop the agent actually runs
- [linting-ci.md](docs/developer-experience/linting-ci.md) — ✨ Biome · 🔨 Blacksmith CI
- [ai-first-cicd.md](docs/developer-experience/ai-first-cicd.md) — 🤖 **AI-first CI/CD** — one gate ≡ CI, parity tests, gates that aren't tests, agent-readable failures
- [smart-builds.md](docs/developer-experience/smart-builds.md) — 🧠 **smart change detection & caching** — one detector, build-vs-roll content hashes, per-unit images, cache keys

### 5. 🤖 AI agents
- [ai-agents/README.md](docs/ai-agents/README.md) — the index
- [agent-sdk.md](docs/ai-agents/agent-sdk.md) — 🧬 building agents with the AI Agent SDK
- [orchestration.md](docs/ai-agents/orchestration.md) — 🎼 Planner → Worker → Reviewer, AI Task Master, z.ai subs
- [hive-mind.md](docs/ai-agents/hive-mind.md) — 🐝 many agents, **one checkout**: when to hive, the file set as the lock, the 9-point brief
- [tools-and-mcp.md](docs/ai-agents/tools-and-mcp.md) — 🔧 tool design + MCP integration
- [agent-work-limits.md](docs/ai-agents/agent-work-limits.md) — 🔓 **freedom + access** — never cap the work, stall-fence instead, diagnose the terminator
- [context-budget.md](docs/ai-agents/context-budget.md) — 🧮 standing context, prompt-cache prefix order, lazy surfaces, chunked external results
- [media-generation.md](docs/ai-agents/media-generation.md) — 🎬 images, video, audio & speech via OpenRouter

### 6. ☸️ Infrastructure
- [infrastructure/README.md](docs/infrastructure/README.md) — the index
- [kubernetes-gitops.md](docs/infrastructure/kubernetes-gitops.md) — ☸️ k3s HA + ArgoCD GitOps (for big projects)
- [servers-and-dns.md](docs/infrastructure/servers-and-dns.md) — 🖥️ OVH VPS/dedicated · 🌐 ClouDNS · ✉️ self-hosted mail
- [sso-zitadel.md](docs/infrastructure/sso-zitadel.md) — 🔑 one SSO for many apps (Zitadel + GitHub)
- [secrets.md](docs/infrastructure/secrets.md) — 🔐 sealed-secrets, Vaultwarden, `.env.example`
- [observability.md](docs/infrastructure/observability.md) — 🚨 Sentry error monitoring as an agent work-queue

### 7. 🎨 Frontend craft
- [frontend-craft/README.md](docs/frontend-craft/README.md) — the index
- [i18n.md](docs/frontend-craft/i18n.md) — 🌍 internationalization
- [theming-dark-mode.md](docs/frontend-craft/theming-dark-mode.md) — 🌗 auto theme & dark mode
- [dates-money-timezones.md](docs/frontend-craft/dates-money-timezones.md) — 🕰️ per-user timezones · 💰 money formatting
- [captcha.md](docs/frontend-craft/captcha.md) — 🛡️ hCaptcha (and keeping the app AI-testable)
- [css-scss-craft.md](docs/frontend-craft/css-scss-craft.md) — 💅 CSS3 + SCSS showcase: animations, gradients, tokens
- [pwa-offline.md](docs/frontend-craft/pwa-offline.md) — 📲 PWA, responsive, web workers & offline-first
- [assets-optimization.md](docs/frontend-craft/assets-optimization.md) — 🗜️ optimized images/video/WebP & fonts, optimize before commit
- [seo.md](docs/frontend-craft/seo.md) — 🔍 SEO: meta, Open Graph, JSON-LD, sitemap, media, Core Web Vitals

### 8. 🔄 Workflow
- [workflow/README.md](docs/workflow/README.md) — the index
- [project-kickoff.md](docs/workflow/project-kickoff.md) — 🌱 zero → running project, fast
- [github-issues-milestones.md](docs/workflow/github-issues-milestones.md) — 🎫 issues & milestones at agent speed
- [shipping-doctrine.md](docs/workflow/shipping-doctrine.md) — 🚢 no legacy, no shims, **no feature flags**, no prod A/B — one path, replaced completely

## 🤝 How to use this repo

1. **As an agent:** read [00-philosophy.md](docs/00-philosophy.md), then the section relevant to your task. Treat the patterns as defaults — deviate only with a reason.
2. **As a human:** clone it into your workspace next to your projects. Reference it from your own `CLAUDE.md` (e.g. `Conventions: see ../gold-standards-in-ai/docs/`).
3. **Keep it alive:** when a pattern proves itself in a real project, fold it back here. Delete what goes stale — a wrong doc costs more than a missing one.

---

*Brainstorming + docs only. No build step, no dependencies — just signal.* ✨
