# ♻️ Resource Lifetimes — the creator releases, on every exit

A resource acquired for one unit of work and released by the process is a leak. Daemons run for days, so the leak is invisible in tests, in typecheck and in a short manual run — it shows up as a full disk, a pool at its cap or an OOMKill a week later. As of 2026-11.

Memory-specific symptoms and how to measure them → [memory-and-leaks.md](memory-and-leaks.md).

## Two lifetimes, never mixed

| Lifetime | Examples | Created | Released |
|---|---|---|---|
| **Unit of work** — a request, a job, a run, a stream connection | temp dir, worktree, child process, lock, lease, subscription, timer, per-job store, file handle | inside the unit | by the unit, on every exit path |
| **Boot** — the process | DB pool, queue connection, HTTP server, an SDK client keyed on config | once, in the composition root | by the [bounded drain](#-bounded-drain-on-shutdown) |

**Ask what the resource's unit of work is, then what its actual lifetime is.** A mismatch is the bug: a per-job helper built once at boot and shared across jobs grows forever; a client built per request leaks a socket pool per request. Pick one row and make the code say which.

## Ownership rules

| Rule | Mechanism | Fails when |
|---|---|---|
| **One owner, named at acquisition** | two code paths that *could* release it means neither reliably does | a cleanup lives "somewhere downstream" |
| **Release on every exit** — success, throw, cancel, timeout, drain, signal | a scope-exit disposer, not happy-path cleanup | the throw path skips the release |
| **Acquire inside the scope that releases** | the acquire is the first statement inside the guard | the acquire sits above the `try` / scope and a throw between them leaks it |
| **Keeping it is a hand-over, not an exit** | the owner records the new owner and an expiry; the taker records it took it | a "kept for retry" resource has no recorded owner |
| **What outlives a crash has an on-disk record** | a `finally` does not run in a dead process; register before use, settle the record at next startup | the only recovery is "the sweeper will find it" |
| **Release saves first** | a release that could destroy work parks it first and removes nothing if the park failed | cleanup deletes the only copy |
| **Cannot release → say so once, stop the damage** | raise a fault the platform reads, alert once, stop taking work that fails the same way | it retries silently forever, or waits on a person |

## 🔒 `using` / `await using` over `try/finally`

Any resource with a lifetime shorter than the process is a `Disposable` / `AsyncDisposable`, acquired with `using` / `await using` (TypeScript 5.2+, `Symbol.dispose` / `Symbol.asyncDispose`).

| `try/finally` | `using` |
|---|---|
| correct by **enumeration** — every exit path must remember the release | correct by **construction** — scope exit releases, whatever the path |
| a new early `return` or `throw` can skip it with no failing test | a new branch cannot skip it |
| teardown order is whatever the blocks happen to nest | reverse declaration order, a language guarantee |
| a throwing cleanup replaces the original error | a throwing disposer surfaces as `SuppressedError`, original kept in `.suppressed` |

```ts
// one construction site for disposers — never an ad hoc object with a [Symbol.asyncDispose]
export function asyncDisposable(label: string, close: () => Promise<void>): AsyncDisposable {
  let done = false;
  return {
    async [Symbol.asyncDispose]() {
      if (done) return;           // idempotent: explicit release + scope exit closes once
      done = true;
      try { await close(); }
      catch (e) { log.warn(`${label}_failed`, { error: String(e) }); } // never throws
    },
  };
}

export async function runJob(job: Job) {
  await using dir = await scratchDir("job-");   // declared first → released last
  await using lock = await acquireLock(job.key);
  return await build(job, dir.path);            // `return await`, never `return promise`
}
```

The disposer contract — each clause is a unit test on the primitive:

| Clause | Why |
|---|---|
| **Never throws** — log `<label>_failed`, swallow | a failing `close()` must not skip the next disposer or mask the caller's error |
| **Idempotent** | an explicit early `release()` followed by scope exit runs once |
| **Order is declared** | where it is load-bearing, say so at the declaration; one handle owning several resources writes its order as code |
| **A destructive disposer records what it did** | a `kill` / `rm` / shred returns a record that is logged; an unlogged deletion did not happen |
| **A partial acquisition rolls back** | a factory that acquires A, then fails acquiring B, releases A — register each on an `AsyncDisposableStack`, hand the caller the whole handle or nothing |

**The one trap:** `return somePromise` ends the scope — every `using` in it disposes while the work is still running. Write `return await` in any scope that holds a handle, and pin it with a test.

```ts
// partial acquisition: the caller gets everything or nothing stays acquired
export async function openWorkspace(repo: string): Promise<Workspace & AsyncDisposable> {
  await using stack = new AsyncDisposableStack();
  const dir = stack.use(await scratchDir("ws-"));
  const tree = stack.use(await checkout(repo, dir.path)); // throws → dir released
  const owned = stack.move();                              // success → ownership leaves the scope
  return { dir, tree, [Symbol.asyncDispose]: () => owned.disposeAsync() };
}
```

Make the primitive the one implementation, then guard it: a lint or grep test that refuses a `finally` naming a teardown verb (`kill`, `rm`, `unlink`, `close`, `wipe`) in new code, and a ratchet on raw `mkdtemp` sites that only falls → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md#-dont-weaken-the-gate--a-ratchet-not-a-rule).

## 🧹 A sweeper is the backstop, never the release

Age- or pressure-driven sweeps (boot GC, nightly temp cleanup, a reaper cron) exist for a record that was **lost**. A design whose normal path waits for the sweeper is the bug: a 14-day backstop holds 14 days of every closed unit's resources, and on a busy box that is the disk.

| Sweeper rule | Why |
|---|---|
| Runs at startup for crash records, not after a retention window | the dead process's record is known garbage the moment the new one boots |
| Paced and bounded per tick, with a doubling backoff per record | a record that cannot be released must not be retried every tick forever |
| Gives up loudly after N attempts — alert naming the record, release it | silent infinite retry hides a broken owner |
| Releases only what a re-read proved stale | a sweep racing a live owner deletes live work |
| Logs what it reaped, as a counter you watch | a non-zero steady reap rate means an owner is leaking — fix the owner |

## ⏱️ Bounded drain on shutdown

`SIGTERM` starts a drain; the orchestrator's grace period ends it with `SIGKILL`. An unbounded `await worker.close()` waits for the active job — which may run for hours — so the kill decides the outcome: no log line, no closed sockets, no record of what died, and the lease lapses so a second worker re-runs the job. Health endpoints and the deregister step → [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md#-graceful-shutdown-on-sigterm); grace period → [../infrastructure/kubernetes-gitops.md](../infrastructure/kubernetes-gitops.md#-workload-runtime-contract).

| Part | Does | Without it |
|---|---|---|
| **Deadline** | force-exit, logged, **under** the grace period (25 s inside a 30 s grace) | the `SIGKILL` lands first, unlogged |
| **Force phase** (optional, at `forceAfterMs`) | abandons in-flight work deliberately so the rest of teardown runs — e.g. close the worker without waiting, so lock renewal stops | the drain waits out the job and the deadline eats the teardown |
| **Close pooled handles after quiesce, best-effort** | queues, DB, cache, server — closed even if quiesce threw; a stubborn socket never flips the exit code | a clean drain reports as a crash |
| **Re-entrancy guard** | a second `SIGTERM` is logged and ignored | two drains race two deadline timers |

```ts
let draining = false;
export async function drain(deps: DrainDeps, { deadlineMs = 25_000, forceAfterMs }: DrainOpts) {
  if (draining) { deps.log("drain_already_running"); return; }
  draining = true;
  const killTimer = deps.timer(() => { deps.log("drain_deadline_exit"); deps.exit(1); }, deadlineMs);
  const forceTimer = forceAfterMs ? deps.timer(() => void deps.force(), forceAfterMs) : () => {};
  try { await deps.quiesce(); }                       // stop intake, settle in-flight work
  catch (e) { deps.log(`drain_quiesce_failed ${String(e)}`); }
  finally { forceTimer(); }
  await closeQuietly(deps.handles, deps.log);         // never throws
  killTimer();
  deps.exit(0);
}
```

- **The drain never writes the abandoned work's terminal state.** The handler is still running and still the single writer for that row; a drain that appends becomes a second writer. Abort the work and let it write its own terminal row.
- **Raising the deadline is not the fix.** Sized to the longest job, every deploy blocks that long — and only works if the grace period is raised with it. Raise both together, app deadline under the platform's. A job longer than the grace period must be resumable → [data-and-scale.md](data-and-scale.md#-long-jobs-a-lease-is-not-a-timeout).
- **A malformed deadline knob throws at boot**, naming the variable and the value — never a silent fallback to the default you meant to raise.
- **Timers the drain sets are `unref`'d** so they never keep a drained process alive.
- **The force is never a second call to an idempotent close.** A close that memoizes its first call (BullMQ's `Worker.close()` returns the in-flight close promise) makes a later `close(force)` a no-op: quiesce with a wait that is not a close (`pause()`), then close once, forced → [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md#-graceful-shutdown-on-sigterm).
- **One drain implementation per repo.** Every long-lived process uses it; a hand-rolled one beside it is refused by a one-implementation guard.

## 🧪 Testing teardown

A release is only proven on the paths a test drove. Inject the failure, assert the release.

| Test | Asserts |
|---|---|
| **Throw at each await** between acquire and release (a fake that rejects on call N) | release ran **exactly once** on every N |
| Cancel / abort mid-work | release ran; no second release from the later scope exit |
| Double dispose | second call is a no-op |
| Disposer that throws | the caller's original error propagates by identity; later disposers still ran |
| `return await` vs `return promise` | the handle is still open while the returned work runs |
| Drain with a fake timer: quiesce never settles | deadline exit fires at `deadlineMs`, logged; handles closed only on the clean path |
| Drain: second signal | logged and ignored; one deadline timer |
| Crash record: write it, "kill" the process, boot again | startup settles it, without waiting for the sweeper's window |

```ts
it.each([1, 2, 3])("releases the lock when step %i throws", async (n) => {
  const lock = fakeLock();
  const steps = failingAt(n);
  await expect(runJob(job, { lock, steps })).rejects.toThrow(`step ${n}`);
  expect(lock.releases).toBe(1);
});
```

Inject the clock, never `sleep` → [testing.md](testing.md#-never-sleep-in-a-test--inject-the-timer).

## Review questions

| Ask | A bad answer |
|---|---|
| What is this resource's unit of work? | "it's just a helper" |
| Who owns it, and where is that written? | two callers both "clean up" |
| What releases it on the throw path? On cancel? On `SIGTERM`? | "the finally at the end" with an early return above it |
| What survives a crash, and what record lets the next boot settle it? | "the nightly sweep" |
| What bounds the shutdown? | an unbounded `close()` |

---

**Related:** [memory-and-leaks.md](memory-and-leaks.md) · [query-shape.md](query-shape.md) — pool pressure · [data-and-scale.md](data-and-scale.md) — leases and resumable jobs · [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md) — SIGTERM and health endpoints · [../workflow/bug-sweep.md](../workflow/bug-sweep.md) — the scheduled leak checkup
