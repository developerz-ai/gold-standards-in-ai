# 🐘 Databases & Storage

PostgreSQL is the default. Scale out to YugabyteDB. Cache and queue on Dragonfly. Store blobs in R2.

## The set
| Need | Use | Why |
|---|---|---|
| Relational data | **PostgreSQL 15+** | the default, everywhere |
| Distributed SQL at scale | **YugabyteDB** | Postgres-wire compatible → drop-in horizontal scale |
| Cache / queues / ephemeral | **Dragonfly** | Redis-compatible, faster, less memory |
| Object storage | **Cloudflare R2** | S3-compatible, **no egress fees** |
| ORM | **Drizzle** | type-safe, schema-first |
| Driver | **postgres.js** | promise-based, fast |

## Drizzle config
```ts
import type { Config } from "drizzle-kit";
export default {
  schema: "./src/schema/index.ts",
  out: "./migrations",
  dialect: "postgresql",
  dbCredentials: { url: process.env.DATABASE_URL! },
  strict: true,
  verbose: true,
} satisfies Config;
```

## Schema by domain, one re-export
```ts
// packages/db/src/schema/index.ts
export * from "./accounts.ts";
export * from "./wallets.ts";
export * from "./ledger.ts";
export * from "./sessions.ts";
```

## Migrations
- Generate from schema diff: `drizzle-kit generate` → numbered SQL in `migrations/`.
- Apply at runtime:
```ts
import { migrate } from "drizzle-orm/postgres-js/migrator";
import { db } from "./client";
await migrate(db, { migrationsFolder: "./migrations" });
```
- Never edit a shipped migration; add a new one.

## Connection
```ts
import postgres from "postgres";
import { drizzle } from "drizzle-orm/postgres-js";
export const db = drizzle(postgres(process.env.DATABASE_URL!));
```

## YugabyteDB — when & how
Yugabyte speaks the Postgres wire protocol, so the same Drizzle schema and `postgres.js` client work unchanged. Switch when you need multi-node HA, horizontal write scaling, or multi-region. Locally it runs in one container:
```yaml
yugabyte:
  image: yugabytedb/yugabyte:latest
  command: [bin/yugabyted, start, --daemon=false, --advertise_address=0.0.0.0]
  ports: ["5433:5433", "15433:15433"]
```

## Dragonfly — cache + jobs
Drop-in Redis replacement. One process handles what would need a Redis cluster.
```yaml
dragonfly:
  image: docker.dragonflydb.io/dragonflydb/dragonfly:latest
  ports: ["6379:6379"]
  ulimits: { memlock: -1 }
```
- App cache + BullMQ live here.
- Durable job queues use a **noeviction** instance (so jobs aren't dropped under memory pressure); ephemeral cache uses an eviction instance.
- Give each app its own DB index — no shared keys.

| Rule | Why / how |
|---|---|
| **Every cache key gets a TTL** — `SET k v EX 900`, never bare `SET` | keys without one accumulate until eviction picks for you |
| **`SCAN`, never `KEYS`** in app code | `KEYS` walks the whole keyspace in one blocking call |
| **Stampede protection on hot keys** | single-flight: one caller recomputes, the rest await the same promise; or refresh early before TTL — never N parallel recomputes on expiry |
| **A lock is released only by its holder** | `SET lock:x <token> NX PX 30000`; release = compare token then `DEL`, atomically (Lua `EVAL`) — a plain `DEL` frees someone else's lock after yours expired |
| Large blobs (> ~100KB) | store in R2, cache the key |

## Production hygiene
- **Pool through pgcat** on `:6432`; apps never connect directly to the primary.
- **Per-app database + role** — one DB and a least-privilege role per app; read-only roles for analytics/agents.
- **Daily `pg_dump` → R2**, with a weekly restore drill. A backup you've never restored is a hope, not a backup.
- **Read-only DB access for agents** goes through an audited gateway, never a raw connection string → [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md).

## Postgres hygiene
- **Timeouts on the role, not the session** — a pooler hands sessions around, so a `SET` doesn't stick:
```sql
ALTER ROLE app SET statement_timeout = '15s';
ALTER ROLE app SET idle_in_transaction_session_timeout = '30s';
```
  Long jobs/migrations override per transaction (`SET LOCAL statement_timeout = '10min'`) — an explicit exception beats a high global.
- **`pg_stat_statements` on from day one** — `order by total_exec_time desc` is the first query of any "it's slow" session.
- **Index the query, not the column:**

| Shape | Index |
|---|---|
| `where status = $1 and created_at > $2` | composite, equality first: `(status, created_at)` |
| hot subset (`where deleted_at is null`, `status = 'pending'`) | partial: `… where deleted_at is null` |
| read a few columns by key | covering: `(email) include (name)` — index-only scan |
| keyset pagination | the exact `order by` tuple: `(tenant_id, created_at, id)` |

- **Job claim = `FOR UPDATE SKIP LOCKED`** — workers never block on each other's rows:
```sql
update jobs set status = 'running', claimed_at = now()
where id = (select id from jobs where status = 'pending'
            order by created_at limit 1 for update skip locked)
returning *;
```
- Types: `timestamptz` not `timestamp`, `text` not `varchar(n)`, money as integer minor units + currency code, never float (→ [../frontend-craft/dates-money-timezones.md](../frontend-craft/dates-money-timezones.md)). IDs are UUIDv7 generated app-side → [../architecture/data-and-scale.md](../architecture/data-and-scale.md#-your-dev-engine-is-not-your-prod-engine).
- Lock-safe DDL and deploy ordering → [../architecture/data-and-scale.md](../architecture/data-and-scale.md#-lock-safe-ddl--deploy-ordering).

## R2 for blobs
S3-compatible API (use any S3 SDK), zero egress fees — ideal for user uploads, generated assets, and backups. Keep bucket creds in [sealed secrets / Vaultwarden](../infrastructure/secrets.md), never in code.
