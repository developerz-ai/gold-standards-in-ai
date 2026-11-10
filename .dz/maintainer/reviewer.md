# Reviewer instructions — developerz-ai/gold-standards-in-ai

> Read by the developerz.ai reviewer at EVERY review of this repository, at the reviewed
> commit, as this repo's own layer of the reviewer's prompt — established by dz-runner setup mode.
> It refines the platform rules (a finding is cited or withheld; the bot always discloses)
> and never overrides them. Edit freely; keep it under 12,000 characters — past that it is cut.

## Stack

Markdown only (no runtime). The gate is `bin/check` — it also grades the tracked mode of the `bin/` scripts (a script tracked without the executable bit is a finding naming the path) before `bin/check-docs` (index + relative-link + `#fragment` validation) and `bin/test` (its regression fixtures), and is silent on success. It exits 0 when the change passes, 75 when a gate precondition is unmet and the gate could not grade, and any other nonzero when validation, a mode finding or regression fails. A change that does not pass it is not proven, whatever its description says.

Read the gate, never a copy of it. `bin/check-docs` is the single source of truth for the rules it enforces:

- **Indexing** — a doc under `docs/` is indexed only when the root `README.md` **and** its section `README.md` both carry it. A doc in only one of the two is a finding.
- **Links** — a relative link is resolved against the target file's headings, and the `#fragment` is the slug of the heading text: lowercase, every character that is not a letter, a digit, `-` or `_` deleted, spaces turned into `-`. Emoji, arrows and dashes are deleted, not turned into hyphens; a literal `-` and `_` are kept. So `## ⌨️ Keyboard paths` is `-keyboard-paths`, and `## 6. Merge → deploy without a human bottleneck` is `6-merge--deploy-without-a-human-bottleneck`.
- **Fences and inline code** are stripped first — a link inside one is an example, not a link.

Never reproduce the slug algorithm in prose or in a one-liner: a copy drifts from the checker, and a drifted copy makes a green tree look red. Run `bin/check-docs` and report what it prints.

## Review conventions

- TODO(setup): the conventions a reviewer must hold this repo to (from CLAUDE.md / CONTRIBUTING), each as a rule.

## Gates that matter

- TODO(setup): which checks decide a change here, and what "proven" means for a pull request.

## Never flag

- TODO(setup): what never to flag — generated or vendored files, style a formatter owns, conventions this repo chose on purpose.
