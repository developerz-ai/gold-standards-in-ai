# 🔌 `.mcp.json` — the repo's MCP servers, committed

`.mcp.json` at the repo root is **project-scoped MCP config**: every agent that clones the repo gets the same external reach — issues, browser, errors, DB, code graph — with **zero per-dev setup**. Commit it. It is DX, not a personal preference.

| Scope | File | Committed | Use for |
|---|---|---|---|
| **Project** | `.mcp.json` (repo root) | ✅ yes | servers *every* agent on this repo needs |
| User | `~/.claude/settings.json` | no | your personal servers/keys across all repos |
| Local override | `.claude/settings.local.json` | no (gitignored) | one box, one experiment |

## The shape

```json
{
  "mcpServers": {
    "sentry":   { "type": "http", "url": "https://mcp.sentry.dev/mcp" },
    "linear":   { "type": "http", "url": "https://mcp.linear.app/mcp" },
    "codegraph":{ "type": "stdio", "command": "codegraph", "args": ["serve", "--mcp"] },
    "app-admin": {
      "type": "http",
      "url": "${ADMIN_MCP_URL:-https://mcp.example.com/mcp/admin}",
      "headers": { "Authorization": "Bearer ${ADMIN_MCP_BEARER}" }
    },
    "ui-debugger": {
      "command": "npx",
      "args": ["-y", "@your-org/ui-debugger-mcp@1.4.2"],
      "env": {
        "OPENAI_API_KEY": "${UI_DEBUGGER_API_KEY}",
        "OPENAI_BASE_URL": "${UI_DEBUGGER_BASE_URL:-https://openrouter.ai/api/v1}"
      }
    }
  }
}
```

## Rules

- **Never inline a secret.** Use `${VAR}` interpolation — the file is committed, the value lives in the shell/`.env`. A token in `.mcp.json` is a token in git history.
- **`${VAR:-default}` for anything environment-dependent.** Staff URL, region, base URL: default to prod/public, let a dev override to localhost. No fork of the file per box.
- **Every `${VAR}` you introduce goes in `.env.example`** the same commit, exactly like a runtime env var. An agent that can't tell *which* variable is missing burns a turn guessing.
- **`http` for hosted, `stdio` for local binaries.** `stdio` servers must be installable by one command (`npx -y …`, a `bin/` tool, a released binary) — if setup takes a page of instructions, wrap it in `bin/setup` first.
- **Pin every `stdio` package to an exact version.** `npx -y pkg@latest` runs whatever was published last, on every dev box, with every `${VAR}` in its `env` block and your shell's reach — a hijacked maintainer account is code execution on all of them. Exact version (`@1.4.2`, no `^`/`latest`); bump in a PR like any dependency, reading the changelog. Same for `bunx`/`uvx`. `http` servers are the vendor's deploy — nothing to pin, but the token you send them is scoped (→ [../infrastructure/secrets.md](../infrastructure/secrets.md#-agent-identities--its-own-accounts-never-yours)).
- **`.mcp.json` changes are reviewed like code that runs on every box** — because it is. A new server or a changed `command`/`url` in a PR is a supply-chain change → [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md#-repo-config-is-executable-surface).
- **Name servers for what they reach**, not the vendor: `db-gateway`, `ui-debugger`, `app-admin`. Tool names surface to the model as `mcp__<server>__<tool>` — the server name is prompt real estate.

## What earns a slot

Every connected server costs tokens on **every turn** (its tools are listed in the standing surface — see [../ai-agents/context-budget.md](../ai-agents/context-budget.md)). So the bar is: *does an agent working in this repo reach for it weekly?*

| Server | Buys the agent |
|---|---|
| Error monitor (Sentry) | prod stack traces → a fix + reproducing test, unprompted → [../infrastructure/observability.md](../infrastructure/observability.md) |
| Issue tracker (Linear/Jira) | read the ticket it's implementing, close it on merge |
| Browser (Playwright / UI debugger) | *see* the UI it just changed |
| DB gateway | audited read-only SQL against real data → [../ai-agents/tools-and-mcp.md](../ai-agents/tools-and-mcp.md#audited-capability-access-the-gateway-pattern) |
| [CodeGraph](../developer-experience/codegraph.md) | structural code lookup instead of grep sweeps |
| Your own product's admin MCP | operate the thing it builds — dogfooding, and the fastest prod debugging you own |

**Second test — does the work need a server?** MCP earns the slot when the job is **stateful**: a held-open session (live browser/devtools), streaming, an auth handshake (OAuth, SSO session), or a gateway that must hold the credential. Stateless request/response against a tool with a good CLI → **CLI + a skill** instead: zero standing schema cost, and the model already knows the CLI from training.

| Job | Surface |
|---|---|
| GitHub PRs, issues, Actions | `gh` CLI + a skill — not the GitHub MCP (~dozens of schemas every turn) |
| Docs lookup against a REST API | a skill with the `curl` call |
| Live browser debugging, perf traces | MCP — the held session is the value |
| DB gateway, SSO-gated admin | MCP — the auth handshake + credential custody is the value |

"Popular" isn't an argument for a slot; "stateful and most agents here need it weekly" is.

Anything used monthly: connect it per-session (`claude mcp add …`) or leave it to the user config. A 47-tool connector nobody calls is a permanent tax.

## Pair each server with a skill

A server gives the agent *capability*; a skill tells it **when and how**. One skill per non-obvious server, naming the scope and the auth dance:

```markdown
---
name: db-gateway
description: Debug THIS repo's Postgres with read-only SQL via the db-gateway MCP. Use when
  inspecting a table/schema, checking row counts, or confirming what's actually in the DB.
---
Target server `main`, database `app`. Read-only, audited, SSO-gated.
First call opens SSO in the browser; session ~8h. 403 after login → ask an admin for the grant.
Do NOT use it against other products' databases.
```

Without that, the agent either ignores the server or points it somewhere it shouldn't. See [skills-commands-agents.md](skills-commands-agents.md).

## Gotchas (each one has cost a real session)

- **A remote OAuth flow "hangs" on a headless box** — the callback is `localhost` on a machine you're SSH'd into. Paste the callback URL into the server's `complete_authentication` tool instead.
- **In-memory MCP sessions die on redeploy.** If the server is yours and it drops sessions, the agent sees a mystery 401 — say so in the skill.
- **`tools/list` is not free to re-fetch.** Cache the tool list per session; HTTP streamable transports dislike repeated listing.
- **A server that's down must degrade, not block.** Agents building on MCP should `try/catch` the connection and continue with local tools ([tools-and-mcp](../ai-agents/tools-and-mcp.md)).

## Allow the tools you trust

The agent runs with full permissions, so every server's tools work without prompts → [permissions.md](permissions.md). Make writes safe in the server (scoped tokens, audit, reversible verbs), not with a prompt.

---

*Building your own MCP server?* The surface-size rules — few parameterized tools, lazy schemas, the 3-tool meta pattern — are in [memory-and-mcp.md](memory-and-mcp.md#building-mcp-servers--few-parameterized-tools-beat-many).
