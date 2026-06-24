# 🐧 Every Dev on a Linux VPS with Claude Code

**Standard:** every developer works on a Linux VPS (or dedicated box) with Claude Code installed and the workspace cloned. No exceptions.

## The problem it kills
"Works on my machine." Path separators, line endings, missing system deps, a script that runs on Mac but not Windows, Docker behaving differently on three OSes — every one of those is a tax the team and the agents pay repeatedly.

> **One OS. One setup. One set of commands.** The Linux/Windows/Mac fragmentation problem disappears.

## Why it's a force multiplier for agents
- **Identical environment for everyone** — a script that works for one dev (or agent) works for all.
- **Same env as production** — dev and deploy targets are both Linux. Fewer "works locally, breaks in prod" surprises.
- **Claude Code runs where the code runs** — full filesystem, real services, real network. No bridging from a laptop.
- **Disposable + reproducible** — a box is cattle, not a pet. Rebuild from a script in minutes.
- **Safe autonomy** — the VPS is the blast radius, so you can run the agent with [broad permissions](../writing-for-agents/hooks-and-permissions.md); you still review every PR.
- **Reachable from anywhere** — phone, tablet, any laptop → SSH/web into the same powerful box.

## What a dev box has
- Ubuntu LTS (or similar), hardened SSH (key-only, modern crypto), fail2ban.
- Per-handle Linux user (not a shared `ubuntu@`) — rotate one without touching others.
- Bun, Docker, git, gh, and Claude Code preinstalled.
- The workspace cloned via the org `repos.json` manifest ([monorepo](../architecture/monorepo.md)).
- Access to internal services as needed (reached over a mesh VPN or via a bastion ProxyJump) → [../infrastructure/servers-and-dns.md](../infrastructure/servers-and-dns.md).

## One-command onboarding
A new dev should be productive in minutes, not days. A single onboarding script provisions, idempotently:
1. GitHub org membership.
2. Mailbox on the self-hosted mail server.
3. SSO identity (Zitadel, federated to GitHub) → [../infrastructure/sso-zitadel.md](../infrastructure/sso-zitadel.md).
4. Secrets-manager access (Vaultwarden collections by role) → [../infrastructure/secrets.md](../infrastructure/secrets.md).
5. The dev VPS: Unix user, SSH key, dotfiles, workspace clone.
6. Claude Code config + MCP servers (audited DB gateway, secrets broker).

Offboarding is the same script in reverse. State of record is a small committed mapping (`gh_login → handle, role`), not tribal knowledge.

## Sizing
Give dev boxes real resources — an agent running tests, a dev server, and Docker services at once wants CPU and RAM. A cramped box reintroduces the slow inner loop you're trying to kill. Source them from a value VPS/dedicated provider (e.g. OVH) → [../infrastructure/servers-and-dns.md](../infrastructure/servers-and-dns.md).
