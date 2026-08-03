# ☁️ Free Cloud Initiative

[![Documentation](https://img.shields.io/badge/docs-MkDocs%20Material-blue.svg)](docs/)
[![Terraform](https://img.shields.io/badge/IaC-Terraform-purple.svg)](terraform-automation/)
[![Ansible](https://img.shields.io/badge/Automation-Ansible-red.svg)](ansible-automation/)
[![Kubernetes](https://img.shields.io/badge/Orchestration-K3s-326CE5.svg)](k3s-manifests/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Empowering Cloud Native Engineers through Open Infrastructure and Hands-on DevOps.**

---

## 🎯 About The Project

**Free Cloud Initiative** is an open-source, hands-on cloud platform project designed to demonstrate production-grade **DevOps methodologies**, **Infrastructure as Code (IaC)**, **Configuration Management**, and **Cloud Architecture** using lightweight Kubernetes (**K3s**).

This project serves a dual purpose:
1. **DevOps Showcase**: Demonstrating end-to-end automation, cloud infrastructure provisioning, and container orchestration across multi-cloud environments (GCP, Azure, AWS).
2. **Educational Hub**: Providing a clear, step-by-step learning resource to teach developers, sysadmins, and aspiring DevOps engineers how to build, secure, and operate a self-hosted cloud infrastructure from scratch.

---

## 🏗️ Architecture & Core Components

The repository is modularly structured into dedicated domains:

```
freecloudinitiative/
├── 🌐 terraform-automation/  # Multi-cloud IaC provisioning (GCP, Azure, AWS)
├── 🔧 ansible-automation/    # System configuration, security, & K3s node bootstrapping
├── ☸️ k3s-manifests/          # Kubernetes architecture, infrastructure & app manifests
└── 📚 docs/                  # MkDocs project source & technical documentation
```

### 1. 🌐 Infrastructure as Code (Terraform)
Located in [`terraform-automation/`](../terraform-automation/):
* Declarative resource provisioning across **Google Cloud Platform (GCP)**, **Microsoft Azure**, and **Amazon Web Services (AWS)**.
* Automated compute instances, virtual private networks (VPCs), firewalls, and storage management.

### 2. 🔧 Configuration Management & Orchestration (Ansible)
Located in [`ansible-automation/`](../ansible-automation/):
* Automated node bootstrapping, SSH hardening, firewall management (`ufw`), and dependency installation.
* Automated K3s cluster deployment (control plane and worker nodes setup).
* Operational playbooks for system maintenance, thermal monitoring, and automated cluster teardown/re-bootstrapping.

### 3. ☸️ K3s Kubernetes Architecture & Manifests
Located in [`k3s-manifests/`](../k3s-manifests/):
* **Infrastructure Stack**:
  * **Ingress & TLS**: Traefik, Cert-Manager with Let's Encrypt integration.
  * **Observability & Monitoring**: Prometheus, Grafana, OpenTelemetry, Loki, Tempo, and Grafana Alloy.
  * **Security & Governance**: Kyverno policy engine.
  * **Artifact Registry**: Harbor self-hosted registry setup.
* **GitOps & App Deployment**: Application manifests organized into infrastructure and user application layers.

### 4. 📚 Documentation (MkDocs)
Located in [`docs/`](../docs/):
* Complete technical documentation, setup guides, and architectural decisions built with **MkDocs** and the **Material for MkDocs** theme.

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
* How to write clean, modular Terraform code for cloud provisioning.
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