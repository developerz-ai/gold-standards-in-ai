# 🖥️ Servers, DNS & Mail

Self-hosted on value hardware. Own the stack, control the cost, run where you deploy.

## Servers — OVH & Hetzner (VPS / dedicated)
- **OVH** and **Hetzner** for VPS and dedicated boxes — both give strong price/performance with no per-request cloud tax. Pick whichever has the right box/region; mix freely.
- A small project = **one VPS + Docker Compose**. A big one = a pool of VPSes running [k3s](kubernetes-gitops.md).
- Each [dev gets their own VPS](../developer-experience/dev-vps.md) from the same providers.
- The hosts run the workloads; **container images live in a registry** (below), pulled at deploy time.

### Server inventory as data
Keep servers in a committed inventory (`servers/<region>/<pool>/<id>.yml`), never hand-parsed — load it through a `lib/inventory/load.ts` helper:
```yaml
id: control-1
provider: ovh
role: control-plane
pool: control-plane
public_ip: 203.0.113.10
wg_ip: 100.64.0.8
dns: control-1.infra.example.com
os: ubuntu-24.04
ssh_keys: [alice, bob]      # per-handle users, not a shared ubuntu@
```

### SSH & users
- **Per-handle Linux user** per GitHub login — rotate one key without touching others. NOPASSWD sudo, key-only auth.
- Sync keys idempotently from a script that rewrites `authorized_keys` authoritatively.
- Harden: modern-crypto-only SSH + fail2ban via a `scripts/servers/harden.ts`.

### Bootstrap with TS scripts
```bash
bun scripts/servers/list.ts                 # inventory + live provider state
bun scripts/servers/bootstrap.ts <selector> # hostname, keys, sysctls, ufw, docker
bun scripts/servers/exec.ts <selector> <cmd># SSH (ProxyJump-aware for mesh hosts)
```
Selectors (`--all`, `--pool <p>`, `--role <r>`, `--tag <t>`) come from the YAML inventory. All SSH/HTTP goes through `scripts/lib/` ([DX scripts](../developer-experience/dx-scripts.md)).

## Container registries — DigitalOcean (DOCR) + GHCR
Hosts are cattle; the **image is the artifact**. Store images in a registry and pull them at deploy time — CI builds + pushes, the cluster pulls.

| Registry | Use for |
|---|---|
| **GHCR** (GitHub Container Registry) — `ghcr.io/<org>/<app>` | **public** images, OSS, anything tied to the GitHub org |
| **DOCR** (DigitalOcean Container Registry) — `registry.digitalocean.com/<org>/<app>` | **private** app images |

- Tag every push `latest` + `<sha>` (multi-arch amd64+arm64 via Buildx when targets differ).
- The cluster pulls with a registry pull-secret per app (a [sealed secret](secrets.md)); **CI never holds cluster credentials** — the handoff is image-only.
- [ArgoCD Image Updater](kubernetes-gitops.md) watches both registries' `:latest` digest and rolls the new image in-cluster.
- Build/push pipeline + layer caching: [../developer-experience/linting-ci.md](../developer-experience/linting-ci.md).

## Network — mesh VPN + bastion
- **Mesh VPN** (e.g. Headscale/WireGuard) for **server-to-server** traffic only (app → database over internal IPs).
- **Humans don't join the mesh.** Internal UIs are public but **SSO-gated** (Zitadel + MFA is the perimeter) → [sso-zitadel.md](sso-zitadel.md).
- Dev VPSes stay off-mesh, reachable via `ProxyJump bastion`.
- **UFW per host:** 22 everywhere; 80/443 only on the public ingress node; mail ports only on the mail host; VPN port only on mesh hosts.

## DNS — ClouDNS
Manage DNS as code through a script, not a console:
```bash
bun scripts/dns/upsert.ts example.com A @ 203.0.113.10
```
ClouDNS for authoritative DNS; TLS is automatic via Let's Encrypt (cert-manager in k3s, or a companion container with Compose).

## Mail — self-hosted
Run your own mail server (e.g. **maddy**) on the bastion/vpn host — multi-domain, full control, no per-seat SaaS cost.
```bash
bun scripts/mail/provision-domain.ts example.com
bun scripts/mail/create.ts alice@example.com
```
Mailboxes back the [SSO identities](sso-zitadel.md) and the [onboarding flow](../developer-experience/dev-vps.md). Open only the needed ports (25, 143, 465, 587, 993) and only on the mail host.
