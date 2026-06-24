# ☸️ Infrastructure

How we host, deploy, secure, and observe. Self-hosted on value hardware, GitOps for the big stuff, SSO across everything, secrets never in the clear.

| File | Topic |
|---|---|
| [kubernetes-gitops.md](kubernetes-gitops.md) | ☸️ k3s HA + ArgoCD GitOps (for big projects) |
| [servers-and-dns.md](servers-and-dns.md) | 🖥️ OVH & Hetzner VPS/dedicated · 📦 DOCR + GHCR registries · 🌐 ClouDNS · ✉️ self-hosted mail · 🔗 mesh VPN |
| [sso-zitadel.md](sso-zitadel.md) | 🔑 One SSO for many apps (Zitadel + GitHub) |
| [secrets.md](secrets.md) | 🔐 Sealed-secrets · Vaultwarden · `.env.example` |
| [observability.md](observability.md) | 🚨 Sentry error monitoring as an agent work-queue |

## Choose the right size
| Project size | Deploy |
|---|---|
| Small / one service | A single OVH/Hetzner VPS + Docker Compose. Done. |
| Big / many services | **k3s HA + ArgoCD** GitOps → [kubernetes-gitops.md](kubernetes-gitops.md) |

Don't reach for Kubernetes until the project actually needs it — a VPS with Compose ships faster and is easier for an agent to operate.

## Principles
- **Self-hosted, value hardware.** OVH & Hetzner VPS/dedicated, ClouDNS, self-hosted mail — own the stack, control the cost. Images in DOCR (private) + GHCR (public).
- **GitOps is the source of truth.** The cluster reflects git; you change infra by merging a PR.
- **One SSO perimeter.** Every internal UI sits behind Zitadel (federated to GitHub + MFA).
- **No plaintext secrets, ever.** Sealed-secrets in-cluster, Vaultwarden for humans, `.env.example` as the only committed env file.
- **Deploys are agent-run, human-supervised** (the "Ana" pattern) → [../00-philosophy.md](../00-philosophy.md).
