# Gold Standards in AI

> Knowledge base for AI coding agents — Markdown only, no code to build, no tests to run.

## What this is

| Question | Answer |
|---|---|
| What | Knowledge base for AI coding agents — how developerz-ai builds software fast with agents |
| Who reads it | An agent, before it starts a project: conventions, stack, infra, how to set a repo up so agents thrive |
| What is inside | `docs/**/*.md` — self-contained, dense, copy-paste oriented |
| What it is not | A codebase — no runtime, no dependencies, no build step |
| One brain | `AGENTS.md` points here — a pointer, never a second copy |

## Stack

| Piece | Reality |
|---|---|
| Content | Markdown only — no runtime, no dependencies |
| Scripts | bash under `bin/` |
| Gate | `bin/check` — the same gate CI runs |
| CI | `.github/workflows/ci.yml` — `bin/setup` + `bin/check` on push and pull_request |
| Pre-commit | `.githooks/pre-commit` → `bin/check`; activate once per clone: `git config core.hooksPath .githooks` |

## Commands

Run from the repo root; each script resolves the repo root itself.

| Command | Does | Run |
|---|---|---|
| `bin/setup` | `mise install` when mise is present; echoes `gold-standards-in-ai: markdown only, no dependencies to install` | once, after clone |
| `bin/dev` | echoes `gold-standards-in-ai: no app to run - read README.md for the map` | every session |
| `bin/check` | the gate — `bin/check-docs` + `bin/test`; silent on success | before every commit |
| `bin/check-docs` | map + relative-link validation; one line per finding, exit 1 on any | while editing docs |
| `bin/test` | regression fixtures for the validator; no downloaded dependencies | when `bin/check-docs` changes |
| `bin/stop` | no-op — nothing to stop, never removes data | session end (the platform runs it) |

## Structure

| Path | What |
|---|---|
| `README.md` | the map — every doc is linked from it |
| `AGENTS.md` | pointer to this file — one brain, never two |
| `docs/README.md` | docs index |
| `docs/00-philosophy.md` | the core thesis; indexed in the root `README.md` and in `docs/README.md` — its section index, both required indexes |
| `docs/<section>/README.md` | index for each section |
| `docs/<section>/<topic>.md` | one topic per file |
| `docs/<section>/` | 8 sections — `ai-agents`, `architecture`, `developer-experience`, `frontend-craft`, `infrastructure`, `stack`, `workflow`, `writing-for-agents` |
| `.claude/agents/` | specialists — `writing-rules.md`, `readme-index.md`, `link-integrity.md`, `dated-claims.md` |
| `.claude/commands/` | `/feature` — the doc workflow · `/planx` — plan slices |
| `.claude/skills/` | `verify-change` · `stale-claims` · `plan-slices` |
| `.dz/` | maintainer bot — `maintainer/maintainer.yml`, `briefs/`, `onboarding/` |
| `.mcp.json` | MCP servers — `sentry-mcp`, `codegraph`, `developerz` |
| `.env.example` | names only — copy to `.env`; never commit `.env` |
| `bin/` | the scripts above |

## Writing rules

| Rule | Fails when |
|---|---|
| Lead with the rule, not the reason. Fragments over sentences. Tables over paragraphs. | a section opens with a tour of the repo |
| Keep file paths, commands, env vars and model IDs verbatim — compress prose only. | a path or a command is paraphrased |
| One topic per file. Keep each file scannable in one pass. | a file answers two questions |
| Use a table for any list of three or more rows. | rows are flattened into prose |
| No meta-framing, no rhetoric, no trailing summary paragraph. | "This section covers…" appears |
| Use emojis in headings/READMEs for navigation; keep code blocks clean. | a heading loses its marker |
| Cross-link a sibling with a relative link instead of restating it. | the same text is copied twice |
| Date load-bearing claims (`As of 2026-06`). | a load-bearing claim carries no date |
| Keep examples vendor-neutral and reusable. Prefer the latest Claude models when showing model IDs. | an example names a client |
| Index rows carry a link + a one-line hook; a section row uses the bare filename, relative to the section directory. | a section row spells its own section path |

Long form and self-check list: [docs/writing-for-agents/compressed-config.md](docs/writing-for-agents/compressed-config.md).

## Hard rules

| Rule | Fails when |
|---|---|
| No secrets, no private hostnames/IPs, no client names. | a doc names a token, an internal host or a customer |
| Every doc is linked from `README.md` *and* its section `README.md`. | `bin/check-docs` prints the path for a doc in neither index |
| Delete stale docs rather than let them rot — a wrong doc costs more than a missing one. | a doc contradicts the tree it describes |
| Never `--force`, never `git stash`, never a git worktree. | another writer's checkout moves under them |

## Where to look

| Need | Read |
|---|---|
| The map of every doc | [README.md](README.md) |
| The thesis | [docs/00-philosophy.md](docs/00-philosophy.md) |
| The house writing rules, long form | [docs/writing-for-agents/compressed-config.md](docs/writing-for-agents/compressed-config.md) |
| How to write a project brain | [docs/writing-for-agents/claude-md.md](docs/writing-for-agents/claude-md.md) |
| The doc workflow, end to end | [.claude/commands/feature.md](.claude/commands/feature.md) |
| Plan slices for a doc set | [.claude/commands/planx.md](.claude/commands/planx.md) |
| Prove a change before committing | [.claude/skills/verify-change/SKILL.md](.claude/skills/verify-change/SKILL.md) |
| Monthly upkeep of dated claims and links | [.claude/skills/stale-claims/SKILL.md](.claude/skills/stale-claims/SKILL.md) |
| House style specialist | [.claude/agents/writing-rules.md](.claude/agents/writing-rules.md) |
| README map specialist | [.claude/agents/readme-index.md](.claude/agents/readme-index.md) |
| Link resolution specialist | [.claude/agents/link-integrity.md](.claude/agents/link-integrity.md) |
| Claim freshness specialist | [.claude/agents/dated-claims.md](.claude/agents/dated-claims.md) |
| The policy the maintainer bot obeys | [.dz/maintainer/maintainer.yml](.dz/maintainer/maintainer.yml) |

## Git rules

| Rule | Fails when |
|---|---|
| One checkout, many hands — work in this checkout; split a team into disjoint path sets so no two agents touch the same file. | two writers meet in one file |
| Name the paths on `git add`; never `git add -A`. | an editor's scratch file ships |
| Wait for every writer before committing. | a half-written doc lands |
| `git fetch` before each slice; a file that moved on `main` is three-way merged, never overwritten wholesale. | `main`'s index rows vanish silently |
| One PR in flight; cap a PR at ~25–30 changed files. | review becomes a formality |
| Scratch notes, draft plans and agent summaries never ship. | the PR carries files nobody meant to add |
| Merge only on a green gate — `bin/check` local, CI green, review approved. | a red check is merged around |
