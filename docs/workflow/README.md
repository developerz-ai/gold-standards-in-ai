# 🔁 Workflow

How work moves from **idea → plan → issues/milestones → autonomous PRs → machine gates → deploy** — at agent speed. A human writes a paragraph; an agent turns it into a spec and a multi-file execution plan; the plan becomes GitHub issues under a milestone; autonomous workers pick up issues and open PRs; CI and an automated AI reviewer gate each PR; the merge is the deploy consent and the agent verifies the roll. Humans bring ideas and report problems — they don't type the code or approve it.

| File | Topic |
|---|---|
| [project-kickoff.md](project-kickoff.md) | 🌱 Zero → running project, fast — spec, scaffold, the `planx` plan pattern |
| [project-from-zero.md](project-from-zero.md) | 🧭 The greenfield order — docs first, then tooling, then code; the stack proposal |
| [bug-sweep.md](bug-sweep.md) | 🧹 Bug sweeps on a cadence — find the defects yourself, ship them big, prove each in prod |
| [dz-contract.md](dz-contract.md) | 🤖 The `.dz/` contract — policy file, briefs and their caps, pinned in CI |
| [github-issues-milestones.md](github-issues-milestones.md) | 🎫 Issues & milestones at agent speed with the `gh` CLI |
| [shipping-doctrine.md](shipping-doctrine.md) | 🚢 No legacy, no shims, no feature flags, no prod A/B — one path, replaced completely |
| [opportunistic-improvement.md](opportunistic-improvement.md) | 🎯 Take the opportunity a paid cost opens — fold in what has a switch back and a gate, file the rest |
| [data-risk-ladder.md](data-risk-ladder.md) | 🪜 Stop writers, prove zero writes, back up, restore-test, switch — never lose data, fix forward |
| [decision-records.md](decision-records.md) | 🧭 One record per one-way door — rejected options written down so agents stop re-proposing them |

## The loop
The execution layer is [Planner → Worker → Reviewer](../ai-agents/orchestration.md): the **plan** (see `planx` in [project-kickoff.md](project-kickoff.md)) is what a Planner produces and a Worker consumes, and the docs are written the way an agent reads them ([../writing-for-agents/planning-and-docs.md](../writing-for-agents/planning-and-docs.md)). Decouple **author** from **executor**: one agent (or human) writes the plan, another implements it — with zero shared context beyond the files on disk.
