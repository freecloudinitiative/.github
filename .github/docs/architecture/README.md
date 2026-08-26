# FreeCloud Initiative

## What Code Do

Self-hosted cloud platform on bare-metal Kubernetes. Users get:

- **Compute** — managed engines (containers; optional Kata `dedicated`)
- **Databases** — managed PostgreSQL clusters (CloudNativePG)
- **Object storage** — S3-compatible buckets (Garage). Prefix `acct/<accountID>/<bucketID>`
- **Networks** — VPCs and firewall rules → Kubernetes NetworkPolicy
- **Terminal** — browser shell into a running engine
- **Identity and access** — accounts, API keys, users, quotas, policies

Dashboard or HTTP API. Cluster is K3s on Raspberry Pi (arm64). `ansible-automation` installs K3s and Argo CD. Helm charts live in `k3s-manifests`, not in service repos.

## Why Need It

Managed clouds cost money and require third-party trust. FCI keeps compute, storage, and networking on owned hardware, with a self-hosted API and dashboard.

## How Start

**Prerequisites**: bare-metal nodes (Raspberry Pi or similar), this workspace cloned.

**Step 1 — Bootstrap the cluster**

```bash
cd ansible-automation
ansible-playbook playbook.yml --ask-vault-pass
```

Installs K3s, Argo CD, `root-app`. OpenBao seed waits until OpenBao is initialized, unsealed, and `OPENBAO_BOOTSTRAP_TOKEN` is set. See [ansible-automation/README.md](ansible-automation/README.md).

**Step 2 — Argo CD takes over**

`root-app` syncs `k3s-manifests`. Infrastructure (Traefik, cert-manager, Longhorn, Garage, CloudNativePG, Postgres, Valkey, Authentik, OpenBao, Zot, monitoring) and application charts deploy from Git.

**Step 3 — Develop locally (optional)**

Each Go service has `.env.example`. `go.work` lists the seven Go modules. Root `go build ./...` fails — prefix `.` is not a module. Build a module path or `cd` in:

```bash
# From workspace root
go build ./api-gateway/...

# Per service — api-gateway entrypoint is cmd/gateway
cd api-gateway
cp .env.example .env
go run ./cmd/gateway

# Other Go services
cd ../compute-service
cp .env.example .env
go run ./cmd/server
```

Frontend:

```bash
cd frontend
npm install
npm run dev   # MSW mock API — no backend needed
```

## Language

| Repo                         | Language                                |
| ---------------------------- | --------------------------------------- |
| `ansible-automation`         | Ansible YAML                            |
| `api-gateway`                | Go                                      |
| `compute-service`            | Go                                      |
| `database-service`           | Go                                      |
| `iam-service`                | Go                                      |
| `platform-common`            | Go (shared library)                     |
| `storage-service`            | Go                                      |
| `terminal-gateway`           | Go                                      |
| `frontend`                   | TypeScript / React                      |
| `k3s-manifests`              | YAML (Helm charts, ArgoCD Applications) |
| `terraform-cloudflare-infra` | Terraform HCL                           |

`go.work` at workspace root. Modules: the seven Go dirs above. Go 1.26.6.

## Folders

```
ansible-automation/     K3s + Argo CD bootstrap (`playbook.yml`). Seeds OpenBao. Then stops.
api-gateway/            HTTP reverse proxy and auth. Mints internal JWT and console tickets.
compute-service/        Engine lifecycle. Reconciler, reaper, backups. exec-target + namespace API.
database-service/       CNPG database lifecycle. SQL + import. Optional Barman backups.
frontend/               React SPA. TUI aesthetic. HTTP and /ws/ via api-gateway.
iam-service/            Accounts, API keys, users, quotas, audit. Authentik sync.
k3s-manifests/          GitOps. Infrastructure + application Helm charts.
platform-common/        Shared library: auth, cache, config, httpx, obs, storage (Postgres), testing.
storage-service/        Buckets (Garage) + networks/firewall. Backup-bucket for compute.
terminal-gateway/       WebSocket → pods/exec. Redeems api-gateway tickets. No Postgres.
terraform-cloudflare-infra/ Cloudflare DNS + Zero Trust tunnel. Not the daemon.
```

## Read More

Workspace: [ROLES](ROLES.md) · [FILES](FILES.md) · [caveman](caveman.md)

| Repo                         | Docs                                                                                                                                                                                    |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ansible-automation`         | [README](ansible-automation/README.md) · [ROLES](ansible-automation/ROLES.md) · [FILES](ansible-automation/FILES.md)                                                                    |
| `api-gateway`                | [README](api-gateway/README.md) · [ARCHITECTURE](api-gateway/ARCHITECTURE.md) · [API](api-gateway/API.md) · [FILES](api-gateway/FILES.md)                                               |
| `compute-service`            | [README](compute-service/README.md) · [ARCHITECTURE](compute-service/ARCHITECTURE.md) · [API](compute-service/API.md) · [FILES](compute-service/FILES.md)                               |
| `database-service`           | [README](database-service/README.md) · [ARCHITECTURE](database-service/ARCHITECTURE.md) · [API](database-service/API.md) · [FILES](database-service/FILES.md)                           |
| `frontend`                   | [README](frontend/README.md) · [ARCHITECTURE](frontend/ARCHITECTURE.md) · [API](frontend/API.md) · [FILES](frontend/FILES.md)                                                           |
| `iam-service`                | [README](iam-service/README.md) · [ARCHITECTURE](iam-service/ARCHITECTURE.md) · [API](iam-service/API.md) · [FILES](iam-service/FILES.md)                                               |
| `k3s-manifests`              | [README](k3s-manifests/README.md) · [APPS](k3s-manifests/APPS.md) · [CHARTS](k3s-manifests/CHARTS.md) · [ARCHITECTURE](k3s-manifests/ARCHITECTURE.md) · [FILES](k3s-manifests/FILES.md) |
| `platform-common`            | [README](platform-common/README.md) · [ARCHITECTURE](platform-common/ARCHITECTURE.md) · [API](platform-common/API.md) · [FILES](platform-common/FILES.md)                               |
| `storage-service`            | [README](storage-service/README.md) · [ARCHITECTURE](storage-service/ARCHITECTURE.md) · [API](storage-service/API.md) · [FILES](storage-service/FILES.md)                               |
| `terminal-gateway`           | [README](terminal-gateway/README.md) · [ARCHITECTURE](terminal-gateway/ARCHITECTURE.md) · [API](terminal-gateway/API.md) · [FILES](terminal-gateway/FILES.md)                           |
| `terraform-cloudflare-infra` | [README](terraform-cloudflare-infra/README.md) · [FILES](terraform-cloudflare-infra/FILES.md)                                                                                           |
