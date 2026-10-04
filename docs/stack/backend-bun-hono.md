# 🦊 Backend — Bun + Hono + Drizzle

The default server stack. Same language as the frontend, sub-100ms cold start, type-safe to the database.

## The pieces
- **Bun** runtime (1.3+) — fast, native test runner, `--hot` reload.
- **Hono** — lightweight HTTP framework.
- **Drizzle ORM** + **postgres.js** — type-safe SQL.
- **BullMQ** + **Dragonfly/Redis** — background jobs.
- **Zod** — runtime validation at every boundary.

## App structure
```
apps/api/src/
├── main.ts            # boot server
├── app.ts             # Hono app factory
├── routes/            # health.ts (/livez, /readyz), users.ts, …
├── middleware/        # auth, logging, error
├── services/          # business logic (SRP)
├── db/                # Drizzle queries
└── test/{unit,integration}/
```

```json
{
  "name": "@product/api",
  "scripts": {
    "dev": "bun --watch src/main.ts",
    "start": "bun src/main.ts",
    "test": "bun test test/unit",
    "test:integration": "bun test test/integration"
  },
  "dependencies": {
    "@product/db": "workspace:*",
    "hono": "^4.6", "drizzle-orm": "^0.45", "bullmq": "^5", "ioredis": "^5", "zod": "^4"
  }
}
```

## Hono app + validated route
```ts
// app.ts
import { Hono } from "hono";
import { z } from "zod";
import { validate } from "./middleware/validate"; // zValidator wrapper → our error body

export function createApp() {
  const app = new Hono();

  const createUser = z.object({ email: z.string().email() });
  app.post("/users", validate("json", createUser), async (c) => {
    const body = c.req.valid("json");      // typed + validated
    const user = await userCreator.run(body); // delegate to a service
    c.header("Location", `/users/${user.id}`);
    return c.json(user, 201);
  });
  return app;
}
```
Routes stay thin: parse → validate → call a service → render. All logic lives in `services/` ([SRP](../architecture/solid-srp.md)). Errors, status codes, pages, idempotency → [api-contracts.md](../architecture/api-contracts.md). Authz in the data layer, sessions/CSRF/CSP, uploads, rate limits → [app-security.md](../architecture/app-security.md).

## 🩺 Health: `/livez` ≠ `/readyz`
| Endpoint | Checks | Fails → | Probe |
|---|---|---|---|
| `/livez` | nothing — the event loop answered | process restarted | liveness |
| `/readyz` | `select 1` + Redis `PING`, each with a short timeout; `503` while draining | pod pulled from the load balancer | readiness |

Never put a dependency in `/livez`: a DB blip would restart every pod at once and turn an outage into a crash loop.

```ts
// routes/health.ts
let draining = false;
export const startDraining = () => { draining = true; };

app.get("/livez", (c) => c.text("ok"));
app.get("/readyz", async (c) => {
  if (draining) return c.text("draining", 503);
  const ok = await Promise.all([
    sql`select 1`.then(() => true, () => false),
    redis.ping().then(() => true, () => false),
  ]).then((r) => r.every(Boolean));
  return c.text(ok ? "ok" : "deps down", ok ? 200 : 503);
});
```

## 🛑 Graceful shutdown on SIGTERM
Rolling deploys send `SIGTERM`. Exit clean or drop requests and jobs:

```ts
// main.ts
const server = Bun.serve({ port: Number(process.env.PORT), fetch: app.fetch });

process.on("SIGTERM", async () => {
  startDraining();                        // 1. /readyz → 503
  await Bun.sleep(5_000);                 // 2. let the ingress stop routing here
  await server.stop();                    // 3. refuse new, finish in-flight
  await Promise.all(workers.map((w) => w.close()));  // 4. BullMQ: finish current jobs
  await sql.end({ timeout: 5 });          // 5. close the DB pool
  await redis.quit();
  process.exit(0);
});
```

Total drain time must fit inside the pod's `terminationGracePeriodSeconds` → [../infrastructure/kubernetes-gitops.md](../infrastructure/kubernetes-gitops.md#-workload-runtime-contract). A job longer than that must be resumable, not "allowed to finish" → [data-and-scale.md](../architecture/data-and-scale.md#-long-jobs-a-lease-is-not-a-timeout).

## Background jobs (BullMQ)
```ts
import { Queue, Worker } from "bullmq";
const connection = { url: process.env.REDIS_URL };
export const emails = new Queue("emails", { connection });

new Worker("emails", async (job) => {
  await emailSender.run(job.data);   // a service, again
}, { connection });
```
Point BullMQ at **Dragonfly** in prod → [databases.md](databases.md).

## Drizzle schema
```ts
import { pgTable, uuid, text, timestamp } from "drizzle-orm/pg-core";

export const wallets = pgTable("wallets", {
  id: uuid("id").primaryKey().defaultRandom(),
  accountId: uuid("account_id").notNull(),
  name: text("name").notNull(),
  createdAt: timestamp("created_at", { withTimezone: true }).notNull().defaultNow(),
});
```
Split schema by domain, re-export from `packages/db/src/schema/index.ts`. Migrations: `drizzle-kit generate` → `bun run src/migrate.ts`. Full detail in [databases.md](databases.md).

## Conventions
- **postgres.js**, not node-postgres.
- **Zod at every external boundary** — requests, env, webhooks.
- **Connect through a pooler** (pgcat) in prod, never direct to the primary.
- **Custom error classes** mapped to stable HTTP codes through one registry + `app.onError` → [api-contracts.md](../architecture/api-contracts.md).
- **Tests** with TestContainers for anything touching the DB → [../architecture/testing.md](../architecture/testing.md).

When this isn't fast enough for a specific endpoint, that endpoint becomes a [Rust service](rust-apis.md) — in the same monorepo.
