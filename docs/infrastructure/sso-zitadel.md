# 🔑 SSO — One Login for Many Apps

One identity provider in front of everything: **Zitadel** (OIDC), federated to **GitHub**. Devs log in once, with MFA, and reach every internal app and UI. New apps add SSO by registering an OIDC client — no per-app user database.

## Why one SSO
- **One perimeter.** Internal UIs (ArgoCD, Grafana, mail, secrets, dashboards) can be public because Zitadel + MFA is the gate.
- **No password sprawl.** Devs never set a Zitadel password — GitHub federation + org-enforced MFA is the auth.
- **One offboard.** Deactivate in Zitadel → access to every app is gone.

## The flow
```
laptop → "Login with GitHub" → Zitadel (OIDC) → GitHub OAuth (2FA) → Zitadel JWT → app
```

## Per-app integration — register an OIDC client
Each app/UI gets a Zitadel OIDC client. Provision idempotently from a script:
```bash
bun scripts/sso/create-oidc-client.ts <app>
# convention: redirect URI https://<app>.example.com/oauth2/callback
# outputs client_id + client_secret → into a sealed-secret
```
Wire GitHub as the IdP once (`scripts/sso/wire-github-idp.ts`); seed users from GitHub org membership (`scripts/sso/seed-devs-from-github.ts`). State of record is a small committed mapping (`gh_login → handle, role`).

## App-side OIDC
Standard OIDC/OAuth2 — use a library for your runtime. The app trusts Zitadel's JWT and reads groups/roles from claims. For an agent or service that needs identity end-to-end, pass the **bearer token** to downstream gateways rather than minting new credentials.

## SSO-gated capability access for agents
SSO is also how an agent gets *audited* access to sensitive systems without ever holding a credential. The [DB gateway](../ai-agents/tools-and-mcp.md#audited-capability-access-the-gateway-pattern) pattern:
- The agent's MCP call carries only an SSO session token.
- The gateway validates the token + the user's groups, checks config-as-code grants, runs the query with a least-privilege role, writes a synchronous audit row, and returns rows.
- Every query traces **SSO user → group → grant → audit row**. No DB URL ever reaches the client.

This is the concrete answer to "give the agent maximum access, leak nothing" → [../00-philosophy.md](../00-philosophy.md) principle #2.
