# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | `bin/check-docs` | it prints nothing |
| every doc is indexed | `bin/check-docs` | it prints nothing |
| red default-branch CI | `gh run list --branch main --workflow check --limit 5 --json headSha,conclusion,status,url` | the newest completed run is `success` at the sha `main` names now |
| stuck Dependabot PR | `gh pr list --state open --author app/dependabot --json number,title,createdAt,url` | no open PR is older than 7 days, one `.github/dependabot.yml` schedule interval |

One validator: `bin/check-docs` — the gate and this checkup run the same code, so a finding here is a finding there.

## What to file

For each unhealthy signal, one ask, unless an open issue already carries its exact title
(`gh issue list --state open --search "<title> in:title" --json number,title`, then compare byte for byte).
Titles are stable: nothing that changes run to run (sha, run URL, version) goes in one; that goes in the body,
with the command and its output. Unreadable evidence is no verdict: re-run it, never file it as a defect.
A red `main` comes first: nothing releases from it. A stuck Dependabot PR comes next.

## Rows

- `relative-links` — Every relative link resolves; use `bin/check-docs`.
- `docs-indexed` — Every doc is indexed; use `bin/check-docs`.
- `main-ci` — The newest completed `check` run is `success` at the sha `main` names now; a finding carries the run URL, the head sha and `main`'s current sha; a red run on a superseded sha and an errored `gh` call are not findings (re-run, never file); use the existing default-branch CI command in the Signal table above.
- `dependabot-prs` — No open Dependabot PR is older than 7 days, one `.github/dependabot.yml` schedule interval; a finding carries the PR number, its age in days and its URL; a younger PR is normally progressing and an errored `gh` call is missing evidence (re-run, never file); use the existing Dependabot PR command in the Signal table above.
