# 🎼 Multi-Agent Orchestration

One agent loop is the atom. Orchestration composes loops into autonomous systems that turn a **goal into merged PRs** — with humans reviewing, not typing. Living example: [`ai-task-master`](https://github.com/developerz-ai/ai-task-master).

**Scope:** this is the *unattended machine you build*. Coordinating subagents yourself, in a session, in one checkout, is a different discipline → [hive-mind.md](hive-mind.md).

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

> ⚠️ **Worktrees are for THIS machine only** — an unattended orchestrator whose workers each own a branch and a PR, with no human working the tree. Inside an interactive session, **never**: one checkout, many hands, the file set is the lock. See [hive-mind.md](hive-mind.md).

### Reviewer — threads → fixes
For each unresolved review thread: decide `fixed` (make the change, push), `replied` (answer), or `wontfix` (justify) — and resolve the thread via the platform API.

### Orchestrator — the glue
Owns a `StateStore` (resumable across crashes), spawns N workers concurrently (one per group, dependencies respected), watches CI per PR (green → auto-merge or hold for human; red → Reviewer loop), and handles retries.

## Autonomous loop patterns
- **Loop-until-goal:** keep working until acceptance criteria pass.
- **Loop-until-count:** accumulate to a target (e.g. find 10 bugs).
- **State machine:** `SETUP → PLANNING → CODING → REVIEW → COMPLETE/FAILED`, with a runaway backstop sized from observed healthy max × margin ([agent-work-limits](agent-work-limits.md#unattended-runs-keep-a-backstop--which-is-the-same-rule-not-an-exception)) that escalates instead of spinning forever.
- **Mailbox:** inject plan updates while it runs (mid-run guidance from [agent-sdk](agent-sdk.md#memory-across-turns)).

## 🎯 The loop's exit — "done" a machine can judge

The loop converges or spins on one thing: its exit condition. Write it before the loop.

| Rule | Why |
|---|---|
| **Done = yes/no by one command** ("`bin/check` green AND every group has a merged PR"). Never "make it good" | A vague goal never passes, or passes at random |
| **"Done" ships with "must not"**: no test deleted/skipped/weakened, coverage not lower, no acceptance file touched | "All tests pass" alone is a license to delete tests |
| **Prefer an external oracle over self-assertion** — diff vs a known-good sample, tie-out to upstream totals, a golden file | The agent's own asserts can be loosened; an outside number can't |
| **The builder never edits the acceptance checks.** Planner writes them; Worker writes code | Grading against a moved goalpost always passes |
| **A separate checker runs acceptance** — a different agent/process, deterministic tools (tests, diff, typecheck), not "looks right" | Grading your own homework inflates → [../writing-for-agents/reviewer-agents.md](../writing-for-agents/reviewer-agents.md) |
| **Every question answered before launch** — ambiguities resolved in the plan | An unattended loop never stops to ask; it runs the wrong reading to the end |
| **Stop only after N consecutive "done" signals** (e.g. 3 iterations in a row find nothing left) | One premature "done" ends a run with work left |
| **Non-progress kills it, not a retry count** — same failure fingerprint, no new state, identical calls → stop and report ([stall rule](agent-work-limits.md#the-one-bound-that-stays-lack-of-progress)) | A cap kills honest slow work and lets fast spinning through |

The human step is **PR review** — the loop opens PRs, it does not sign off its own work. Kill the **whole process group**, never just the parent; a missed heartbeat = stall.

## ✂️ Strip the harness as models improve

Every scaffold encodes "the model can't X alone": sprint decomposition, context resets between phases, a per-step evaluator, a file-manifest pre-pass. On every model change, **re-run the [evals](evals.md) with each scaffold removed** and delete the ones no longer load-bearing. A harness built for last year's model is a tax on this year's.

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
