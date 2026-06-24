# 🧠 Memory & 🔌 MCP

How an agent remembers across sessions, and how it reaches external systems.

## Memory — cross-conversation learning
File-based memory persists between conversations. The agent learns from corrections and confirmed approaches over time.

| Type | Stores | Example |
|---|---|---|
| **user** | who you are, preferences | "Senior backend dev, prefers terse answers." |
| **feedback** | corrections AND confirmed approaches | "Use integration tests for DB ops — mocks burned us." |
| **project** | ongoing work, decisions, deadlines | "Auth rewrite is compliance-driven, not tech-debt." |
| **reference** | pointers to external systems | "Pipeline bugs tracked in issue tracker project INGEST." |

Format (markdown + frontmatter), one fact per file:
```markdown
---
name: testing-approach
description: Prefer integration tests over mocks for database operations
type: feedback
---
Use real DB connections in tests, not mocks.
**Why:** mocked tests passed but a prod migration failed.
**How to apply:** all service tests that touch the DB use the test database.
```

Save from **both failure and success** — if you only record corrections, the agent avoids mistakes but drifts from validated approaches. An index file lists all memories for quick lookup.

## MCP servers — external tools, no tab switching
MCP (Model Context Protocol) gives the agent native tools for outside systems.

| Server | The agent can… |
|---|---|
| Issue tracker (Linear/Jira) | create/update issues, search, comment |
| Playwright | open browsers, click, fill forms, screenshot |
| Sentry/Bugsnag | search errors, view stack traces |
| Slack / Gmail / Calendar | read/send messages, events |
| **Custom** | anything you build — DB gateway, knowledge base, CI |

```bash
claude mcp add playwright -- npx @playwright/mcp
```

This is principle #2 from the [philosophy](../00-philosophy.md): *the more the agent can reach, the more it solves end-to-end.* For audited, SSO-gated DB access without handing out credentials, see [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md) and [../ai-agents/tools-and-mcp.md](../ai-agents/tools-and-mcp.md).

## Building MCP servers — few parameterized tools beat many
One tool per action = hundreds of tools sitting in context every turn. The cost is brutal:

| Resources (6 verbs) | One-tool-per-verb | `tools/list` every turn |
|---|---|---|
| 50 | 300 tools | ~60k tokens |
| 500 | 3,000 tools | won't fit |

### The 3-tool meta pattern — constant surface at any scale
| Tool | Purpose |
|---|---|
| `list_resources` | catalog: `[{name, description, actions}]`. Cheap. |
| `describe_resource` | schema for one resource/action. Lazy — only what's needed. |
| `manage_resource` | dispatcher: `{resource, action, ...params}` → registered handler. |

```
manage_resource({ resource: "posts", action: "list",
                  filters: { status_eq: "published" }, sort: "-created_at" })
```
Add 50 resources tomorrow — `tools/list` doesn't grow. Only schemas the model asks about enter context.

### Push composition into params, not tools
Don't ship `list_recent_posts`, `list_published_posts`, `posts_by_tag`. Ship one `list` with composable primitives: `filters` (`_eq`, `_gt`, `_in`, `_cont`), `query`, `sort`, `scope`, `fields` (sparse responses), `page`/`per_page`, `include` (eager-load).

Rules:
- **Whitelist, don't blacklist** filters/sorts/scopes per resource.
- **Auth in the handler, not the dispatcher** — same ability/scoping as the rest of the app.
- **Hidden resources return ToolNotFound, not Forbidden** — users can't enumerate what's behind the curtain.
- Keep genuinely non-CRUD tools separate (`send_slack_message`, `reset_password`) and filter them from `tools/list` by role.

## Make your app AI-debuggable
Captchas block Playwright/MCP browser flows. Add a query-param escape hatch that skips *captcha verification only* (not auth) when present — the widget still renders so screenshots match real users. Credentials, rate limits, lockouts all still apply. Gate behind `ALLOW_AI_DEBUG_LOGIN` so it's off in prod. Details: [../frontend-craft/captcha.md](../frontend-craft/captcha.md).
