# 🎫 Issues & Milestones at Agent Speed

An **issue is the unit of work** an autonomous agent picks up; a **milestone** groups them into a release or epic. Move fast with the `gh` CLI — scriptable, no clicking, and an agent can drive it directly.

## `gh` recipes
Real, copy-pasteable. `{owner}/{repo}` = your repo.

```bash
# Create an issue with body, labels, milestone
gh issue create \
  --title "Approval service: approve/reject a receipt" \
  --body "From plan slice 02-backend-service.md. Closes the domain logic." \
  --label "type:feat,area:receipts,priority:high" \
  --milestone "approvals"

# Create a milestone (no first-class gh command — use the API)
gh api repos/{owner}/{repo}/milestones \
  -f title="approvals" \
  -f description="Receipt approval flow" \
  -f due_on="2026-07-15T00:00:00Z"

# Edit: add to milestone + labels
gh issue edit 142 --milestone "approvals" --add-label "status:ready"

# Open a PR linked to an issue (auto-closes on merge)
gh pr create --title "feat: approval service" --body "Closes #142" --base main

# Comment / triage
gh issue comment 142 --body "Picked up by worker-3."
gh issue list --milestone "approvals" --label "status:ready" --state open
```

### Bulk-create a milestone + its issues
One milestone, a batch of issues under it — straight from a `planx` plan:

```bash
#!/usr/bin/env bash
set -euo pipefail
REPO="{owner}/{repo}"
MILESTONE="approvals"

# 1. Milestone (idempotent-ish: ignore "already_exists")
gh api "repos/$REPO/milestones" -f title="$MILESTONE" \
  -f description="Receipt approval flow" 2>/dev/null || true

# 2. Issues — one per plan slice
issues=(
  "01 data model|type:feat|area:db"
  "02 backend service|type:feat|area:receipts"
  "03 api routes|type:feat|area:api"
  "04 frontend|type:feat|area:web"
  "05 tests|type:test|area:receipts"
)

for row in "${issues[@]}"; do
  IFS='|' read -r title type area <<<"$row"
  gh issue create --repo "$REPO" \
    --title "$title" \
    --body "From plan 101-receipt-approval-flow / slice ${title%% *}-*.md" \
    --label "$type,$area,priority:med" \
    --milestone "$MILESTONE"
done
```

## Map a `planx` plan → GitHub
The plan structure maps 1:1 onto GitHub:

| `planx` | GitHub |
|---|---|
| The plan dir (`1NN-<slug>/`) | A **milestone** |
| Each `<NN>-<aspect>.md` slice | An **issue** |
| `status.yml` `evidence` | The PR / commit refs that close each issue |

So a plan is the source of truth; the issues/milestone are its projection onto the tracker. See [project-kickoff.md](project-kickoff.md).

## Labels convention
| Group | Values |
|---|---|
| `type:` | `feat` · `fix` · `chore` · `docs` · `test` |
| `priority:` | `high` · `med` · `low` |
| `area:` | `api` · `web` · `db` · `infra` · `<package>` |
| `status:` | `ready` · `in-progress` · `blocked` · `review` |

**Milestones = releases or epics** (e.g. `v1.0`, `approvals`). Issues = slices.

## Let the agent run `gh`
Add to the allow list so the agent operates the tracker without prompts → [../writing-for-agents/hooks-and-permissions.md](../writing-for-agents/hooks-and-permissions.md):

```json
{
  "permissions": {
    "allow": [
      "Bash(gh issue *:*)",
      "Bash(gh pr *:*)",
      "Bash(gh api:*)"
    ]
  }
}
```

## End to end
```
idea
  └─ /initial-idea ─▶ spec + milestone list
        └─ /planx ─▶ multi-file plan + status.yml
              └─ gh ─▶ milestone + issues (one per slice)
                    └─ Planner / Worker / Reviewer ─▶ PRs ("Closes #N")
                          └─ automated AI review + human review
                                └─ supervised deploy
```

- Plan: [project-kickoff.md](project-kickoff.md)
- Autonomous execution: [../ai-agents/orchestration.md](../ai-agents/orchestration.md)
- The review gate (AI + human): [../developer-experience/linting-ci.md](../developer-experience/linting-ci.md)
- Why deploy stays human-supervised: [../00-philosophy.md](../00-philosophy.md)
