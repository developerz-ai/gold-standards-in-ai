# 💸 Cheap, Fast CI

CI is a bill that multiplies by every push an agent makes. **Measure where the minutes go, then remove the runs that should never have started** — a faster runner on a wasted run is still waste. Every number here was measured on our own repos on 2026-11-28 (Actions API, 7-day window); measure yours the same way before copying a change.

The pipeline's shape → [ai-first-cicd.md](ai-first-cicd.md). What to build and restart → [smart-builds.md](smart-builds.md). The agent-token half of the bill → [../ai-agents/maintainer-token-spend.md](../ai-agents/maintainer-token-spend.md).

## 🧮 The cost model

| Runner | Billed | Consequence |
|---|---|---|
| GitHub-hosted **standard** runner, **public** repo | free (As of 2026-11); a larger hosted runner is billed even there | the default for open source; pay elsewhere only for measured wall-clock |
| GitHub-hosted, **private** repo | per minute; macOS and Windows cost the most | an OS matrix is the most expensive thing a PR can run |
| Third-party per-minute runner | per minute, by size. The one measured here scales with vCPU: 2 vCPU = 1×, 4 = 2×, 8 = 4×, 16 = 8× | a bigger runner pays only when the wall-clock falls by more than the multiplier |

**Billing minute** = wall minute × the runner's price multiplier — on the provider measured here, vCPU ÷ 2. It is a normalization for that price list, not a universal formula: read your provider's multipliers and substitute them. Count in billing minutes, never runs or wall minutes: there, a 1-minute job on 16 vCPU costs what an 8-minute job costs on 2.

One monorepo measured **≈145,000 billing min/week (~$180/week)** in total. PR runs were ≈121,000 of it: 1,098 runs/week at ~110 billing min each. Free runners are not free either — a slow free run is paid in the agent's waiting.

## 🕳️ Where the minutes went

Ordered by measured size where a number exists. Each row is a mechanism, not a repo.

| # | Mechanism | Measured | Fix |
|---|---|---|---|
| 1 | **Merge-main commits re-run the full PR pipeline.** A "branch must be up to date" rule forces a merge of `main` into every open PR after each merge; the commit changes nothing the PR owns | 358 of 1,098 PR runs (33%), ~40,000 billing min/week | a [merge queue](#-merge-main-reruns-and-merge-queues) |
| 2 | **Oversized runners.** Lint and typecheck on 4 vCPU finished in ~1 min while the critical path was 5+ min | moved to 2 vCPU: same wall-clock, ~2,900 billing min/week saved | [size by the real constraint](#-runners--size-by-the-real-constraint) |
| 3 | **A single-threaded tool on a big runner.** A linter documented as per-core parallel ran one file at a time | an 8-vCPU runner sat on one core for 8 min | read utilization before buying cores |
| 4 | **Expensive OS matrix on every PR** — macOS, Windows, sanitizer builds | one repo: 17 → 9 jobs per PR | [full matrix on `main`, tags and weekly](#-matrices) |
| 5 | **Duplicate triggers.** `push` on every branch plus `pull_request` | every PR commit ran everything twice | [one trigger shape](#-triggers) |
| 6 | **Image builds regardless of inputs** | one repo built 9 images on every merge | [build only when the image's inputs changed](#-images--build-only-when-inputs-changed) |
| 7 | **A scanner that gates nothing, on every PR** | one code scanner: ~460 wall-min over 300 runs | `main` + weekly |
| 8 | **Nightly crons that could be weekly** — determinism, Lighthouse, audits, smoke | — | [crons](#-crons) |
| 9 | **A flaky scheduled job feeding an AI maintainer** | a nightly job failed 4 of 40 runs; each red became a fix task: 547M tokens on 9 tasks, one of them a 795-line download mirror written for a single HTTP 504 | re-run first; fix the flake → [maintainer-token-spend](../ai-agents/maintainer-token-spend.md#-flaky-schedulers-feed-agents) |

The reverse case is real too: one public repo moved its Linux jobs **off** the free runner to a paid 4-vCPU one — 12.5 min against 31–35 min. Speed was worth paying for there. Decide per repo by measured wall-clock × price.

## 🎯 Triggers

```yaml
on:
  pull_request:
  push:
    branches: [main]
```

| Rule | Fails when |
|---|---|
| One shape: `pull_request` + `push: [main]` | `on: [push, pull_request]` runs every PR commit twice |
| Add `merge_group:` when a merge queue is on | the queue waits forever on a check that never runs |
| Never a workflow-level `paths:` filter on a **required** check | the check never reports and the PR is blocked forever — filter [inside the workflow](#-change-detection-that-fails-closed) |
| Never rename a required job | its name **is** the check context; the rule keeps waiting on the old one |

## 🔀 Concurrency — and the third-push trap

```yaml
concurrency:
  # A PR gets one group: a newer push cancels the superseded run.
  # Every other run gets a group of its own: nothing cancels it, nothing drops it.
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.run_id }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
```

**The trap:** `group: ci-${{ github.ref }}` with `cancel-in-progress: false` looks safe for `main` and is not. A group holds **one running and one pending** run; a third push replaces the pending one. Three merges in a row and the middle commit never gets a verdict — and `cancel-in-progress: true` on `main` is the same loss, sooner. A merged commit with no verdict is a red `main` nobody can bisect.

A job that **promotes** — moves `latest`, deploys, publishes — gets its own per-environment group, and inside it promotes **the tip's verified image, not its own**. Runs do not reach `promote` in commit order, and the group's one pending slot goes to whichever run arrived last:

| Written as | Fails when |
|---|---|
| "promote my image" | an older run whose build finished late overwrites `latest` after a newer one |
| "promote my image if I am still the tip" | the older run takes the pending slot from the tip's run, then declines — neither image ships |
| "promote the tip's image if it exists" | an older run publishes the tip before the tip's own checks finish — or after they failed |
| "promote the tip's image once the tip's own verdict is green" | — any run that holds the lock does the right thing, whichever one it is |

```yaml
  promote:
    needs: [build, ci-ok]            # ci-ok needs build too: a green ci-ok means the image is pushed
    if: github.ref == 'refs/heads/main'
    concurrency:
      group: promote-production      # one writer per environment; the newest pending run wins
      cancel-in-progress: false      # never kill a promotion mid-write
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
      actions: read
      packages: write
    steps:
      - uses: docker/login-action@<sha>     # a fresh runner is logged in to nothing; `packages: write` alone pushes nothing
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ github.token }}
      - name: Move latest to the tip's verified image
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          set -euo pipefail
          image="ghcr.io/${GITHUB_REPOSITORY,,}"      # an image reference is lowercase; an owner name need not be
          tip=$(gh api "repos/$GITHUB_REPOSITORY/commits/main" --jq .sha)
          # The tag existing proves a build, not a verdict: only the tip's own green ci-ok makes it promotable.
          # Bound to THIS workflow's push run on main - a check that merely shares the name proves nothing.
          run=$(gh api -X GET "repos/$GITHUB_REPOSITORY/actions/workflows/ci.yml/runs" \
            -f head_sha="$tip" -f event=push -f branch=main --jq '.workflow_runs[0].id // empty')
          verdict=pending
          if [ -n "$run" ]; then
            verdict=$(gh api "repos/$GITHUB_REPOSITORY/actions/runs/$run/jobs" \
              --jq '[.jobs[] | select(.name == "ci-ok")][0].conclusion // "pending"')
          fi
          if [ "$verdict" != success ]; then
            echo "tip $tip: ci-ok is $verdict - not promotable; its own run promotes it when it is"; exit 0
          fi
          # Verified, so the image must exist: any failure from here is real and fails the job.
          docker buildx imagetools create -t "$image:latest" "$image:sha-$tip"
```

The lock serializes promoters, not pushes. The guarantee is a snapshot: `latest` is the verified tip **as read inside the lock**, it never moves backwards, and a push that lands after the read is promoted by its own run. Exact-tip at every instant needs the branch update itself coordinated with promotion — a merge queue, not a tag.

This assumes every `main` commit builds an image. Where builds are skipped by input ([below](#-images--build-only-when-inputs-changed)), promote the image carrying the tip's **build hash** instead of `sha-$tip`.

## 🚪 Change detection that fails closed

**A skipped required job counts as passing.** So a gate written `if: needs.changes.outputs.code == 'true'` **fails open**: when the `changes` job errors, or its file list is truncated, the output is empty, every gated job is skipped, the checks go green, and untested code merges.

| Truncation | Limit (As of 2026-11) |
|---|---|
| GitHub's own `paths` filters | read the first 300 changed files of a diff |
| The REST "list pull request files" endpoint | returns at most 3,000 files |

Diff with git, and make every failure mean **run everything**:

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    outputs:
      code: ${{ steps.diff.outputs.code }}
    steps:
      - uses: actions/checkout@<sha>
        with: { fetch-depth: 2 }       # a pull_request checkout is the merge commit; HEAD^1 is the base
      - id: diff
        if: github.event_name == 'pull_request'
        run: |
          set -euo pipefail            # any error fails this job - and a failed job runs everything
          # --no-renames: a source file moved INTO docs/ must list its old path too, or the move skips the tests
          files=$(git diff --no-renames --name-only HEAD^1 HEAD)
          # code=false only when EVERY changed path is one named here; an empty list counts as code
          if grep -qvE '^(docs/|README\.md$|LICENSE$)' <<< "$files"; then code=true; else code=false; fi
          echo "code=$code" >> "$GITHUB_OUTPUT"

  test:
    needs: changes
    # !cancelled(), not always(): a superseded run must stop, not burn a runner
    if: >-
      !cancelled() && (
        github.event_name != 'pull_request' ||
        needs.changes.result != 'success' ||
        needs.changes.outputs.code == 'true'
      )
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@<sha>
      - run: bin/check
```

`push` to `main` skips detection and runs everything: the verdict on a merged commit is not the place to save a minute.

With several gated jobs, require one aggregator instead of each job: one check context, so adding or renaming a job never touches the branch rule. It grades **only** the jobs in its `needs` — a gated job left out can fail while the required check stays green, so pin the list with a test:

```yaml
  ci-ok:
    if: always()                       # must report on every run: a skipped aggregator would pass
    needs: [changes, lint, test]
    runs-on: ubuntu-latest
    timeout-minutes: 2
    steps:
      - env:
          NEEDS: ${{ toJSON(needs) }}
        run: |
          jq -e '.changes.result == "success"
                 and all(.[]; .result == "success" or .result == "skipped")' <<< "$NEEDS"
```

| Rule | Fails when |
|---|---|
| Name the skippable paths; never skip by extension | a docs site *is* markdown → [the markdown trap](smart-builds.md#the-markdown-trap) |
| Check what the tests read before a docs-only skip | a test that reads `docs/` or the README makes "docs-only" a code change |
| One detector, shared with the local gate | CI and `:changed` disagree → [smart-builds](smart-builds.md#1-one-detector-three-consumers) |
| Every job except `ci-ok` is in `ci-ok.needs`, asserted by a test that parses the workflow | a new job fails and the one required check stays green |
| **Observe it red**: break the `changes` step on a throwaway PR and assert **both** results | the gated jobs skip instead of run, **or** they run and `ci-ok` still reports success |

## 📐 Runners — size by the real constraint

| Constraint | Evidence | Size |
|---|---|---|
| Not on the critical path | the job finishes minutes before the slowest one | smallest runner — a faster lint does not make the PR green sooner |
| CPU, parallel | wall-clock falls as cores rise; the suite uses them | the size where the wall stops falling → [four measured techniques](linting-ci.md#-four-measured-techniques) |
| CPU, single-threaded | one core busy, the rest idle | smallest runner; fix the tool's parallelism first |
| Disk or memory | the job fails on space, not on time | the size that carries it, and say so |

**Write the constraint next to the label.** One E2E job stays on 4 vCPU for the 40 GiB of free disk, not the cores; unlabelled, the next cost sweep downsizes it and it fails on space.

```yaml
    runs-on: <provider>-4vcpu   # DISK: the stack needs 40 GiB free. Not a CPU choice - do not downsize by utilization.
```

## 🧱 Matrices

| Run | Cells |
|---|---|
| Pull request | one representative cell **plus the declared minimum runtime** |
| `main`, tags, weekly | the full matrix — every OS, every runtime, sanitizer builds |

The trade is stated, not hidden: a break only the full matrix catches now shows on `main`, not on the PR. Take it when such breaks are rare and the matrix is expensive.

The floor is not optional. One library's PR runs dropped its minimum supported runtime while trimming the matrix; the package's declared floor went untested until review caught it. "Latest only" on PRs ships a syntax the floor cannot parse.

## 🗃️ Caching and timeouts

| Rule | Shape |
|---|---|
| The ecosystem's own cache first | the `setup-*` action's built-in cache, `Swatinem/rust-cache`, `type=gha` for Docker layers |
| Restore everywhere, save on the default branch only | → [linting-ci](linting-ci.md#-four-measured-techniques) |
| Keys and layers | → [smart-builds](smart-builds.md#4-cache-everything-that-didnt-change) |
| `timeout-minutes` on **every** job, ~3× its normal time | the default is 6 hours; one hung job on a paid 8-vCPU runner bills all of them |

## ⏰ Crons

| Rule | Because |
|---|---|
| Weekly unless a nightly run has a record of catching things | determinism, Lighthouse, audit and smoke runs rarely change overnight |
| A scanner whose result blocks nothing runs on `main` + weekly | on every PR it is cost with no gate |
| A scheduled job must be deterministic, or retried before it is believed | one red is a task for whoever watches it — human or [agent](../ai-agents/maintainer-token-spend.md#-flaky-schedulers-feed-agents) |
| Stagger off `:00` | GitHub delays scheduled runs under load, and the start of an hour is the loaded minute |

## 🐳 Images — build only when inputs changed

| Rule | Mechanism |
|---|---|
| No image build on a PR unless the PR changed that image's inputs | Dockerfile, lockfile, the paths it `COPY`s — the build hash → [smart-builds](smart-builds.md#2-build-vs-roll-are-two-different-questions) |
| **Build?** — does an image with this build hash exist in the registry. Never "did the previous commit change it" | a failed or skipped build on the previous commit is never retried — the change never ships and nothing errors |
| **Promote?** — does `latest` point at that image. A separate question, asked on every `main` run | a build that pushed and then failed to promote leaves the next unchanged commit to skip both — the rollout stays pending forever |
| Promote `latest` in its own serialized job, reconciling to the tip | → [concurrency](#-concurrency--and-the-third-push-trap) |
| `latest` is a channel, the content tag is the identity | → [three tag meanings](smart-builds.md#3-one-repository-per-rollable-unit) |

## 🚦 Merge-main reruns and merge queues

A "require branches to be up to date" rule turns every merge into a full re-run of every other open PR: 33% of one repo's PR runs were merge-`main`-only commits, the largest single line of its CI bill.

| Option | Cost | Safety |
|---|---|---|
| Up-to-date rule, re-run each PR after each merge | O(open PRs) full runs per merge | each PR is tested against the `main` it merges into |
| No rule | none | two green PRs can merge into a red `main` |
| **Merge queue** | one run per merge, on the exact merge result | same guarantee as the rule, without the re-runs |

The queue is the direction, not yet a measured result here (As of 2026-11-28). It needs `merge_group:` in the workflow's triggers, and the required check must run there. Until it is on, the cheap half is free: never merge `main` into a PR "to refresh it" — only when the merge gate asks.

## ✅ Checklist

| Check | Pass |
|---|---|
| Billing minutes per week, by workflow and by job | a number with its date and query |
| Triggers | `pull_request` + `push: [main]` (+ `merge_group`); no duplicate run on a PR commit |
| Concurrency | PR runs cancel superseded; a non-PR run's group is unique; promotion is serialized and promotes the tip's image |
| Change detection | a failed or truncated detector runs everything; every gated job is in the aggregator's `needs`; observed red once |
| Required checks | no workflow-level `paths:` filter; job names unchanged |
| Runner size | each label above the smallest carries its constraint in a comment |
| Matrix | PR: one cell + the declared minimum; full matrix on `main`, tags, weekly |
| Timeouts | `timeout-minutes` on every job |
| Crons | each has a cadence reason; none nightly by default; flaky ones retried before believed |
| Images | built only when no image carries the build hash; `latest` checked separately on every `main` run |
| Merge rule | a merge queue, or a counted share of merge-`main` reruns |
| Local parity | one `bin/check` that CI also calls; toolchain pinned once (`.bun-version`, `.ruby-version`, `rust-toolchain.toml`) and read by CI → [dx-scripts](dx-scripts.md) |

Where a repo's own guard test pins the workflow's text, the guard wins: change both, or keep `bin/check` as the local mirror.

---

**Related:** [ai-first-cicd.md](ai-first-cicd.md) — the pipeline's shape · [smart-builds.md](smart-builds.md) — build vs roll · [linting-ci.md](linting-ci.md) — runners and caches · [../ai-agents/maintainer-token-spend.md](../ai-agents/maintainer-token-spend.md) — the token half of the bill
