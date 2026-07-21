# 🛠️ Workflow Commands — `/planx` and `/feature`

Two commands carry the delivery flow in every repo: one writes a plan, one ships. Mechanics of commands live in [skills-commands-agents.md](skills-commands-agents.md); this is the pair worth standardizing on.

## Why split planning from building
| | `/planx` | `/feature` |
|---|---|---|
| Output | files under `docs/plans/…` | merged PRs in prod |
| Touches source | never | always |
| Cost of being wrong | one exploration pass | a revert |
| Rerunnable | yes, cheaply | no |
| Consumed by | a different agent/person, later | the repo |

A plan that lives only in an agent's context dies with the session. Written to disk it becomes reviewable, correctable, handoff-able — and the expensive half becomes restartable: an agent that dies mid-`/feature` resumes from the plan files, not from zero.

## `/planx` — plans are files
- **Path:** `docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/`. Never a chat message.
- **Multiple files, never one `plan.md`** — `overview.md` index + one `NN-<aspect>.md` per separable area (data model, API, UI, tests). Each slice independently executable.
- **Reference, don't restate** — `file:line`, `Class#method`. Pasted code goes stale immediately.
- **Self-contained per slice** — executor reads `overview.md`, its slice, and the files those cite.
- **No checkboxes in `.md`** — those are reference maps. Tracking lives in one place only.

### `status.yml` — the only tracker
```yaml
plan: 101-internal-exchange-rates
status: in_progress        # not_started | in_progress | blocked | complete | superseded
created_by: sebi           # authored the plan
worked_by: ""              # executing it; empty = unclaimed
percent: 40
current_focus: "03-api-routes.md — rate lookup endpoint"
slices:
  - { file: 01-data-model.md, status: complete, percent: 100 }
evidence: ["#324", "abc1234"]
last_updated: 2026-07-21
```
Machine-readable so an orchestrator can query it. `created_by` ≠ `worked_by` is what lets one person plan and another execute. `evidence` holds commits/PRs — "80% done" becomes checkable.

## `/feature` — idea to deployed
1. **Understand** — restate the goal in one line; fetch cited URLs, extract the mechanism, translate onto this stack.
2. **Explore in parallel** — read-only agents map every affected surface → worklist in PR-sized batches.
3. **Track** — one sub-issue per slice; `Fixes #NNN` auto-closes on merge.
4. **Build primitive-first** — land one reusable primitive with its first real caller, then adopt everywhere. No abstractions before consumers.
5. **Verify** — typecheck + lint + test as the green gate; user-facing changes driven in a real browser.
6. **PR + merge sequentially** — never in parallel; each merge rebases `main` under the others.
7. **Deploy + watch** — confirm the roll landed with a live probe, not a bundle grep.
8. **Close** — verify auto-closes fired; close the parent by hand.

**Read autonomy from the prompt.** "Just ship it" → run start-to-finish, decide everything, surface decisions in the PR body. Tentative ask → clarify the genuinely ambiguous, stop before merge. Always stop for a real blocker: irreversible prod action, data-integrity or auth risk, policy violation.

## 🚫 No git worktrees — agents share the checkout
Fan-out is for reading and building, not for cloning the repo N times. Worktrees look like free parallelism and aren't:

- Work lands in a directory you aren't looking at — `git status` reads clean while three branches of real work sit elsewhere.
- Each worktree needs its own install, its own copied `.env*`, its own test DB. Minutes and gigabytes per agent on any real monorepo.
- Commits get stranded — remove a worktree before its branch is pushed and the work goes with it.
- Your language server, test watcher, and dev stack point at one directory. Agents elsewhere run half-blind.

One checkout, one branch at a time. Parallel agents coordinate over disjoint file sets — a scheduling problem you can see, not an isolation problem you can't. Genuine isolation (destructive migration rehearsal, a dependency bump you expect to explode) is a container or a throwaway clone, not a worktree hanging off the repo you're working in.

State the rule inside the command file, not only in `CLAUDE.md` — the agent reading `/feature` is the one about to spawn the fan-out.

## Keep them adapted, keep them short
Command budget from [compressed-config.md](compressed-config.md) is <30 lines; these two are the justified exception, but the pressure stands — link to skills and docs instead of inlining them. A `/feature` that restates the deploy runbook rots the day the runbook changes.

Both files are per-repo. The flow generalizes; stack names, commands, and directory layout do not. Copying another project's `/feature` verbatim hands the agent confidently wrong build commands — adapt it or skip it.
