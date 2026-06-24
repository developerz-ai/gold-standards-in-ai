# ✍️ Writing for Agents

The highest-leverage thing you can do is make a repo a place where an AI agent does excellent work. This section is the playbook — distilled from running Claude Code in production every day.

The throughline: **an agent is only as good as the context and tooling you hand it.** Each file you make it read costs tokens every turn, so write dense; each command you document saves a guess; each rule you state prevents a class of mistakes.

## Chapters

| # | File | What it covers |
|---|---|---|
| 1 | [claude-md.md](claude-md.md) | 🗂️ `CLAUDE.md` — the project brain, read every session |
| 2 | [skills-commands-agents.md](skills-commands-agents.md) | 🧩 Skills (domain expertise), slash commands (workflows), subagents (specialists) |
| 3 | [hooks-and-permissions.md](hooks-and-permissions.md) | 🔒 Auto-allow safe ops, gate quality with hooks |
| 4 | [memory-and-mcp.md](memory-and-mcp.md) | 🧠 Cross-session memory · 🔌 MCP servers · the 3-tool MCP pattern |
| 5 | [compressed-config.md](compressed-config.md) | 🗜️ Write config files that cost fewer tokens, same signal |
| 6 | [behavioral-rules.md](behavioral-rules.md) | 🎯 Steer *how* the agent codes (assumptions, simplicity, surgical diffs) |
| 7 | [planning-and-docs.md](planning-and-docs.md) | 📐 Plan before coding · docs vs CLAUDE.md · the initial idea |

## TL;DR — the quick wins
80% of the value, under an hour:

1. **`CLAUDE.md` with exact commands** — test, lint, build, deploy. The agent stops guessing your CLI.
2. **Broad permissions + a pre-commit lint hook** — zero approval friction, but it physically can't commit unlinted code.
3. **Response + coding rules** in `CLAUDE.md` — "no preamble, lead with action, disagree when wrong" + surgical-diff rules.
4. **One lint command** — `bin/lint` auto-fixes everything.

## Anti-patterns
- `CLAUDE.md` over 600 lines — read every turn, trim ruthlessly.
- Pasting whole files into `CLAUDE.md` — reference paths instead.
- A skill for every task — only for repeated, complex workflows.
- No tests — the agent's biggest edge is verifying its own work.
- 20 approval prompts in a row — your permission setup is wrong.
