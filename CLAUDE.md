# Gold Standards in AI

Knowledge base for AI coding agents. **Markdown only — no code to build, no tests to run.** Distills how developerz-ai builds software fast with agents.

## What this repo is for
- An agent reads this before starting/working on a project to learn our conventions, stack, infra, and how to set a repo up so agents thrive.
- Each `docs/**/*.md` is self-contained, dense, copy-paste oriented.

## Structure
- `README.md` — the map. Every doc is linked from it.
- `docs/00-philosophy.md` — the core thesis.
- `docs/<section>/README.md` — index for each section.
- `docs/<section>/<topic>.md` — one topic per file.

## Writing rules (this repo dogfoods its own advice — see docs/writing-for-agents/compressed-config.md)
- Lead with the rule, not the reason. Fragments over sentences. Tables over paragraphs.
- File paths and exact commands stay verbatim. Compress prose only.
- One topic per file. Keep each file scannable in one pass.
- Use emojis in headings/READMEs for navigation; keep code blocks clean.
- Cross-link related docs with relative links.
- Date load-bearing claims (`As of 2026-06`). Delete stale docs rather than letting them rot.

## When adding/editing docs
- Update `README.md` and the section `README.md` with a link + one-line hook.
- Keep examples vendor-neutral and reusable. Prefer the latest Claude models when showing model IDs.
- No secrets, no private hostnames/IPs, no client names.
