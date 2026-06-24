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
