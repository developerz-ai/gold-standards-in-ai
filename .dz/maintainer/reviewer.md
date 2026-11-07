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
- Hold every doc to the writing rules in `CLAUDE.md`: lead with the rule, fragments over sentences, tables over paragraphs.
- Reject a file that merges several topics — one topic per file.
- Require exact paths, commands, env vars and model IDs to stay verbatim; only prose is compressed.
- Reject any secret, private hostname/IP or client name; placeholders are allowed.
- Require a date on every load-bearing claim (`As of 2026-06`); prefer deleting a stale doc over patching it.
- Require the doc to be linked from `README.md` **and** its section `README.md` — an unwired doc does not exist.
- Keep examples vendor-neutral; require the latest Claude model IDs when a model is named.
- Reject meta-framing, rhetoric and a trailing summary paragraph.
- Cite `file:line` for every finding and drop the finding if you cannot — the precision gate in `docs/writing-for-agents/reviewer-agents.md`.
- Treat zero findings as a valid review; never hold back approval to look rigorous.

## Gates that matter
- CI runs the same gate — the job `check` in `.github/workflows/ci.yml` runs `bin/setup` + `bin/check` on push and pull_request; a green local run cannot land red there.
- Prose, link integrity and the map staying true are the reviewer's own gate (`.claude/commands/feature.md`).
- "Proven" means both final checks pass — every relative link resolves, every doc is indexed in both READMEs. `bash bin/check-docs` prints one line per finding; empty output is the proof.
- The link sweep strips fenced code blocks and inline code before it extracts links — every printed hit is a real markdown link; never dismiss one as fenced-code noise.
- `maintainer.yml` sets `required_checks: ["check / check"]` — workflow `check` + job `check` in `.github/workflows/ci.yml` — and `auto_merge_scope: own_and_dependency_bots`: an outside contributor's PR waits for a member.
- A red `main` comes first; nothing else ships until it is green.

## Never flag
- Emoji headings and one-line hooks in the READMEs — house navigation style, not decoration to clean up.
- Tables of exact paths, commands or model IDs — the compression rule keeps them verbatim.
- `As of <date>` stamps — the mechanism for dated claims, not staleness.
- The `description` / `argument-hint` / `allowed-tools` frontmatter in `.claude/commands/*.md` — required by the command format.
- `.claude/` and `.dz/` files — platform-owned layout, edited by the maintainer bot, not by a doc PR.
- Plan artifacts under `docs/plans/` — transient `/planx` output, not published docs.
- A sibling link written as `../<section>/<file>.md` instead of restating the sibling's content — cross-linking is the rule.

