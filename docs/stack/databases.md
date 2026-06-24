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

## Production hygiene
- **Pool through pgcat** on `:6432`; apps never connect directly to the primary.
- **Per-app database + role** — one DB and a least-privilege role per app; read-only roles for analytics/agents.
- **Daily `pg_dump` → R2**, with a weekly restore drill. A backup you've never restored is a hope, not a backup.
- **Read-only DB access for agents** goes through an audited gateway, never a raw connection string → [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md).

## R2 for blobs
S3-compatible API (use any S3 SDK), zero egress fees — ideal for user uploads, generated assets, and backups. Keep bucket creds in [sealed secrets / Vaultwarden](../infrastructure/secrets.md), never in code.
