# 🔒 Hooks & Permissions — Quality Gates and Trust

The goal: **maximum autonomy on safe operations, automated quality gates, human approval only for the genuinely risky.** An agent that pauses for approval 20 times in a row is misconfigured.

## The trust model
```
┌─────────────────────────────┐
│  Full autonomy              │  read, grep, test, lint, edit, git
│  (pre-allowed, no prompts)  │
├─────────────────────────────┤
│  Quality gates              │  hooks fire on commit / push
│  (automated enforcement)    │
├─────────────────────────────┤
│  Approval required          │  destructive / unknown / prod-mutating ops
└─────────────────────────────┘
```

## Permissions — allow the safe stuff
Safe because the *tools* are safe: `scripts/*` have read-only DB roles and no destructive ops; `bin/*` are your own commands; `git` outcomes are reviewed at the PR. So broad allow is fine in practice.

```json
{
  "permissions": {
    "allow": [
      "Bash(bin/*:*)",
      "Bash(bun *:*)",
      "Bash(scripts/*:*)",
      "Bash(git add:*)", "Bash(git commit:*)", "Bash(git push:*)",
      "Bash(git fetch:*)", "Bash(git checkout:*)", "Bash(git rebase:*)",
      "Bash(gh pr create:*)", "Bash(gh pr view:*)", "Bash(gh api:*)",
      "Bash(docker compose:*)",
      "WebSearch"
    ],
    "defaultMode": "acceptEdits"
  }
}
```

`defaultMode: "acceptEdits"` lets the agent edit files freely — you review the diff. Many run with full bypass in a sandboxed [dev VPS](../developer-experience/dev-vps.md); the VPS *is* the blast radius, and you still review every PR.

Settings live in `.claude/settings.local.json` (project) and `~/.claude/settings.json` (user). Project overrides user.

## Hooks — automated quality gates
Shell commands that run before/after tool calls. They enforce quality at the tool level, so the agent *physically cannot* skip them.

Pre-commit lint hook:
```bash
# .claude/hooks/pre-commit.sh
#!/bin/bash
set -e
bun run lint:fix          # or bin/lint, make lint
if [ -n "$(git diff --name-only)" ]; then
  git add -u              # auto-stage what the linter changed
fi
```

Wire it:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit:*)",
        "hooks": [{ "type": "command", "command": ".claude/hooks/pre-commit.sh" }]
      }
    ]
  }
}
```

Now every commit passes through the linter. Lint changes are auto-staged. **Unlinted code cannot be committed.**

| Trigger | Hook | Purpose |
|---|---|---|
| `Bash(git commit:*)` | run linter, auto-stage | style enforcement |
| `Bash(git push:*)` | run full test suite | no broken pushes |
| `Bash(*migrate*)` | back up schema | migration safety net |

> ⚠️ Hooks are how *automated behavior* ("always do X before Y") gets enforced — the harness runs them, not the agent's good intentions. If you need something to happen every time, make it a hook.

## The iteration rule
- Explain something twice? → make it a skill.
- Agent asks the same question? → add it to `CLAUDE.md`.
- Approve the same command 5×? → add it to the allow list.
- Agent repeats a mistake? → save a feedback [memory](memory-and-mcp.md).
