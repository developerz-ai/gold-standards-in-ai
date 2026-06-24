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

## Keep it small — index, don't inline
The root file is a **router, not an encyclopedia.** Put the always-needed rules + exact commands inline; for everything deep, **point at where it lives** so the agent loads it *only when relevant*. `CLAUDE.md` is read every turn; `docs/` is read on demand — that split is what keeps it lean ([planning-and-docs](planning-and-docs.md)).

```markdown
## Where to look (load on demand)
- Architecture & ADRs → `docs/architecture/`
- API reference        → `docs/api.md`
- Deploy runbook       → `docs/deploy.md`
- Auth model           → `packages/auth/CLAUDE.md`
- Per-app specifics    → `apps/<app>/CLAUDE.md`
- Scripts catalog      → run `bun scripts/help.ts`
```

Rules:
- **Reference, don't paste.** `Webhook flow → src/payments/stripe_webhook.ts` beats pasting the code.
- **Push detail down to where it's used** — a per-subdir `CLAUDE.md` next to the code it describes; the root just names it. Same [layered pattern](#layered-claudemd-monorepo) as a monorepo.
- **One source of truth.** If it's in a doc, link the doc — don't duplicate it in `CLAUDE.md` (two copies drift).
- **A link earns its place** only if the agent will need that detail sometime. Otherwise drop it.

> **Same principle for scripts.** Don't list every script in `CLAUDE.md` — expose a `scripts/help.ts` catalog and a `scripts/lib/` of reusable modules; `CLAUDE.md` just says "run `bun scripts/help.ts`." The doc stays small, the agent discovers depth on demand → [../developer-experience/dx-scripts.md](../developer-experience/dx-scripts.md).

### CLAUDE.md vs AGENTS.md
Same file, two names — `CLAUDE.md` is read by Claude Code; `AGENTS.md` is the cross-tool convention other agents read. Keep **one** as the source of truth and make the other a one-line pointer (`See CLAUDE.md`) or a symlink, so they never drift. The content rules here apply to both.

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
