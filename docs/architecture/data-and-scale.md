# 📈 Data & Scale — shape now, capacity later

An agent will happily write the naive version of anything. Most of the time that's correct. The exception is a small set of decisions that **cost the same today as the naive version and are expensive to retrofit** — get those right up front, and defer everything else behind a written trip condition.

## Shape vs capacity

| | **Shape** — decide now, always | **Capacity** — defer, always |
|---|---|---|
| Costs today | ~nothing | real work |
| Costs later | a migration + a backfill + an outage | just the work you deferred |
| Examples | keyset vs offset · watermark vs recomputed window · per-tick bounds · partition key · idempotency key · retention policy · the *contract* a subsystem promises (exhaustive vs sampled) | sharded workers · rollup tables · archival tiering · streaming reads · caching layers |

**Never build capacity on a forecast.** Write the trip condition instead — in the module, next to the code:

```ts
/** Trip condition: shard this worker when `select count(*) from events where …` > 5M
 *  (measured 2026-08-05: 38k — 130× away). Until then a single worker is correct. */
```

**Measure before you bound.** One `count(*)` settles most "it won't scale" arguments — and usually shows the threshold is 13×–200× away. A trip query in the module beats a plan document nobody reads.

The anti-forgetting mechanism is a **written threshold + an assertion that throws + a lint guard** — never a dashboard, never a monitor, never a `TODO`.

## Work per run is bounded by a constant, not by table size

A background read's cost must track **change volume, not corpus size**.

- **Keyset, never offset.** `where (created_at, id) > ($1,$2) order by … limit N`.
- **Every cross-tenant read carries a `LIMIT` or a time window.** Every fan-out loop carries a cap.
- **A watermark table is the generic cursor** — one row per sweep, advanced only on success, so a restart resumes instead of recomputing.
- **A sweep that cannot be exhaustive says so in its header.** "Bounded, best-effort: at most N per tick" is a contract; discovering it at 3am is an incident.
- **Report saturation, don't hide it.** A bound hit *every* tick means permanently behind — but a `candidates: 200, enqueued: 200` log looks healthy. Return distinct values for *finished* and *ran out of budget*, and alert on the second.
- **A silent clamp is worse than the unbounded read it replaced.** If a caller asks for more than the cap, throw or report — never quietly return 100 rows and let the fleet believe that's all of them.

```ts
const { rows, exhausted } = await drain(cursor, { limit: 500, deadlineMs: 30_000 });
if (!exhausted) log.warn("sweep budget hit", { cursor, rows: rows.length }); // resumable, not silent
```

**Fail fast on anomaly, never on ordinary growth.**

## ⏳ Long jobs: a lease is not a timeout

In most queue systems the "lock duration" is a **liveness heartbeat** — it renews on a timer while the job runs. It does **not** bound job duration. The lease only lapses when the process dies or the event loop is starved, and then a **still-executing** job gets handed to a second worker.

Consequences to design for:
- A hung network call runs until the pod does. Bound it yourself: an `AbortSignal` on anything awaiting the network, plus a wall-clock deadline inside any `for(;;)` loop.
- Derive the deadline from the **lease**, not from typical duration (e.g. `lease/2`).
- **A budget must reject, not merely abort.** If the drain checkpoint keys on "aborted" without inspecting *why*, a timeout looks like a cooperative stop — the job is re-delayed forever, never failing, never paging. Throw when a single unit blows its own budget, naming the seam and the budget.
- **Stopping early with work remaining is correct and resumable.** That's the watermark's job.

## 🧱 Migrations vs backfills

They look alike and behave completely differently. Split them, mechanically.

| | **Migration** | **Backfill** |
|---|---|---|
| Is | DDL | a rewrite of existing rows |
| Runs | once, forever, in every environment, on the deploy path | as a job, resumable, off the deploy path |
| Never | seeds business entities · rewrites existing rows | changes schema |
| Lives in | `migrations/NNNN_*.sql` | `src/backfills/NNNN-slug.ts` + a registry + a `backfill_runs` ledger |

- **A migration never seeds business rows.** It applies once forever and cannot be opted out of — a seeded org/user/entitlement can never be taken back. New business rows belong to the **seed layer** (idempotent, re-runnable).
- **A migration never rewrites rows.** A rewrite inside DDL is dead weight on every dev box and CI database, and it holds the deploy open.
- **A merged backfill runs itself** if you wire the post-sync job — so write it as if it executes the moment it merges, because it does. Post-sync precisely so a failure marks the job failed and *leaves the release up*.
- **Converging contract**: every backfill exposes `remaining()` and `step()`, is dry-run by default, and stamps who ran it.
- **Never hand-edit the migration journal's timestamps.** A future stamp makes the runner skip every later migration — silently, in prod.
- **A hand-written data migration ships with an integration test that executes the shipped `.sql` file** over fixtures. Nothing else parses that SQL, so a plain typo ships green locally and breaks every DB-backed CI job at once.

## 🧪 Your dev engine is not your prod engine

If local/CI is stock Postgres and production is a distributed SQL engine (Yugabyte, CockroachDB, Aurora DSQL, Spanner-ish), then **a violation is green on both gates and red in the deploy** — after merge, because migrations self-deploy.

- Encode the known-bad shapes as a **static guard** on migration SQL: foreign keys, `serial`/identity ids, in-DB UUID defaults, extensions prod doesn't ship.
- Tenant tables: composite PK `(tenant_id, id)` so the engine co-locates a tenant's rows. UUID v7 generated **app-side**.
- Keep transactions small — distributed commits are expensive.
- **Some behaviors cannot be proven on a dev box.** Write-conflict retries are the classic: a distributed engine aborts same-row races with a serialization error; stock Postgres under READ COMMITTED blocks and applies. So the retry wrapper and its regression test pass *trivially* locally. Say that in the test file, and get the real signal from a staging engine (or by raising the local isolation level for those suites).

## 🔁 Idempotency & exactly-once-enough

- **Every enqueue carries a deterministic id** built from the domain key, so a redelivery is a no-op. Validate the id at the **single enqueue seam** — never let each producer construct its own queue client (that path also silently drops your default retry options).
- **Watch the separator.** Many queue systems use `:` in key composition; an id containing one can make a sweep enqueue **zero** jobs, silently. Assert the character set at the seam and lint the shape repo-wide.
- **Optimistic concurrency on a timestamp column is a trap.** Databases store microseconds; JS `Date` truncates to milliseconds, so `where updated_at = $read` matches **0 rows** and the guarded write silently never fires. Use a version column, or truncate both sides explicitly.

---

**Related:** [testing.md](testing.md) · [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) · [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md) — how each rule above becomes a guard
