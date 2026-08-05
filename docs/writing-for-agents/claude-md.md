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

### CLAUDE.md vs AGENTS.md — symlink for one source of truth
Same file, two names — `CLAUDE.md` is read by Claude Code; `AGENTS.md` is the cross-tool convention other agents read. **Don't maintain two copies — symlink them** so there's exactly one source of truth:
```bash
ln -s CLAUDE.md AGENTS.md      # AGENTS.md → CLAUDE.md; edit one, both update
git add AGENTS.md              # git stores the symlink itself
```

### Symlink all shared AI config
Apply the same `ln` trick to anything an agent reads that should be identical across a project — so you edit once and every consumer sees it:
- **`AGENTS.md` → `CLAUDE.md`** (above).
- **Per-app files that mirror the root** in a [monorepo](../architecture/monorepo.md): if every app should see the org rules, symlink rather than copy.
- **Shared skills/commands/hooks:** keep canonical ones in a root `.claude/` and symlink the dir (or individual files) into apps that need them:
  ```bash
  ln -s ../../.claude/skills/deploy apps/web/.claude/skills/deploy
  ```
- **The gold standards themselves:** symlink this repo's docs into a project so the agent reads them in-tree:
  ```bash
  ln -s ../gold-standards-in-ai/docs docs/standards
  ```

Rules:
- **One real file, many symlinks** — never duplicate; copies drift, symlinks can't.
- **Use relative symlinks** inside a repo so they resolve after clone anywhere.
- **Symlinks-in-git just work** because [everyone's on a Linux dev VPS](../developer-experience/dev-vps.md) — no Windows checkout quirk. (Set `git config core.symlinks true`.)
- The content rules in this doc apply to whatever the canonical file is.

## Preventive rules — the section that earns its tokens

Beyond commands and layout, the highest-value block in a mature `CLAUDE.md` is a list of **rules that bite before any symptom appears** — the ones an agent would otherwise violate *while writing new code*, with no error message to warn it.

The test for inclusion: **"would an agent write this bug without the rule?"** If the answer is "no — it would only hit it while debugging," it belongs in `docs/gotchas.md` instead → [guards-and-gotchas.md](guards-and-gotchas.md#where-knowledge-lands-claudemd-vs-docsgotchasmd).

Write each as **mechanism + cost**, not manners, and name the guard that enforces it:

```markdown
## Preventive rules (these bite before any symptom appears)
- **No queue key may contain `:`.** It's the key separator: an id with one makes a sweep silently
  enqueue 0 jobs. Build ids with `-`, go through `QueueRegistry.enqueue`. Guard: `scripts/lint/no-unguarded-bullmq-id.ts`.
- **A migration is DDL.** It never seeds business rows and never rewrites existing ones — it applies
  once, forever, everywhere, and can't be taken back. Rows → the seed layer. Rewrites → a backfill.
- **Local is Postgres, production is Yugabyte** — a YSQL violation is green on BOTH gates and red in
  the deploy, after merge. Guard: `scripts/lint/migration-ysql-compat.ts`.
- **Never `git stash` here.** Agents share this checkout; the stash is one global stack.
```

Every entry that *can* be mechanized names its guard — the prose is then a shortcut, not the enforcement.

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

## Section order that works in production
A mature root `CLAUDE.md` in this house style, in this order — each section earns its place by changing the *next action*:

| # | Section | Holds |
|---|---|---|
| 1 | Response rules | how it talks: no preamble, lead with action, disagree when wrong |
| 2 | Coding rules | fail fast · simplicity · surgical · verifiable done · proactive · [how to say no](behavioral-rules.md#-proactive--but-about-the-problem-not-the-diff) |
| 3 | Project | what the product is, what it optimizes for, the doctrine decisions that outrank instinct |
| 4 | Monorepo table | one row per app/package: path · what · stack |
| 5 | Stack | runtime, framework, DB, queue — with the non-obvious constraint on each |
| 6 | The loop | the exact inner-loop and full-gate commands, and which to prefer |
| 7 | Commands | copy-pasteable, grouped by scope (root · per workspace · DB · infra) |
| 8 | Code patterns | layering, where SQL lives, DI, file-size limit |
| 9 | Hard boundaries | the handful of things that are never OK |
| 10 | **Preventive rules** | the bite-before-symptom list above |
| 11 | Docs index | one line per architecture doc: *read this before touching that area* |
| 12 | PR conventions | branch naming, commit format, one concern per PR |
| 13 | What NOT to build | deferred scope + **rejected** ideas, with the reason (a rejected idea keeps coming back without one) |

Sections 3, 9, 10, and 13 are what a generic "good practices" file lacks — they're the ones that stop an agent doing something locally reasonable and globally wrong.

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
