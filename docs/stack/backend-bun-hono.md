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
├── routes/            # health.ts, users.ts, …
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
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";

export function createApp() {
  const app = new Hono();
  app.get("/healthz", (c) => c.json({ ok: true }));

  const createUser = z.object({ email: z.string().email() });
  app.post("/users", zValidator("json", createUser), async (c) => {
    const body = c.req.valid("json");      // typed + validated
    const user = await userCreator.run(body); // delegate to a service
    return c.json(user, 201);
  });
  return app;
}
```
Routes stay thin: parse → validate → call a service → render. All logic lives in `services/` ([SRP](../architecture/solid-srp.md)).

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
- **Custom error classes** mapped to stable HTTP codes.
- **Tests** with TestContainers for anything touching the DB → [../architecture/testing.md](../architecture/testing.md).

When this isn't fast enough for a specific endpoint, that endpoint becomes a [Rust service](rust-apis.md) — in the same monorepo.
