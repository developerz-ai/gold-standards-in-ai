# 🤖 Building AI Agents

Patterns for building agents — the systems that *do the work* — with the AI Agent SDK. Vendor-neutral; defaults to the latest Claude models and an OpenAI-compatible provider layer.

| File | Topic |
|---|---|
| [agent-sdk.md](agent-sdk.md) | 🧬 The agent loop, tools, structured output, streaming, retries |
| [orchestration.md](orchestration.md) | 🎼 Multi-agent: Planner → Worker → Reviewer, autonomous loops, model tiers, z.ai subs |
| [hive-mind.md](hive-mind.md) | 🐝 Running a team of agents in ONE checkout — when to hive, the file set as the lock, the 9-point brief |
| [tools-and-mcp.md](tools-and-mcp.md) | 🔧 Tool design + MCP integration + audited capability access |
| [media-generation.md](media-generation.md) | 🎬 Images, video, audio & speech via OpenRouter |

## Building blocks
- **SDKs:** the **Vercel AI SDK** (`ai` + `@ai-sdk/*`) for the loop and tools; the **Claude Agent SDK** / Anthropic SDK when you want Claude's native agent surface; `@openrouter/ai-sdk-provider` for provider routing.
- **Models:** prefer the latest Claude (Opus/Sonnet/Haiku/Fable) for the hard reasoning; cheaper tiers for mechanical steps.
- **Providers:** abstract behind one interface so you can swap providers per tier and per cost target. **OpenRouter** is the default router — one key reaches almost every model (LLMs *and* [generated media](media-generation.md)); **z.ai** subscriptions are great for the high-volume coding tier; Anthropic direct / OpenAI when you want first-party.

## Living examples (public org repos)
- [`ai-task-master`](https://github.com/developerz-ai/ai-task-master) — autonomous task orchestrator, Planner/Worker/Reviewer on the Vercel AI SDK + OpenRouter.
- [`claude-task-master`](https://github.com/developerz-ai/claude-task-master) — keep an agent working until a goal is met.
- [`db-mcp-gateway`](https://github.com/developerz-ai/db-mcp-gateway) — audited, SSO-gated DB access as an MCP server (Rust).
- [`ai-designer`](https://github.com/developerz-ai/ai-designer) — chat-to-redesign Chrome extension shipping real code via MCP (Bun + WXT + SolidJS + Vercel AI SDK).
- [`nexus-engine`](https://github.com/developerz-ai/nexus-engine) — AI-first, spec-driven game engine.

## The core idea
An agent is a **loop**: model → picks a tool → tool runs → result feeds back → repeat until a stop condition. Everything else (multi-agent orchestration, providers, memory, MCP) is structure around that loop. Start at [agent-sdk.md](agent-sdk.md).
