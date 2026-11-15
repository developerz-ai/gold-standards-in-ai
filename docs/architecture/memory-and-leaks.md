# 🫗 Memory & Leaks — find the growth before the OOMKill

A leak in a long-lived process does not fail a test, a typecheck or a short manual run. It grows for days under a generous memory limit, a shallow health check reports healthy right up to the kill, and the evidence dies with the container unless you read it first. As of 2026-11.

Who releases a resource, and on which exit → [resource-lifetimes.md](resource-lifetimes.md). This page is the memory side: shapes, limits, evidence, measurement.

## The leak shape that actually bites

**A short-lived helper given a process lifetime.** Unbounded module-level `Map`s, missing `clearInterval`s and stacked listeners are real but rare in code that keeps locks, counters and cooldowns in a TTL'd external store. The common one is a lifetime mismatch:

| Shape | What is retained | Why it grows forever |
|---|---|---|
| Per-job store built once at boot, shared by every consumer | one entry (with its whole payload graph) per job | each job writes under a fresh key; nothing ever ends the "session" |
| SDK client built inside a per-request resolver | a client + its keep-alive agent and socket pool per request | the constructor is the expensive part, and it runs on every call |
| Client-side store fed by a server diff stream | every row from every snapshot ever received | the server's diff state is per connection; after a reconnect it cannot emit a delete for a row the client still holds |

The third is not a cache at all: a store correct for one connection, wrong across a reconnect.

## Process lifetime vs unit-of-work lifetime caches

| Rule | Mechanism | Fails when |
|---|---|---|
| **Bound the primitive, not the call site** | a helper that *can* be handed a longer lifetime bounds itself inside — idle TTL + hard entry cap | eleven call sites are fixed and the twelfth is added next month |
| **Eviction cannot corrupt live state** | size the TTL off the slowest consumer (a 2 h job → TTL above 2 h); write the safety argument in a comment | an entry still being appended to is evicted |
| **Cache what is expensive, re-read what is authoritative** | cache the client construction; still read the "is this integration still connected" row every call | a revoked credential keeps working until restart |
| **Key on everything the resource is built from** | config, credential version, region — all in the key | a rotated secret serves the stale client |
| **A per-unit cache is a local, not a module field** | create it in the handler, let it go out of scope | "memoise across calls" turns into "retain forever" |
| **Drain a response body you discard** | an unread `fetch` body holds its socket until GC — `await res.body?.cancel()` before a `throw` on `!res.ok` | a failing upstream leaks one socket per call |
| **Timers and listeners are released by their owner** | `clearInterval` / `removeEventListener` in the disposer; `unref()` a boot-lived timer | a per-connection interval keeps running after the client left |

## 📦 Cgroup caps — the limit is a backstop, not a detector

Every container and every per-run job gets a hard memory cap, so a leak kills one process instead of the node. Requests and limits → [../infrastructure/kubernetes-gitops.md](../infrastructure/kubernetes-gitops.md#-workload-runtime-contract).

| cgroup v2 file | Means |
|---|---|
| `memory.max` | hard cap — crossing it invokes the OOM killer inside the group |
| `memory.high` | soft cap — the kernel throttles and reclaims; set it below `max` for an early, survivable signal |
| `memory.current` | usage now, page cache included |
| `memory.events` | counters: `high`, `max`, `oom`, `oom_kill` — a rising `oom_kill` is the definitive record |

- **Headroom hides a leak.** A process using 1/10 of its limit grows for a long time before the kill. Watch the **trend** of usage, not the restart count.
- **Cap the children too.** A tool the daemon spawns (a compiler, `git`, a test runner) outside the per-run cgroup can starve the daemon itself; route every spawn through the one capped spawn seam.
- **Never raise the limit as the fix** without a measured working set that justifies it; a bigger limit buys a longer time-to-kill, not a bounded process.

## 💀 What OOMKill evidence looks like

| Where | Evidence |
|---|---|
| `kubectl describe pod <pod>` | `Last State: Terminated`, `Reason: OOMKilled`, `Exit Code: 137`, `Restart Count` climbing |
| `kubectl get events` | `OOMKilling` / container restarted |
| `kubectl logs <pod> --previous` | the killed container's last lines — **read it before the next restart replaces it** |
| kernel log (`dmesg`, `journalctl -k`) | `Memory cgroup out of memory: Killed process <pid> (<name>)` with the cgroup path |
| systemd unit | `systemctl status <unit>` → `Result: oom-kill` |
| the cgroup | `oom_kill` count in `memory.events` |

- **Exit `137` alone is `128 + SIGKILL`**, not proof of OOM — a grace-period kill gives the same code. `Reason: OOMKilled` or an `oom_kill` count is the proof.
- **The process cannot log its own OOMKill.** `SIGKILL` runs no handler. The record comes from the platform or a periodic memory sample written before the kill.

## 🩺 Shallow health checks hide leaks

`/livez` answers "the event loop responded" — correct, and blind to memory → [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md#-health-livez--readyz). A leaking pod is healthy until the instant it is killed.

| Do | Don't |
|---|---|
| export RSS, heap used and `memory.current` as metrics; alert on slope over hours | put a memory threshold in the liveness probe — the leak becomes a silent restart loop that drops in-flight work and erases the slope |
| alert on `oom_kill` increments and restart count | treat "no restarts this week" as "no leak" — the limit may just be large |
| emit a per-unit metric for the store sizes you bound (entries, bytes) | rely on a dashboard nobody opens; the alert is the mechanism |

## Working set vs occupancy

Two different numbers, both called "memory":

| Number | Is | Read it for |
|---|---|---|
| **RSS** | resident pages of the process, shared libs included | the OS view; what `top` shows |
| **Heap used** | live JS objects after GC | your objects; a leak in your code shows here first |
| **Container working set** | cgroup usage minus inactive file cache | what eviction and the OOM decision act on |
| **Page cache** | file pages the kernel may reclaim | not a leak; inflates `memory.current` |
| **Cache occupancy** | entries / bytes a store holds | compare to its **working set** — the entries actually hit in one window |

A cache whose occupancy keeps climbing past its working set is retaining, not caching. A heap that returns to the same floor after GC is fine at any height; a floor that rises every hour is the leak.

## 📈 Measuring over time

**Sample, then compare floors.** One reading proves nothing; the slope of the post-GC floor does.

```ts
// boot-lived sampler: one line per minute, unref'd so it never holds the process
const t = setInterval(() => {
  const m = process.memoryUsage();
  log.info("mem_sample", { rss: m.rss, heapUsed: m.heapUsed, external: m.external });
}, 60_000);
t.unref();
```

**Soak test** — run the unit of work N times in one process, force GC, assert the floor is bounded:

```ts
it("handleJob retains nothing per job", async () => {
  await warmUp(50);
  Bun.gc(true);
  const before = process.memoryUsage().heapUsed;
  for (let i = 0; i < 5_000; i++) await handleJob(fakeJob(i));
  Bun.gc(true);
  const grown = process.memoryUsage().heapUsed - before;
  expect(grown).toBeLessThan(5 * 1024 * 1024); // budget named per job class, not "small"
});
```

| Tool | When |
|---|---|
| post-GC heap floor over N units (above) | a regression test for a fixed leak — pins it |
| two heap snapshots, diffed by constructor count | finding *what* grows; take them N units apart after GC |
| a bounded store's own `size` as a test assertion | proving the cap and TTL work: insert past the cap, assert `size <= cap` |
| per-minute samples in prod logs | the slope, and the last value before a kill |

Greps produce noise: a repo-wide scan for `new Map` or `.on(` returns function-local aggregation maps and one-time startup listeners. There is no cheap lint for this class — the review questions are the tool.

## Review questions

| Ask | A bad answer |
|---|---|
| What is this cache's unit of work, and what is its actual lifetime? | "it lives on the module" |
| What bounds it — TTL, cap, both — and why is eviction safe? | "it won't get big" |
| Is it keyed on everything the value is built from? | keyed on the account id only, built from a rotating secret |
| What does the throw path leave behind — a body, a timer, a listener? | "we throw on non-200" |
| What metric would show it growing, and what alert fires? | "we'd see restarts" |

---

**Related:** [resource-lifetimes.md](resource-lifetimes.md) · [query-shape.md](query-shape.md) — wide reads that load payloads into memory · [../infrastructure/observability.md](../infrastructure/observability.md) · [testing.md](testing.md) · [../workflow/bug-sweep.md](../workflow/bug-sweep.md) — the scheduled leak checkup
