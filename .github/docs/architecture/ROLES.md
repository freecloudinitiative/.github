# ROLES — What Each Part Does

Each repo is a distinct role in the FCI platform. This document covers what each one owns, what it depends on, and what talks to it.

Helm charts live in `k3s-manifests/applications/<service>/`. Cluster bootstrap lives in `ansible-automation`.

---

## ansible-automation

**Role**: Bare-metal cluster bootstrap. Installs K3s, Argo CD, `root-app`. Stops there.

**Does**:
- `playbook.yml` — three plays. Masters: swap off, `k3s server --cluster-init` (no Traefik, no ServiceLB, embedded registry). Workers join with slurped node-token. First master labels `node-tier`, taints low-memory and masters, installs Argo CD, applies `root-app`.
- `openbao-secrets-init` — after OpenBao is initialized and unsealed, seed KV, Kubernetes auth, External Secrets policy. Needs `OPENBAO_BOOTSTRAP_TOKEN`. Never writes that token into Kubernetes.
- Inventory groups: `masters`, `workers`, `high_memory`, `mid_memory`, `low_memory`. Vault file `group_vars/all/secret.yml`.

**Does not**: install Traefik, cert-manager, OpenBao, or FCI apps. That is `k3s-manifests` after `root-app`. Does not keep running after bootstrap.

**Depends on**: SSH to Pis. `k3s_master1_public_ip` set. Ansible Vault password. OpenBao init/unseal (operator, out of band) before the seed run.

**Talks to**: nodes over SSH. Kubernetes API on first master. OpenBao ClusterIP on the seed run. Applies `root-app` → `https://github.com/freecloudinitiative/k3s-manifests.git`.

**Docs**: [README](ansible-automation/README.md) · [ROLES](ansible-automation/ROLES.md) · [FILES](ansible-automation/FILES.md)

---

## k3s-manifests

**Role**: GitOps source of truth. Declares everything that runs in the cluster, including Helm charts.

**Does**:
- Declares infrastructure: Traefik, cert-manager, Longhorn, MetalLB, Valkey, Garage, CloudNativePG operator, platform-postgresql, Authentik, External Secrets, Kyverno (+ policies), kube-prometheus-stack, Loki, Tempo, Alloy, OpenTelemetry, Cloudflare tunnel, Zot Registry.
- Declares FCI applications as ArgoCD `Application` resources. Chart files sit next to each `app.yaml` (`Chart.yaml`, `values.yaml`, `templates/`).
- ArgoCD syncs this repo continuously. Push deploys. Drift self-heals.

**Does not**: contain application Go/TypeScript. Does not init/unseal OpenBao (operator) or seed KV (that is `ansible-automation` `openbao-secrets-init`).

**Depends on**: Argo CD from `ansible-automation` `playbook.yml`. OpenBao (GitOps chart here; External Secrets reads it after ansible seed).

**Docs**: [README](k3s-manifests/README.md) · [APPS](k3s-manifests/APPS.md) · [CHARTS](k3s-manifests/CHARTS.md) · [ARCHITECTURE](k3s-manifests/ARCHITECTURE.md) · [FILES](k3s-manifests/FILES.md)

---

## platform-common

**Role**: Shared Go library. Building blocks for every FCI backend service.

**Does**:
- `auth` — internal Ed25519 JWT mint/verify, Authentik OIDC validation, `Actor` / `AuditInfo`, HTTP middleware. No API-key hashing.
- `cache` — Valkey client (TLS, fail-open primitives). Rate limit, JWKS cache, idempotency, Pub/Sub.
- `config` — env-var struct load. `Base` most services embed. Fails closed.
- `httpx` — HTTP server, middleware, health probes, uniform error envelope.
- `obs` — `slog`, OpenTelemetry, span helpers.
- `storage` — Postgres pool (`pgx`), schema pin, goose migrations, error map to `httpx`. Not S3.
- `testing` — Testcontainers Postgres/Valkey, HTTP contract asserts, auth fixtures, stub OIDC.

**Does not**: run as a standalone service. Imported as a Go module.

**Depends on**: nothing at runtime. Compile-time dependency only.

**Used by**: `api-gateway`, `compute-service`, `database-service`, `iam-service`, `storage-service`, `terminal-gateway`.

**Docs**: [README](platform-common/README.md) · [ARCHITECTURE](platform-common/ARCHITECTURE.md) · [API](platform-common/API.md) · [FILES](platform-common/FILES.md)

---

## api-gateway

**Role**: Single entry point for all client API traffic. Auth enforcer. Reverse proxy.

**Does**:
- Validates OIDC Bearer tokens (Authentik JWKS) or API keys (via `iam-service`).
- Mints internal Ed25519 JWTs for HTTP upstreams.
- Routes `/api/*` to the correct backend.
- Per-account, per-class rate limiting (Valkey, fixed window).
- `Idempotency-Key` on mutating methods.
- Mints console tickets (`POST /api/console/tickets`) into Valkey. Proxies `/ws/terminal/` to `terminal-gateway` with ticket only — no internal JWT on that path.
- Strips `X-FCI-*` and inbound identity headers. Adds internal JWT on HTTP only.
- Circuit breakers, retries, timeouts per upstream.

**Does not**: implement product business logic. Pure proxy and auth layer.

**Depends on**: Authentik (JWKS), Valkey, internal signing key. All five HTTP/WS backends as upstreams.

**Talks to**: `iam-service`, `compute-service`, `database-service`, `storage-service`, `terminal-gateway`.

**Docs**: [README](api-gateway/README.md) · [ARCHITECTURE](api-gateway/ARCHITECTURE.md) · [API](api-gateway/API.md) · [FILES](api-gateway/FILES.md)

---

## iam-service

**Role**: Identity and access. Owns accounts, users, API keys, quotas, policies, audit.

**Does**:
- Provisions account on first OIDC login (`POST /internal/accounts/resolve`). Advisory lock. Owner user, default quotas, admin policy — one transaction.
- Account settings. IAM users (`admin` / `editor` / `viewer` / `auditor`). Never remove last active admin.
- API keys: Argon2id hash, prefix lookup, plaintext returned once.
- `GET /internal/accounts/{id}/quotas` — compute, storage, api-gateway, database call this.
- Writes audit rows. Accepts `POST /internal/audit` from `terminal-gateway`.
- After user write commits, best-effort Authentik user + group sync. Background Authentik drift reconciler.

**Does not**: issue OIDC tokens (Authentik does). Store plaintext keys after create.

**Depends on**: `platform-postgresql` (`iam` schema). Internal JWT from `api-gateway`. Optional Authentik admin token.

**Talks to**: Authentik admin API (user/group sync). Called by `api-gateway` and other services.

**Docs**: [README](iam-service/README.md) · [ARCHITECTURE](iam-service/ARCHITECTURE.md) · [API](iam-service/API.md) · [FILES](iam-service/FILES.md)

---

## compute-service

**Role**: Compute engine lifecycle. Desired state in Postgres, observed state from Kubernetes.

**Does**:
- HTTP API: CRUD engines, metrics, backup list/get. Wire status is `running` / `stopped` / `pending` only (`ComposeStatus`; observed `failed` maps to `pending`).
- Reconcile worker: queue in Postgres. Per engine: Deployment + Service + PVC. Per namespace: ResourceQuota + LimitRange, RBAC, ImagePullSecrets. Observe pod, write back.
- `instanceType` `shared` | `dedicated` (Kata). Frozen `runtime_class` at create.
- Nightly disk backups when `BACKUP_ENABLED` and `autoBackups`. Restore is `RestoreToPVC` — no HTTP route.
- `POST /internal/accounts/{accountID}/namespace` for `database-service`. `GET /internal/compute-engines/{id}/exec-target` for `terminal-gateway` (separate verifier).
- Namespace reaper (dry-run default). Quota check against `iam-service` (409 `quota_exceeded`).

**Does not**: mint terminal tickets (api-gateway does). Run inside guest VMs.

**Depends on**: `platform-postgresql` (`compute` schema), `iam-service` (quotas), Kubernetes API. Optional Valkey, Prometheus, `storage-service` (backup target). Internal JWT from `api-gateway`.

**Talks to**: `iam-service`, Kubernetes, `storage-service` (backup bucket). Called by `api-gateway`, `terminal-gateway`, `database-service`.

**Docs**: [README](compute-service/README.md) · [ARCHITECTURE](compute-service/ARCHITECTURE.md) · [API](compute-service/API.md) · [FILES](compute-service/FILES.md)

---

## database-service

**Role**: Managed Postgres lifecycle. CloudNativePG `Cluster` per customer database.

**Does**:
- HTTP API: CRUD databases, execute SQL (timeout / row / byte limits), CSV/JSON import, connections, metrics.
- Reconcile worker: server-side apply CNPG `Cluster`, observe, write `database_status`.
- Reads credentials live from Kubernetes Secrets. Never caches. Never stores password in platform Postgres. Isolated pool per database.
- Quota via `iam-service`. Ensures customer namespace via `compute-service` `POST /internal/accounts/{accountID}/namespace`.
- Optional Barman Cloud backups when `BACKUP_ENABLED`.

**Does not**: cache credentials. Share one connection pool across customers.

**Depends on**: `platform-postgresql` (`database` schema), Kubernetes (CNPG), `iam-service` (quotas). Optional `compute-service` + signing key for namespace provision. Internal JWT from `api-gateway`.

**Talks to**: `iam-service`, `compute-service` (namespace), Kubernetes.

**Docs**: [README](database-service/README.md) · [ARCHITECTURE](database-service/ARCHITECTURE.md) · [API](database-service/API.md) · [FILES](database-service/FILES.md)

---

## storage-service

**Role**: Object storage and virtual networks. One Garage bucket; tenant prefix isolation.

**Does**:
- HTTP API: buckets, objects, access-policy rows, networks, firewall rules, usage metrics.
- Prefix isolation: `acct/<accountID>/<bucketID>`. Derived server-side. Key sanitise rejects `../`.
- Bucket kinds: `customer` (default) and `platform-backup`. Customer list hides platform-backup.
- `POST /internal/accounts/{accountID}/backup-bucket` for `compute-service`.
- Usage collector snapshots object count and bytes into Postgres.
- Reconcile worker: firewall rules on **networks** → Kubernetes `NetworkPolicy`.
- Quota via `iam-service`. `/api/*` mounts only when `OBJECT_STORE_ENABLED=true`.

**Does not**: expose Garage to users. Access-policy rows are not enforced on the object store (shared platform credential).

**Depends on**: Garage S3 (`internal/objectstore`, not `platform-common/storage`), `platform-postgresql`, `iam-service` (quotas), Kubernetes (NetworkPolicy). Internal JWT from `api-gateway`.

**Talks to**: Garage, `iam-service`, Kubernetes. Called by `compute-service` for backup target.

**Docs**: [README](storage-service/README.md) · [ARCHITECTURE](storage-service/ARCHITECTURE.md) · [API](storage-service/API.md) · [FILES](storage-service/FILES.md)

---

## terminal-gateway

**Role**: Browser terminal proxy. WebSocket → Kubernetes `pods/exec`.

**Does**:
- `GET /ws/terminal/{id}?ticket=`. Usually reached via api-gateway `/ws/terminal/` proxy.
- Redeems single-use ticket from Valkey (minted by **api-gateway**).
- Authorizes against `compute-service` `GET /internal/compute-engines/{id}/exec-target` with self-minted JWT (issuer `terminal-gateway`).
- Opens `pods/exec` (SPDY). Relays stdin/stdout/resize.
- Per-account session cap (Valkey). Ticket GetDel is fail-closed. Session-cap Acquire fails open.
- `POST /internal/audit` to `iam-service` on session start/end.

**Does not**: validate OIDC. No Postgres. Does not embed `config.Base`. Does not store session beyond the connection.

**Depends on**: Valkey (tickets, session slots), Kubernetes `pods/exec`, `compute-service`, `iam-service` (audit). Internal signing key.

**Talks to**: `compute-service`, `iam-service`, Kubernetes, Valkey.

**Docs**: [README](terminal-gateway/README.md) · [ARCHITECTURE](terminal-gateway/ARCHITECTURE.md) · [API](terminal-gateway/API.md) · [FILES](terminal-gateway/FILES.md)

---

## frontend

**Role**: User-facing dashboard SPA. React, TUI aesthetic.

**Does**:
- OIDC login (`react-oidc-context`, Authentik).
- Compute engines, databases (including active connections), buckets, networks (VPC / firewall), IAM users, account settings.
- In-browser terminal (`xterm.js`). Ticket from `POST /api/console/tickets`. WS URL `/ws/terminal/{id}?ticket=`.
- SQL editor (Monaco) via `api-gateway` → `database-service`.
- `nonprod`: MSW mocks `/api/*`. `prod`: real api-gateway. Runtime config from `/config.js` (mounted, not baked).

**Does not**: talk to compute / iam / storage / database / terminal-gateway by service name. HTTP and (in prod) `/ws/` go through `api-gateway` (nginx same-origin `/ws/` unless `wsBaseUrl` set).

**Depends on**: `api-gateway`, Authentik (OIDC). `terminal-gateway` only as api-gateway WS upstream.

**Docs**: [README](frontend/README.md) · [ARCHITECTURE](frontend/ARCHITECTURE.md) · [API](frontend/API.md) · [FILES](frontend/FILES.md)

---

## terraform-cloudflare-infra

**Role**: Public DNS and Zero Trust tunnel provisioner.

**Does**:
- Creates `freecloud-k3s-tunnel` Zero Trust tunnel.
- Creates DNS CNAME records for root domain and subdomains.
- Configures tunnel ingress rules. Routes to Traefik `websecure` entrypoint.

**Does not**: Run `cloudflared` daemon (`k3s-manifests` does). Terminate HTTP. Commit state.

**Depends on**: Cloudflare API token.

**Talks to**: Cloudflare API.

**Docs**: [README](terraform-cloudflare-infra/README.md) · [FILES](terraform-cloudflare-infra/FILES.md)
