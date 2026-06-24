# 🖥️ Servers, DNS & Mail

Self-hosted on value hardware. Own the stack, control the cost, run where you deploy.

## Servers — OVH VPS / dedicated
- **OVH** for VPS and dedicated boxes — strong price/performance, no per-request cloud tax.
- A small project = **one VPS + Docker Compose**. A big one = a pool of VPSes running [k3s](kubernetes-gitops.md).
- Each [dev gets their own VPS](../developer-experience/dev-vps.md) from the same provider.

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
