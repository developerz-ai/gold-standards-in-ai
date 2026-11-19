# 🧰 Multi-Agent Ops — one box, many sessions, a clean repo

**Admit heavy work against what the machine has; leave nothing behind that a later session must clean up.** Parallel agents fail on shared memory, shared cores and leftover state long before they fail on the code.

Who edits what and who runs which check → [hive-mind.md](hive-mind.md). The unattended machine → [orchestration.md](orchestration.md). This page is what both need from the box and the repo.

As of 2026-11 every script here ran on Linux 7.0 with bash 5.2, util-linux 2.39 and `gh` 2.105, and is `shellcheck -S warning` clean. `prune-merged --apply` ran against a local remote with `gh` stubbed; `level-next --apply` has not been run. Put them on `PATH`, or under the repo's `bin/`.

## 🚨 What it looks like without this

| Measured | On | Result |
|---|---|---|
| 11 agents running gates at once (2026-11) | 8 cores | load **93**, CPU pressure **97%** |
| Agent sessions with no shared memory budget (17 days to 2026-09-10) | 46 GB, no swap | **9** whole-box OOM kills (`oom_kill` in `/proc/vmstat`) |
| A session running 11 read-only agents (2026-11-10) | a shared dev box | killed by `systemd-oomd` |
| CPU-bound tests beside 0 / 12 / 36 busy loops (2026-10-15) | 12 cores | pass / pass / **fail at their per-test timeouts**: a load failure that reads as a test failure |

## 🧠 Memory: one ledger per machine

**Every heavy command reserves its measured peak before it starts, and waits when the machine has no room.** Every session, checkout and worktree shares the one ledger.

| Rule | Why |
|---|---|
| **One ledger per machine and user**, at a fixed path, never under `$TMPDIR` | a tool can set `$TMPDIR` per session; two ledgers are two budgets |
| **Reserve the measured peak**, never typical use | a typical figure bounds nothing |
| **`flock`, not pid files** | the kernel releases the lock when the last process holding it is gone, by any signal. No stale lock, no liveness probe, no two waiters both taking over one dead pid. Local filesystems only |
| **Admit up to 50% of RAM** | derived on a 46 GB box idling at 12.7 GB: half leaves about 10 GB for the sessions and editors the ledger cannot see. Re-derive it from your idle figure |
| **Read the cgroup**: the lowest `memory.max` up the chain, not just host RAM | a ledger sized from the host admits work into a cgroup OOM kill. Above the limit, refuse with exit 75 before starting |
| **Children inherit the hold** | through the lock descriptor, so killing the wrapper does not free memory the workload still uses; through the environment, so a nested run does not reserve again and wait on its own ancestor. A process that closes inherited descriptors drops out of the hold |
| **Oversized work runs alone**, never refused | clamp to capacity, wait for an empty ledger |
| **The waiter at the gate keeps it** | a stream of small jobs cannot starve a large one |

```bash
#!/usr/bin/env bash
# box-run <peak-mb> <label> -- <command...>   Linux, cgroup v2, util-linux flock.
# Admits a command against the one memory budget every session on this machine shares.
set -euo pipefail
weight=$1 label=$2; shift 3

# Already inside a reservation: reserving again can wait on its own ancestor forever.
if [ -n "${BOX_LEDGER_HOLDER:-}" ]; then exec "$@"; fi

# Fixed path, never $TMPDIR: two sessions reading two ledgers are two budgets.
dir=/tmp/box-ledger-$(id -u)
mkdir -p "$dir/holds"
[ -O "$dir" ] || { echo "box-run: $dir is not yours" >&2; exit 75; }
chmod 700 "$dir"

# The tightest of host RAM and every memory.max up the cgroup chain.
limit=$(awk '/^MemTotal:/ { print int($2 / 1024) }' /proc/meminfo)
cg=$(sed -n 's/^0:://p' /proc/self/cgroup)
while :; do
  max=$(cat "/sys/fs/cgroup$cg/memory.max" 2>/dev/null || echo max)
  if [ "$max" != max ] && [ $((max / 1048576)) -lt "$limit" ]; then limit=$((max / 1048576)); fi
  if [ -z "$cg" ] || [ "$cg" = / ]; then break; fi
  cg=${cg%/*}
done
if [ "$weight" -gt "$limit" ]; then
  echo "box-run: $label needs $weight MB, this cgroup allows $limit MB" >&2
  exit 75
fi
capacity=$((limit / 2))
# Bigger than the budget still runs, alone.
if [ "$weight" -gt "$capacity" ]; then weight=$capacity; fi

# Whoever holds the gate is head of the line, so small jobs cannot starve a large one.
exec 8>"$dir/gate"
flock 8
said=
while :; do
  used=0
  for hold in "$dir"/holds/*; do
    [ -e "$hold" ] || continue
    if flock -n "$hold" true; then rm -f "$hold"   # lock was free: its owner is dead
    else used=$((used + $(cat "$hold"))); fi
  done
  if [ $((used + weight)) -le "$capacity" ]; then break; fi
  if [ -z "$said" ]; then echo "box-run: $label waits: $used+$weight > $capacity MB" >&2; said=1; fi
  sleep 0.2
done

# The kernel drops this lock when the last process holding fd 9 is gone, by any signal.
hold=$dir/holds/$$-${label//[^A-Za-z0-9_.-]/_}
exec 9>"$hold"
flock 9
echo "$weight" >"$hold"
exec 8>&-

# The command inherits fd 9: if this wrapper is killed, the reservation lasts as long as
# the workload does. Anything it leaves running keeps holding memory, and the reservation.
status=0
BOX_LEDGER_HOLDER=$$ "$@" || status=$?
exec 9>&-
flock -n "$hold" rm -f "$hold" || true
exit "$status"
```

```bash
box-run 5120 typecheck -- npm run typecheck
box-run 2048 "tests:$PWD" -- npm test -- src/orders.test.ts
ls "/tmp/box-ledger-$(id -u)/holds"              # <pid>-<label>, each file holding its MB
fuser -v "/tmp/box-ledger-$(id -u)/holds/"*      # which processes keep each reservation alive
```

A queue that never drains usually means a daemon left running: it still holds its memory and its reservation. `fuser` names it.

**Measure the weight; do not guess it.** `/usr/bin/time -v` reports the largest single process: two 60 MB processes read 73 MB, and this script read 147 MB.

```bash
#!/usr/bin/env bash
# peak-mb <command...>   Peak summed RSS of the command's session, sampled every 0.5 s.
# Shared pages count once per process, so it reads high: the safe side for a reservation.
set -euo pipefail
setsid "$@" &
pid=$!
peak=0
while kill -0 "$pid" 2>/dev/null; do
  kb=$({ ps -o rss= -s "$pid" || true; } | awk '{ sum += $1 } END { print sum + 0 }')
  if [ "$kb" -gt "$peak" ]; then peak=$kb; fi
  sleep 0.5
done
status=0
wait "$pid" || status=$?
echo "peak $((peak / 1024)) MB, exit $status" >&2
exit "$status"
```

Take several runs on real input, record N, the p99 and the maximum with the date, and round up. Re-measure when a run exceeds its reservation.

| The ledger does not bound | Cover it with |
|---|---|
| the agent sessions themselves | the other 50%; count sessions against it |
| a command run without `box-run` | wrap it in the repo's scripts, so the normal command is the admitted one |
| watchers and dev servers | one per checkout, outside the budget, on purpose |
| containers, which run under the Docker daemon (`peak-mb` misses them too) | `docker stats` for the weight, `--memory` for the cap |

## ⚙️ CPU: cap locally, every core in CI

**A dev box is shared; a CI runner runs nothing else.** Cap native tools too, or the cap stops at your test runner.

```bash
#!/usr/bin/env bash
# dev-jobs [own-runnable]   How many workers may run NOW. Ask again before each new worker.
set -euo pipefail
own=${1:-0} cap=4 floor=2

cores=$(nproc)
if read -r quota period 2>/dev/null </sys/fs/cgroup/cpu.max && [ "$quota" != max ]; then
  limit=$((quota / period)); [ "$limit" -ge 1 ] || limit=1
  [ "$limit" -ge "$cores" ] || cores=$limit
fi
case ${CI:-} in true | 1) echo "$cores"; exit 0 ;; esac

psi() { awk '/^some/ { sub("total=", "", $5); print $5 }' /proc/pressure/cpu 2>/dev/null || true; }
p0=$(psi) t0=$(date +%s%N)
samples=()
for _ in 1 2 3 4 5; do
  read -r _ _ _ running _ </proc/loadavg
  samples+=("${running%/*}")
  sleep 0.1
done
p1=$(psi) t1=$(date +%s%N)

median=$(printf '%s\n' "${samples[@]}" | sort -n | sed -n 3p)
other=$((median - 1 - own))    # 1 is this script reading the file
[ "$other" -ge 0 ] || other=0
jobs=$((cores - other))

# Pressure includes our own workers, so it only speaks before they start.
if [ "$own" -eq 0 ] && [ -n "$p0" ] && [ -n "$p1" ]; then
  stalled=$(((p1 - p0) * 100 / ((t1 - t0) / 1000)))
  [ "$stalled" -le 100 ] || stalled=100
  delivered=$((cores * (100 - stalled) / 100))
  [ "$delivered" -ge "$jobs" ] || jobs=$delivered
fi

[ "$jobs" -ge "$floor" ] || jobs=$floor
[ "$jobs" -le "$cap" ] || jobs=$cap
[ "$jobs" -le "$cores" ] || jobs=$cores
echo "$jobs"
```

```bash
export GOMAXPROCS=4 RAYON_NUM_THREADS=4     # locally only; Go and Rust tools otherwise take every core
```

| Rule | Evidence |
|---|---|
| **Cap local work** (4 cores is the starting value) | 12 cores at load 8-14, 2026-10-24: cold typecheck 87 s / 7,882 MB uncapped, 109 s / 6,605 MB capped; linter 19 s → 34 s. The cap costs time and buys room for everyone else |
| **Re-ask before each worker** | a pool sized once from a busy moment stayed at 2 workers while the 16-core box later read 16% busy: 721 s, against 444 s for the same leg at 4 workers (2026-11-16) |
| **Back off on other sessions' runnable tasks** | the load average lags and includes your own previous step; field 4 of `/proc/loadavg` is the count right now |
| **Keep a floor of 2** | a busy box slows the run instead of stalling it |

| Signal | Read | Means |
|---|---|---|
| Runnable now | `cut -d' ' -f4 /proc/loadavg` | `running/total` scheduling entities. Far above cores = queueing |
| Load | `uptime` | 1, 5, 15 minute trend. Counts I/O wait too, so it never stands alone |
| CPU pressure | `cat /proc/pressure/cpu` | `some`: share of time at least one task waited for a CPU. For a decision, use the change in `total` over a short window; `avg10` lags |
| Memory pressure | `cat /proc/pressure/memory` | `full`: every non-idle task stalled at once, i.e. thrashing |
| One group | `cat /sys/fs/cgroup/<path>/cpu.pressure` | the same figures for one session or service |
| Stolen CPU, on a VM | `steal` in `top`, or `/proc/stat` | the host is starving the guest, which no in-guest count of tasks shows |
| OOM kills so far | `grep oom_kill /proc/vmstat` | a rising count is the record |

## 🚦 Who may spend what

| Role | Runs | Budget |
|---|---|---|
| **Worker** | its own files' tests and typecheck, [scoped](hive-mind.md#-who-runs-which-checks) | concurrency 1, through `box-run` |
| **Coordinator** | the whole gate, **once, before the PR** | `dev-jobs` workers, through `box-run` |
| **Read-only agent** | one command at a time, bounded output; no build, test, typecheck or index run | its own limit, separate from writers: the session count that fits the unledgered half of RAM at your measured session size (about 1 GB each on the box above) |
| **Second gate on the box** | nothing yet | it queues. The ledger stops the OOM by making gates wait, so never start two on purpose |
| **CI** | everything | every core |

## 🔒 Coordination

| Question | Answer lives in |
|---|---|
| What stops two agents overwriting each other | [the file set is the lock](hive-mind.md#-the-file-set-is-the-lock) |
| When a worktree is earned | [worktrees: when](hive-mind.md#-worktrees-when) — a second branch in flight, not one per agent |
| Who runs git | [the coordinator](hive-mind.md#-the-coordinator-coordinates-it-does-not-code); workers run none |
| How many PRs at once | [PR cap by repo kind](../writing-for-agents/workflow-commands.md#-pr-cap-by-repo-kind) |

## 🧹 Repo and box hygiene

**Whoever creates it removes it, and a scheduled job catches what a crashed session left.**

| Rule | How |
|---|---|
| **Delete head branches on merge** | `gh repo edit --delete-branch-on-merge` |
| **Delete any other branch only when its exact tip merged through a PR** | `prune-merged`. `git branch --merged` misses squash merges, and a same-named branch with newer commits is unmerged work |
| **Remove worktrees when the work ends** | `git worktree remove <path>` refuses a dirty tree; [archive it first](hive-mind.md#-worktrees-when), never `--force` past it. Then `git worktree prune` |
| **Remove scratch clones** | `reap-clone` |
| **Sweep orphan processes** | end a run by killing its [whole process group](orchestration.md#-the-loops-exit--done-a-machine-can-judge); `orphans` finds what escaped. Review before `kill`: an editor's helper can match too |
| **Keep the next green PR level with `main`** | `level-next`, on a schedule or after each merge, so a merge bot requiring the current tip can take it |

`level-next` moves **one** PR because each merge moves `main` again: updating all N re-runs CI N times for one merge.

```bash
#!/usr/bin/env bash
# prune-merged [--apply]   Default prints. Run from a clone of the repository.
set -euo pipefail
default=$(gh repo view --json defaultBranchRef --jq .defaultBranchRef.name)
git fetch --quiet --prune origin
while read -r branch tip <&3; do
  case $branch in HEAD | "$default") continue ;; esac
  [ "$(gh api "repos/{owner}/{repo}/branches/$branch" --jq .protected)" = false ] || continue
  # An open PR from it, or based on it, still needs it.
  [ "$(gh pr list --state open --head "$branch" --json number --jq length)" = 0 ] || continue
  [ "$(gh pr list --state open --base "$branch" --json number --jq length)" = 0 ] || continue
  merged=$(gh pr list --state merged --head "$branch" --limit 100 \
    --json headRefOid,isCrossRepository \
    --jq "[.[] | select(.headRefOid == \"$tip\" and (.isCrossRepository | not))] | length")
  if [ "$merged" = 0 ]; then echo "keep   $branch (tip not merged through a PR)"; continue; fi
  if [ "${1:-}" != --apply ]; then echo "delete $branch"; continue; fi
  # Compare-and-delete: the server refuses if the branch moved after the check above.
  if git push --quiet --force-with-lease="refs/heads/$branch:$tip" origin ":refs/heads/$branch"; then
    echo "deleted $branch"
  else
    echo "keep   $branch (delete refused)"
  fi
done 3< <(git for-each-ref --format='%(refname:strip=3) %(objectname)' refs/remotes/origin)
```

```bash
#!/usr/bin/env bash
# reap-clone <dir>   Deletes a scratch clone only when it holds nothing unique. Do not fetch first.
set -euo pipefail
dir=$1
[ -d "$dir/.git" ] || { echo "keep $dir: not a standalone clone" >&2; exit 1; }
[ ! -d "$dir/.git/worktrees" ] || { echo "keep $dir: it has worktrees" >&2; exit 1; }
dirty=$(git -C "$dir" status --porcelain)
# Local refs, HEAD and the reflog: branches, stashes and detached commits.
unpushed=$(git -C "$dir" log --all --reflog HEAD --not --remotes --oneline)
# A tag on a pushed commit is still unique: compare names and tag objects with the remote.
remote_tags=$(git -C "$dir" ls-remote --tags --refs origin | awk '{ print $1, $2 }' | sort)
local_tags=$(git -C "$dir" for-each-ref --format='%(objectname) %(refname)' refs/tags | sort)
tags=$(comm -23 <(printf '%s\n' "$local_tags") <(printf '%s\n' "$remote_tags"))
# Ignored files can be the only copy (.env). Only rebuildable directories may go.
ignored=$(git -C "$dir" ls-files --others --ignored --exclude-standard --directory |
  grep -Ev '(^|/)(node_modules|dist|build|target|\.cache)/$' || true)
if [ -n "$dirty$unpushed$tags$ignored" ]; then echo "keep $dir: unsaved work" >&2; exit 1; fi
rm -rf -- "$dir"
```

```bash
#!/usr/bin/env bash
# orphans [min-age-seconds]   Lists; killing stays a decision. Edit `tools` for your stack.
set -euo pipefail
age=${1:-3600}
tools='^(bun|node|tsc|tsgo|biome|esbuild|vite|playwright|chrome|cargo|rustc|pytest)'
adopters=" 1 $(pgrep -u "$(id -u)" -x systemd | tr '\n' ' ')"
ps -u "$(id -u)" -o pid=,ppid=,etimes=,rss=,comm= |
  awk -v age="$age" -v tools="$tools" -v adopters="$adopters" '
    index(adopters, " " $2 " ") && $3 >= age && $5 ~ tools {
      printf "%d\t%ds\t%dMB\t%s\n", $1, $3, $4 / 1024, $5
    }'
```

```bash
#!/usr/bin/env bash
# level-next [--apply]   Brings the oldest green PR that is behind main level with it.
set -euo pipefail
default=$(gh repo view --json defaultBranchRef --jq .defaultBranchRef.name)
green='map(select((.isDraft | not) and (.statusCheckRollup | length > 0) and
  ([.statusCheckRollup[] | .conclusion // .state] |
    all(. == "SUCCESS" or . == "NEUTRAL" or . == "SKIPPED"))))
  | sort_by(.number)[] | "\(.number) \(.headRefOid)"'
while read -r pr head <&3; do
  behind=$(gh api "repos/{owner}/{repo}/compare/$default...$head" --jq .behind_by)
  [ "$behind" -gt 0 ] || continue
  echo "#$pr is $behind behind $default"
  if [ "${1:-}" = --apply ]; then gh pr update-branch "$pr"; fi
  break
done 3< <(gh pr list --state open --base "$default" --limit 100 \
  --json number,headRefOid,isDraft,statusCheckRollup --jq "$green")
```

## 🗄️ Retention

**Delete by what is still needed, never by age alone.** Day counts are starting points; set them from how far back you actually debug.

| Thing | Keep | Why / how |
|---|---|---|
| **Public packages and tags** | **every version** | others pin them. Deprecate a bad one; do not delete it |
| **Private images** | the **running digest + the last 2 that worked** + anything in a live rollout | "worked" = passed the deploy health check, so rollback lands on a good image. [Protect the digest, not the date](../developer-experience/smart-builds.md). If the running digest cannot be read, delete nothing |
| **CI artifacts** | about 7 days | `retention-days: 7` on `actions/upload-artifact` |
| **CI caches** | let eviction work; delete a closed PR's and any poisoned key | GitHub evicts after 7 days unused, 10 GB per repo by default (as of 2026-11). `gh cache list --sort last_accessed_at --order asc` · `gh cache delete --all --ref refs/pull/<n>/merge` |
| **Local Docker** | a week of unused layers | `docker image prune -f --filter until=168h` · `docker builder prune -f --filter until=168h` |
| **Per-agent test databases, ports, dirs** | the run | dropped by the step that created them |

## ✅ Merged is not finished

**A check that runs only on `main` has not passed until it passes on the merge commit.** As of 2026-11, GitHub reports a job skipped by `if:` as **Success**, and it does not block a merge even when required. Real incident: a Windows export job looked fine on its PR, then went red on `main`.

```bash
#!/usr/bin/env bash
# after-merge <pr>   Exit 0 only when every push run on the merge commit is green.
set -euo pipefail
sha=$(gh pr view "$1" --json mergeCommit --jq .mergeCommit.oid)
[ -n "$sha" ] || { echo "#$1 is not merged" >&2; exit 75; }
ids=$(gh run list --commit "$sha" --event push --json databaseId --jq '.[].databaseId')
[ -n "$ids" ] || { echo "no run on $sha yet" >&2; exit 75; }
for id in $ids; do
  gh run watch "$id" --exit-status >/dev/null || { gh run view "$id"; exit 1; }
done
echo "$sha green"
```

Exit 75 means no verdict yet: retry it, never read it as green. Run it after every merge, and fix a red `main` before starting the next PR.

When a diff touches what a main-only job covers, run that job before merging: add `workflow_dispatch` to the workflow, include `github.event_name == 'workflow_dispatch'` in the job's `if:`, then:

```bash
gh workflow run ci.yml --ref "$(git branch --show-current)"
```

---

**Related:** [hive-mind.md](hive-mind.md) — running the team · [orchestration.md](orchestration.md) — the unattended machine · [../architecture/memory-and-leaks.md](../architecture/memory-and-leaks.md) — memory inside a long-lived service · [../developer-experience/smart-builds.md](../developer-experience/smart-builds.md) — image tags and caching
