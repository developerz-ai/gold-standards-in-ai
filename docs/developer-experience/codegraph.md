# 🕸️ CodeGraph — A Knowledge Graph for Agents

**CodeGraph** is a tree-sitter-parsed knowledge graph of every symbol, edge, and file in a codebase. It gives agents *structural* answers (what calls what, where X is defined, what breaks if you change Z) in sub-millisecond lookups — instead of a slow grep-and-read crawl.

Repo: <https://github.com/colbymchenry/codegraph> · Docs: <https://colbymchenry.github.io/codegraph/>

## Why it matters for AI-first repos
The more an agent can *cheaply* understand the codebase, the better it builds ([philosophy #2](../00-philosophy.md)). CodeGraph is a pre-built index, so the agent stops re-deriving structure with dozens of file reads.

Reported across real codebases: far fewer tool calls, faster task completion, fewer file reads, lower token cost — because one structural query replaces a grep→read→grep→read loop.

## Setup
```bash
# install the CLI
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
# wire it into your agent (Claude Code, Cursor, etc.)
codegraph install
# index a project
cd your-project && codegraph init
```
Once initialized it watches files and keeps the graph fresh — the index is never stale (the watcher debounces ~500ms behind writes, so don't re-query in the same instant you edit).

## When to use it vs grep
Use CodeGraph for **structural** questions; use grep/read for **literal text**.

| Question | Tool |
|---|---|
| "Where is X defined?" | `codegraph_search` |
| "What calls Y?" | `codegraph_callers` |
| "What does Y call?" | `codegraph_callees` |
| "What breaks if I change Z?" | `codegraph_impact` |
| "Show me Y's signature/source" | `codegraph_node` |
| "Focused context for a task/area" | `codegraph_context` |
| "Several related symbols' source at once" | `codegraph_explore` |
| Find a string/comment/log message | grep |

## Rules of thumb
- **Trust the results** — they come from a full AST parse. Don't re-verify with grep.
- **Don't grep first** to find a symbol — `codegraph_search` returns kind + location + signature in one call.
- **One `codegraph_context`** beats chaining search + node; **one `codegraph_explore`** beats looping `node` over many symbols.
- If `.codegraph/` doesn't exist, the server reports "not initialized" — run `codegraph init`.

## Make it part of the repo
Add CodeGraph to the project's MCP setup and mention it in `CLAUDE.md` so every agent uses it by default. It pairs naturally with the [predictable file layout](../architecture/solid-srp.md) — structure the code well *and* index it, and the agent navigates instantly.
