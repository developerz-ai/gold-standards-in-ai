# 🌱 Project Kickoff

Zero → running project, fast. Idea → spec → scaffold → a multi-file plan an agent can execute. The faster the gap from "paragraph in a chat" to "agent opening PRs," the more you ship.

## Idea → spec
The first artifact is **not code** — it's a tight spec an agent can execute without you in the room. Capture:

| Section | What goes here |
|---|---|
| Problem | One paragraph. What hurts, for whom. |
| Users | Who uses it + their one core job. |
| Core flows | 3–5 bullet flows, happy path only. |
| Data model sketch | Entities + key fields + relations (not full DDL). |
| Tech choices | Default to the standard stack → [../architecture/tech-stack.md](../architecture/tech-stack.md). Only note **deviations**. |
| Out of scope | What you're explicitly *not* building yet. |

Don't hand-write it. An **`/initial-idea`** slash command turns one paragraph into this spec **plus a milestone list** (the epics / releases). Those milestones become GitHub milestones in [github-issues-milestones.md](github-issues-milestones.md).

```
/initial-idea A tool where teams submit expense receipts, an approver
              signs off, and finance exports a monthly report.

→ writes docs/spec.md (problem, users, flows, data sketch, stack deltas)
→ proposes milestones: [auth, receipts, approvals, export, billing]
```

## Scaffold
Same shape every time so the agent learns one repo and knows them all:

| Piece | What | Reference |
|---|---|---|
| `apps/*` + `packages/*` | Monorepo: user-facing apps, shared domain/infra packages | [../architecture/monorepo.md](../architecture/monorepo.md) |
| `bin/setup` `bin/dev` `bin/check` | Setup, run, gate — the core trio | [../developer-experience/dx-scripts.md](../developer-experience/dx-scripts.md) |
| `CLAUDE.md` | The repo's contract for agents | [../writing-for-agents/claude-md.md](../writing-for-agents/claude-md.md) |
| `.claude/` | Skills, commands, hooks | [../writing-for-agents/skills-commands-agents.md](../writing-for-agents/skills-commands-agents.md) |
| `biome.json` + CI | Format/lint + the green-gate pipeline | [../developer-experience/linting-ci.md](../developer-experience/linting-ci.md) |

After scaffold, `bin/setup && bin/dev` should boot the stack with zero manual steps.

## The `planx` pattern — multi-file execution plans

> The single most leveraged habit in an AI-first repo. A plan written by **one** agent/human is implemented by **another** with no extra context. Plan only — **no implementation**.

Ship it as a **`/planx`** slash command.

### Why multi-file
- **Decoupled author/executor** — the plan is the handoff. Author and executor never share memory, only the files on disk → [../ai-agents/orchestration.md](../ai-agents/orchestration.md).
- **Parallelism** — one slice = one unit of work. A fleet of Workers takes a slice each → [../writing-for-agents/planning-and-docs.md](../writing-for-agents/planning-and-docs.md).
- **Coordination** — a machine-readable `status.yml` lets the fleet claim, track, and report without stepping on each other.

### Path
```
docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/
```
- Date-based dir, e.g. `docs/plans/2026/06/24/`.
- `1NN` auto-increments per day, starting **101** (`101-`, `102-`, …).
- `slug` is kebab-case, **max 5 words**.

Example: `docs/plans/2026/06/24/101-receipt-approval-flow/`.

### Procedure
1. **Explore first (read-only).** Use a read-only subagent or codegraph to find **files-to-touch (`file:line`)**, existing tests, and shared contracts/types. Don't guess — cite real locations.
2. **Write multiple files, never one `plan.md`:**
   - `overview.md` — the index.
   - One `<NN>-<aspect>.md` per separable area, each independently executable and short.
   - `status.yml` — the only tracker.

```
docs/plans/2026/06/24/101-receipt-approval-flow/
├── overview.md
├── 01-data-model.md
├── 02-backend-service.md
├── 03-api-routes.md
├── 04-frontend.md
├── 05-tests.md
└── status.yml
```

### Rules
- **Compact English.** Fragments, not essays.
- **Reference, don't paste.** Point at code with `path:line` and `Class#method` — never copy source into the plan.
- **Self-contained per slice.** An executor reads `overview.md` + its one slice + the cited files. That's enough.
- **Respect `CLAUDE.md`** — the plan assumes the repo's conventions, doesn't restate them.
- **No checkboxes in `.md` slices.** Slices are reference maps; `status.yml` is the **only** tracker.

### `overview.md` template
```markdown
# <Plan title>

## Goal
What we're building and **why** (1–3 sentences).

## Context
- Stack facts: Bun + Hono API, SolidJS web, Drizzle/Postgres.
- Reference patterns:
  - Service shape → `packages/billing/src/invoice.service.ts:14`
  - Route registration → `apps/api/src/routes/index.ts:30`
  - Existing migration → `packages/db/migrations/0007_receipts.sql`

## Plan files (execute in order)
1. [01-data-model.md](01-data-model.md) — schema + migration
2. [02-backend-service.md](02-backend-service.md) — approval domain logic
3. [03-api-routes.md](03-api-routes.md) — HTTP endpoints
4. [04-frontend.md](04-frontend.md) — approver UI
5. [05-tests.md](05-tests.md) — unit + integration

## Done when
- [ ] Approver can approve/reject a receipt; status persists.
- [ ] `GET /receipts?status=pending` returns only pending.
- [ ] `bin/check` and `bun run test:integration` green.

## Risks / open questions
- Multi-approver later? Out of scope — model single approver now.
- Notification on approval — deferred.
```

### `<NN>-<aspect>.md` template
```markdown
# 02 — Backend service

> Part of [overview.md](overview.md). Depends on: 01

## Files to change
- `packages/receipts/src/approval.service.ts` (new) — approve/reject domain logic.
- `packages/receipts/src/receipt.repo.ts:42` — add `updateStatus()`.
- `packages/receipts/src/errors.ts:8` — add `ReceiptAlreadyDecided`.

## Steps
1. Mirror `InvoiceService` shape from `invoice.service.ts:14` (constructor-injected repo).
2. `ApprovalService#approve(id, approverId)` → guard non-pending via the new custom error.
3. Persist through `ReceiptRepo#updateStatus`; return updated entity.

## Tests
- Unit: `approve` on pending → approved; on approved → throws. `bun test packages/receipts`
- Integration: covered in [05-tests.md](05-tests.md).

## Done when
- [ ] `ApprovalService` approve/reject implemented + unit-tested.
- [ ] Re-deciding a decided receipt throws `ReceiptAlreadyDecided`.
```

### `status.yml` — the single tracker
Machine-readable so a fleet of agents can coordinate. `worked_by` empty = **unclaimed**; an executor sets it to **their own** `git config user.name` before starting a slice.

```yaml
plan: 101-receipt-approval-flow
title: Receipt approval flow
status: in_progress        # not_started | in_progress | blocked | complete | superseded
created_by: planner-agent
worked_by: ""              # empty = unclaimed; executor sets to their git user.name
owner: admin
percent: 40
current_focus: 03-api-routes
slices:
  - file: 01-data-model.md
    status: complete
    percent: 100
  - file: 02-backend-service.md
    status: complete
    percent: 100
  - file: 03-api-routes.md
    status: in_progress
    percent: 30
  - file: 04-frontend.md
    status: not_started
    percent: 0
  - file: 05-tests.md
    status: not_started
    percent: 0
evidence:
  - "commit a1b2c3d — add receipts schema + migration"
  - "PR #142 — approval service"
notes: >
  Blocked nothing. API slice in progress; frontend waits on routes.
last_updated: 2026-06-24T14:05:00Z
```

## Next
Turn the plan into work: each slice → an issue, the plan → a milestone → [github-issues-milestones.md](github-issues-milestones.md).
