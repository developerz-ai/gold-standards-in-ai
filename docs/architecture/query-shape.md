# 🔎 Query Shape — one statement per question, not per row

The slow-database incident is rarely one bad query. It is a fast query called once per box, per repo, per account, every tick — each call cheap, the sum saturating a small connection pool shared with every background job. Query count is a **shape** decision: settle it when the port is written → [data-and-scale.md](data-and-scale.md#shape-vs-capacity). As of 2026-11.

Keyset paging, bounded sweeps and watermarks → [data-and-scale.md](data-and-scale.md#work-per-run-is-bounded-by-a-constant-not-by-table-size). Pooler, role timeouts, indexes → [../stack/databases.md](../stack/databases.md#postgres-hygiene).

## The shapes

| Shape | Looks like | Costs | Fix |
|---|---|---|---|
| **N+1** | `for (const id of ids) await findX(db, id)` | N round trips; latency × N | a batched port: one `= ANY($1)` |
| **Parallel N+1** | `Promise.all(ids.map((id) => findX(db, id)))` | N statements **and** up to N pool connections at once — starves every other caller | same batched port; never "fixed" by `Promise.all` |
| **List-then-filter** | `(await listByAccount(db, a)).find((r) => r.name === n)` | every row, every column, to use one | an indexed lookup port; a `count` port for `.length` |
| **Wide read** | `select *` on a table carrying payload / JSON / blob columns | bytes over the wire and into the heap → [memory-and-leaks.md](memory-and-leaks.md) | project the columns the caller reads |
| **Unbounded read** | multi-row read with no `LIMIT` | grows with the table, not with the request | a page limit asserted at the port |
| **Write per item** | one `insert` per touched row in a loop | N statements, N WAL flushes | one multi-row `insert … values (…), (…)` or `unnest` |
| **Fire-and-forget write on a read path** | `void db.update(…)` per recall / view | unbounded concurrent writes, no back-pressure on the pool | coalesce into one batched write per tick, or measure and write the trip condition |

**Root cause is almost always the port, not the caller.** When the data layer only exposes `findX(db, key)`, every caller with a list writes a loop. Ship the batched sibling with the single-row one.

## Batched ports

```ts
/** Past this a batch is a different query plan: the caller pages its keys instead. */
export const RUNNER_STATUS_MAX_KEYS = 500;

/** One statement for N keys. Missing keys map to `undefined`, never dropped silently. */
export async function findRunnerStatusMany(
  db: Db, accountId: AccountId, ids: readonly RunnerId[],
): Promise<Map<RunnerId, RunnerStatus | undefined>> {
  if (ids.length === 0) return new Map();
  if (ids.length > RUNNER_STATUS_MAX_KEYS) {     // the cap the rule below demands: throw, never truncate
    throw new Error(
      `findRunnerStatusMany: ${ids.length} ids > RUNNER_STATUS_MAX_KEYS (${RUNNER_STATUS_MAX_KEYS}); page the keys`,
    );
  }
  const rows = await db.select({ id: runners.id, status: runners.status })
    .from(runners)
    .where(and(eq(runners.accountId, accountId), inArray(runners.id, [...ids])))
    .orderBy(runners.id);                         // stable order: deterministic tests and plans
  const byId = new Map(rows.map((r) => [r.id, r.status]));
  return new Map(ids.map((id) => [id, byId.get(id)]));
}
```

| Rule | Why |
|---|---|
| Tenant scope stays in the batched `WHERE` | a batch over ids alone crosses tenants → [app-security.md](app-security.md) |
| Return a map keyed by the input, missing keys explicit | a caller comparing `rows.length` to `ids.length` cannot tell which was missing |
| Stable `ORDER BY` | a test asserting a list, and a plan cache, both need it |
| Cap the key array; throw past it | `ANY` with 100k keys is a different query plan and a different incident |
| Return a domain shape, not the row | the projection is the one read seam; a second projection of the same table drifts |

A loop over a **fixed-cardinality** set (a 4-member enum, a config list) is not an N+1 — do not "fix" it → [../writing-for-agents/reviewer-agents.md](../writing-for-agents/reviewer-agents.md#-skip-list-llm-false-positives-on-our-stack).

## Keyset, not offset

The rule and its `WHERE` shape → [data-and-scale.md](data-and-scale.md#work-per-run-is-bounded-by-a-constant-not-by-table-size). What bites when you implement it:

| Trap | Mechanism | Do |
|---|---|---|
| Cursor and seek built separately | the seek compares keys in one order, the cursor writes them in another; pages skip or repeat | one declaration emits both the `WHERE` seek and the `keyOf(row)` cursor extractor |
| Non-unique order | `(created_at)` alone ties; rows on the page boundary vanish | always end with a unique, non-null tiebreak: `(created_at, id)` |
| **Nullable sort key** | `x < NULL` is `NULL`, not false — the seek drops every null row | `NULLS LAST` in both directions, plus an explicit `OR col IS NULL` branch; or refuse a nullable sort key |
| Timestamp precision | the DB stores µs, the client truncates to ms; the cursor never matches its own row | ordering columns at millisecond precision, asserted at declaration |
| Index does not match the order | a nullable `desc NULLS LAST` key gets a seq scan + full sort | index the exact order tuple: `(scope, col DESC NULLS LAST, id DESC)` |
| "Seek is free at any depth" | the expanded `(k < v OR (k = v AND id < x))` reaches the planner as a **filter** on an ordered scan, not an index range | measure deep pages with `EXPLAIN (ANALYZE, BUFFERS)`; when every key sorts the same direction, try the row-value form `(k, id) < (v, x)` and confirm in the plan that it became an index bound |
| A sort field with no key mapping | silently orders by fewer keys than the cursor carries | throw at declaration |

Offset that remains is listed in one place with why, never discovered.

## 🚰 Pool pressure

A pool is a shared budget across requests, crons, consumers and the outbox relay. One handler that fans out takes it from all of them.

| Rule | Mechanism |
|---|---|
| **No unbounded `Promise.all` over a DB port** | N in flight takes N connections; the next request waits on `acquire`, not on the DB |
| **Bound fan-out with a concurrency limit** when a batch is genuinely impossible | `pMap(items, fn, { concurrency: 4 })` — a number with a reason, below the pool size |
| **No network call inside a transaction** | the connection is held for the call's latency; a hung upstream pins a pool slot |
| **Short transactions** | a long one holds locks and a connection, and blocks vacuum |
| **Size the pool against the pooler and the replica count** | `pods × pool size` must fit the pooler's server-side limit |
| **Measure acquire wait** | a pool-wait metric rising before query latency is pool pressure, not a slow query |

## ⏲️ Statement timeouts

Role-level `statement_timeout` and `idle_in_transaction_session_timeout`, with a `SET LOCAL` override for long jobs → [../stack/databases.md](../stack/databases.md#postgres-hygiene). Add on the client side:

| Layer | Bounds |
|---|---|
| Role `statement_timeout` | one statement, server-side — the floor everything inherits |
| `SET LOCAL statement_timeout` inside a transaction | an explicit, reviewed exception for a migration or backfill |
| `AbortSignal` / deadline on the call | the caller's wait, including pool acquire — a server timeout does not cover a connection you never got |
| Ad-hoc operator queries | a wrapper that adds a short timeout and a time window — a read-only scan with no `ts` bound can run for most of an hour on a shared primary |

A timeout is a **rejection with the seam's name**, never a retry that re-queues the same statement forever.

## 🔬 Detect it

**In tests — count statements.** A batched port's test asserts the shape, not just the result.

```ts
it("loads N runners in one statement", async () => {
  const db = countingDb(testDb);                      // wraps the driver, counts executed statements
  await findRunnerStatusMany(db, account, ids(50));
  expect(db.statements).toBe(1);
});

it("the fleet tick is O(1) in boxes", async () => {
  const small = await statementsFor(() => tick(withBoxes(2)));
  const large = await statementsFor(() => tick(withBoxes(200)));
  expect(large).toBe(small);
});
```

**In CI — guards, each a ratchet that only falls** → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md#custom-lint-guards--institutional-memory-that-executes):

| Guard | Refuses |
|---|---|
| query-in-loop | a data-layer port awaited inside `for` / `.map` / `.forEach` outside the data package |
| fan-out | `Promise.all` over a port with an unbounded array |
| unbounded read | a multi-row read with no `LIMIT` or page assertion |
| wide read | `select *` on a named payload-bearing table |

**In prod — `pg_stat_statements`.** The N+1 fingerprint is many calls, about one row each, sub-millisecond mean:

```sql
select calls,
       round(mean_exec_time::numeric, 2)  as mean_ms,
       round(total_exec_time::numeric)    as total_ms,
       rows / nullif(calls, 0)            as rows_per_call,
       left(query, 120)                   as query
from pg_stat_statements
order by calls desc
limit 20;
```

| Signal | Reads as |
|---|---|
| top by `calls`, `rows_per_call` ≈ 1 | an N+1 — find the loop |
| top by `total_exec_time` | where the database's time goes; fix these first |
| high `mean_ms`, few calls | a genuinely slow query — `EXPLAIN (ANALYZE, BUFFERS)` it |
| `pg_stat_user_indexes.idx_scan = 0` over a **stated window** | an unused index taxing every write; drop it in a migration, behind a backup |

Record before/after numbers with the query and the date next to them; a number without its query cannot be re-checked.

## Review questions

| Ask | A bad answer |
|---|---|
| How many statements does this run for N items? | "one per item, but it's fast" |
| Is there a batched port, and if not, why does the caller loop? | "the port only takes one key" |
| What does this read select, and does the caller use it? | `select *` to read `hash` |
| How many pool connections can this hold at once? | an unbounded `Promise.all` |
| What bounds this statement — server timeout, client deadline, `LIMIT`? | "it's always small" |

---

**Related:** [data-and-scale.md](data-and-scale.md) · [../stack/databases.md](../stack/databases.md) · [api-contracts.md](api-contracts.md) — keyset pages on the wire · [resource-lifetimes.md](resource-lifetimes.md) · [memory-and-leaks.md](memory-and-leaks.md)
