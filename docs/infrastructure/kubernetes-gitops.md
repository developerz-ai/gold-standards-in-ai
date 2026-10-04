# ☸️ Kubernetes + GitOps (for big projects)

For multi-service production, run **k3s (HA)** with **ArgoCD** GitOps. The cluster is a pure function of git: merge a manifest → it goes live. CI never touches the cluster.

> Only reach for this when the project needs it. A single VPS + Docker Compose is the right answer for most apps → [servers-and-dns.md](servers-and-dns.md).

## Topology
```
vpn / bastion            control-plane (3× HA)        workers
├─ mesh VPN              ├─ k3s control + etcd        ├─ worker-1 (public ingress)
├─ mail                  └─ (quorum of 3)             └─ worker-db (tainted, state)
└─ SSO (public)

Internet → worker-1:80/443 (Traefik ingress, Let's Encrypt HTTP-01)
```
- k3s installed with `--disable servicelb`; **Traefik** runs as a DaemonSet pinned to the public worker, `hostNetwork: true`, binds 80/443 directly — no cloud LB.
- State (Postgres, Dragonfly) lives on a **tainted** `worker-db` node so it never gets evicted.

## Install order (hard dependency)
```bash
bun scripts/k8s/install-cluster.ts    # k3s on all 3 control planes
bun scripts/k8s/install-platform.ts   # sealed-secrets → cert-manager → Traefik
bun scripts/k8s/install-stateful.ts   # Postgres (CNPG) → pgcat pooler → Dragonfly
```

## ArgoCD — app-of-apps
`stacks/` is the source of truth. A root Application recurses into every `*/application.yml`:
```
stacks/
├── platform/   app-of-apps.yml, traefik, cert-manager, sealed-secrets, argocd, image-updater
├── stateful/   postgres-cluster, postgres-instance, pgcat, dragonfly-{prod,staging,jobs}
├── sso/        zitadel, vaultwarden
├── mail/       maddy
└── apps/
    ├── _template/   namespace, deployment, service, certificate, ingressroute, sealed-secret, cnpg-database
    └── <app>/       application.yml + manifests/
```
```yaml
syncPolicy:
  automated: { prune: true, selfHeal: true }
  syncOptions: [CreateNamespace=true, ServerSideApply=true]
```

## Image Updater — deploy without git write-back
The killer pattern: **ArgoCD Image Updater patches the live Application CR in-cluster** when a new image digest appears. No "digest PR → merge → sync" wait; new image → live in ~30s.
```yaml
metadata:
  annotations:
    argocd-image-updater.argoproj.io/image-list: <app>=registry/<app>:latest
    argocd-image-updater.argoproj.io/<app>.update-strategy: digest
```
It watches both DOCR (private) and GHCR (public) `:latest`. CI's only job is to build + push — **CI holds zero cluster credentials.** → [../developer-experience/linting-ci.md](../developer-experience/linting-ci.md)

## 🫀 Workload runtime contract
Every app Deployment in `stacks/apps/_template/` ships these. Rolling deploys then drop zero requests.

| Field | Rule | Why |
|---|---|---|
| `livenessProbe` → `/livez` | no dependency checks | restart only a hung process, never the fleet on a DB blip |
| `readinessProbe` → `/readyz` | DB + cache; `503` while draining | stop routing to a pod that can't serve |
| Liveness ≠ readiness endpoint | never the same path | same path = a dependency outage becomes a restart loop |
| `startupProbe` | instead of `initialDelaySeconds` | slow boot gets a budget; no guessed delay, no early kill |
| `resources.requests` | always set; memory limit = request | unset → scheduler guesses, noisy-neighbour OOM evictions |
| `terminationGracePeriodSeconds` | ≥ the app's full drain time | otherwise `SIGKILL` lands mid-request / mid-job |
| `replicas: 2`+ and a `PodDisruptionBudget` | for anything user-facing | a node drain can't take the last pod |
| `strategy.rollingUpdate` | `maxUnavailable: 0`, `maxSurge: 1` | new pod Ready before an old one leaves |

```yaml
spec:
  replicas: 2
  strategy: { type: RollingUpdate, rollingUpdate: { maxUnavailable: 0, maxSurge: 1 } }
  template:
    spec:
      terminationGracePeriodSeconds: 45     # 5s deregister + in-flight + worker close + pool close
      containers:
        - name: app
          resources:
            requests: { cpu: 100m, memory: 256Mi }
            limits:   { memory: 256Mi }
          startupProbe:   { httpGet: { path: /livez,  port: http }, periodSeconds: 2,  failureThreshold: 30 }
          livenessProbe:  { httpGet: { path: /livez,  port: http }, periodSeconds: 10, failureThreshold: 3 }
          readinessProbe: { httpGet: { path: /readyz, port: http }, periodSeconds: 5,  failureThreshold: 2 }
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata: { name: app }
spec:
  maxUnavailable: 1
  selector: { matchLabels: { app: app } }
```
App side (`/livez`, `/readyz`, SIGTERM drain) → [../stack/backend-bun-hono.md](../stack/backend-bun-hono.md#-health-livez--readyz).

## Per-app isolation
- **One Postgres database + role per app**, one Dragonfly DB index per app — no shared keys. `bun scripts/db/init-app-db.ts <app>` provisions both and emits a sealed-secret stub with `DATABASE_URL` + `REDIS_URL`.
- Apps connect through the **pgcat** pooler on `:6432`, never direct to the primary.
- TLS per app via cert-manager + Let's Encrypt (IngressRoute + Certificate).

## Operate it with TS scripts
Everything is a Bun TS script (`scripts/<resource>/<verb>.ts`, reusable `scripts/lib/`) → [../developer-experience/dx-scripts.md](../developer-experience/dx-scripts.md):
```bash
bun scripts/k8s/pods.ts <selector>
bun scripts/k8s/logs.ts <pod>
bun scripts/argocd/sync.ts <app>
```
Render-check manifests before pushing: `kubectl kustomize stacks/apps/<app>/manifests`.

## Deploys are supervised
The deploy itself is agent-run (the "Ana" pattern), with a human watching the rollout and able to stop it. Single environment, no staging — validate pre-DNS-flip with `curl --resolve` against the ingress node.

After every roll, smoke the live URL before calling it done: key routes return `200`, `/readyz` is `ok`, the page loads with **zero console errors** and its key assets (JS, CSS, fonts) resolve. Failure → `git revert` the PR, don't patch forward blind → [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md#definition-of-done).
