# 🔓 Permissions — full power, gates in `bin/check`

**Rule: the agent runs with full permissions and zero prompts. Quality is enforced by the gate it runs (`bin/check`) and the gate CI runs — never by harness hooks, never by approval prompts.** An agent that pauses for approval is misconfigured; an agent fenced by hooks is slower, dumber, and fails in ways nobody can see.

## ⚙️ The setup
Dev agents run on a [dev VPS](../developer-experience/dev-vps.md) — the VPS is the blast radius, the PR is the review.

```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

- `~/.claude/settings.json` (user) or `.claude/settings.json` (project, committed). Project overrides user.
- Or per run: `claude --dangerously-skip-permissions`.
- No `allow` lists to curate, no `deny` lists, no `allowed-tools` fences on skills or subagents. Every approval prompt is a bug in the setup.

## 🛡️ What makes full power safe
Safety lives in the **tools and the pipeline**, not in withholding capability → [agent-work-limits.md](../ai-agents/agent-work-limits.md).

| Risk | Handled by |
|---|---|
| Bad code lands | `bin/check` before every commit (a `CLAUDE.md` rule) + the same gate in CI + PR review → [ai-first-cicd.md](../developer-experience/ai-first-cicd.md) |
| Prod data damage | audited DB gateway, per-grant roles, backups — the tool, not a prompt → [tools-and-mcp.md](../ai-agents/tools-and-mcp.md#audited-capability-access-the-gateway-pattern) |
| Leaked credentials | creds never reach the agent; it gets its own identity → [secrets.md](../infrastructure/secrets.md) |
| Hostile input steering the agent | data-flow design → [untrusted-input.md](../ai-agents/untrusted-input.md) |
| Box compromised | the VPS is disposable; rebuild from `bin/setup` |

## 🚫 Why no hooks
| Hooks | `bin/check` + CI |
|---|---|
| Invisible — fire from harness config the agent didn't write and can't see in the repo | One command, in `CLAUDE.md`, readable, runnable by anyone |
| Harness-specific — another agent, a teammate's laptop, a script skips them | Same gate everywhere: agent, human, CI |
| Block mid-loop with output the agent didn't ask for | The agent runs the gate when the work is ready and reads the failure |
| Enforcement you can't trust (hookless runs happen) → you need CI anyway | CI is the enforcement; local is the fast copy |

Automated behaviour ("always lint before commit") → a `CLAUDE.md` rule + the gate in CI that fails if it didn't happen. Mechanical checks → a [lint guard](guards-and-gotchas.md) inside `bin/check`.

## 🔁 The iteration rule
- Explain something twice → a [skill](skills-commands-agents.md).
- Agent asks the same question → `CLAUDE.md`.
- Agent repeats a mistake → a [lint guard](guards-and-gotchas.md) in `bin/check`; prose only if it can't be checked.
- An approval prompt appears → fix the setup, don't click through it.
