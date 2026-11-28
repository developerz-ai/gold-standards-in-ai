# 🪙 AI-Maintainer Token Spend

An AI maintainer's bill is cut by **removing work that should never start**, never by capping work that should finish. Measure by lane first: the lane everyone suspects is rarely the big one. Numbers are one fleet's maintainer runs over 30 days, measured 2026-11-28 from the run records; measure yours before acting on them.

Why a cap is the wrong tool → [agent-work-limits.md](agent-work-limits.md). Harness-level savings inside a run → [harness-techniques.md](harness-techniques.md). The CI half of the bill → [../developer-experience/cheap-cicd.md](../developer-experience/cheap-cicd.md).

## 📊 Measure by lane first

54,188M input tokens in 30 days, over 90% of them cache reads.

| Lane | Input tokens | Share |
|---|---|---|
| Issue-driven coding | 20,182M | 37% |
| PR shepherding — fix rounds on open PRs | 13,340M | 25% |
| Survey scouts | 7,595M | 14% |
| Platform care | 5,051M | 9% |
| Code review | 4,070M | 8% |
| Scheduled checkups | 2,654M | 5% |
| Setup | 1,124M | 2% |
| Triage | 64M | 0.1% |

The sweep started from "the schedules waste tokens". True, and small: the schedule waste measured **1.3%** of tokens. The big pools were PR shepherding and failed coding runs. Cut by the table, not by the hunch.

| Per lane, record | Decides |
|---|---|
| Tokens, split input / cache-read / output | where the bill is |
| Outcome per run: merged, filed, no-op, failed, cancelled | which tokens bought nothing |
| The input each run started from (PR head, check states, thread ids) | whether a run was a repeat |

## ✅ Keep review

Code review cost **0.33M tokens per review with 2% wasted**, and it is the lane people value. It is the last thing to cut and the first thing to protect when a key runs short.

## 🗑️ Kill the schedule that finds nothing

Score every schedule by **product merges per firing**, over a window long enough to hold a bad week.

| Schedule | Firings (30 d) | Result | Verdict |
|---|---|---|---|
| daily security / release check | 170 | 3 product merges + 17 PRs that only rewrote a report file to say "no release" | deleted |
| daily health check | 270 | 8 asks filed, 0 merges | deleted |
| improvement pass | 59 | 25 product merges | kept, slower cadence |
| hygiene pass | 10 | 45 product merges | kept |
| docs-drift lane | 47 tasks | 8 docs-only merges, 33 failed | turned off |
| advisory watch (14 d) | 401 tasks | 240 failed or cancelled | off — the owner had already switched it off, and an owner's setting outranks a default |

| Rule | Fails when |
|---|---|
| **A no-op verdict ships nothing** — no PR, no commit, no report rewrite | a "nothing to release" run merges a dated report into the repo every day |
| A schedule with no merge on record gets slower or goes | it stays because it is cheap per firing |
| A lane that fails most of its runs is turned off before it is tuned | a stream of "fix stale citation" PRs trains reviewers to skim |
| Daily only with a record of merges | "daily" was the template default |

## 🔁 Kill the same-input rerun

The largest removable pool. PR shepherding ran **453 fix rounds on 146 PRs — 1,491M tokens, ~$834 in 30 days** — repeating the same action on an **unchanged PR head**, after a round that had pushed nothing. Worst case: 33 rounds in two days on one review thread.

| Rule | Mechanism |
|---|---|
| **No new round until an input changes** | fingerprint what the round reads — head SHA, check conclusions, unresolved thread ids; same fingerprint as the last round that changed nothing → no run |
| A round that pushed nothing records why | the next wake-up reads the reason instead of re-deriving it |
| A blocked item waits on an event, not a poll | the wake-up is the push, the check result, the reply |

This is the [lack-of-progress bound](agent-work-limits.md#the-one-bound-that-stays-lack-of-progress) applied across runs: identical input, no new state. It names a failure, not a bill.

## 🔂 Retries are not the main waste

Issue-driven coding spent 37% of its tokens on runs that failed or were cancelled. The reflex is "stop retrying". The data says otherwise:

| Attempt | Measured |
|---|---|
| Below the third | a retry succeeds 53–60% of the time |
| The third | 2 of 157 merged |

Keep early retries. By the third attempt on the same input the run is a same-input rerun: change what it starts from — the failure, what was tried, a narrower task — or hand it up with that record.

## 🗓️ Cadence — weekly, fortnightly, monthly

Schedule keys → [the `.dz/` contract](../workflow/dz-contract.md#the-policy-file).

| Schedule | Cadence | Config |
|---|---|---|
| Release-due check | weekly, Monday — **only in a repo that publishes** | `days: [mon]` |
| Improvement pass, active repo | fortnightly | `days: [mon]` + `weeks_of_month: [1, 3]` |
| Improvement pass, idle repo | monthly | `weeks_of_month: [1]` |
| Survey of an idea-stage repo | off | nothing is broken in a repo with nothing running |
| Any daily checkup | only with a record of merges | — |

```yaml
schedules:
  - name: improve
    days: [mon]
    weeks_of_month: [1, 3]     # week n = days 7n-6..7n, UTC: the 1st and 3rd Monday
    hours_utc: [8]
    brief_file: .dz/schedules/improve.md
```

Put the cadence in the schedule, not in the brief. A brief that opens "act only on the first Monday" still fires — and bills — the other three.

## 🧯 Why a cap is the wrong tool

[Never cap the work](agent-work-limits.md#-freedom-never-cap-the-work) stands; this page is how to spend less without breaking it.

| A token cap | Removing the work |
|---|---|
| bills twice: the tokens before the cut, then the rerun | bills zero: the run never starts |
| stops real work mid-task and escalates it to a human | touches only runs that had nothing to do |
| cannot tell a 33rd identical round from a hard migration | separates them by input and outcome |

Every cut on this page is one of three: a **same-input rerun**, a **no-op schedule**, a **noisy lane**. None limits how much a run that should exist may do.

## 🎲 Flaky schedulers feed agents

A red scheduled job is an instruction to whoever watches it. A nightly determinism job failed **4 of 40 runs**; each red became a fix task: **547M tokens on 9 tasks**, including a 795-line download mirror written to "fix" a single HTTP 504.

| Rule | Because |
|---|---|
| Re-run before any model reads the failure | deterministic before AI → [harness-techniques](harness-techniques.md#the-techniques-ranked) row 16 |
| A transient — 5xx, timeout, rate limit — is never a code task | the "fix" is new code guarding against a failure that already went away |
| Fix the flake at its source, or take the job off the schedule | every red is billed twice: the run, then the agent |
| The fix task carries the failure history | "1 of the last 10" reads differently from "red" |

The CI side of the same job → [cheap-cicd](../developer-experience/cheap-cicd.md#-crons).

## 🌊 Bursts and shared keys

A fleet-wide sweep — ~10 agents, ~60 PRs open at once — is its own load test.

| What broke | Rule |
|---|---|
| The shared inference keys saturated; unrelated runs stalled behind the sweep | stagger a fleet-wide sweep; a key shared with production is production capacity |
| One rate-limited post to the code host froze review delivery for every tenant, ~5 min per throttle: the client slept through its retries inside a serial drain | a throttled call re-queues itself; it never sleeps in a shared queue |
| Agents polled "still blocked?" every few minutes | hand the wait to a token-free shell loop that wakes the agent on the event → [harness-techniques](harness-techniques.md#the-techniques-ranked) row 13 |
| A scripted merge loop merged 16 PRs carrying 36 open review findings, several of them real bugs | "mergeable" is not "reviewed" — see below |

`mergeStateStatus: CLEAN` ignores unresolved review threads on a repo without required conversation resolution. A scripted merge checks the threads itself, and pins the commit it checked:

```bash
unresolved=$(gh api graphql -F owner="$OWNER" -F repo="$REPO" -F pr="$PR" -f query='
  query($owner: String!, $repo: String!, $pr: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) { reviewThreads(first: 100) { nodes { isResolved } } }   # page past 100
    }
  }' --jq '[.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved | not)] | length')
[ "$unresolved" = 0 ] || { echo "PR $PR: $unresolved unresolved review threads - not merging"; exit 1; }
gh pr merge "$PR" --squash --match-head-commit "$HEAD_SHA"
```

---

**Related:** [agent-work-limits.md](agent-work-limits.md) — never cap the work · [harness-techniques.md](harness-techniques.md) — savings inside a run · [multi-agent-ops.md](multi-agent-ops.md) — many agents, one machine · [../workflow/dz-contract.md](../workflow/dz-contract.md) — the schedule keys · [../developer-experience/cheap-cicd.md](../developer-experience/cheap-cicd.md)
