# 🪜 Abstractions & growth — pre-MVP to really big

The right amount of abstraction is a function of **stage**. Getting it wrong in either direction is expensive: a framework at pre-MVP is a product you didn't ship; hand-written mirrors at scale is a bug factory.

This doc is the ladder, plus the one thing that is different when **an AI writes ~100% of the code**.

## The one rule

> **Repetition of a *declaration* is free. Repetition of a *derived fact* is a bug farm.**

Copy-paste is fine — Rails proved it. `rails g scaffold` still writes eight files. What it never does is make you re-type the column list: `ActiveRecord` reflects the schema, so no layer can silently drop a field, because no layer re-declares one.

| Kind | Example | Verdict |
|---|---|---|
| **Declaration** — states a decision | `belongs_to :account`, a route's path, a policy | duplicate freely |
| **Derived fact** — restates something already known | a wire type mirroring a table, a serializer copying fields, a param list written beside its schema | **derive it or you will drift** |

A 16-line serializer that copies `record.x → dto.x` for every field is the second kind. It cannot be right; it can only be *currently not yet wrong*.

## ⚠️ Why AI-written codebases get this wrong

This is the part that isn't in older engineering books, and it's the reason this doc exists.

**1. The AI never grumbles.** In a human team, abstraction pressure comes from irritation. Someone writing the twelfth near-identical mirror gets annoyed and goes looking for a helper. **That irritation is the design signal.** An agent writes the thirtieth propagation site cheerfully and never once says "this is absurd." Copying stops costing anything — while the cost that *doesn't* fall is the one that matters: a missed site is still a silent production bug.

**2. No memory across sessions.** Abstractions emerge at the third or fourth instance — but only if one person writes all three and remembers the first two. Across hundreds of independent sessions, instance #3 looks exactly like instance #1. Measured in one production monorepo (2026-09): **96 independently hand-rolled `makeDeps` helpers, 62 `fakeDb`, 101 `makeService`** — ~250 builders, each written by a session that correctly concluded "there's no helper for this."

**3. Every local decision was right.** Hexagonal ports for testability, an HTTP layer separate from the domain, a lint guard when a bug recurs, "mirror the nearest existing service for consistency" — all defensible. The failure is **emergent**: six correct layer boundaries × one new field = six hand-written declarations. Nobody made a bad call; the calls composed badly.

### The consequence: the rule of three doesn't fire

You can't wait for someone to notice the third instance. Replace the trigger:

| Old trigger | AI-first trigger |
|---|---|
| "I've written this three times, time to abstract" | **"A new field/variant must be declared in more than one place"** → derive it, now |
| Someone complains it's tedious | **Nothing complains. Measure it instead** (see [the metric](#-measure-files-per-decision-not-loc)) |

The second column is a property of the *code*, not of anyone's patience — which is the only kind of trigger that works when the author has no patience to run out of.

## 🪜 The ladder

| Stage | Codebase | Abstraction posture | What you build |
|---|---|---|---|
| **Pre-MVP** | < ~10k LOC, 1 surface | **Duplicate freely.** Do not build a framework | Nothing. Inline it. A second copy is cheaper than a wrong abstraction |
| **MVP** | ~10–50k, 2 surfaces (API + web) | **Derive the entity shape only** | One source for each entity's fields; the wire type is generated from it. Nothing else |
| **Product-market fit** | ~50–300k, 3–4 surfaces | **Seams for the repeating shapes** | Route factory, scoped query builder, one deps builder for tests, error→status on the class |
| **Scale** | 300k+, 5+ surfaces | **Declare once, project everywhere** | A resource protocol; every surface is a projection. Adding a surface adds zero lines per entity |

**The stage-skip mistake, both directions:**
- Building the resource protocol at pre-MVP → you shipped a framework, not a product.
- Still hand-writing mirrors at 5 surfaces → measured cost: **two nullable columns = 34 files**; one enum field on a nested entity = **80 files** (40 source + 40 test).

**The trigger to move up a rung is a count, not a feeling:** when adding one field touches more than ~3 files, you are one rung below where you should be.

## 🎯 Declare once, project everywhere

The end state, and it's the same idea in every framework that survived at scale:

| Framework | The move |
|---|---|
| **Ash** (Elixir) | declare a resource — attributes, relationships, actions, policies — and `AshPostgres` / `AshJsonApi` / `AshGraphql` / `AshPhoenix.Form` / `AshAdmin` each **project** it |
| **`jsonapi-resources`** (Rails) | `attributes` + `creatable_fields` / `updatable_fields` / `fetchable_fields` + `records()` on one class |
| **Rails itself** | `Rails::Application < Rails::Engine` — **the core is a protocol, not a pile of shared code**; the framework's own parts are ordinary engines |
| **Django** | `INSTALLED_APPS`; `django.contrib.*` are just apps |
| **Phoenix** | `MyApp` (core) vs `MyAppWeb` (projection) |

Shape it as:

```
packages/kernel          the protocol. Imports NO entity, ever. Guarded.
packages/domain/<entity> attributes derived from the table, actions, policies
projections              REST · RPC/MCP · agent tools · client types · forms
apps/*                   thin hosts that mount projections
```

**A core is defined by what it refuses to know.** The common failure is a `shared/` package that accumulates entity knowledge until it's a grab-bag — measured in one repo: 143 files, 1,598 exports, and 24 files that knew about specific entities. Ship the no-entity-import guard **in the same PR as the core**, or you get a second grab-bag with a nicer name.

Two fields worth stealing verbatim from `jsonapi-resources`:

- **`creatable` / `updatable` / `fetchable` declared together.** A writable field with no readable counterpart becomes a *compile error*. That kills an entire real bug class — a capability you can write but never read back.
- **One `records()` tenancy hook.** Every projection reads through it; no projection carries its own tenant id. Otherwise the tenant predicate is hand-written per query (measured: **456 times**) and you end up policing it with regex.

### Derive at the type level, not at runtime

Ash and Rails buy generality with runtime metaprogramming and pay in debuggability. **When an AI writes the code, pay the other way.**

The agent has no memory of pain: a convention it must remember is one it will silently violate. **A compile error is the only feedback channel it reliably receives.** So: derived types + `satisfies Record<Kind, Handler>` exhaustiveness, and a missing field or arm fails the build. See [very low undefined behavior](../00-philosophy.md#-very-low-undefined-behavior).

### Ship the escape hatch, and say it isn't a dual path

Every framework that survives has a documented drop-to-raw for one case — Ash `manual` actions, Rails `find_by_sql`. Without it, the first awkward entity forks your framework.

State it explicitly against a [no-dual-paths doctrine](../workflow/shipping-doctrine.md): a **declared override**, named and typed at the declaration site, is current code using a documented seam. A second copy of the machinery is a dual path and is banned.

> If **more than ~3 entities** need the hatch, the abstraction is wrong. Stop and redesign rather than widen it.

## 🛡️ A guard is not an abstraction

The most expensive habit an AI-first codebase falls into:

> **Duplicate the fact, then write a lint guard to check the copies agree.**

Guards feel like progress — they're cheap to write, they catch a real bug once, and a PR that adds one looks productive. But **a guard is a tax paid on every commit, forever**, while a type that makes the bug unrepresentable is free after the day you write it.

Measured in one production monorepo (2026-09): **178 lint guards, ~118k LOC — larger than the entire HTTP layer they policed.** About 29% of them were retired outright by a type.

The finding that matters:

> In **12 of 13** cases the correct abstraction **already existed in the repo**. The guard wasn't covering for a missing abstraction — it was covering for the abstraction being **optional**.

**The guard is the tax paid for not deleting the bypass.**

Adoption numbers from that same repo, each a seam that was built and then left co-resident with the thing it replaced:

| Safe seam | Uses | Unsafe original | Uses |
|---|---|---|---|
| `asId()` checked brand | 102 | raw `as SomeId` cast | **606** |
| `looseUuid()` | 26 | `z.string().min(1)` on an id | **191** |
| shared `<Input>` | 16 | raw `<input>` | **114** |
| the sweep primitive | 2 | hand-rolled loop | **13** |

### The rule

| Invariant crosses… | Mechanism |
|---|---|
| something the type system can see | **change the type, and delete the bypass** |
| a language / file-format boundary (nginx, Dockerfile, Kotlin, SQL text) | a lint guard — legitimate, keep it |
| a budget over time (LOC caps, coverage floors, byte ceilings) | a ratchet — legitimate |
| English prose (copy rules, claim integrity) | a guard — legitimate |
| **an absence** ("a second thing should also exist") | a guard or a registry test — **no type can require that something else was written** |

Roughly 70% of guards are legitimately in the bottom four rows. The problem is never guards; it's guards standing in for a type.

### Delete the bypass in the same PR

This is the whole difference between an abstraction that lands and one that becomes shelfware. When you ship the seam:

1. Make the old path **not resolve** — don't re-export the raw primitive, narrow the injected deps so the unsafe name isn't in scope, brand the id so a bare string won't typecheck.
2. Delete the guard, its tests, and any `CLAUDE.md` bullet pointing at it.
3. Ship a **compile-fail fixture** proving the type catches what the guard caught — *before* deleting the guard.

Never remove a guard without step 3. A guard removed without its replacing type is a silently dropped invariant, which is strictly worse than the tax.

## 📊 Measure files-per-decision, not LOC

LOC is a vanity metric and will actively mislead you. Track the thing that costs time:

```
add a nullable column to <entity>     34 files  →  ceiling 4
add a field to <nested entity>        80 files  →  ceiling 5
add a UI modal / route variant        16 files  →  ceiling 2
add a background queue                 9 registries  →  ceiling 1
```

Commit it as a ledger with a `--check` mode in lint, baselined from **real past commits**, and let it ratchet **down only** — same mechanism as [coverage floors](testing.md) and context byte budgets.

Measure by *executing* the change on a scratch branch and counting files. A grep-based estimate is the exact failure this metric exists to catch.

**Why this metric and not size:** a full de-duplication pass on that monorepo projected **~8–10% fewer lines** but **~70–85% fewer files per cross-layer feature**. The first number wouldn't justify the work. The second is a permanent change to the cost of everything you haven't built yet.

Expect the ratio to be lopsided in tests, and don't fight it: **tests are structured per hand-written layer, so N layers ⇒ N test files.** Delete a layer and its tests go with it. That is why test code reached **1.68× source** with an **11.3% assertion density** — roughly 17 lines of setup per 2 lines of assertion. You don't fix that by writing fewer tests; you fix it by deleting layers and sharing one deps builder per bounded context.

## 🚫 What NOT to abstract

Restraint is half the skill. From the same audit, all of these were already correct and would have been damaged by "improvement":

- **A working seam.** One error→HTTP mapping used by 270 handlers with 6 deliberate local overrides. Correct. Leave it.
- **Middleware that already won.** Auth/scope resolved once for the whole tree; zero routes repeated it.
- **Chrome that's already shared.** One modal shell used by 59 components — only the *bodies* were bespoke.
- **Genuinely mechanical generated output.** 176 migrations, ~100% generated DDL. The gap was the TypeScript *around* them, not the SQL.
- **Anything with one call site.** Two is a coincidence. The trigger is a *derived fact in two places*, not a shape you find pleasing.

> **Speculative abstraction is worse than duplication**, because duplication is visible and a wrong abstraction is load-bearing. Defer behind a written trip condition, exactly as with [capacity](data-and-scale.md).

## ✅ Checklist

- [ ] Each entity's field list exists in **one** place; every other shape derives from it
- [ ] Adding a field touches **≤ 3 files**; a `feature-tax` ledger enforces it in CI
- [ ] The core package imports **no entity** — guarded
- [ ] Writable-without-readable is a **compile error**
- [ ] Tenancy has **one hook**; no projection carries its own tenant id
- [ ] Every new variant lands as a **table row**, and a missing row fails the build
- [ ] Every lint guard is either irreducible (crosses a boundary a type can't see) or **scheduled for deletion with the type that replaces it**
- [ ] When a seam ships, the **bypass stops resolving in the same PR**
- [ ] One deps builder per bounded context; assertion density is tracked and rising

---

Related: [shape now, capacity later](data-and-scale.md) · [SOLID / SRP](solid-srp.md) · [testing](testing.md) · [guards & gotchas](../writing-for-agents/guards-and-gotchas.md) · [shipping doctrine](../workflow/shipping-doctrine.md)

*Evidence dated 2026-09, from a ~2M-LOC production AI-first monorepo (~100% agent-written) audited by four read-only agents.*
