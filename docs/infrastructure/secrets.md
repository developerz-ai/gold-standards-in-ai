# 🔐 Secrets

**Never commit a plaintext secret.** Three mechanisms, by audience.

| Audience | Mechanism |
|---|---|
| In-cluster apps | **Sealed-secrets** (encrypted at rest, decrypted only in-cluster) |
| Humans | **Vaultwarden** (zero-knowledge, role-scoped collections) |
| Local dev | **`.env.example`** committed, `.env` gitignored |
| CI | GitHub Actions secrets |

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

## Rules
- **No secret in source, logs, or error messages** — ever. (Rust: never put a credential in a typed error.)
- **Output a generated credential once** for copy-paste, then never again.
- **Rotate regularly**; least-privilege roles (read-only where possible).
- **Scan history** for leaked secrets in CI (gitleaks); fail the build on a finding.
- DB access for agents goes through the [audited gateway](sso-zitadel.md), not a connection string.
