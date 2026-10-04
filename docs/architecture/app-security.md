# 🛡️ App Security — authz, input, sessions, uploads, logs

The app-layer rules for what we ship on Bun + Hono + Drizzle + SolidJS, R2, and [Zitadel SSO](../infrastructure/sso-zitadel.md). Each rule is a default an agent applies without being asked, and each one has a test or a gate behind it. As of 2026-10.

**Not here:** secrets storage → [../infrastructure/secrets.md](../infrastructure/secrets.md) · dependency/supply chain + workflow security → [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) · prompt injection / untrusted text fed to an LLM → [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md) · error shape + status codes → [api-contracts.md](api-contracts.md).

## ✅ The checklist

| Area | Do | Never |
|---|---|---|
| Authorization | tenant/ownership filter **inside the data-access layer**; one cross-tenant test per resource | a per-route `if (row.ownerId !== user.id)` an agent can forget |
| Not yours | `404` — same as missing → [status codes](api-contracts.md#-status-codes-we-use) | `403` (leaks that the id exists) |
| Input | Zod at the edge via `validate(...)`; Drizzle query builder / `sql` template | string-built SQL, `sql.raw(userInput)` |
| Output | Solid JSX escaping; sanitize before any `innerHTML` | `innerHTML={userText}`; user URL in `href` unchecked |
| Sessions | opaque id in `__Host-` cookie: `HttpOnly; Secure; SameSite=Lax` | tokens in `localStorage` / readable cookies |
| CSRF | JSON-only mutations + `csrf()` + exact-origin CORS | `Access-Control-Allow-Origin: *` with credentials |
| Headers | `secureHeaders()` + strict CSP, no `'unsafe-inline'` / `'unsafe-eval'` | CSP loosened "for now" with no removal plan |
| Uploads | presigned PUT direct to R2, verify size + magic bytes after, serve from a separate origin | trusting extension or `Content-Type`; serving uploads from the app origin |
| Rate limits | Dragonfly counter on auth + expensive routes → `429` + `Retry-After` | unthrottled login, OTP, export, LLM-backed routes |
| Logs | redaction configured **in the logger**; log ids, not values | passwords, tokens, cookies, `Authorization`, PII in logs or Sentry |
| Frontend env | only public values in `VITE_*`; explicit `define` keys | secrets in any var the bundle can see |
| Source maps | `sourcemap: "hidden"` → upload to Sentry → delete from `dist/` | `.map` files on the public origin |
| Server fetch of user URLs | resolve, block private ranges, no blind redirects | `fetch(userUrl)` from inside the cluster |

## 🔐 Authorization lives in the data layer

Routes never touch raw `db`. Auth middleware builds a `Scope` from the session once; every repo function takes it and filters by it. Forgetting the check becomes a type error, not a review comment.

```ts
// packages/db/src/repos/wallets.ts
import { and, eq } from "drizzle-orm";
import { NotFoundError } from "@product/core/errors";

export type Scope = { accountId: string; userId: string; roles: string[] };

export const walletsRepo = (db: Db, s: Scope) => ({
  async get(id: string) {
    const [row] = await db.select().from(wallets)
      .where(and(eq(wallets.id, id), eq(wallets.accountId, s.accountId))).limit(1);
    if (!row) throw new NotFoundError("wallet");   // missing OR not yours → 404
    return row;
  },
  async rename(id: string, name: string) {
    const [row] = await db.update(wallets).set({ name })
      .where(and(eq(wallets.id, id), eq(wallets.accountId, s.accountId))).returning();
    if (!row) throw new NotFoundError("wallet");
    return row;
  },
});
```

- **Writes filter too.** `update … where id = $1` without the tenant column is the classic IDOR — put the tenant in the `where`, not in a prior read.
- **Roles from Zitadel claims** go into `Scope.roles`; role checks live in the service, next to the action (`403` is for "authenticated, this tenant, not allowed").
- **Cross-tenant code is named.** Admin/ops reads go through an `unscoped/` module; a lint guard fails any import of the raw `db` client outside `packages/db` → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md).
- Postgres RLS is an optional second wall, not the first. The repo filter is what tests prove.

**One test per resource, per verb** — the gate that makes the rule real → [testing.md](testing.md):

```ts
test("wallets: another tenant gets 404 on read, rename, delete", async () => {
  const [a, b] = [await seedAccount(), await seedAccount()];
  const w = await seedWallet(a);
  for (const [method, body] of [["GET"], ["PATCH", { name: "x" }], ["DELETE"]] as const) {
    const res = await app.request(`/wallets/${w.id}`, {
      method, headers: { ...authAs(b), "content-type": "application/json" },
      body: body && JSON.stringify(body),
    });
    expect(res.status).toBe(404);
  }
  expect((await getWallet(w.id)).name).toBe(w.name);   // untouched
});
```

Lists are the same rule: a list endpoint scoped by the repo never returns another tenant's rows — seed two tenants, assert the count.

## 🧼 Input and output

- **Validate at the edge, once.** `validate("json" | "query" | "param", schema)` from [api-contracts.md](api-contracts.md#-wire-it-once-in-hono). Services receive typed data; they don't re-validate.
- **Bound everything.** `z.string().max(…)`, `z.array(…).max(…)`, enums over free text, `z.uuid()` for ids (UUIDv7). An unbounded string is a DoS and a log-flood.
- **SQL:** Drizzle builder or the `sql` tagged template (parameterized). `sql.raw()` only with constants; a lint guard flags `sql.raw(` with a non-literal arg.
- **Error bodies** never carry stacks, SQL, or hostnames — `onError` already enforces this → [api-contracts.md](api-contracts.md#-one-error-body).
- **Solid escapes** text children and attributes. It does **not** protect:

| Sink | Fix |
|---|---|
| `innerHTML={html}` | render text instead; if HTML is the product (rich text, markdown), `DOMPurify.sanitize(html)` first |
| `href={userUrl}` / `src={userUrl}` | allow only `https:` / `mailto:` — `new URL(u).protocol` check; `javascript:` passes JSX escaping |
| `style={userCss}` | never user-controlled |
| `window.open(userUrl)` / `location.href = …` | same protocol check |

## 🍪 Sessions, CSRF, headers

Behind-login apps are SPAs ([frontend-solidjs.md](../stack/frontend-solidjs.md)); the API does the Zitadel OIDC code flow (PKCE) server-side and hands the browser an **opaque session id** — tokens stay on the server (session record in Dragonfly).

```ts
// apps/api/src/middleware/security.ts
import { secureHeaders } from "hono/secure-headers";
import { csrf } from "hono/csrf";
import { cors } from "hono/cors";
import { bodyLimit } from "hono/body-limit";
import { setCookie } from "hono/cookie";

const APP_ORIGIN = env.APP_ORIGIN;                     // https://app.example.com

app.use(secureHeaders({                               // HSTS, nosniff, Referrer-Policy, frame opts
  contentSecurityPolicy: { defaultSrc: ["'none'"], frameAncestors: ["'none'"] }, // JSON API
}));
app.use(cors({ origin: [APP_ORIGIN], credentials: true }));
app.use(csrf({ origin: APP_ORIGIN }));                // Origin / Sec-Fetch-Site on form-type bodies
app.use(bodyLimit({ maxSize: 1024 * 1024 }));         // 1 MB JSON; uploads don't come through here
app.on(["POST", "PUT", "PATCH", "DELETE"], "*", async (c, next) => {
  if (!c.req.header("content-type")?.startsWith("application/json"))
    return c.json({ error: { code: "validation_failed", message: "JSON only", request_id: c.get("requestId") } }, 415);
  await next();                                       // cross-site forms can't send JSON without a preflight
});

export const startSession = (c: Context, id: string) =>
  setCookie(c, "__Host-session", id, {
    httpOnly: true, secure: true, sameSite: "Lax", path: "/", maxAge: 60 * 60 * 12,
  });
```

- `SameSite=Lax`, not `Strict` — `Strict` drops the cookie on the redirect back from Zitadel and the user lands logged out.
- `__Host-` prefix = `Secure`, `Path=/`, no `Domain` — a sibling subdomain can't overwrite it.
- **Rotate the session id on login**, delete the record on logout, idle + absolute expiry server-side.
- **SPA shell CSP** (set where the HTML is served): `default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'; connect-src 'self' <api> <sentry ingest>`. Static shell → no inline scripts, no nonce needed.
- **SSR pages** (SolidStart, public): per-request nonce + `'strict-dynamic'` — Hono's `secureHeaders` takes `scriptSrc: [NONCE]` and exposes `c.get("secureHeadersNonce")`.

## 📤 Uploads — straight to R2, verify after

Upload bytes never stream through the API. R2 supports presigned `PUT` (not presigned `POST`), and a presigned PUT can pin `Content-Type` but **not size** (As of 2026-10, [R2 docs](https://developers.cloudflare.com/r2/api/s3/presigned-urls/)) — so verify after.

```ts
import { S3Client } from "bun";
const r2 = new S3Client({ endpoint: env.R2_ENDPOINT, bucket: "uploads",
  accessKeyId: env.R2_ACCESS_KEY, secretAccessKey: env.R2_SECRET_KEY });

// 1. POST /uploads → row { id, accountId, status: "pending", declaredType }
const key = `${scope.accountId}/${uuidv7()}`;            // server picks the key, never the client
const url = r2.file(key).presign({ method: "PUT", expiresIn: 300, type: "image/png" });

// 2. POST /uploads/:id/complete → verify, then mark ready
const file = r2.file(key);
const { size } = await file.stat();
const head = new Uint8Array(await file.slice(0, 4100).arrayBuffer());
const sniffed = await fileTypeFromBuffer(head);            // `file-type` package: magic bytes
if (size > MAX_BYTES || sniffed?.mime !== row.declaredType) {
  await file.delete(); throw new AppError("validation_failed", "Upload rejected");
}
```

- **Type = magic bytes**, checked server-side. Extension and browser `Content-Type` are attacker input.
- **Serve from a separate origin** (R2 public bucket on its own domain, or short-lived presigned `GET`) — never from the app origin, where an uploaded HTML/SVG runs as your app. Force `Content-Disposition: attachment` for anything that isn't an image you re-encode.
- **Images:** re-encode (strips EXIF + polyglots) before marking `ready`.
- A sweep deletes `pending` rows + objects older than the presign window → [data-and-scale.md](data-and-scale.md).

## 🚦 Rate limits (Dragonfly)

```ts
export const rateLimit = (name: string, limit: number, windowSec: number, key: (c: Context) => string) =>
  createMiddleware(async (c, next) => {
    const now = Math.floor(Date.now() / 1000);
    const k = `rl:${name}:${key(c)}:${Math.floor(now / windowSec)}`;
    const n = await redis.incr(k);
    if (n === 1) await redis.expire(k, windowSec);
    if (n > limit) {
      return c.json({ error: { code: "rate_limited", message: "Too many requests",
        request_id: c.get("requestId") } }, 429, { "Retry-After": String(windowSec - (now % windowSec)) });
    }
    await next();
  });
```

| Route class | Key | Starting limit |
|---|---|---|
| login / OIDC callback / OTP / password-ish | IP **and** account identifier, separately | 10 / 15 min |
| expensive (export, search, LLM-backed, upload presign) | user id | per cost — measure, then set |
| everything authenticated | user id | generous; catches runaway clients |

Client IP comes only from the header your ingress sets — never a raw `X-Forwarded-For` the client controls. Status + retry semantics → [api-contracts.md](api-contracts.md).

## 📜 Logs and errors

Redaction lives **in the logger config**, not at call sites — call sites will forget.

```ts
import pino from "pino";
export const log = pino({ redact: { censor: "[redacted]", paths: [
  "password", "*.password", "token", "*.token", "*.accessToken", "*.refreshToken", "*.secret",
  "req.headers.authorization", "req.headers.cookie", "res.headers['set-cookie']", "*.email", "*.phone",
] } });
```

- Log **ids** (`userId`, `accountId`, `request_id`), never the value an id stands for.
- Sentry: leave `sendDefaultPii` off; scrub request bodies in `beforeSend` → [../infrastructure/observability.md](../infrastructure/observability.md).
- A test asserts a login log line contains no password/token → the redaction list can't silently regress.

## 🌐 Frontend env + source maps

| Rule | Why |
|---|---|
| `VITE_*` = public, always | statically inlined into the bundle; minify/base64/no-maps don't hide it |
| `loadEnv(mode, root, '')` is fine **for config-time use** (port, proxy target) | it returns every env var, `process.env` included |
| …but never spread it into `define` | `define: { "process.env": env }` ships every server secret to the browser; one explicit key per public value |
| never set `envPrefix: ''` | Vite throws on it — don't work around it |
| `build.sourcemap: "hidden"` + `@sentry/vite-plugin` with `sourcemaps.filesToDeleteAfterUpload: ["./dist/**/*.map"]` | Sentry gets maps; the public origin doesn't |

## 🕳️ SSRF — server fetches of user URLs

Webhooks, link previews, "import from URL": resolve the host, reject loopback / private / link-local / metadata ranges (`127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `::1`, `fc00::/7`, `fe80::/10`), connect to the **resolved IP**, `redirect: "manual"` and re-check each hop, `https:` only, timeout + response size cap. Better still: run these fetches from a worker with no cluster network access.

**Related:** [api-contracts.md](api-contracts.md) · [testing.md](testing.md) · [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md) · [../stack/frontend-solidjs.md](../stack/frontend-solidjs.md) · [../stack/databases.md](../stack/databases.md) · [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md) · [../infrastructure/observability.md](../infrastructure/observability.md)
