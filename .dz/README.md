# `.dz/`

The developerz.ai maintainer bot's configuration for this repo. The platform owns the layout.

| Path | What |
|---|---|
| `maintainer/maintainer.yml` | the policy: what the bot triages, closes, reviews, merges and releases. [Reference](https://developerz.ai/docs/maintainer-yml) |
| [maintainer/reviewer.md](maintainer/reviewer.md) | reviewer instructions — this repo's own layer of the reviewer's prompt, read at every review; refines the platform rules, never overrides them; NOT the policy |
| `briefs/` | what each `schedules:` checkup works from, read at `main`'s HEAD when it fires: `improve.md` (monthly: accuracy upkeep, a stuck Dependabot PR, the Sentry MCP pin's advisories). Broken relative links and an unindexed doc are the gate (external links stay with `improve`)'s (`bin/check`); a red default branch is the platform's red-main ask. The daily `health` schedule was deleted (270 firings in 30 days, 8 asks, 0 merges). No code and no release, so no `release-due` |
| [onboarding/](onboarding/) | the AI-first scorecard from the latest onboarding run (each run rewrites it whole, latest wins) |
