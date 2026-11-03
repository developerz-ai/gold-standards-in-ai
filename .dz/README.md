# `.dz/`

The developerz.ai maintainer bot's configuration for this repo. The platform owns the layout.

| Path | What |
|---|---|
| `maintainer/maintainer.yml` | the policy: what the bot triages, closes, reviews, merges and releases. [Reference](https://developerz.ai/docs/maintainer-yml) |
| `briefs/` | what each `schedules:` checkup works from, read at `main`'s HEAD when it fires: `health.md` (broken links, an unindexed doc, red default-branch CI, stuck Dependabot PRs), `improve.md` (monthly: accuracy upkeep). No code and no release, so no `release-due` |
