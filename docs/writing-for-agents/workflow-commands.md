# 🛠️ Workflow Commands — `/planx` and `/feature`

Two commands carry the delivery flow in every repo: **one writes a plan, one ships it.** Mechanics of commands live in [skills-commands-agents.md](skills-commands-agents.md); this is the pair worth standardizing on. Fan-out discipline inside `/feature` lives in [../ai-agents/hive-mind.md](../ai-agents/hive-mind.md).

## Why split planning from building
| | `/planx` | `/feature` |
|---|---|---|
| Output | files under `docs/plans/…` | merged PRs in prod |
| Touches source | never | always |
| Cost of being wrong | one exploration pass | a revert |
| Rerunnable | yes, cheaply | no |
| Consumed by | a different agent/person, later | the repo |

A plan that lives only in an agent's context dies with the session. Written to disk it becomes reviewable, correctable, handoff-able — and the expensive half becomes restartable: an agent that dies mid-`/feature` resumes from the plan files, not from zero.

**Skip `/planx` for small work.** Same judgement call as [whether to hive](../ai-agents/hive-mind.md#-when-to-hive-at-all): a one-file fix does not need a plan dir, and writing one costs more than the change.

## 📐 `/planx` — plans are files

**Plan only.** No implementation, no code execution, no edits outside the plan dir.

| Rule | Shape |
|---|---|
| **Dated, numbered dir** | `docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/` — resolve with `date +%Y %m %d`, then `Glob docs/plans/<YYYY>/<MM>/<DD>/1*` → highest `1NN-*` + 1, else `101`. Sorts chronologically, never collides. Slug kebab-case, ≤5 words. |
| **Multiple files, never one `plan.md`** | `overview.md` (the map) + one `<NN>-<aspect>.md` per separable area — `01-data-model.md`, `02-backend-service.md`, `03-api-routes.md`, `04-frontend.md`, `05-tests.md`. |
| **Each slice independently executable** | And short. Split by *area of work*, not by chapter. |
| **Reference, don't restate** | `file:line`, `Class#method`. Point at code — never paste it, never re-explain it ("follow `x.ts:12` but …"). Pasted code goes stale immediately. |
| **Self-contained** | The executor reads `overview.md`, its own slice, and the files those cite. Nothing else. |
| **No checkboxes in `.md`** | Those are reference maps. Tracking lives in exactly one place. |
| **Cross-repo work** | One `<NN>-<aspect>.md` per repo/app, and say which repo each is edited from. |

`overview.md` carries: **Goal** (1–2 sentences, what + why) · **Context** (only the stack facts the executor needs + reference patterns at `file:line`) · **Plan files, in execution order**, each with a one-line hook · **Done when** (verifiable acceptance criteria spanning the feature) · **Risks / open questions**.

Each `<NN>-<aspect>.md` carries: a `> Part of overview.md. Depends on: <NN or none>` line · **Files to change** (`path:line` — what, why) · **Steps** (ordered, concrete) · **Tests** (what to add, which command) · **Done when**.

### `status.yml` — the only tracker
```yaml
plan: 101-internal-exchange-rates
title: Internal exchange rates
status: in_progress        # not_started | in_progress | blocked | complete | superseded
created_by: sebi           # authored the plan
worked_by: ""              # executing it; empty = unclaimed; executor fills with their git user.name
owner: sebi
percent: 40
current_focus: "03-api-routes.md — rate lookup endpoint"
slices:
  - { file: 01-data-model.md, status: complete, percent: 100 }
evidence: ["#324", "abc1234"]
last_updated: 2026-07-21
```
- Machine-readable (valid YAML, the enums above) so an orchestrator can query it.
- **`created_by` ≠ `worked_by`** is what lets one person plan and another execute; `owner` is who answers for it.
- `evidence` holds commits/PRs — "80% done" becomes checkable.
- It is the **one** tracker in the dir. The `.md` slices never grow a checkbox.

## 🚚 `/feature` — idea to deployed

**Done means deployed and verified.** A green local gate is not done; an open PR is not done; a merged PR whose roll you did not confirm is not done. Report what you *verified*, not what you assume happened.

1. **Understand** — restate the goal in one line; fetch cited URLs, extract the mechanism, translate onto this stack.
2. **Distrust the paperwork** — see below. Do this *before* planning work off any plan or doc.
3. **Explore in parallel** — read-only agents map every affected surface → a ranked worklist in PR-sized batches. ([hive rules](../ai-agents/hive-mind.md))
4. **Track** — one sub-issue per slice; `Fixes #NNN` auto-closes on merge.
5. **Build primitive-first** — land one reusable primitive with its first real caller, then adopt everywhere. No abstractions before consumers.
6. **Verify** — typecheck + lint + test as the green gate; user-facing changes driven in a real browser.
7. **PR + merge sequentially** — never in parallel; each merge rebases `main` under the others.
8. **Deploy + watch** — confirm the roll landed with a live probe, not a bundle grep.
9. **Close** — verify auto-closes fired; close the parent by hand. Re-check the original symptom in prod.

**Read autonomy from the prompt.** "Just ship it" → run start-to-finish, decide everything, surface decisions in the PR body. Tentative ask → clarify the genuinely ambiguous, stop before merge. Always stop for a real blocker: irreversible prod action, data-integrity or auth risk, policy violation.

## 🔗 The seam — one continuous line

**The slice boundary in the plan IS the file-set boundary the `/feature` hive locks on, and the file set IS the PR.**

```
/planx slice          →   agent file set        →   PR
01-data-model.md          exclusive paths           one concern, ≤~110–120 files
02-backend-service.md     exclusive paths           …
```

That is why the plan split is not cosmetic. Split by *separable area of work* and the boundaries come free at build time: disjoint by construction, so no two agents ever contend, and each slice becomes a PR with no extra carving. Split by chapter and you invent boundaries the code does not have — and pay for it twice.

**Cap a PR at ~110–120 changed files**, which is what keeps the last hop reviewable:

- Automated reviewers refuse outright past a limit (CodeRabbit: >150 changed files → "Review skipped"), so the biggest, riskiest PR gets the *least* review — exactly backwards.
- A human cannot hold 279 files either. Approval becomes a formality, which is the same as no review.
- One red CI job blocks everything: a 279-file PR failing two specs holds ~90 fixes hostage; four 70-file PRs would have landed three of them.
- Bisecting a later production issue lands on one enormous commit instead of a slice.

Exceeding the cap? Split it **even if the user asked for one PR**, and say why.

## 🕵️ Distrust the paperwork

**A plan is paperwork, and paperwork rots.** `/feature` step 2 exists to catch it: before planning work off a `status.yml`, a plan doc, or an architecture doc, **check it against the code and `git log`.**

Trackers rot in **both** directions:

| Rot | Real case (2026-07) | Cost if believed |
|---|---|---|
| Says not-done, is done | `percent: 10`, six of seven slices `not_started` — the work had shipped across eight merged PRs | Re-implementing shipped work |
| Says decided, is superseded | An architecture doc recorded a decision on the same page that marked it superseded | "Fixing" working code to match a dead design |

Merged PR titles for the area are the cheapest ground truth. **State plainly which claims you falsified**, so nobody re-does shipped work — and update the doc in the same change. A doc that lies costs the next person a full re-audit.

## 🐝 Fan-out belongs to `/feature`

`/planx` explores read-only; `/feature` builds. Both fan out, both do it in **one checkout** — the coordinator owns git and the merge, every brief names its exclusive paths, and agents are re-tasked with `SendMessage` rather than replaced. Full doctrine, the 9-point brief, and the failure modes: [../ai-agents/hive-mind.md](../ai-agents/hive-mind.md).

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

Both files are per-repo. **The flow generalizes; stack names, commands, and directory layout do not.** Copying another project's `/feature` verbatim hands the agent confidently wrong build commands — adapt it or skip it. As of 2026-07 the pair ships in every active developerz-ai repo, each carrying its own real commands.
