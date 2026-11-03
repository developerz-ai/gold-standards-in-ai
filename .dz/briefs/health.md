# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | `bin/check-docs` | it prints nothing |
| every doc is indexed | `bin/check-docs` | it prints nothing |

One validator: `bin/check-docs` — the gate and this checkup run the same code, so a finding here is a finding there.

## What to file

For each unhealthy signal, one ask, unless an open issue already carries its exact title
(`gh issue list --state open --search "<title> in:title" --json number,title`, then compare byte for byte).
Titles are stable: nothing that changes run to run (sha, run URL, version) goes in one; that goes in the body,
with the command and its output. Unreadable evidence is no verdict: re-run it, never file it as a defect.
A red `main` comes first: nothing releases from it.

## Rows

- `relative-links` — Every relative link resolves; use `bin/check-docs`.
- `docs-indexed` — Every doc is indexed; use `bin/check-docs`.
