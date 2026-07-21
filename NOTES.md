# gold-standards-in-ai

A knowledge base — not a codebase — that distills how developerz-ai builds software with AI coding
agents. It is written to be read by an agent before it starts or joins a project, so the agent
inherits the org's conventions, stack choices, infrastructure patterns, and repo-setup practices
from the first prompt. The target audience is AI-first teams where agents write nearly all the code
and humans review, steer, and operate; the humans running those agents are the secondary audience.

**Stack:** Markdown only. No languages, frameworks, datastores, build, or deploy target — there is
nothing to compile and nothing to run.

**Key commands:** None. Contributions are edits to `.md` files; there is no package.json, Makefile,
or test suite.

**Layout:**
- `README.md` — the map; every doc in the repo is linked from here with a one-line hook
- `docs/00-philosophy.md` — the core thesis (AI-first development, great DX = fast agents)
- `docs/writing-for-agents/` — the most important section: `CLAUDE.md` design, skills/commands/subagents, hooks and permissions, memory and MCP, compressed config, behavioral rules, planning docs
- `docs/architecture/`, `docs/stack/`, `docs/developer-experience/`, `docs/ai-agents/`, `docs/infrastructure/`, `docs/frontend-craft/`, `docs/workflow/` — one topic per file, each with its own section `README.md` index

**Conventions worth knowing:** lead with the rule not the reason; fragments over sentences; tables
over paragraphs; keep paths and commands verbatim; date load-bearing claims (`As of 2026-06`) and
delete stale docs rather than letting them rot; no secrets, private hostnames, or client names.

**State as of 2026-07-21:** branch `main`, working tree clean before this note was added.
