---
description: End-to-end doc workflow for gold-standards-in-ai — scope a topic, research it, write or revise the doc, wire it into the READMEs, review against the house rules, PR, merge. Markdown only; no code, no tests.
argument-hint: <the doc or topic you want written/revised> [+ reference URL(s)]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, WebFetch
---

# /feature

Writer on **Gold Standards in AI** — the knowledge base agents read before starting a project. Markdown only. No code to build, no tests to run. Read [`CLAUDE.md`](../../CLAUDE.md) and `docs/00-philosophy.md` first.

## Request
$ARGUMENTS

Infer scope from the prompt. "Just ship it" → write, wire, merge. A tentative ask → confirm the topic boundary before writing. Stop for a real blocker: a claim you can't source, a topic that duplicates an existing doc, anything that would leak secrets/hostnames/client names.

## No worktrees

**Never use `isolation: worktree`, `git worktree add`, or per-agent worktree directories.** Parallel agents share this one checkout. Coordinate instead:

| Rule | Why |
|---|---|
| One branch per feature; all agents commit into it | No cross-tree merges |
| One doc per agent, disjoint paths | Two agents never edit one file |
| Announce owned paths before writing | Prevents silent clobbering |
| Re-read a file before editing it | A sibling may have touched it |
| README index edits land **last**, by one agent | `README.md` + section `README.md` are shared choke points |
| Never `git checkout` / `git stash` under a sibling | Moves the tree out from under them |

## Flow

1. **Scope.** One sentence: what an agent will be able to do after reading this. One topic per file. Duplicate of an existing doc → revise that doc instead of adding one.

2. **Locate.** `Glob docs/**/*.md`, read the section `README.md`. Decide: new `docs/<section>/<topic>.md`, or an edit to an existing file. New section → it needs its own `README.md`.

3. **Research.** Cited URLs → `WebFetch`. Fan out `Task` Explore agents for a multi-doc sweep. Keep examples vendor-neutral and reusable. Prefer the latest Claude model IDs in examples.

4. **Write.** House rules, non-negotiable:

   - Lead with the rule, not the reason.
   - Fragments over sentences. Tables over paragraphs.
   - File paths, commands, env vars, model IDs verbatim — compress prose only.
   - Dense and copy-paste oriented. Scannable in one pass.
   - Emojis in headings/READMEs for navigation; code blocks stay clean.
   - Cross-link related docs with relative links.
   - Date load-bearing claims (`As of 2026-07`).
   - No meta-framing, no rhetoric, no trailing summary paragraph.

5. **Wire it in.** Add a link + one-line hook to `README.md` **and** the section `README.md`. A doc not linked from the map does not exist.

6. **Review before PR.** Check the doc against: house rules above · every relative link resolves · no secrets, private hostnames/IPs, or client names · no stale claim left undated · no duplication of a sibling doc. Stale doc found along the way → delete it rather than letting it rot.

7. **PR + merge.** Conventional Commit (`docs: …`), branch from `main`, `gh pr create` with Summary + the doc-map rows changed. Merge one at a time.

## Output

```
Docs:   <files added/edited>
Map:    README.md + <section>/README.md updated  <yes/no>
Review: house rules <ok>  links <ok>  dates <ok>
```
