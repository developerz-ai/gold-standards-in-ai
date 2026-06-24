# 🎼 Multi-Agent Orchestration

One agent loop is the atom. Orchestration composes loops into autonomous systems that turn a **goal into merged PRs** — with humans reviewing, not typing. Living example: [`ai-task-master`](https://github.com/developerz-ai/ai-task-master).

## Planner → Worker → Reviewer
```
goal ─▶ Planner ─▶ task groups (DAG) ─▶ Orchestrator ─▶ Worker ─▶ PR
         (read-only   (PR-sized,           (state +         (code +
          survey)      with deps)          concurrency)     commits)
                                              │                │
                                              ▼                ▼
                                          StateStore ◀── CI/Review ◀─ Reviewer
                                                      (auto-merge green / fix red)
```

### Planner — goal → task groups
Surveys the repo with **read-only tools** (grep, glob, read), then emits a Zod-validated DAG of ~PR-sized groups with `dependsOn` edges so independent work parallelizes.
```ts
const Plan = z.object({ groups: z.array(z.object({
  id: z.string(), title: z.string(),
  tasks: z.array(z.object({ description: z.string(), acceptance: z.string().optional() })),
  dependsOn: z.array(z.string()),
})) });
```

### Worker — group → branch + PR
Runs in an isolated **git worktree** (no branch trampling). For big PRs, two phases: (1) plan a file manifest via `submit`, (2) run per-file editors in parallel. Then format → run changed-file tests → push. The orchestrator opens the PR.

### Reviewer — threads → fixes
For each unresolved review thread: decide `fixed` (make the change, push), `replied` (answer), or `wontfix` (justify) — and resolve the thread via the platform API.

### Orchestrator — the glue
Owns a `StateStore` (resumable across crashes), spawns N workers concurrently (one per group, dependencies respected), watches CI per PR (green → auto-merge or hold for human; red → Reviewer loop), and handles retries.

## Autonomous loop patterns
- **Loop-until-goal:** keep working until acceptance criteria pass.
- **Loop-until-count:** accumulate to a target (e.g. find 10 bugs).
- **State machine:** `SETUP → PLANNING → CODING → REVIEW → COMPLETE/FAILED`, with a hard runtime ceiling that escalates instead of spinning forever.
- **Mailbox:** inject plan updates while it runs (mid-run guidance from [agent-sdk](agent-sdk.md#memory-across-turns)).

## Two-command surface, hidden complexity
Expose almost nothing:
```bash
aitm start "add password reset" --max-prs 3 --no-automerge
aitm merge-pr
```
Planning, branching, concurrency, review threads, retries — all inside those two commands.

## Provider abstraction & model tiers
Abstract behind one interface so you swap providers per tier and cost target (OpenRouter, Anthropic, **z.ai**, OpenAI — any OpenAI-compatible endpoint). A profile system works like `nvm use`:
```bash
aitm profile add zai --preset zai --api-key "$ZAI_KEY"
aitm profile use zai           # run the fleet on a z.ai coding subscription
```
Pin a model per tier — the **z.ai subscription** is a cost-effective default for the high-volume coding/worker tier, with a top Claude model reserved for planning/hard reasoning:
```json
{ "models": { "smart": "<top-claude-for-planning>", "coding": "<zai-coding-model>", "fast": "<cheap-fast-model>" } }
```

| Tier | Phase | Pick |
|---|---|---|
| smart | planning, hard fixes | latest Claude (Opus-class) |
| coding | the bulk of worker edits | z.ai coding model (volume-friendly) |
| fast | review, triage, mechanical | cheap fast model |

## Cost & safety
- **Track `totalUsage` per phase**, attach a cost table to the PR body.
- **Retry transient, fail fast on credits-exhausted** ([agent-sdk](agent-sdk.md#streaming-retries-cost-control)).
- **Read the repo's `CLAUDE.md`/`AGENTS.md`** and feed it to every sub-agent as a system-prompt prefix — the agent inherits your conventions for free.
- **Deployments stay supervised** (the "Ana" pattern) — see [../00-philosophy.md](../00-philosophy.md) and [../infrastructure/README.md](../infrastructure/README.md).
