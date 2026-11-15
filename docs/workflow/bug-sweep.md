# 🐛 Bug Sweep — find the defects yourself, ship them big

`/feature` is handed one stated fix. **A sweep is handed a goal or an area and finds the work itself.** The deliverable is merged PRs, each deployed and proven in production — a ranked list of findings, a plan or a proposal is a failed sweep.

## Cadence — schedule it, never wait for it

Fix work overtakes feature work within a month or two of a build phase (as of 2026-11, `git log` of one product monorepo: fix commits outnumbered feature commits 1.7:1 overall and 4:1 in its two heaviest months). A sweep that happens by accident happens late.

| Trigger | Sweep | Scope |
|---|---|---|
| A feature merge window closes | post-merge sweep | the area the merges touched — defects hide one hop outside the diff |
| Weekly | scheduled checkup: leaks, slow algorithms, duplicated helpers, missing indexes, drifted docs, dead code | the whole repo, from a committed brief → [dz-contract.md](dz-contract.md) |
| Overnight, on demand | full campaign | the whole platform or one named area |
| After an incident | class sweep | every file that shares the incident's shape, not just the one that fired |

## The operating contract

| Rule | Because |
|---|---|
| No check-ins, no questions, no progress pauses | scope, severity, slicing, merges and rollbacks are the sweep's calls; each goes in the epic issue and the PR body |
| Never halt on a failure | route around it (§ Running unattended) and keep shipping |
| Nothing rationed — no step, turn or token budget | deep over three areas beats shallow over ten → [../ai-agents/agent-work-limits.md](../ai-agents/agent-work-limits.md) |
| Prove the mechanism before changing a line | a plausible patch over an unproven diagnosis leaves the defect alive under a fix that looks done, and nobody re-audits a recently touched file |
| A clean area reported clean is a result | "nothing breaks by leaving it" → leave it and say so |

## The target resolves to a path set first

Print it with its file count before anyone is dispatched. **A lane name expands to the whole lane**, never the same-named folder: "billing" is the billing package + its webhook route + its API routes + the views that render it. A too-narrow glob is how an area gets declared clean with the defect one hop outside it.

## Shape of a campaign

```
campaign (the goal)
└─ area                    ← cut on failure boundaries, contracts first; one at a time
   ├─ field walk           ← the coordinator, on the ground
   └─ PR                   ← big, goal-scoped
      ├─ waves of agents   ← parallel, path-disjoint: find + fix + test, no execution
      ├─ coordinator gate  ← lint, typecheck, tests, anti-vacuity — one holder
      └─ merge → deploy → prove → re-measure
   └─ sweep-up PR          ← the gate's fallout, the collisions, the homeless findings
   └─ …                    ← until a wave finds nothing new
```

The hive mechanics — file set as the lock, waves, the brief, interrogating the report — are [../ai-agents/hive-mind.md](../ai-agents/hive-mind.md). Plan area 1, not the campaign: a class confirmed in one area becomes a grep across the rest.

## Phase 1 — Census

```bash
git ls-files <resolved paths> | wc -l
git log --oneline -40 -- <resolved paths>        # cheapest ground truth for an area
gh issue list --search "<area>" --state all --limit 40
```

**Distrust the paperwork** — check every doc claim you act on against the code and `git log`; each falsified claim ships as a docs fix → [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md). Default area order: shared contracts (domain, DB, policy) → inbound → dispatch and jobs → workers → the agent lane → the public API → front ends → tooling and gates.

## Phase 2 — Walk the field, take the baseline

The coordinator's one reading phase: the failure's shape and the seams to cut on, not every bug.

- **Take the production baseline and write it down** — logs, a read-only DB query, the live endpoint. Phase 6 re-runs the same commands.
- **Measure with a denominator**: `N of M`, never a bare count.
- **Trace one real request end to end** (a webhook, a job, a page load). The hop with no observable is usually the defect.

## Phase 3 — One PR, divided across agents

**A PR is a goal, not a folder**: "fix the job that runs on the worker box" is ONE PR spanning the claim path, the transport, the credential seam and the stuck rows. Contracts land first, consumers after. Size and in-flight rules → [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md).

Every brief adds to the hive brief:

| Line | Why |
|---|---|
| the bar: prove the mechanism, fix the cause, delete rather than abstract | "I could not reproduce it, here is what I ruled out" beats a guess |
| the production read path and the area's baseline | count the rows your code claims to write before believing it works |
| cleanup in the same slice; permission to fix a worse bug found beside the target | nobody re-audits a recently touched file |
| it runs no tests, lint or typecheck; it writes to no production system | the gate is a mutex with one holder; unlocked writers race each other's reversal |
| a bounded report: fixed, dropped and why, premises falsified, each test with its assertion | no file dumps, no diffs, no narration |

## What to hunt — run the detector, do not just read

| Class | Detector |
|---|---|
| **Dead producer** — mechanism complete, observable hop missing, everything green | grep the consumer, then count its production rows; zero for a shipped feature is the finding |
| **Guard that proves nothing** — substring match, hard-coded list, vacuous assert | break the guarded thing; a guard that stays green is the bug |
| **Silent degrade** — invalid config capped, defaulted, fallen back | trace a bad value from the seam it enters |
| **Surviving flag / legacy branch** | there should be none → [shipping-doctrine.md](shipping-doctrine.md) |
| **Short-lived helper given a process lifetime** | a per-job cache or client at module scope in a long-running daemon |
| **Refusal collapsed into `null`** | three reasons returning one `null`; wants a discriminated verdict |
| **Per-item query / unbounded read** | a query inside a loop; a list read with no limit or window |
| **Doc asserting a guarantee the code does not provide** | especially a universal with no enumeration — produce the counterexample |
| **Shape defect** — offset paging, missing idempotency, no retention | outranks any capacity concern → [../architecture/data-and-scale.md](../architecture/data-and-scale.md) |
| **Retention that stopped running** | a growing table whose prune job exists = a dead producer |
| **Missing test** | where the untested branch can actually be wrong |

## Every fix ships its guard

| Bar | Rule |
|---|---|
| Test | the test that would have caught it, in the same PR |
| Anti-vacuity | revert the fix, run the test: **red**; restore: green. Green with the fix reverted is green over the bug |
| Guard | a class that can recur gets a mechanical guard wired to a command that runs; seen once → fix the instance and write the trip condition down → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md) |
| Refactor | only on evidence: the class already recurred, the fix needs three files, the invariant has no single seam — then the refactor IS the fix, as its own PR |
| Cleanup | dead code, dead branches, stale comments, bare `TODO`s and agent leftovers ship in the same PR as the fixes |

Decide on three horizons, and name the one that was not obvious in the PR body:

| Horizon | Question | Consequence |
|---|---|---|
| Short — tonight | does it fix the failing thing, and does one command revert it? | both yes → ship |
| Medium — next month | will this come back under another file name? | the guard lands either way |
| Long — irreversible | a stored shape, public contract, id scheme? | spend the hour now → [decision-records.md](decision-records.md) |

## Phase 4 — The gate, one holder

1. Cross-link the reports: three findings across inbound → queue → worker are often one bug seen three times.
2. Sweep the leftovers (scratch tests, debug logging, stray probes).
3. Lint, typecheck, the full test suite — once, on the whole PR.
4. Anti-vacuity on the highest-severity fixes (above).
5. A reviewer reads the diff before the PR opens → [../writing-for-agents/reviewer-agents.md](../writing-for-agents/reviewer-agents.md).

## Writing to production

| Step | Rule |
|---|---|
| Backup gate | before anything not key-bounded and reversible, verify a **completed** backup inside its schedule window — a scheduled job object proves nothing |
| Before | `SELECT` the exact rows with a count; no count, no write |
| Reversal first | write the reversal SQL before the forward SQL; both go in the PR body |
| Bounded | key-bounded `WHERE`, one statement, one transaction |
| After | `SELECT` the same rows; before/after in the PR body |
| Fix the writer | the code that wrote the bad row changes in the same PR; a hand-written `DELETE` is a bug report about a missing prune job |

## Phase 5–6 — Merge, deploy, prove, re-measure

**Done means deployed and verified** → [shipping-doctrine.md](shipping-doctrine.md). Per PR: confirm the running version moved, re-run the Phase 2 baseline, put before/after in the epic. A 200 is not an answer — check the body. Then the sweep-up PR, then **keep going until a wave finds nothing new**, close the area, take the next.

## Running unattended

| At 3am | Do |
|---|---|
| An agent dies mid-edit | its work is uncommitted in the tree; resume that agent with what survived — never a fresh agent on its paths |
| The gate is red, cause unclear | bisect by slice; drop the offending slice to the sweep-up PR, ship the rest |
| A merge breaks production | roll back first, diagnose second; forward-fix as its own PR |
| A production write went wrong | run the reversal you wrote first |
| A finding "needs a human decision" | decide, log the reasoning, ship; defer only a step that spends money, with the exact ask |
| Context running low | update the ledger on disk and continue — never end the campaign to report progress |

## The ledger and the report

Keep a ledger on disk (scratch, never the checkout), updated after every wave: areas with baselines, PRs with deploy status, findings with `file:line` + class + evidence, production writes with reversals, falsified doc claims, deferred items with why. Mirror it into the epic issue.

```
Per area —
  <area>     PRs:       <n> merged, <n> deployed, <n> verified
             Root cause: <one-line mechanism, where findings shared one>
             Fixed:     <n> findings -> #… #…
             Cleaned:   <dead code deleted, branches removed, docs corrected>
             Guards:    <test / lint rule / assertion, or none>
             Deferred:  <n> — what, and why not now
             Baseline:  <before> -> <after>
```

**Fixing 40 of 90 findings is a success only if the other 50 are named.**

---

**Related:** [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md) — `/feature`, PR sizing · [dz-contract.md](dz-contract.md) — the scheduled checkup brief · [../ai-agents/evals.md](../ai-agents/evals.md) — measuring before and after
