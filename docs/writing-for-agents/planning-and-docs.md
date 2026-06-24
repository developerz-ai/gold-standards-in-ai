# 📐 Planning & Docs

Plan before coding, and write docs the way an agent reads them. This is what separates "AI helps me write code" from "AI builds my product."

## Plan before you code
`/plan` exists for a reason — planning before implementation is dramatically more effective with an agent.

```
/plan add Stripe payment processing

→ explores the codebase (grep, glob, read)
→ identifies existing patterns to follow
→ maps dependencies + affected files
→ writes .plans/stripe-payments/plan.md
```

A good plan is ~30 lines, all signal:
```markdown
# Stripe Payment Processing

## Files to create
- src/services/payment/stripe_processor.ts
- src/services/payment/webhook_handler.ts
- src/services/payment/errors.ts
- test/services/payment/stripe_processor.test.ts

## Files to modify
- src/routes/api.ts — add /payments endpoint
- src/config.ts — add STRIPE_API_KEY

## Pattern: follows src/services/invoice/generator.ts

## Steps
1. Error classes        → verify: typecheck
2. StripeProcessor      → verify: unit test passes
3. Webhook handler      → verify: signature test passes
4. Routes               → verify: integration test
5. Full suite green
```
Attach a `verify:` to each step. Verifiable criteria let the agent loop to completion without you adjudicating "done" — see [behavioral-rules.md](behavioral-rules.md).

## The initial idea → spec
For a brand-new project, capture the idea as a tight spec the agent can execute against. An `/initial-idea` command turns a paragraph into: problem, users, core flows, data model sketch, tech choices (default to the [stack](../architecture/tech-stack.md)), and a milestone list. Then `/plan` each milestone. See [../workflow/project-kickoff.md](../workflow/project-kickoff.md).

## Docs vs CLAUDE.md
| `CLAUDE.md` (execution) | `docs/` (context) |
|---|---|
| Commands the agent runs | Architecture decisions + WHY |
| Patterns to follow | API documentation |
| Tech stack versions | Onboarding guides |
| Testing strategy | Design docs, ADRs |

`CLAUDE.md` is read every conversation; `docs/` only when relevant. That split keeps `CLAUDE.md` lean.

```
docs/
├── architecture.md
├── api.md
├── onboarding.md
└── decisions/            # ADRs
    ├── 001-use-postgres.md
    └── 002-service-pattern.md
```

## Concise documentation — the golden rule
Every markdown file the agent reads costs tokens. Write like you pay per word.

**Bad** (prose): a paragraph explaining the auth service exists and follows SRP.
**Good** (signal):
```markdown
# User Auth
Login, register, password reset, sessions.
Pattern: `services/auth/authenticator.ts`, `services/auth/registrar.ts`
Tests: `test/services/auth/`
Deps: injected via constructor.
```
Same info, 80% fewer tokens. Rules: lead with the answer · file paths over descriptions · tables over paragraphs · no filler · `SKILL.md` < 80 lines · `CLAUDE.md` < 600.

## <a name="docs-decay"></a>Docs decay — write for the next session
- **Write for the next action, not posterity.** "Stripe webhook → `src/payments/stripe_webhook.ts`" earns its tokens; "comprehensive payments overview" is a liability.
- **Date load-bearing claims.** `As of 2026-06` beats `currently`.
- **Gap statements rot fastest.** "We haven't done X" is wrong the day someone does. Prefer "When you do X, follow `path/to/pattern.ts`".
- **Delete > update** when in doubt. A wrong doc costs more than a missing one.

## Prompts follow the same rules
| Weak | Strong |
|---|---|
| "Can you help with the auth flow?" | "Login returns 500 when email has `+`. Fix in `src/auth/login.ts`." |
| "Refactor the user model." | "User model is 1200 LOC. Split into `User`, `Profile`, `Preferences`. Tests in `test/models/user/`." |

Lead with the constraint, the file, or the goal. Drop throat-clearing.
