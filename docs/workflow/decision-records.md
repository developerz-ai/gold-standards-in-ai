# 🧭 Decision Records

**One short record per one-way door, in the repo, with the rejected options written down.** An agent has no memory of last month's debate — without the record it re-proposes the option you already killed, confidently, every session.

## 📁 Where they live
```
docs/decisions/
├── README.md                  # the index — one row per record
├── 001-postgres-primary.md    # status: superseded by 004
├── 002-uuidv7-app-side.md
├── 003-keyset-pagination.md
└── 004-yugabyte-primary.md    # supersedes: 001
```
- `NNN-<slug>.md`, 3-digit, monotonically increasing, never renumbered. Slug kebab-case, ≤ 5 words.
- Point at the folder from `CLAUDE.md` "where to look" → [../writing-for-agents/claude-md.md](../writing-for-agents/claude-md.md). Folder convention: [../writing-for-agents/planning-and-docs.md](../writing-for-agents/planning-and-docs.md).

## ✍️ When to write one
| Write a record | Don't |
|---|---|
| A **one-way door** → [shipping-doctrine.md](shipping-doctrine.md): stored data shape, id scheme, public contract, promise to users | Naming, formatting, file layout — the linter/`CLAUDE.md` owns those |
| Picking a datastore, framework, auth model, tenancy model, queue | Anything cheap to reverse next week |
| Deviating from the standard stack → [../architecture/tech-stack.md](../architecture/tech-stack.md) | Restating the stack default (no deviation = no record) |
| Deleting a module/approach on purpose (pairs with the "deleted on purpose" list) | Bug fixes, refactors with no choice between options |
| Rejecting an option someone (or an agent) keeps proposing | |

## 🤖 Agent rules
| Rule | Shape |
|---|---|
| **Write it, don't ask** | Agent makes a one-way-door call → writes the record in the same PR as the change. No draft-for-approval step; PR review is the review. |
| **Read before reopening** | Before proposing to change a datastore/pattern/contract → scan `docs/decisions/README.md`, read the matching record. Re-proposing a rejected option requires naming what changed since its "Why not". |
| **Alternatives are mandatory** | ≥ 2 real alternatives, each with a one-line **why not**. "We just picked it" is not a record. The "why not" lines are what stop the next agent re-litigating. |
| **Supersede, never edit history** | Decision changes → new record; old one gets `Status: superseded by NNN`, new one gets `Supersedes: NNN`. Link both ways. Old body untouched. |
| **Index in the same PR** | New/changed status → update the `README.md` row in the same commit. |
| **Backfill honestly** | Recording an old choice → `Date:` = original date if known, add `Backfilled: YYYY-MM-DD`. |
| **2-minute read** | Context ≤ ~10 lines. Longer → it's a design doc; link it. |

## 🔄 Status lifecycle
```
proposed → accepted → superseded by NNN
                    ↘ deprecated        (the thing it governed is gone)
```
- `proposed` — open in a PR. Merged = `accepted`; agents set it in the merging PR.
- Only `accepted` records bind. Agents follow them without re-deriving them.

## 📄 Template — `docs/decisions/NNN-<slug>.md`
```markdown
# NNN — <Decision in present tense: "Primary store is Postgres">

Status: accepted            <!-- proposed | accepted | deprecated | superseded by NNN -->
Date: YYYY-MM-DD
Supersedes: —               <!-- NNN, or — -->

## Context
2–10 lines. The forces: load, data shape, team, deadline, constraint that made this a choice.

## Decision
1–3 sentences. Specific: "Postgres 17 via Drizzle", not "a relational DB".

## Alternatives considered
- **<Option A>** — why not: <one specific reason it lost>.
- **<Option B>** — why not: <one specific reason it lost>.

## Consequences
- Easier: <what this unlocks>.
- Harder: <the cost we accept>.
- Revisit when: <observable trigger, e.g. "single-region write p95 > 50ms" or "multi-region writes required">.
```

## 🗂️ Index — `docs/decisions/README.md`
```markdown
# Decisions

| # | Decision | Status | Date |
|---|---|---|---|
| [001](001-postgres-primary.md) | Primary store is Postgres | superseded by [004](004-yugabyte-primary.md) | 2026-01-15 |
| [002](002-uuidv7-app-side.md) | IDs are UUIDv7 generated app-side | accepted | 2026-01-15 |
| [003](003-keyset-pagination.md) | All list endpoints use keyset pagination | accepted | 2026-02-03 |
| [004](004-yugabyte-primary.md) | Primary store is YugabyteDB (Postgres wire) | accepted | 2026-09-10 |
```

## 🔗 Related
- One owning doc per fact; "why" lives here, "where" in the map → [../writing-for-agents/planning-and-docs.md](../writing-for-agents/planning-and-docs.md)
- Spec "open questions" that get answered become records → [project-kickoff.md](project-kickoff.md)
- Which doors are one-way → [shipping-doctrine.md](shipping-doctrine.md)
