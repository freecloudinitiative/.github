# FILES — Where Code Live

## Root

```
README.md                       What FCI does. How to start. Folder map. Links to all docs.
ROLES.md                        What each repo does. Dependencies. Who talks to who.
FILES.md                        This file.
../../instructions/caveman.md   Documentation style guide for all repos.
go.work                         Go workspace. Lists the seven Go modules.
go.work.sum                     Go workspace checksum file. Auto-generated.
```

No `PLAN`. No `deploy/` in any service repo. Charts live in `k3s-manifests/applications/<service>/`. Cluster bootstrap is `ansible-automation/playbook.yml`. Public DNS + Zero Trust tunnel is `terraform-cloudflare-infra`.

---

## Repos

Each repo is a subdirectory. Each has its own `README.md` and `FILES.md`. Go services also have `ARCHITECTURE.md` and `API.md`. `ansible-automation` uses `ROLES.md` instead of those two. `terraform-cloudflare-infra` is README + FILES only.

```
ansible-automation/             Bare-metal K3s + Argo CD bootstrap. Then GitOps takes over.
  README.md                     What `playbook.yml` does. Two-run OpenBao seed.
  ROLES.md                      Plays, roles, token handoff, why GitOps after Argo CD.
  FILES.md                      Every playbook-related file, one line.
  playbook.yml                  Three plays: masters, workers, first-master components.
  ansible.cfg                   Inventory, vault prompt, `roles_path`.
  inventory.ini                 Host groups and memory tiers.
  group_vars/all/               IPs + Vault `secret.yml`.
  roles/                        Roles `playbook.yml` calls (`k3s-*`, `argocd-*`, `openbao-secrets-init`).

api-gateway/                    HTTP reverse proxy and auth gateway. Entry point for all API traffic.
  README.md                     What it does. How to start.
  ARCHITECTURE.md               How auth, routing, and resilience work.
  API.md                        Endpoints, request/response, errors.
  FILES.md                      Core files, one line.
  cmd/gateway/                  Binary entrypoint (`go run ./cmd/gateway`).
  internal/                     All private packages (proxy, console tickets, circuit breaker).

compute-service/                Compute engine lifecycle manager.
  README.md                     What it does. How to start.
  ARCHITECTURE.md               HTTP API + reconcile + reaper + backup planes.
  API.md                        Endpoints, state machine, errors.
  FILES.md                      Core files, one line.
  cmd/server/                   Binary entrypoint.
  internal/                     All private packages (backup, instancetype, reconcile).
  migrations/                   SQL migrations (goose).
  images/compute-os/            Customer instance boot images.

database-service/               Managed database lifecycle manager.
  README.md                     What it does. How to start.
  ARCHITECTURE.md               HTTP API + CNPG reconcile worker.
  API.md                        Endpoints, SQL execution, import, errors.
  FILES.md                      Core files, one line.
  cmd/server/                   Binary entrypoint.
  internal/                     All private packages.
  migrations/                   SQL migrations (goose).

frontend/                       React SPA dashboard.
  README.md                     What it does. Dev vs prod mode.
  ARCHITECTURE.md               Startup, OIDC, Axios, MSW, runtime config.
  API.md                        API calls made, WebSocket, browser events.
  FILES.md                      Core files, one line.
  src/                          React source: app, features, components, lib, store.
  public/                       Static assets including runtime config.js placeholder.
  package.json                  Dependencies and npm scripts.

iam-service/                    Identity and access management service.
  README.md                     What it does. How to start.
  ARCHITECTURE.md               Postgres-centric design, authz, Authentik sync.
  API.md                        Endpoints, permissions, quotas, audit, errors.
  FILES.md                      Core files, one line.
  cmd/server/                   Binary entrypoint.
  internal/                     All private packages.
  migrations/                   SQL migrations (goose).

k3s-manifests/                  GitOps source of truth for the cluster.
  README.md                     What it does. How ArgoCD picks it up.
  APPS.md                       What each app does and its key config.
  CHARTS.md                     Chart layout and helm-unittest contract.
  ARCHITECTURE.md               Sync order, traffic flow, secret flow, storage.
  FILES.md                      Core files, one line.
  infrastructure/               Platform tools: ingress, TLS, storage, secrets, observability.
  applications/                 FCI service ArgoCD Applications + Helm charts.

platform-common/                Shared Go library for all backend services.
  README.md                     What packages it provides.
  ARCHITECTURE.md               Package roles and design rationale.
  API.md                        Exported types and functions.
  FILES.md                      Core files, one line.
  auth/                         OIDC validation, internal JWT mint/verify. No API-key hash.
  cache/                        Valkey client wrapper.
  config/                       Config loading from environment.
  httpx/                        HTTP server, middleware, error envelope.
  obs/                          OpenTelemetry, structured logging.
  storage/                      Postgres pool + goose. Not S3.
  testing/                      Testcontainers, HTTP contracts, auth fixtures.

storage-service/                Object storage + virtual networks (Garage).
  README.md                     What it does. How to start.
  ARCHITECTURE.md               Prefix isolation, networks, NetworkPolicy, usage collection.
  API.md                        Buckets, objects, networks, firewall rules, errors.
  FILES.md                      Core files, one line.
  cmd/server/                   Binary entrypoint.
  internal/                     All private packages (`objectstore`, `prefix`, reconcile).
  migrations/                   SQL migrations (goose).

terminal-gateway/               WebSocket-to-Kubernetes exec terminal proxy.
  README.md                     What it does. How to start.
  ARCHITECTURE.md               Connection lifecycle, ticket flow, session limits.
  API.md                        WebSocket endpoint, frame protocol, session limits.
  FILES.md                      Core files, one line.
  cmd/server/                   Binary entrypoint.
  internal/                     All private packages. No migrations. No Postgres.

terraform-cloudflare-infra/     Cloudflare DNS + one Zero Trust tunnel. Not the daemon.
  README.md                     What Terraform does. Token env. Local backend warning.
  FILES.md                      Core `.tf` files, one line.
  main.tf                       Root `@` + per-service CNAME records. Point at tunnel.
  tunnel.tf                     Tunnel `freecloud-k3s-tunnel`, secret, ingress, catch-all 404.
  variables.tf                  account_id, zone_id, domain, traefik_service, services map.
  providers.tf                  Cloudflare + random. Token from `CLOUDFLARE_API_TOKEN`.
  backend.tf                    Local backend. No remote backend in this file.
  outputs.tf                    Hostnames, tunnel_id, tunnel_cname, tunnel_token.
```
