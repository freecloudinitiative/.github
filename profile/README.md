# ☁️ Free Cloud Initiative

[![Documentation](https://img.shields.io/badge/docs-MkDocs%20Material-blue.svg)](https://github.com/freecloudinitiative/docs)
[![Frontend Console](https://img.shields.io/badge/Console-React%2019%20%7C%20TypeScript-61DAFB.svg)](https://github.com/freecloudinitiative/frontend)
[![Multi-Cloud IaC](https://img.shields.io/badge/IaC-Multi--Cloud%20Terraform-purple.svg)](https://github.com/freecloudinitiative/terraform-multicloud-infra)
[![Cloudflare IaC](https://img.shields.io/badge/IaC-Cloudflare%20Terraform-orange.svg)](https://github.com/freecloudinitiative/terraform-cloudflare-infra)
[![Ansible](https://img.shields.io/badge/Automation-Ansible-red.svg)](https://github.com/freecloudinitiative/ansible-automation)
[![Kubernetes](https://img.shields.io/badge/Orchestration-K3s-326CE5.svg)](https://github.com/freecloudinitiative/k3s-manifests)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Empowering Cloud Native Engineers through Open Infrastructure and Hands-on DevOps.**

---

## 🎯 About The Project

**Free Cloud Initiative** is an open-source, hands-on cloud platform project designed to demonstrate production-grade **DevOps methodologies**, **Infrastructure as Code (IaC)**, **Configuration Management**, and **Cloud Architecture** using lightweight Kubernetes (**K3s**).

This project serves a dual purpose:
1. **DevOps Showcase**: Demonstrating end-to-end automation, cloud infrastructure provisioning, and container orchestration across multi-cloud environments (GCP, Azure, AWS) and edge services (Cloudflare).
2. **Educational Hub**: Providing a clear, step-by-step learning resource to teach developers, sysadmins, and aspiring DevOps engineers how to build, secure, and operate a self-hosted cloud infrastructure from scratch.

---

## 🏗️ Architecture & Core Components

The repository ecosystem is modularly structured into dedicated domains:

```
freecloudinitiative/
├── 🌐 terraform-multicloud-infra/ # On-demand multi-cloud IaC (GCP, Azure, AWS) [FinOps strategy]
├── ☁️ terraform-cloudflare-infra/ # Always-on Cloudflare DNS & edge security via Terraform Cloud
├── 🔧 ansible-automation/        # System configuration, security, & K3s node bootstrapping
├── ☸️ k3s-manifests/              # Kubernetes architecture, infrastructure & app manifests
├── 🖥️ frontend/                   # Web Console SPA (React 19, TypeScript, Vite)
├── 📚 docs/                      # MkDocs project source & technical documentation
│
│   # 🔒 Backend Microservices (In Active Development)
├── 🚪 api-gateway/               # Central API gateway, AuthN & rate-limiting proxy
├── 🔐 iam-service/               # Identity & Access Management, multi-tenant RBAC
├── ⚡ compute-service/           # Compute instance lifecycle & Kubernetes reconciliation
├── 🗄️ database-service/          # CloudNativePG database orchestration & SQL management
├── 📦 storage-service/           # Object storage & virtual network management
├── 💻 terminal-gateway/          # Secure WebSocket terminal proxy for pod execution
└── 🧩 platform-common/           # Shared Go libraries, models & telemetry primitives
```

### 1. 🌐 Multi-Cloud Infrastructure as Code (Terraform)
Located in [`terraform-multicloud-infra`](https://github.com/freecloudinitiative/terraform-multicloud-infra):
* **On-Demand & FinOps Driven**: Provision compute instances, VPCs, firewalls, and storage across **Google Cloud Platform (GCP)**, **Microsoft Azure**, and **Amazon Web Services (AWS)** strictly when needed to optimize cloud costs.
* Allows spin-up/spin-down of cluster nodes on demand without affecting persistent domain services.

### 2. ☁️ Cloudflare Infrastructure as Code (Terraform)
Located in [`terraform-cloudflare-infra`](https://github.com/freecloudinitiative/terraform-cloudflare-infra):
* **Always-On Continuous Management**: Powered by **Terraform Cloud** for continuous automation and state management.
* Keeps DNS records, domain routing, SSL/TLS certificates, and edge security settings active and up-to-date independently of cloud server lifecycles.

### 3. 🔧 Configuration Management & Orchestration (Ansible)
Located in [`ansible-automation`](https://github.com/freecloudinitiative/ansible-automation):
* Automated node bootstrapping, SSH hardening, firewall management (`ufw`), and dependency installation.
* Automated K3s cluster deployment (control plane and worker nodes setup).
* Operational playbooks for system maintenance, thermal monitoring, and automated cluster teardown/re-bootstrapping.

### 4. ☸️ K3s Kubernetes Architecture & Manifests
Located in [`k3s-manifests`](https://github.com/freecloudinitiative/k3s-manifests):
* **Infrastructure Stack**:
  * **Ingress & TLS**: Traefik, Cert-Manager with Let's Encrypt integration.
  * **Observability & Monitoring**: Prometheus, Grafana, OpenTelemetry, Loki, Tempo, and Grafana Alloy.
  * **Security & Governance**: Kyverno policy engine.
  * **Artifact Registry**: Harbor self-hosted registry setup.
* **GitOps & App Deployment**: Application manifests organized into infrastructure and user application layers.

### 5. 🖥️ Web Console (React & TypeScript)
Located in [`frontend`](https://github.com/freecloudinitiative/frontend):
* **Modern Cloud Management SPA**: Built with **React 19**, **TypeScript**, **Vite**, and **TanStack Query**.
* Features a sleek terminal-aesthetic user interface, interactive command palette, real-time terminal emulator sessions, SQL query editor, and multi-tenant resource management.

### 6. 📚 Documentation (MkDocs)
Located in [`docs`](https://github.com/freecloudinitiative/docs):
* Complete technical documentation, setup guides, and architectural decisions built with **MkDocs** and the **Material for MkDocs** theme.

### 7. 🚧 Backend Microservices *(In Active Development)*
> *Note: The following backend services are currently in internal development and will be released as they mature.*

* **🚪 api-gateway**: Central API gateway handling external routing, token validation (OIDC/JWT), tenant resolution, and account rate limiting.
* **🔐 iam-service**: Identity and Access Management service controlling accounts, users, policies, roles, API key issuing/verification, and audit trails.
* **⚡ compute-service**: Compute Engine orchestration managing virtual compute lifecycles, Kata container runtime isolation, and disk backups.
* **🗄️ database-service**: Managed PostgreSQL database orchestration via CloudNativePG (CNPG) operator, query execution, and data import tooling.
* **📦 storage-service**: Object storage bucket/object lifecycle management and software-defined network/firewall controls.
* **💻 terminal-gateway**: Lightweight, high-security WebSocket gateway enabling interactive browser-based shell access (`pods/exec`) to workloads.
* **🧩 platform-common**: Foundational Go library providing standard data models, error envelopes, logging, database drivers, and telemetry middleware.

---

## 🚀 Getting Started

### Prerequisites
Make sure you have the following installed on your control machine:
* [Terraform](https://www.terraform.io/downloads) (>= 1.5.0)
* [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) (>= 2.15.0)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [Python 3](https://www.python.org/) & `pip` (for MkDocs documentation site)

---

### 📖 Running Documentation Locally

To preview the documentation website locally using MkDocs:

1. Navigate to the documentation directory:
   ```bash
   cd docs
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Start the live preview server:
   ```bash
   mkdocs serve
   ```
4. Open `http://127.0.0.1:8000` in your browser.

---

## 🎓 Educational Goals & Learning Outcomes

By exploring and utilizing this repository, you will learn:
* How to implement **FinOps strategies** by decoupling on-demand cloud infrastructure provisioning from always-on Cloudflare edge services using **Terraform Cloud**.
* How to structure reusable Ansible roles for server configuration and Kubernetes cluster bootstrapping.
* How to design a cloud-native Kubernetes architecture with full observability (metrics, logs, traces).
* How to adopt GitOps and declarative manifest management for cluster components.
* How to maintain production-ready project documentation using MkDocs.

---

## 🤝 Contributing & Feedback

Contributions, suggestions, and feedback are welcome! Whether you are fixing a typo, adding a new Ansible role, or expanding the documentation to help others learn, feel free to open an issue or submit a pull request.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.