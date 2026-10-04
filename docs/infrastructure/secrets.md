# 🔐 Secrets

**Never commit a plaintext secret.** Three mechanisms, by audience.

| Audience | Mechanism |
|---|---|
| In-cluster apps | **Sealed-secrets** (encrypted at rest, decrypted only in-cluster) |
| Humans | **Vaultwarden** (zero-knowledge, role-scoped collections) |
| Local dev | **`.env.example`** committed, `.env` gitignored |
| CI | GitHub Actions secrets |
| Agents | Their own identities + short-lived scoped tokens → [below](#-agent-identities--its-own-accounts-never-yours) |

## `.env` pattern
```
project/
├── .env.example          # committed — documents every key, no values
├── .env.development       # committed — non-secret local defaults only
├── .env.development.local # gitignored — per-box secrets/overrides, wins
└── .env                   # gitignored — real secrets
```
`.env.example` is the contract: every key, grouped and described, zero values.
```bash
# === Infrastructure ===
DATABASE_URL=
REDIS_URL=
# === External ===
OPENROUTER_API_KEY=
R2_ACCESS_KEY=
```
The agent never sees real secrets — it runs scripts that load them. New box: copy `.env.example` → `.env`, fill in.

## Sealed-secrets (in-cluster)
A `SealedSecret` is encrypted to a `(namespace, name)` and only the in-cluster controller can decrypt it — so it's safe to commit to git.
```bash
kubectl create secret generic <app>-secrets -n <app> \
  --from-literal=API_KEY='value' --dry-run=client -o yaml \
| kubeseal --controller-namespace=sealed-secrets -o yaml > manifests/sealed-secret.yml
```
Seal offline from a laptop by fetching the controller's public cert first. A reloader restarts pods automatically on secret change → rotation is a re-seal + commit.

## Vaultwarden (humans)
Role-scoped collections, provisioned as code:
```
cto              → provider tokens, registry creds, admin
infrastructure   → cluster/API tokens (devops)
onboarding       → SSO PAT, per-dev mailbox passwords
apps/<app>       → that app's secrets
```
A `warden-mcp` broker lets an agent fetch a needed secret at call time without it ever landing in a file.

## 🤖 Agent identities — its own accounts, never yours
**Give the agent broad reach under its own name.** Separate identity is what makes wide access safe: every action is traceable to the bot, and a hijacked turn is the bot, not you.

| System | Agent gets | Never |
|---|---|---|
| GitHub | a bot **GitHub App** → installation tokens (expire in 1h, scoped to listed repos + permissions) | a dev's personal PAT |
| Email | `agent@<domain>` mailbox | a human's inbox |
| Chat | a bot user | a human's session token |
| DB / internal APIs | SSO identity through the [audited gateway](sso-zitadel.md) | a shared connection string |
| Cloud / registry | a dedicated service account, least privilege per job | the org admin key |

- **Short-lived, minted per job.** Mint at session start, let it expire. Nothing long-lived in the agent's env or memory.
- **Repo-scoped beats org-wide.** A token for the repo it's working on; mint another for the next repo.
- **Audit by identity.** Commits, comments, queries all carry the bot's name → you can grep what it did.
- A missing grant is a setup bug: add it to the bot's role, don't hand over your own credentials. → [../ai-agents/agent-work-limits.md](../ai-agents/agent-work-limits.md)
- Why it matters for injected input: [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md).

## 🚨 Incident: a credential or install got compromised
Order matters — rotating first while a persistence payload is still running just hands it the new token.

1. **Stop publish/deploy** from the affected box or runner.
2. **Preserve evidence**: lockfile (`bun.lock`, `package-lock.json`, …), shell/package-manager history, CI run URLs + runner logs, outbound network logs.
3. **Treat the box as compromised** if install/lifecycle scripts could have run.
4. **Remove persistence before rotating**: `~/.claude/settings.json` and project `.claude/settings.json` (injected env/commands), `.mcp.json` entries you didn't add, `.vscode/tasks.json` folder-open tasks, `~/.config/systemd/user/*.service` units, cron entries, unfamiliar binaries in `~/.local/bin/`, `/tmp` payloads.
5. **Rotate everything the process could reach**: registry/npm tokens, GitHub tokens + deploy keys + Actions secrets, cloud + k8s service-account tokens, SSH keys, `.env` values, **and MCP / agent-harness credentials** in env or user config. Re-seal changed in-cluster secrets.
6. **Purge CI dependency caches.**
7. **Reinstall clean, scripts off**: `bun install --ignore-scripts` (or `npm ci --ignore-scripts`) on pinned known-good versions; re-enable scripts after.
8. **Ship the guard** that would have caught it (pin, scan, lint) in the same PR as the cleanup → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md).

## Rules
- **No secret in source, logs, or error messages** — ever. (Rust: never put a credential in a typed error.)
- **Output a generated credential once** for copy-paste, then never again.
- **Rotate regularly**; least-privilege roles (read-only where possible).
- **Scan history** for leaked secrets in CI (gitleaks); fail the build on a finding.
- DB access for agents goes through the [audited gateway](sso-zitadel.md), not a connection string.
