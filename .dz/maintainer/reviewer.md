# Reviewer instructions — developerz-ai/gold-standards-in-ai

> Read by the developerz.ai reviewer at EVERY review of this repository, at the reviewed
> commit, as this repo's own layer of the reviewer's prompt. It refines the platform rules
> (a finding is cited or withheld) and never overrides them. `CLAUDE.md` is the source of
> truth; this file is its review-time distillation. Keep it under 12,000 characters — past
> that it is cut.

## Stack

Markdown only (no runtime, no dependencies, no build). Scripts are bash under `bin/`. The gate is `bin/check` — it runs `bin/check-docs` (index + relative-link + `#fragment` validation), then `bin/check-public-tree` (this tree only), then `bin/test` (the validator's regression fixtures), and is silent on success. It exits 0 when the change passes, 75 when a gate precondition is unmet and the gate could not grade, and any other nonzero when validation or regression fails. A change that does not pass it is not proven, whatever its description says.

Read the gate, never a copy of it. `bin/check-docs` is the single source of truth for the rules it enforces:

- **Indexing** — a doc under `docs/` is indexed only when the root `README.md` **and** its section `README.md` both carry it. A doc in only one of the two is a finding.
- **Links** — a relative link is resolved against the target file's headings, and the `#fragment` is the slug of the heading text: lowercase, every character that is not a letter, a digit, `-` or `_` deleted, spaces turned into `-`. Emoji, arrows and dashes are deleted, not turned into hyphens; a literal `-` and `_` are kept. So `## ⌨️ Keyboard paths` is `-keyboard-paths`, and `## 6. Merge → deploy without a human bottleneck` is `6-merge--deploy-without-a-human-bottleneck`.
- **Fences and inline code** are stripped first — a link inside one is an example, not a link.

Never reproduce the slug algorithm in prose or in a one-liner: a copy drifts from the checker, and a drifted copy makes a green tree look red. Run `bin/check-docs` and report what it prints.

## What to reject

**Public tree (`CLAUDE.md` § Hard rules, `bin/check-public-tree`)** — this repository is public; every tracked file is a public document.
- A secret, a private hostname or IP, a client or customer name, a home-directory path, or a link to a private repository or its tracker. `bin/check-public-tree` refuses most of these; a finding names the file and the rule, never quotes the matched text.
- A committed `.env` (only `.env.example`, names only, is tracked; see `.gitignore`).
- A vendor-specific example where a neutral one works (`CLAUDE.md` § Writing rules; policy `non_goals` in `.dz/maintainer/maintainer.yml`).

**Structure (`CLAUDE.md` § Structure)**
- A new doc that is not `docs/<section>/<topic>.md` under a section directory, or a new section without its own `docs/<section>/README.md` and a row in `docs/README.md`.
- A doc not linked from both the root `README.md` and its section `README.md`.
- A section index row that spells its own section path: a section row uses the bare filename, relative to the section directory, and carries a link + a one-line hook.
- Two topics in one file; the same text copied into two docs instead of a relative cross-link.
- Code to build or tests to run added outside `bin/` — the repo is Markdown only (`maintainer.yml` `non_goals`).

**Writing rules (`CLAUDE.md` § Writing rules; long form `docs/writing-for-agents/compressed-config.md`)**
- A section that opens with the reason or a tour instead of the rule; meta-framing ("This section covers…"), rhetoric, or a trailing summary paragraph.
- A list of three or more rows written as prose instead of a table.
- A file path, command, env var or model ID paraphrased instead of kept verbatim.
- A load-bearing claim without a date (`As of 2026-06`) — `.claude/agents/dated-claims.md`.
- A model ID that is not current in the Claude lineup when the doc shows one.

**Stale docs**
- A doc that contradicts the tree or the world it describes, kept and patched around instead of corrected or deleted with its index rows in the same PR (`CLAUDE.md` § Hard rules).

**Gate and validator (`bin/`)**
- A change to `bin/check-docs` without a matching regression fixture exercised by `bin/test`.
- A change that collapses exit 75 (refused to grade) into another code, or makes a `bin/check-public-tree` failure print the matched text (CI logs are public).
- A private word written into `bin/check-public-tree`, whole or in parts: the words come only from the `PUBLIC_TREE_PRIVATE_WORDS` CI secret.
- A CI change that stops running the same `bin/check` the pre-commit hook (`.githooks/pre-commit`) runs, or renames the `check` job without updating `required_checks` in `.dz/maintainer/maintainer.yml`.

**PR shape (`CLAUDE.md` § Git rules)**
- More than ~25–30 changed files in one PR.
- Scratch notes, draft plans or agent summaries shipped in the diff.

## Gates that matter

- The `check` job in `.github/workflows/ci.yml` (workflow `check`), which runs `bin/setup` then `bin/check` on push and pull_request. It is the one required check (`required_checks: ["check"]` in `.dz/maintainer/maintainer.yml`); a red or missing `check` is not proven.
- On this repository's own (non-fork, non-Dependabot) runs the public-tree guard requires the `PUBLIC_TREE_PRIVATE_WORDS` secret; a fork or Dependabot PR skipping the private-word pass is expected, not a finding.
- `bin/test` is the proof for a validator change; `bin/check-docs` output is the proof for a links or index claim — cite what it prints, not a hand-computed slug.

## Never flag

- Emojis in headings and READMEs: a deliberate navigation convention (`CLAUDE.md` § Writing rules). Code blocks stay clean; that is the only limit.
- Fragments over sentences, tables over paragraphs: the house style on purpose.
- `bin/dev` and `bin/stop` being no-ops: there is no app to run.
- The `.dz/` directory's own layout (`.dz/onboarding/`, `.dz/briefs/`): the platform owns it.
- The identity preamble repeated across `.claude/agents/*.md`: the same text in every specialist file is intended.
- `.codegraph/` and `.mcp.json`: local tooling config.
