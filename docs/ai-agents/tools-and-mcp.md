# 🔧 Tools & MCP for Agents

Tools are how an agent acts on the world. MCP is how it reaches systems you don't own the code for. The principle from the [philosophy](../00-philosophy.md): **the more the agent can reach, the more it solves end-to-end.**

## Local tools vs MCP tools
- **Local tools** — file, git, shell, http — defined in-process with Zod schemas ([agent-sdk](agent-sdk.md#tools--zod-schemas)).
- **MCP tools** — issue trackers, browsers, error monitors, DB gateways — connected over MCP and merged into the tool set.

```ts
const localTools = { file, git, shell };
let mcpTools = {};
try { mcpTools = await getMCPTools(mcp); } catch { /* degrade gracefully */ }
const tools = { ...localTools, ...mcpTools };
```

## Connecting an MCP server (HTTP)
```ts
import { createMCPClient } from "@ai-sdk/mcp";
const mcp = await createMCPClient({
  transport: { type: "http", url, headers: { Authorization: `Bearer ${jwt}` } },
  name: "agent",
});
const tools = await mcp.tools();   // cache these; don't re-fetch every call
```
Retry the connection with backoff; cache the tool list (HTTP streamable transports dislike repeated `tools/list`).

## High-value MCP servers to wire up
| Server | The agent gains |
|---|---|
| **Playwright** ([playwright-mcp](https://github.com/microsoft/playwright-mcp), headless) | drive a browser, screenshot, read the a11y tree — *see* the UI |
| **Sentry / error monitor** | read production errors + stack traces, triage, fix |
| Issue tracker | open/update issues, milestones |
| **DB gateway** | audited, SSO-gated, read-only DB queries — no credential ever on the client |
| Secrets broker | fetch a secret at call time, never store it |
| [CodeGraph](../developer-experience/codegraph.md) | structural code understanding |

### Sentry → close the loop
With a Sentry MCP server the agent can pull the top production errors, read the stack trace, find the offending code, write a fix + a reproducing test, and open a PR — autonomously. Error monitoring isn't just for dashboards; it's a *work queue* for an agent. Wiring + SDK setup: [../infrastructure/observability.md](../infrastructure/observability.md).

## Designing your own MCP server — the 3-tool pattern
Don't ship one tool per action (hundreds of tools = huge per-turn token cost). Expose a constant surface:

| Tool | Purpose |
|---|---|
| `list_resources` | catalog of resources + their actions |
| `describe_resource` | lazy schema for one resource/action |
| `manage_resource` | dispatcher: `{resource, action, ...params}` |

Push composition into **params** (`filters`, `sort`, `fields`, `include`, `page`) not into more tools. Full rationale, the cost table, and auth/whitelisting rules: [../writing-for-agents/memory-and-mcp.md](../writing-for-agents/memory-and-mcp.md#building-mcp-servers--few-parameterized-tools-beat-many).

## Audited capability access (the gateway pattern)
Giving an agent access ≠ handing it credentials. Put a gateway between the agent and the sensitive system. Real example: a Rust DB-gateway MCP ([`db-mcp-gateway`](https://github.com/developerz-ai/db-mcp-gateway)) where:
- **Credentials never leave the gateway** — the agent sends a `(server, database, query)` triple + an SSO token, never a connection string.
- **Identity is end-to-end** — every query traces SSO user → group → grant → audit row.
- **Permissions are config-as-code** — grants in reviewed YAML; constraints (row limits, timeouts, time windows, allowed schemas) merged "most restrictive wins."
- **Audit is synchronous** — the audit row commits *before* the response; if the audit write fails, the response rolls back.

This is how you satisfy "give the agent maximum access" and "never leak a secret" at the same time. SSO wiring: [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md).
