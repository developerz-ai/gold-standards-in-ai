# 🏗️ Project From Zero — docs first, then tooling, then code

The **order** of a greenfield start. What each artifact contains — the spec sections, the scaffold, the `planx` plan — is [project-kickoff.md](project-kickoff.md); this page is the sequence and the stack proposal. Taken from how one org bootstraps its own repos (as of 2026-11: 6 of 8 app repos opened with a spec of 18–26 pages and no code).

## The sequence

Each step is a first task with a done-condition, not a state machine. Do not start step N+1 on a red step N.

| # | What lands | Done when | How |
|---|---|---|---|
| 1 | **The spec, no code**: `docs/idea/` one page per subsystem (overview, architecture, data model, API, deployment, stack, security, principles, roadmap) + `README.md` + `CLAUDE.md` (`AGENTS.md` a pointer to it) | an agent can answer "what are we building, for whom, on what" from the pages alone | [project-kickoff.md](project-kickoff.md), [../writing-for-agents/claude-md.md](../writing-for-agents/claude-md.md) |
| 2 | **The one-way doors as numbered decisions**: stored shapes, id scheme, tenancy, the stack | every open question that blocks a one-way door is answered or marked `ASSUMPTION:` | [decision-records.md](decision-records.md) |
| 3 | **Agent tooling**: `.claude/{commands,agents,skills}` (`/planx`, `/feature`, `/bug-sweep`, a verify skill), lint config, one verify command, `.mcp.json`, `.dz/` | `bin/setup && bin/check` runs on the empty tree | [../writing-for-agents/skills-commands-agents.md](../writing-for-agents/skills-commands-agents.md), [../developer-experience/dx-scripts.md](../developer-experience/dx-scripts.md), [dz-contract.md](dz-contract.md) |
| 4 | **CI green on the empty scaffold** | the same `bin/check` runs in CI and passes | [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) |
| 5 | **A deployable shell per app**: container, health route, deploy pipeline — before any domain logic | the shell is live and its health route answers in the target environment | [../infrastructure/README.md](../infrastructure/README.md) |
| 6 | **Plans in bulk**: `docs/plans/YYYY/MM/DD/1NN-slug/` per milestone, slices + `status.yml` | every first-milestone slice is executable by an agent with no extra context | [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md) |
| 7 | **Switch from direct-to-main to PRs**; features land titled by plan number | branch protection on; the gate is the merge condition | [github-issues-milestones.md](github-issues-milestones.md) |

Why this order:

- **Spec before code** — the agent's first code is shaped by whatever it reads first. Read nothing and it invents an architecture per file.
- **Tooling before features** — a gate added after 40 PRs grades 40 PRs of debt at once; a gate on the empty tree grades every PR from the first.
- **Deploy before domain** — the infrastructure surprises (ports, secrets, health checks, migrations at boot) arrive on a 20-line app, not inside a feature.
- **Plans in bulk** — one planning pass over the whole first milestone finds the shared contracts; plans written one feature at a time re-discover them per feature.

## The stack proposal

The spec's `docs/idea/stack.md` **proposes** a stack; it never silently picks one.

| Rule | Because |
|---|---|
| The user's stated stack always wins | the proposal answers a question nobody asked otherwise |
| Default to the house stack → [../architecture/tech-stack.md](../architecture/tech-stack.md) | an agent scaffolds without re-litigating every layer |
| Each choice carries its reason **and one alternative** with why not | a choice with no rejected option gets re-proposed next session |
| Fit-check before proposing a full-stack framework: not for a library, CLI, mobile app, non-TypeScript codebase or an existing repo | a web framework's scaffold is dead weight in any of those |
| When a scaffold generator is chosen, steps 3–5 come from **its** scaffold and its verify command becomes `bin/check` | hand-writing what the generator emits is two sources of truth |
| Pin the generator's version; read its `llms.txt` / CLI reference for the scaffold command | an assumed command is a guessed command |
| The stack is a one-way door → a decision record | step 2 |

```markdown
# Stack

| Layer | Choice | Why | Alternative, and why not |
|---|---|---|---|
| Runtime | Bun 1.3 | one toolchain front and back; built-in test runner | Node 24 — second package manager, slower test loop |
| API | Hono | sub-100 ms cold start; typed routes | Express — untyped, middleware-era |
| Data | Postgres 18 + Drizzle | schema-first, typed end to end | a document store — the data is relational (see data-model.md) |
| Web | SolidJS | small runtime, fine-grained reactivity | React — larger runtime, no requirement it serves |

Stated by the user: none. Deviations from the house stack: none → no decision record needed beyond 001-stack.
```

## Greenfield or existing — decide from the tree

| Signal | Greenfield | Existing |
|---|---|---|
| commits / source files | a handful / none | history and code |
| Where pages come from | the user's ask | **what is there** — never invent an architecture the code does not have |
| First tasks | spec → tooling → CI → shell, in order | fill the pages from the survey, then the tooling gaps |

## No placeholder pages

Every page the sequence writes is **filled** — short is fine, empty is not. A `TODO` section, a `<fill in>`, a template heading over nothing: an agent reads each as an instruction or as a fact.

- Gate it: a grep in `bin/check` for the placeholder markers you use, plus a link check that every path a page cites exists ([../writing-for-agents/output-completeness.md](../writing-for-agents/output-completeness.md)).
- Not known yet → `ASSUMPTION:` with what would change it, or leave the section out.

## Anti-patterns

| Anti-pattern | Cost |
|---|---|
| First commit is `npm create …` output with no spec | every later agent reverse-engineers intent from scaffold defaults |
| CI added "once there is something to test" | the first CI run grades a backlog |
| Domain logic before a deployable shell | the first deploy debugs infrastructure and a feature at once |
| One plan per feature, written just in time | shared contracts discovered three times, three ways |
| A stack chosen in chat, never written | the next session proposes a different one |

---

**Related:** [project-kickoff.md](project-kickoff.md) — the spec, scaffold and `planx` templates · [../architecture/monorepo.md](../architecture/monorepo.md) · [../writing-for-agents/planning-and-docs.md](../writing-for-agents/planning-and-docs.md)
