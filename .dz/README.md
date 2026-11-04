# `.dz/`

The developerz.ai maintainer bot's configuration for this repo. The platform owns the layout.

| Path | What |
|---|---|
| `maintainer/maintainer.yml` | the policy: what the bot triages, closes, reviews, merges and releases. [Reference](https://developerz.ai/docs/maintainer-yml) |
| [maintainer/reviewer.md](maintainer/reviewer.md) | reviewer instructions — this repo's own layer of the reviewer's prompt, read at every review; refines the platform rules, never overrides them; NOT the policy |
| `briefs/` | what each `schedules:` checkup works from, read at `main`'s HEAD when it fires: `health.md` (broken links, an unindexed doc, red default-branch CI, stuck Dependabot PRs), `improve.md` (monthly: accuracy upkeep). No code and no release, so no `release-due` |
| [onboarding/](onboarding/) | the AI-first scorecard from the latest onboarding run (each run rewrites it whole, latest wins) |
