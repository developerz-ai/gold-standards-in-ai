# 🔁 Workflow

How work moves from **idea → plan → issues/milestones → autonomous PRs → review → supervised deploy** — at agent speed. A human writes a paragraph; an agent turns it into a spec and a multi-file execution plan; the plan becomes GitHub issues under a milestone; autonomous workers pick up issues and open PRs; an automated AI reviewer plus a human gate each PR; a deploy agent ships under supervision. Humans steer and review — they don't type the code.

| File | Topic |
|---|---|
| [project-kickoff.md](project-kickoff.md) | 🌱 Zero → running project, fast — spec, scaffold, the `planx` plan pattern |
| [github-issues-milestones.md](github-issues-milestones.md) | 🎫 Issues & milestones at agent speed with the `gh` CLI |

## The loop
The execution layer is [Planner → Worker → Reviewer](../ai-agents/orchestration.md): the **plan** (see `planx` in [project-kickoff.md](project-kickoff.md)) is what a Planner produces and a Worker consumes, and the docs are written the way an agent reads them ([../writing-for-agents/planning-and-docs.md](../writing-for-agents/planning-and-docs.md)). Decouple **author** from **executor**: one agent (or human) writes the plan, another implements it — with zero shared context beyond the files on disk.
