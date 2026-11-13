# Reviewer instructions — developerz-ai/gold-standards-in-ai

> Read by the developerz.ai reviewer at EVERY review of this repository, at the reviewed
> commit, as this repo's own layer of the reviewer's prompt — established by dz-runner setup mode.
> It refines the platform rules (a finding is cited or withheld; the bot always discloses)
> and never overrides them. Edit freely; keep it under 12,000 characters — past that it is cut.

## Stack

Markdown only (no runtime). The gate is `bin/check` — it runs `bin/check-docs` (index + relative-link + `#fragment` validation) and then `bin/test` (its regression fixtures), and is silent on success. It exits 0 when the change passes, 75 when a gate precondition is unmet and the gate could not grade, and any other nonzero when validation or regression fails. A change that does not pass it is not proven, whatever its description says.

Read the gate, never a copy of it. `bin/check-docs` is the single source of truth for the rules it enforces:

- **Indexing** — a doc under `docs/` is indexed only when the root `README.md` **and** its section `README.md` both carry it. A doc in only one of the two is a finding.
- **Links** — a relative link is resolved against the target file's headings, and the `#fragment` is the slug of the heading text: lowercase, every character that is not a letter, a digit, `-` or `_` deleted, spaces turned into `-`. Emoji, arrows and dashes are deleted, not turned into hyphens; a literal `-` and `_` are kept. So `## ⌨️ Keyboard paths` is `-keyboard-paths`, and `## 6. Merge → deploy without a human bottleneck` is `6-merge--deploy-without-a-human-bottleneck`.
- **Fences and inline code** are stripped first — a link inside one is an example, not a link.

Never reproduce the slug algorithm in prose or in a one-liner: a copy drifts from the checker, and a drifted copy makes a green tree look red. Run `bin/check-docs` and report what it prints.

## Review conventions

| Rule | A finding when |
|---|---|
| Lead with the rule; fragments over sentences; tables over paragraphs. | prose opens a section, or a list of 3+ rows stays prose |
| Paths, commands, env vars and model IDs verbatim. | a path or a command is paraphrased |
| One topic per file. | a file answers two questions |
| Emojis in headings and READMEs for navigation. | a heading loses its marker |
| Cross-link a sibling instead of restating it. | the same text is copied twice |
| Date load-bearing claims (`As of 2026-06`). | a dated claim carries no date |
| Vendor-neutral, reusable examples. | an example names a client |
| No secrets, private hostnames/IPs or client names. | a doc names a token, an internal host or a customer |
| Every `docs/` doc indexed in the root `README.md` **and** its section `README.md`. | `bin/check-docs` prints the unindexed path |
| Delete a stale doc rather than let it rot. | a doc contradicts the tree it describes |
| Never `--force`, never `git stash`, never a git worktree. | another writer's checkout moves under them |
| Merge only on a green gate. | a red `check` is merged around |

Full tables and the self-check list: [../../CLAUDE.md](../../CLAUDE.md#writing-rules) · [../../CLAUDE.md](../../CLAUDE.md#hard-rules) · long form [../../docs/writing-for-agents/compressed-config.md](../../docs/writing-for-agents/compressed-config.md).

## Gates that matter

| Gate | Decides |
|---|---|
| `bin/check` | the local gate — `bin/check-docs` (index + relative-link + `#fragment` validation) then `bin/test` (regression fixtures); silent on success |
| Its exit contract | `0` pass · `75` an unmet precondition, refused to grade (e.g. a missing `docs/` tree) · any other nonzero red |
| CI job `check` | `.github/workflows/ci.yml`, workflow `check` — `bin/setup` + `bin/check` on push and pull_request |
| `pr.required_checks: ["check"]` | `.dz/maintainer/maintainer.yml` — matches the raw check-run name, never the UI's `check / check` label |

Proven means all three hold: green local `bin/check`, green CI `check`, review approved (`require_review: true`). A red or missing `check` result never qualifies a PR for auto-merge.

## Never flag

| Never flag | Why |
|---|---|
| A missing app, build step or test suite | markdown only by choice; `bin/test`'s mktemp fixtures are the validator's harness, not code to build |
| The policy `.dz/maintainer/maintainer.yml` sets deliberately: `tone.voice: friendly`, triage windows (`close_unanswered_repro_after_days: 14`, `stale_after_days: 30`, `stale_action: comment_then_close`), `review.force: true`, `pr.auto_merge_scope: own_and_dependency_bots`, `release.manager: none` with empty channels | the owner's call, not a defect |
| `triage.non_goals` items, as bugs | triage scope — issues about them qualify into `docs`/`question`, not chased |
| Docs outside `docs/` left unindexed | the two-index rule grades only `./docs/*` |
| A link inside fenced code or inline code | stripped before validation — an example link is not a link |
| Indexing or link findings when `bin/check-docs` is green | the gate arbitrates them; green output, no finding |
