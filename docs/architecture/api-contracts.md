# 🔌 API Contracts — errors, status codes, pages, retries

The HTTP wire contract every service speaks. One error registry, one error body, keyset pages, idempotent writes. The compiler enforces the mapping; the agent doesn't have to remember it.

## 📏 The rules

| Rule | Instead of |
|---|---|
| Every error has a **stable string code** from one registry | ad-hoc `message` strings the client greps |
| Registry maps code → status with `satisfies Record<ErrorCode, …>` — a missing mapping fails the build | a `switch` with a `default: 500` |
| **One** error body shape, everywhere | per-route `{ error }`, `{ msg }`, `{ success: false }` |
| Semantic status codes; never `200` with `success: false` | one status for everything |
| `POST` that creates → `201` + `Location` | `200` + body |
| Lists are **keyset** pages: `next_cursor` + `has_next`, always | offset pages "for small admin tables" |
| `POST` with side effects accepts `Idempotency-Key` | hoping the client never retries |
| Throttled → `429` + `Retry-After` | `503` or silent drop |
| Internal API: no version; both ends change in one PR | `/v2` beside `/v1` |

## 🗂️ One error registry

Domain errors are [custom classes](solid-srp.md#custom-error-classes--never-generic). Each carries a code; the registry owns the status.

```ts
// packages/core/src/errors.ts
import type { ContentfulStatusCode } from "hono/utils/http-status";

export const ERROR_STATUS = {
  validation_failed: 400,
  unauthenticated: 401,
  forbidden: 403,
  not_found: 404,
  conflict: 409,
  idempotency_key_reused: 422,
  insufficient_funds: 422,
  rate_limited: 429,
  internal: 500,
  upstream_unavailable: 503,
} as const satisfies Record<string, ContentfulStatusCode>;

export type ErrorCode = keyof typeof ERROR_STATUS;

export class AppError extends Error {
  constructor(
    readonly code: ErrorCode,
    message: string,
    readonly fields?: FieldError[],
  ) { super(message); }
}
export class NotFoundError extends AppError {
  constructor(what: string) { super("not_found", `${what} not found`); }
}
export class InsufficientFundsError extends AppError {
  constructor() { super("insufficient_funds", "Balance too low for this transfer"); }
}
```

- `ErrorCode` is derived from the registry → a code can't exist without a status; a typo'd code, a removed code still thrown, or a non-status number fails the build.
- TS clients import `ErrorCode` from the same package → `switch (err.code)` is exhaustive too ([monorepo](monorepo.md)). Native apps mirror it in their shared API package.
- **Codes are a public promise.** Rename = breaking change. Add freely; never repurpose.

## 📦 One error body

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Request validation failed",
    "fields": [{ "path": "email", "code": "invalid_format", "message": "Invalid email address" }],
    "request_id": "0192f0c4-7a1e-7c3b-9d2a-5e8f1b6c4a10"
  }
}
```

| Field | Rule |
|---|---|
| `code` | from the registry; the only thing a client branches on |
| `message` | developer-readable, safe to show; never a stack, SQL, or internal hostname |
| `fields` | only for `validation_failed`; built from Zod issues (`path.join(".")`, `code`, `message`) |
| `request_id` | always; same id in the server log line → one grep from a user report to the trace |

User-facing copy is the client's job: map `code` → localized text, never render `message` raw → [../frontend-craft/ux-copy.md](../frontend-craft/ux-copy.md). UI error states per status → [../frontend-craft/design-review-loop.md](../frontend-craft/design-review-loop.md#-harden-design-for-real-data-not-demo-data).

## 🪝 Wire it once in Hono

```ts
// apps/api/src/middleware/validate.ts
import { zValidator } from "@hono/zod-validator";
import type { ZodType } from "zod";
import { AppError } from "@product/core/errors";

export const validate = <T extends ZodType>(target: "json" | "query" | "param", schema: T) =>
  zValidator(target, schema, (result) => {
    if (!result.success) {
      throw new AppError("validation_failed", "Request validation failed",
        result.error.issues.map((i) => ({ path: i.path.join("."), code: i.code, message: i.message })));
    }
  });
```
```ts
// apps/api/src/app.ts
import { requestId } from "hono/request-id";
import { AppError, ERROR_STATUS } from "@product/core/errors";

app.use(requestId());
app.onError((err, c) => {
  const request_id = c.get("requestId");
  if (err instanceof AppError) {
    const status = ERROR_STATUS[err.code];
    if (status >= 500) log.error({ err, request_id });
    return c.json({ error: { code: err.code, message: err.message, fields: err.fields, request_id } }, status);
  }
  log.error({ err, request_id });                       // unknown → full detail in the log only
  return c.json({ error: { code: "internal", message: "Internal error", request_id } }, 500);
});
```

- Routes and services **throw**; only `onError` renders. No `try/catch` in handlers that re-shapes errors.
- Use `validate(...)`, never bare `zValidator` — its default 400 body isn't our shape. A lint guard on `zValidator(` outside `middleware/validate.ts` keeps it that way → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md).

## 🔢 Status codes we use

| Status | When |
|---|---|
| `200` | read, update with body |
| `201` + `Location: /users/<id>` | created; body = the resource |
| `204` | delete; update with no body |
| `400 validation_failed` | malformed JSON or fails the Zod schema |
| `401` / `403` | no/invalid credentials / authenticated but not allowed → [app-security.md](app-security.md) |
| `404` | missing **or not yours** — don't leak existence across tenants |
| `409` | state conflict: duplicate, stale version, same idempotency key still in flight |
| `422` | valid request, business rule says no (`insufficient_funds`) |
| `429` | throttled; always `Retry-After` (seconds) |
| `500` | bug. Generic message; detail in the log |
| `503` | dependency down / draining; `Retry-After` when known |

## 📄 Pagination — keyset, always

```
GET /wallets?limit=50&cursor=0192f0c4-7a1e-7c3b-9d2a-5e8f1b6c4a10
```
```json
{ "data": [/* … */], "next_cursor": "0192f0c5-…", "has_next": true }
```

- IDs are **UUIDv7** (time-ordered, app-side) → the cursor is the last id; `where id > $cursor order by id limit $limit + 1`. Extra row → `has_next`.
- Sorting by another column → cursor encodes `(sort_value, id)`, base64url, opaque to the client.
- `limit` over the max → `400`, never a silent clamp → [data-and-scale.md](data-and-scale.md#work-per-run-is-bounded-by-a-constant-not-by-table-size).
- No offset, not even for admin tables — they grow, and offset cost tracks corpus size. "Jump to page 40" is a search/filter problem.
- No `total` by default; a `count(*)` per page is the offset problem in disguise. Add it only behind a measured need.

## 🔁 Idempotency-Key on writes

Any `POST` that creates, charges, sends, or enqueues accepts `Idempotency-Key: <uuid>`.

| Case | Response |
|---|---|
| New key | run, store `(key, caller, body hash) → (status, body)` for 24h, return it |
| Same key, same body, finished | replay the stored response; do nothing |
| Same key, still running | `409 conflict` |
| Same key, different body | `422 idempotency_key_reused` |

Store in Postgres (unique on `(caller, key)`, inside the write's transaction) so the record and the side effect commit together. Same deterministic-id idea as queue jobs → [data-and-scale.md](data-and-scale.md#-idempotency--exactly-once-enough).

## 🔄 Client retries

- Retry only **network errors, `429`, `502`, `503`, `504`**. Never other `4xx`; never `500` on a non-idempotent call.
- Retry only **safe/idempotent requests**: `GET`, `PUT`, `DELETE`, or a `POST` carrying an `Idempotency-Key` (reuse the same key on every attempt).
- Exponential backoff with **full jitter**: `delay = random(0, min(cap, base * 2^attempt))`. Honour `Retry-After` when present.
- Bound by a **deadline** (e.g. 10s total for a user-facing call), not by hoping. Surface the last error with its `request_id`.
- One shared `ApiClient` does this; no call site rolls its own loop → [solid-srp.md](solid-srp.md#reusable-helpers-not-copy-paste).

## 🏷️ Versioning

| API | Rule |
|---|---|
| **Internal** (our web, mobile, workers) | No version. Server + every client change in the **same PR** — the monorepo makes that one diff → [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md) |
| **Public** (third parties call it) | `/v1` path prefix. Additive changes (new fields, optional params, endpoints) stay in `v1`. Breaking → `/v2`, `Sunset` header on `v1`, then `410` after the date |

Mobile is the one internal client you can't force-upgrade: additive-only for fields shipped apps read, remove them only after the minimum supported app version stops reading them → [../stack/mobile.md](../stack/mobile.md).

---

**Related:** [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md) · [solid-srp.md](solid-srp.md) · [data-and-scale.md](data-and-scale.md) · [abstractions-and-growth.md](abstractions-and-growth.md) — derive the contract from one declaration · [app-security.md](app-security.md) · [testing.md](testing.md)
