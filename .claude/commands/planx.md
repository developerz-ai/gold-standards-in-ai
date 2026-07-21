---
description: Write a concise, self-contained doc plan to docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/ for another AI to execute — which docs to write, in what order, wired into which READMEs
argument-hint: [the doc set or revision you want planned]
allowed-tools: Write, Read, Glob, Grep, Task, Bash
---

# /planx

Plan only. No doc writing outside the plan dir. The executor should need nothing but the plan and the files it cites.

## Goal
$ARGUMENTS

## Steps

1. **Resolve path.** `date +%Y`, `date +%m`, `date +%d`. `Glob docs/plans/<YYYY>/<MM>/<DD>/1*` → next number = highest `1NN-*` + 1, else `101`. Slug = kebab-case, max 5 words. Plan dir: `docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/`.

2. **Explore.** `Task` (subagent_type=Explore, thoroughness="very thorough"): which `docs/<section>/` owns the topic · what existing docs already cover part of it (`file:line`) · whether this is a new file, a revision, or a delete · which READMEs need a new row. **No worktrees** — read this checkout in place.

3. **Write the plan as multiple files.** Always `overview.md` + one `<NN>-<aspect>.md` per doc or doc cluster (e.g. `01-<topic>.md`, `02-<topic>.md`, `03-readme-wiring.md`). Never one big `plan.md`.

   **`overview.md`**:

```markdown
# <Title>

## Goal
1-2 sentences: what an agent can do after reading the finished docs.

## Context
- Section: `docs/<section>/` — its README index.
- Existing docs that overlap: `docs/<section>/<topic>.md` — revise vs. add.
- House style: lead with the rule, fragments over sentences, tables over paragraphs.

## Plan files (execute in order)
1. [`01-<aspect>.md`](01-<aspect>.md) — one line: what it covers.

## Done when
- Every doc written, linked from `README.md` and the section README, links resolve, claims dated.

## Risks / open questions
```

   **Each `<NN>-<aspect>.md`**: `## Files to change` (`path` — new or revised, why) · `## Outline` (the headings the doc should carry, in order) · `## Must include` (rules, tables, verbatim commands/paths/model IDs the doc has to carry) · `## Review` (house rules · links resolve · no secrets/hostnames/client names · load-bearing claims dated) · `## Done when`.

4. **Write `status.yml`** in the plan dir:

```yaml
plan: <1NN>-<slug>
title: <human title from overview.md>
status: not_started        # not_started | in_progress | blocked | complete | superseded
created_by: <git config user.name>
worked_by: ""              # executor fills with their git user.name
owner: <git config user.name>
percent: 0
current_focus: ""
slices:
  - file: 01-<aspect>.md
    status: not_started
    percent: 0
evidence: []
notes: ""
last_updated: <YYYY-MM-DD>
```

## Rules

- The plan dogfoods the house style: lead with the rule, fragments over sentences, tables over paragraphs, no meta-framing.
- Reference-only: point at docs, don't restate their content.
- No checkboxes. Plain bullets. `status.yml` is the only tracker.
- Multiple files always: `overview.md` + `<NN>-<aspect>.md` slices.
- **Execution happens in this checkout — never plan a `git worktree` step or an `isolation: worktree` agent.** Parallel slices = one doc per agent, disjoint paths; README index rows are edited last, by one agent.
- One topic per doc. Duplicate of an existing doc → plan a revision, not an addition. Stale doc → plan its deletion.
- Every new doc gets a row in `README.md` and its section `README.md`. Examples vendor-neutral; latest Claude model IDs. No secrets, private hostnames/IPs, or client names.

## Output
```
✓ docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/overview.md
  + 01-<aspect>.md, 02-<aspect>.md, …
  + status.yml
Next: run an executor on overview.md.
```
