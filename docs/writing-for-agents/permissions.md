# 🔓 Permissions — full power, gates in `bin/check`

**Rule: the agent runs with full permissions and zero prompts. Quality is enforced by the gate it runs (`bin/check`) and the gate CI runs — never by approval prompts, and by a harness hook only when the hook is [earned by a named failure](#-when-a-hook-is-earned).** An agent that pauses for approval is misconfigured; an agent fenced by unearned hooks is slower, dumber, and fails in ways nobody can see.

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
- No `allow` lists to curate, no `allowed-tools` fences on skills or subagents. Every approval prompt is a bug in the setup.
- A `deny` entry passes the [same test as a hook](#-when-a-hook-is-earned): it names an irreversible command (`git push --force`, `git stash` in a shared checkout, `rm -rf /`) — never a preference. Denying never changes `defaultMode`.

## 🛡️ What makes full power safe
Safety lives in the **tools and the pipeline**, not in withholding capability → [agent-work-limits.md](../ai-agents/agent-work-limits.md).

| Risk | Handled by |
|---|---|
| Bad code lands | `bin/check` before every commit (a `CLAUDE.md` rule) + the same gate in CI + PR review → [ai-first-cicd.md](../developer-experience/ai-first-cicd.md) |
| Prod data damage | audited DB gateway, per-grant roles, backups — the tool, not a prompt → [tools-and-mcp.md](../ai-agents/tools-and-mcp.md#audited-capability-access-the-gateway-pattern) |
| Leaked credentials | creds never reach the agent; it gets its own identity → [secrets.md](../infrastructure/secrets.md) |
| Hostile input steering the agent | data-flow design → [untrusted-input.md](../ai-agents/untrusted-input.md) |
| Box compromised | the VPS is disposable; rebuild from `bin/setup` |

## 🪝 When a hook is earned
**Default: no hook.** Each objection below is a cost every hook pays:

| Hook cost | `bin/check` + CI answer |
|---|---|
| Invisible when it lives in user config the repo doesn't carry | One command, in `CLAUDE.md`, readable, runnable by anyone |
| Harness-specific — another agent, a teammate's laptop, a script skips it | Same gate everywhere: agent, human, CI |
| Blocks mid-loop with output the agent didn't ask for | The agent runs the gate when the work is ready and reads the failure |
| Enforcement you can't trust (hookless runs happen) | CI is the enforcement; local is the fast copy |

**A hook is earned when it names a failure the end-of-work gate catches too late — and passes every row:**

| # | Condition | Fails when |
|---|---|---|
| 1 | **A named, recurring failure** the gate finds only after the cause is gone — the edit that caused it is many edits back, or its author was a seat that already reported | the justification is "to be safe" or "remind the agent" |
| 2 | **The fast copy of a check the gate and CI still run** — never the only enforcement | removing the hook would let the defect merge |
| 3 | **Scoped to the edit and fast** — seconds, measured, the number and its date beside the code | it runs the whole-repo suite per keystroke |
| 4 | **Silent when green; hands the error text back when red** to the agent that made the edit (Claude Code `PostToolUse`: exit 2, the output on stderr) | it prints on success, or blocks with no reason |
| 5 | **Committed** in the project `.claude/settings.json`, its script in the repo | it lives in one person's user settings |
| 6 | **Shares one box-wide reservation** when several agents edit on one machine — N seats are N hook runs per edit wave, so an extra run queues instead of OOMing the box | each seat's hook spawns a compiler beside every other |

**Worked example — a post-edit typecheck hook.** Failure named: without it, a type error surfaces only at the coordinator's final gate — many edits after its cause, mixed with every other seat's errors, after the seat that wrote it has reported done. The hook routes each edited file to its owning project and compiles that subgraph only; it compiles against **source**, never stale emitted declaration files (a per-package `--noEmit` against old `.d.ts` reports "has no exported member" for members that exist — a hook that cries wolf is worse than none). Exit 2 with the compiler tail on error, nothing on green. The whole-graph typecheck still runs in the gate and in CI.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          { "type": "command", "command": "cd \"$CLAUDE_PROJECT_DIR\" && bin/typecheck-scoped --hook" }
        ]
      }
    ]
  }
}
```

Not earned, ever: format-on-save (the gate's autofix does it), policy or approval logic, anything whose only output is advice. Automated behaviour ("always lint before commit") → a `CLAUDE.md` rule + the gate in CI that fails if it didn't happen. Mechanical checks → a [lint guard](guards-and-gotchas.md) inside `bin/check`.

## 🔁 The iteration rule
- Explain something twice → a [skill](skills-commands-agents.md).
- Agent asks the same question → `CLAUDE.md`.
- Agent repeats a mistake → a [lint guard](guards-and-gotchas.md) in `bin/check`; prose only if it can't be checked.
- An approval prompt appears → fix the setup, don't click through it.
