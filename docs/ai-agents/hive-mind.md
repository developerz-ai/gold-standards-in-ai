# 🐝 Hive Mind — many agents, one checkout

A big task is not one agent doing more. It is a **team sharing one working tree**, with you as coordinator. The file set is the only lock.

This is the *interactive* pattern — you, in a session, spawning subagents (`Agent`, `SendMessage`). For a built orchestrator that turns a goal into PRs unattended, see [orchestration.md](orchestration.md); that machine spawns process-level Workers on their own branches and the rules below do not apply to it.

## 🎯 When to hive at all

Hiving is a judgement call, not a ritual. **Two things justify it. Nothing else does.**

| Justifies a hive | Why |
|---|---|
| **Searching** — a broad sweep across many files/dirs/naming conventions | You want the *conclusions*, not the file dumps. Reading it yourself burns the one context that must survive. |
| **Scale** — enough independent, **path-separable** work that serialising it costs hours | Real parallelism, no shared file, no shared lock. |

| Do it yourself | Why |
|---|---|
| A single-file fix | Briefing + collision management + report-reading costs more than the change. |
| One bug with one obvious home | You already know the file. Fan-out adds a round trip and a report. |
| A change you already understand | Nothing to discover; a hive only re-derives it. |

You pay the overhead **in the coordinator's context** — the one participant that must survive to the merge. Three agents on a two-file change is a net loss, every time.

## 🚫 Never git worktrees

**One checkout, many hands.** Never pass `isolation: "worktree"` to the `Agent` tool, never `git worktree add`, never per-agent directories.

- The tree fragments: `git status` reads clean while real work sits in a directory you are not looking at — and the final gate never sees it.
- Each worktree costs a full dependency install + copied `.env*` + its own test DB. Minutes and gigabytes **per agent**.

Full reasoning (and the genuine-isolation exceptions) in [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md#-no-git-worktrees--agents-share-the-checkout). State the rule inside the command file that spawns the fan-out, not only in `CLAUDE.md`.

## 👑 The coordinator coordinates; it does not code

- Owns **git, the ledger, and the merge**. Nobody else runs a git command.
- The **only participant that must survive to the end** — so spend that context on routing and judgment, not on reading files an agent will report back.
- Editing app code yourself means you took a slice from someone who had room for it.
- Keep the ledger visible (`TaskCreate` / `TaskUpdate`) so ownership survives a context handoff.

## 🔒 The file set is the lock

- Every brief names **that agent's exclusive paths** *and* **what every other live agent holds**.
- An agent needing a file it does not own **stops and reports the collision**. Never edits across the line. Never negotiates peer-to-peer.
- You mediate: hand the change to the owner, or re-cut the boundary.
- **Two agents that must edit one file are ONE slice**, not two. Combining is honest; splitting invents a boundary that does not exist.
- Branch **before** the fan-out, while the tree is clean. Nobody should ever be writing into `main`.
- Multi-surface sweep: land **one reusable primitive first**, then every surface adopts it. Never convert N surfaces N ways.

## 🧬 Agents are long-lived teammates, not one-shot jobs

- New work in an area someone holds → `SendMessage` to **that** agent. It keeps its context, its reasoning, and its file lock.
- A second agent on the same paths = **two writers and a lost fix**.
- A mid-run user report (a console trace, a transcript) is *confirmed in production* and outranks read-only findings — route it to the agent that already owns those files.

## 🌊 Work in waves

```
wave 1: explore  ─▶ reports ─▶ wave 2: fix ─▶ reports ─▶ wave 3: assemble
        (read-only,          (each wave re-tasks the next)
         disjoint areas)
```

**Do not plan wave 3 before wave 1 reports** — it will be wrong. Wave 1's findings decide wave 2's slices.

## ✅ Who runs which checks

**An agent checks only its OWN files. Whole-repo green is the coordinator's job — once, at the end, in the background.**

| | Agent (per iteration) | Coordinator (once, at the end) |
|---|---|---|
| lint/format | the formatter/linter, **on the paths it edited** | the repo-wide lint task |
| tests | **its own test files, named explicitly** | the full suite, in the **background** |
| typecheck/compile | project-wide by nature — run it **once, when otherwise done** | covered by the full gate |

- Never N agents running the full suite. That is the single biggest time sink in a parallel run.
- **Concurrency 1, per agent.** No `--parallel`, no raised `--concurrency`, no multi-worker test flag. Five agents each fanning out across the cores oversubscribes the box, and the timeouts that follow read as *real test failures*. Saturating the machine is the coordinator's job, once, at the end.
- **Working-tree-diffing "changed"/"affected" commands are WRONG inside a hive.** They diff the tree against `origin/main` — and that tree holds *every* agent's uncommitted work. Each "scoped" run expands to nearly the whole repo, N of them run at once, and they contend on one test DB. As of 2026-07 this turned 2-minute checks into 30-minute ones on a real sweep. Use an explicit path-scoped command instead (`check <files>`, `test <file>`).
- **Shared fixtures bite.** One test DB, one port, one build/`target/` dir, one browser profile: concurrent suites `TRUNCATE` under each other and surface as wandering failures naming tables the suite never writes — even a mid-run auth 401. Read any such failure with this in mind *before* believing it, including in your own final gate.
- **Give each agent its own fixture — it is usually nearly free.** Postgres clones a migrated database via `CREATE DATABASE … TEMPLATE` in milliseconds; Redis-wire stores hand out numbered DBs. Put the per-agent `DATABASE_URL` / port / dir **in the brief**, rather than letting N agents share one and clear it under each other. Cheaper than debugging the wandering failure once.

## 💥 The failure modes, stated as rules

- **Every slice you NAME, you must DISPATCH.** Briefs tell each agent which teammates are live on which paths — so a named-but-unlaunched slice makes agents dutifully defer work to someone who does not exist, and it vanishes. *This really happened:* three briefs referenced an "agent C" that was never spawned; two finished agents left it a combined six items. Keep the roster and the dispatched set as **one list**, and reconcile them **before** reading reports.
- **Reserve an UNOWNED bucket, and expect to fill it mid-run.** The real fix often lands in a file no slice covers — a shared transport, a composition root, another workspace entirely. A homeless finding is the one most likely to be quietly dropped. When a report says "the real fix is outside my set", **assign it immediately**; do not file it.
- **Look for causal chains across reports.** Agents see their own surface; only you see all of them. One run: a missing tool in a background lane removed the very tool an agent used to *discover a response shape*, so it guessed wrong, and every dashboard tile then failed with a different error a *different* agent was independently investigating. Neither could see it. After the reports land, spend one pass asking **"does A explain B?"** — it changes what you fix and what you can drop.

## 📋 The 9-point agent brief

Omitting any one is how a run goes wrong.

| # | The brief must carry | Failure if omitted |
|---|---|---|
| 1 | Its **exclusive file set** + never edit outside it | Two writers, a lost fix |
| 2 | **Which other agents are live, on which paths** | Collisions get silently resolved instead of reported |
| 3 | Each finding: `file:line` + one-sentence defect + **concrete failure scenario** (inputs → wrong outcome) | Agent re-derives the bug, or fixes the wrong thing |
| 4 | **Permission to DROP findings the code contradicts** | It "fixes" working code to satisfy your brief |
| 5 | **Evidence first, diagnosis second** — symptom, fingerprint, failing input; *then* your hypothesis, explicitly labelled **unverified**, to confirm or kill before building | Confident briefs send agents to the wrong file. In one run 2 of 3 headline hypotheses were falsified, and both were stated too confidently to be cheap to abandon |
| 6 | The **house constraints binding its area** (from `CLAUDE.md`) | Work that fails the gate at merge time |
| 7 | **Tests ship with the code, failure case first** | A "fix" nobody can prove |
| 8 | **Checks narrowed to its own files**; never a repo-wide suite | N full gates, contention, 30-minute loops |
| 9 | **No git operations at all** — no branch, commit, checkout, stash. Work is left uncommitted | Interleaved commits, stolen branches, a shared stash stack |

**Never tell an agent to "ask me" — it has no channel to the user.** A question is a dead end: it blocks or it guesses. Give it exactly two legal moves:

| Move | When | What it does |
|---|---|---|
| **Decide and flag** | Proceeding either way is recoverable | Acts on the most defensible reading, states the assumption in its report, marks the artifact so you can overwrite it |
| **Stop and report** | Proceeding either way would be unsafe or wasted | Returns with the evidence |

Then *you* take the question to the user and re-task the agent with `SendMessage`, which resumes it with full context.

## 🤜 Expect the hive to contradict you

Briefs built from a research sweep contain claims the code disproves. A good agent reports **"your premise H1 is false, here is the line."** Drop the premise — that is the agent working correctly, not going off-task.

**Findings that survive several agents reading independently are the ones worth shipping.** Distrust the paperwork the same way: check plan docs, status files and architecture docs against the code and `git log` before planning work off them, and state plainly which claims you falsified.

## 🧾 Report the deferred

A sweep that fixes 40 of 90 findings is a success **only if the other 50 are named**. Never omit the deferred line.

---

**Related:** [orchestration.md](orchestration.md) — the unattended Planner/Worker/Reviewer machine · [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md) — `/planx` + `/feature`, where this doctrine gets written into the command file · [../writing-for-agents/skills-commands-agents.md](../writing-for-agents/skills-commands-agents.md) — subagent mechanics
