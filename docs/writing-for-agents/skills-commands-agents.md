# 🧩 Skills, Slash Commands & Subagents

Three reusable AI capability layers. Use them when a workflow repeats. They're all just markdown files in `.claude/`.

| Layer | Lives in | Is | Triggered by |
|---|---|---|---|
| **Skill** | `.claude/skills/<name>/SKILL.md` | Domain expertise package | The agent matching its `description` |
| **Slash command** | `.claude/commands/<name>.md` | A canned workflow | `/name` typed by the user |
| **Subagent** | `.claude/agents/<name>.md` | A specialist worker with its own tools/model | Delegation from the main agent |

## 🛠️ Skills — domain expertise
One skill per domain, not per task. The `description` is the trigger — pack it with keywords *and* say when to activate.

```yaml
---
name: deploy
description: Deploy app — test, build, push, verify. Triggers: deploy, ship, release.
allowed-tools: [Bash, Read]
---

# Deploy

1. `bin/test` · fail → stop
2. `bin/lint` · fail → stop
3. `scripts/ci/trigger-build production`
4. `scripts/ci/build-log` · tail until done
5. `scripts/hosting/app-status production` · verify healthy
6. Fail → `scripts/hosting/rollback production`
```

Structure for bigger skills:
```
.claude/skills/development/
├── SKILL.md       # main instructions (≤80 lines)
├── patterns.md    # supporting reference
└── testing.md     # supporting reference
```

Tips:
- `allowed-tools` enforces safety (read-only during review, no writes during debugging).
- Inside the body: **instruct, don't explain.** Numbered steps, exact commands, failure branches inline (`fail → stop`).
- A `creating-skills` meta-skill lets the agent author new skills itself.

Common categories: development (your stack's patterns), database-design, performance-optimization, deploy, dns, mail, debug-prod, use-browser (Playwright), migration-management.

## ⚡ Slash commands — workflows
```yaml
---
description: Create a structured implementation plan
argument-hint: [what you want to do]
allowed-tools: [Write, Read, Glob, Grep, Bash, Skill]
---

# /plan

1. Explore the codebase (grep, glob, read). Find patterns to follow.
2. Map affected files + dependencies.
3. Write `.plans/<feature>/plan.md`: files to create/modify, steps, tests, edge cases.
4. Each step gets `step → verify: <check>`.
```

The canonical loop:
```
/plan add Stripe payments   → explores, writes a plan
/implement                  → executes the plan, writes code + tests
/qa                         → reviews for regressions, runs suite + lint
/fix-ci                     → pulls CI logs, fixes failures
/deploy                     → build, monitor, verify health
```
Commands can invoke skills and spawn subagents internally.

## 🤖 Subagents — specialists
```yaml
---
name: backend-expert
description: Backend specialist. Models, services, controllers, migrations, tests.
model: opus
skills: [development, database-design, performance-optimization]
---

Backend specialist. This project.

## Rules
- SOLID. SRP hard.
- Services under `app/services/<domain>/`.
- Custom errors only. No generic Exception.
- Tests required. TDD where possible.
- Files ≤500 LOC. Split → nested modules.
```

Why subagents matter:
- **Parallelism** — `backend-expert` writes code while `infra-expert` handles deploy, simultaneously.
- **Isolation** — each has its own tool restrictions + skill context (and can run in a separate git worktree, no conflicts).
- **Model control** — cheap/fast model for simple work, top-tier model for hard work.

### One subagent per app — the fleet pattern
In a [monorepo](../architecture/monorepo.md), the highest-value roster is **one specialist per app/workspace**, named after the directory it owns: `api-dev`, `frontend-dev`, `jobs-dev`, `router-dev`, `docs-dev`… Each file carries what an expert in *that* app must know:

1. **What the app is and is NOT** — "the edge terminates HTTP and streams; if you're writing a business rule here, you're in the wrong workspace."
2. **How it connects** — who calls in, what it calls out, which contracts are cross-app (change one → grep those consumers).
3. **Hard rules for the area** — its layering, its error types, its quota/auth gate, its file-size limit.
4. **Its exact commands**, run from the repo root so root env files load.

Why it works: the delegation is unambiguous (a path maps to exactly one owner), the file sets are naturally disjoint for a [hive](../ai-agents/hive-mind.md), and the specialist prompt is loaded **only when that app is touched** — laziness by construction.

Add `architect` alongside them for "should this be built, and in what shape" — it produces a design with named tradeoffs, not an implementation.

### Rules for subagents that share one checkout
Agents run in the *same* working tree, so put these in every app agent file (and in the command that spawns them):

- **Scope every command to the files you edited** — never a repo-wide gate, never a working-tree-diffing `:changed` command → [../developer-experience/inner-loop.md](../developer-experience/inner-loop.md#-scoped--safe-when-agents-share-the-checkout).
- **Concurrency 1.** No `--parallel`, no raised worker counts. Saturating the box is the coordinator's job, once.
- **No git operations at all.** Leave work uncommitted; the coordinator owns git — and **never `git stash`** (one global stack, shared).
- **Need a database? Just run the test.** Store isolation is the harness's job; hand-writing a connection string is a harness bug.

### Keep the catalog lazy
Every skill's `description` sits in context on every turn; the **body** should not. Write the description as a trigger line (keywords + when to activate) and let the body load on pickup. If a skill is needed on most turns, that's the rare case for pinning it inline — decide it with an eval, not a hunch → [../ai-agents/context-budget.md](../ai-agents/context-budget.md).

## Writing rules for all three
- Persona/description = one sentence, keyword-rich.
- One rule per line, imperative. Skip `## Overview`/`## Approach`.
- Specify output format literally when it matters.
- Compress — these are read on every activation/spawn. See [compressed-config.md](compressed-config.md).
