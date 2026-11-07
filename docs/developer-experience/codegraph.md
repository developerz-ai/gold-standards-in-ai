# 🕸️ CodeGraph — A Knowledge Graph for Agents

**CodeGraph** is a tree-sitter-parsed knowledge graph of every symbol, edge, and file in a codebase. It gives agents *structural* answers (what calls what, where X is defined, what breaks if you change Z) in sub-millisecond lookups — instead of a slow grep-and-read crawl.

Repo: <https://github.com/colbymchenry/codegraph> · Docs: <https://colbymchenry.github.io/codegraph/>

## Why it matters for AI-first repos
The more an agent can *cheaply* understand the codebase, the better it builds ([philosophy #2](../00-philosophy.md)). CodeGraph is a pre-built index, so the agent stops re-deriving structure with dozens of file reads.

Reported across real codebases: far fewer tool calls, faster task completion, fewer file reads, lower token cost — because one structural query replaces a grep→read→grep→read loop.

## Setup
Pin everything — a `main` ref is a moving target, and a moved ref runs someone else's code. Fetch the installer at an immutable commit SHA, verify its sha256 before executing it, and pin the binary to one exact release version.

```bash
# fetch the installer at an immutable commit SHA — never a branch ref
# verify the checksum, then install one exact release version
# `&&` chains the steps — a checksum mismatch stops the chain before the installer runs
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/d3671faf04991f9d3c0fabc4bd437b952d042d67/install.sh -o /tmp/codegraph-install.sh \
  && (echo 'e8bbcc694584d5c88fa7c25103fc0f9c238d63a86c1494ca03448b680dbcc86e  /tmp/codegraph-install.sh' | sha256sum -c -) \
  && CODEGRAPH_VERSION=v1.6.2 sh /tmp/codegraph-install.sh \
  && codegraph --version   # → 1.6.2
# wire it into your agent (Claude Code, Cursor, etc.)
codegraph install
# index a project
cd your-project && codegraph init
```

The committed [`.mcp.json`](../../.mcp.json) runs `codegraph serve --mcp` from PATH — it pins no version (command, args and env only). This setup expects **v1.6.2** on PATH (as of 2026-11).

| Pinned artifact | Value | As of |
|---|---|---|
| Release | `v1.6.2` | 2026-11 |
| install.sh commit | `d3671faf04991f9d3c0fabc4bd437b952d042d67` | 2026-11 |
| install.sh sha256 | `e8bbcc694584d5c88fa7c25103fc0f9c238d63a86c1494ca03448b680dbcc86e` | 2026-11 |

```bash
# pins still live? each URL must answer 200
curl -sIL --max-time 10 https://raw.githubusercontent.com/colbymchenry/codegraph/d3671faf04991f9d3c0fabc4bd437b952d042d67/install.sh https://github.com/colbymchenry/codegraph/releases/tag/v1.6.2
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
