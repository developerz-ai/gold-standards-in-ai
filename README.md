# 🏆 Gold Standards in AI

> **How we build software fast, with AI agents, at developerz-ai.**
> A knowledge base — not a codebase. Markdown only. Read it, copy the patterns, ship.

This repo is a **knowledge base for coding agents** (and the humans who run them). It distills how we build production systems end to end — the stack, the architecture, the infra, and most importantly *how to set up a project so an AI agent can do excellent work from the first prompt*.

Point any agent at this repo before it starts a new project. The more an agent knows about how we work — conventions, tooling, the exact commands, where things go — the better and faster it builds. 🚀

> 🎯 **Who this is for:** AI-first orgs where **agents write ~100% of the code** and humans review, steer, and operate. Every standard here optimizes for that reality — make the repo so legible and so well-tooled that an agent ships production-quality work autonomously, and a human verifies it at the PR. If humans still write most of your code, much of this still helps, but the rules are tuned for full autonomy.

## 🧭 Start here

| If you want to… | Read |
|---|---|
| Understand the whole philosophy | [docs/00-philosophy.md](docs/00-philosophy.md) |
| Make a repo great for AI agents | [docs/writing-for-agents/](docs/writing-for-agents/README.md) |
| Pick the stack & structure | [docs/architecture/](docs/architecture/README.md) · [docs/stack/](docs/stack/README.md) |
| Get fast feedback loops (DX) | [docs/developer-experience/](docs/developer-experience/README.md) |
| Build AI agents | [docs/ai-agents/](docs/ai-agents/README.md) |
| Deploy & operate | [docs/infrastructure/](docs/infrastructure/README.md) |
| Polish the frontend | [docs/frontend-craft/](docs/frontend-craft/README.md) |
| Move fast on GitHub | [docs/workflow/](docs/workflow/README.md) |

## 📚 The full map

### 0. 🧠 Philosophy
- [00-philosophy.md](docs/00-philosophy.md) — AI-first development. **Great DX = fast agents.** Give the agent access to everything.

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

### 2. 🏗️ Architecture
- [architecture/README.md](docs/architecture/README.md) — the index
- [monorepo.md](docs/architecture/monorepo.md) — 📦 monorepo so **each part uses the best language for the job**
- [tech-stack.md](docs/architecture/tech-stack.md) — 🧰 the default stack & why
- [solid-srp.md](docs/architecture/solid-srp.md) — 🧱 SOLID / SRP, small files, custom errors
- [testing.md](docs/architecture/testing.md) — 🧪 unit + integration testing as a first-class citizen

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
- [linting-ci.md](docs/developer-experience/linting-ci.md) — ✨ Biome · 🔨 Blacksmith CI

### 5. 🤖 AI agents
- [ai-agents/README.md](docs/ai-agents/README.md) — the index
- [agent-sdk.md](docs/ai-agents/agent-sdk.md) — 🧬 building agents with the AI Agent SDK
- [orchestration.md](docs/ai-agents/orchestration.md) — 🎼 Planner → Worker → Reviewer, AI Task Master, z.ai subs
- [tools-and-mcp.md](docs/ai-agents/tools-and-mcp.md) — 🔧 tool design + MCP integration
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

### 8. 🔄 Workflow
- [workflow/README.md](docs/workflow/README.md) — the index
- [project-kickoff.md](docs/workflow/project-kickoff.md) — 🌱 zero → running project, fast
- [github-issues-milestones.md](docs/workflow/github-issues-milestones.md) — 🎫 issues & milestones at agent speed

## 🤝 How to use this repo

1. **As an agent:** read [00-philosophy.md](docs/00-philosophy.md), then the section relevant to your task. Treat the patterns as defaults — deviate only with a reason.
2. **As a human:** clone it into your workspace next to your projects. Reference it from your own `CLAUDE.md` (e.g. `Conventions: see ../gold-standards-in-ai/docs/`).
3. **Keep it alive:** when a pattern proves itself in a real project, fold it back here. Delete what goes stale — a wrong doc costs more than a missing one.

---

*Brainstorming + docs only. No build step, no dependencies — just signal.* ✨
