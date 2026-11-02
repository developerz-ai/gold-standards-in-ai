---
description: Plan a doc set with /planx into path-disjoint slices. Use for planx, doc plan, status.yml, plan dir, slices.
---

> House skill — established by dz-runner setup mode from this repo's own evidence, then owned by its maintainers. Correct it in place; setup never overwrites a skill that exists.

1. Resolve the plan path: `date +%Y`, `date +%m`, `date +%d` -> `docs/plans/<YYYY>/<MM>/<DD>/`; next number = highest `1NN-*` + 1, else `101`; slug = kebab-case, max 5 words.
2. Explore read-only over `docs/<section>/`: which section owns the topic, what already covers it (`file:line`), new file vs revision vs delete, which READMEs need a row. fail -> stop: a duplicate needs a revision, not an addition.
3. Write `overview.md` plus one `<NN>-<aspect>.md` per doc or cluster — never one big `plan.md`.
4. Write `status.yml` in the plan dir: `plan`, `title`, `status` (not_started | in_progress | blocked | complete | superseded), `created_by`, `worked_by`, `owner`, `percent`, `current_focus`, `slices`, `evidence`, `notes`, `last_updated`.
5. Plan only — write nothing outside the plan dir; the executor needs nothing but the plan and the files it cites.
6. No worktrees in the plan: never a `git worktree` step, never `isolation: worktree`. Parallel slices = one doc per slice, disjoint paths; README index rows are edited last, by one agent.
