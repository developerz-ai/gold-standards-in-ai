# 🧱 SOLID / SRP

Clean code an agent can navigate. Small focused files, predictable locations, automated enforcement. **SRP is the one that matters most.**

## Single Responsibility — one module, one job
**Don't** pile verbs into one class:
```ts
class UserManager {
  create() {} update() {} sendNotification() {} generateReport() {}
}
```
**Do** split by responsibility:
```
src/services/user/
├── creator.ts       # ~40 LOC
├── updater.ts       # ~35 LOC
├── notifier.ts      # ~50 LOC
└── errors.ts        # custom error types
```
Smaller files → one-shot reads, isolated changes, focused tests. Every class has *one reason to change*.

## Small files — ~500 LOC max
- A 200-line file = one read, full context.
- A 2000-line file = chunked reads, lost context, mistakes.
- Split when: multiple private methods doing distinct work, complex validation, large test files.

## Thin layers, fat services
| Layer | Does only |
|---|---|
| Controllers / handlers | HTTP: parse → call service → render |
| Models / entities | persistence: validations, associations, queries |
| Jobs / workers | thin wrapper: call a service |
| **Services** | **all business logic** |

```
src/services/
├── payment/{processor,refunder,errors}.ts
├── invoice/{generator,renderer,sender}.ts
└── user/{creator,authenticator,notifier}.ts
```
Domain namespacing (`Payment::Processor`, `User::Creator`) means a grep finds exactly the right file.

## Custom error classes — never generic
```ts
export class PaymentError extends Error {}
export class InsufficientFundsError extends PaymentError {}
export class GatewayTimeoutError extends PaymentError {}
```
`catch (e instanceof InsufficientFundsError)` is self-documenting. `catch (Error)` tells the agent nothing.

## Type-safe everything
- **TypeScript strict** — no `any` (Biome enforces).
- **Zod** for all external data (API requests, env vars, webhook payloads):
```ts
const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(["admin", "user"]),
});
type User = z.infer<typeof userSchema>;
```

## Reusable helpers, not copy-paste
When the same logic shows up a second time, **lift it into a shared helper** — a function, class, or module — instead of duplicating it. DRY keeps behavior in one place: fix a bug once, change it once.

Where shared code lives:
| Scope | Home |
|---|---|
| Across apps in a product | a `packages/*` package (`@product/core`, `@product/ui`, `@product/auth`) → [monorepo](monorepo.md) |
| Across scripts | `scripts/lib/**/*.ts` → [DX scripts](../developer-experience/dx-scripts.md) |
| Within one app | `src/lib/` / `src/utils/` |
| Across web + mobile | a shared package (i18n, models, API client) → [mobile](../stack/mobile.md) |

Rules:
- **Helpers stay SRP** — small, pure where possible, one job, well-named, unit-tested. A "utils" grab-bag of 50 unrelated functions is an anti-pattern; split by concern (`lib/date.ts`, `lib/money.ts`, `lib/http.ts`).
- **Reusable classes over duplicated state machines** — a base `ApiClient`, a `Result<T>` type, a typed error hierarchy ([above](#custom-error-classes-never-generic)) — define once, extend.
- **Export an explicit public API** from each package; don't reach into another package's internals.
- **But don't over-abstract** — see the next section. Extract on the *second* real use, not in anticipation.

This is also a force multiplier for agents: shared helpers mean fewer lines to read, one place to change, and consistent behavior the agent can rely on.

## No premature abstraction
Concrete first. Introduce an interface/trait when the **second implementation** actually arrives — not before. See the simplicity rule in [../writing-for-agents/behavioral-rules.md](../writing-for-agents/behavioral-rules.md).

## Why this compounds for agents
| Practice | Agent benefit |
|---|---|
| SRP / small files | one-shot read, modify, test |
| Service objects | predictable location for logic |
| Strict patterns | follows, doesn't reinvent |
| Automated lint | zero formatting decisions |
| Custom errors | self-documenting error flows |
| Documented commands | exact CLI, no guessing |

Enforce it mechanically: a lint gate + a [pre-commit hook](../writing-for-agents/hooks-and-permissions.md), so the agent literally can't commit code that violates the baseline.
