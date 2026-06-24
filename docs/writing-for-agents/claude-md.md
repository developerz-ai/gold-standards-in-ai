# 🗂️ CLAUDE.md — The Project Brain

`CLAUDE.md` is the single most important file in any repo. The agent reads it **every session**, and it shapes every decision. Get it right and everything downstream gets easier.

## What goes in it
- **Project structure** — directory tree, purpose of each folder.
- **Tech stack** — with versions.
- **All runnable commands** — exact CLI, not descriptions.
- **Architecture decisions** — "business logic in `src/services/`, controllers are thin."
- **Auth model & roles.**
- **Database conventions** — ID types, naming, patterns.
- **Testing strategy** — frameworks, commands, what to test.
- **Infrastructure** — hosting, CI/CD, deploy flow.

## Actionable, not aspirational
Don't write "we value clean code." Write the constraint:

> Business logic belongs in `app/services/<domain>/` following the `Domain::Action` pattern. Controllers are thin — parse, call service, render. Models handle persistence only.

Lead with the constraint, not a tour. The agent reads it to decide the *next action*, not to learn history.

## Exact commands — the agent can't guess your CLI
```bash
bun test                     # unit tests
bun run test:integration     # integration tests
bun run lint:fix             # auto-fix lint
bun run typecheck            # type check
bin/dev                      # boot local stack
scripts/deploy production    # deploy
```

## Document the WHY when non-obvious
- "N+1 prevention: NEVER write manual eager loading — the ORM plugin handles it."
- "Always use domain-specific error classes, never generic `Error`/`Exception`."

## Keep it under 600 lines
It's read every turn — bloat is a token tax forever. 500 well-structured lines cover complex production apps. See [compressed-config.md](compressed-config.md) for how to compress without losing signal.

## Response rules belong at the top
Behavioral instructions shape output style across the whole session:

```markdown
## Response Rules
- Execute. No preamble. No "I'll start by…". No restating the task.
- Lead with action or answer. Reasoning after, only if non-obvious.
- Parallel tool calls when independent.
- Read before speculating.
- Disagree when the user is wrong. State the correction.
- Terse. Fragments OK. Code/commands/paths stay exact.
- End summary: 1–2 sentences max.
```

This alone cuts ~30% of output tokens. Add coding behavior rules too → [behavioral-rules.md](behavioral-rules.md).

## Layered CLAUDE.md (monorepo)
```
workspace/
├── CLAUDE.md              # org-level: stack, repo categories, cross-repo rules
└── repos/
    ├── backend/CLAUDE.md  # project: architecture, commands
    └── frontend/CLAUDE.md # different stack, same pattern
```
The agent loads both, root → project. Keep each layer to its own scope — no duplication. See [../architecture/monorepo.md](../architecture/monorepo.md).

## Self-updating
Add an `/update-claude` slash command so the agent refreshes `CLAUDE.md` when features land. Stale docs are worse than missing ones — [planning-and-docs.md](planning-and-docs.md#docs-decay).

## Minimal skeleton
```markdown
# <App>

<one line: framework · db · cache · what it is>

## Response Rules
- …

## Architecture
- Business logic → `src/services/<domain>/`
- Controllers: thin. Models: persistence only.
- Custom errors only. Files ≤500 LOC.

## Commands
bun test            # tests
bun run lint:fix    # lint
bin/dev             # local stack

## Layout
- `apps/*`     user-facing apps
- `packages/*` shared domain + infra

## Rules
- SOLID/SRP. Tests required. Surgical diffs.
```
