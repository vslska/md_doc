# 🧠 DevOps & Infrastructure Engineering Knowledge Base

Welcome to my personal, production-grade knowledge base. This repository is a meticulously structured compilation of core systems theory, OS internals, container orchestration platforms, and Infrastructure-as-Code (IaC) blueprints. It is built and dynamically updated from real-world enterprise engineering challenges (5+ years of infrastructure experience) and intensive self-directed deep dives.

---

## 🗂️ Detailed Repository Architecture

### 🔬 [0.scnc](./0.scnc) — Computer Science & Distributed Systems
Theoretical foundations for designing scalable, resilient, and fault-tolerant architectures.
* **`lng/`** — Paradigms of concurrency, parallelism, and system scalability primitives.
* **`Распределенные системы/` (Distributed Systems)** — Data consistency and network trade-offs:
  * Theoretical trade-offs via **CAP** and **PACELC** theorems.
  * Distributed consensus algorithms: Detailed analysis of **Paxos**, **Raft**, and **PBFT** (Practical Byzantine Fault Tolerance).
  * System fault models: The Byzantine Generals Problem, Sybil-resistance, and crypto-economic consensus models.

### 🐧 [1.os](./1.os) — Operating Systems Internals & Administration
Bare-metal, virtualized, and cloud OS administration, debugging, and advanced scripting.
* **`1.lin7318/1.sys/` (Linux Internals & System Management)**
  * **Storage Engineers Realm:** Comprehensive **LVM (Logical Volume Manager)** guides, partition creation, structural resizing, and dynamic volume mounting on VMs.
  * **Kernel & Process Deep Dives:** Lifecycle management, resource profiling (`atop`), and **OOM Killer (Out of Memory)** optimization/behavior tuning.
  * **Networking & Access Control:** Advanced networking using `nmcli`, `SSH` infrastructure hardening, and POSIX access rights.
  * **Disaster Recovery:** Live kernel recovery mechanics (`grub-init`), package management troubleshooting (fixing locked `dpkg` states), and **Astra Linux** restoration.
* **`1.lin7318/2.shell/` (Automation, Regular Expressions & Text Processing)**
  * Command-line efficiency, stream redirection, and programmatic text formatting (Heredoc/`EOF` pipelines).
  * Advanced stream editing using **`sed`** (including Backreferences) and **`awk`** text processing patterns.
  * Multi-processing commands with `xargs` and regex patterns.
* **`2.win0x01/2.pwrshll/` (Enterprise Windows Automation)**
  * Storage and Windows Service management via PowerShell.
  * Distributed administration utilizing `Enter-PSSession` and cloud patch deployment via SCCM/PowerShell.

### 📦 [2.contnr.orchstrtn](./2.contnr.orchstrtn) — Containers & Orchestration Engine
Containerization lifecycle, structural image optimizations, and Kubernetes engineering.
* **`1.docker/` (Container Mechanics & Image Building)**
  * Deep-dive architectural path from Docker CLI down to **`runC`** container runtime.
  * Linux kernel isolation primitives: **`cgroups`** and **`namespaces`** underlying mechanics.
  * Advanced build engines: **Docker BuildKit**, layer optimizations, and secure **multi-stage build** architectures.
* **`3.k8s/` (Production-Grade Kubernetes Architecture)**
  * **Cluster Internals:** Detailed control-plane communications, Node mechanics, Controllers, and Garbage Collection.
  * **Workloads & Pod Lifecycle:** Advanced Pod configurations, Init/Sidecar/Ephemeral containers, Probes (Liveness/Readiness/Startup), and QoS classes.
  * **Workload Controllers:** Deployments, StatefulSets, DaemonSets, Jobs, CronJobs, and PDB (Pod Disruption Budgets).
  * **Networking & Storage:** Ingress routing, Services, NetworkPolicies, Gateway API, PersistentVolumes (PV/PVC), and StorageClasses.
  * **Configuration & Security:** Secure ConfigMaps, Secrets management best practices, **RBAC**, PSA (Pod Security Admission), and PSS.
  * **Scheduling & Scale:** Node affinity, Taints/Tolerations, Pod topology spread constraints, and auto-scaling mechanisms (**HPA / VPA**).

### 🛠️ [3.iac](./3.iac) — Infrastructure as Code (IaC) & Configuration Management
Declarative configuration and repeatable cloud provisioning workflows.
* **`1.ansible/` (Configuration Management & IaaS Orchestration)**
  * Infrastructure playbooks, directory design patterns, inventory management, and loop control structures (`until`, `when`, `blocks`).
  * Dynamic template generation using **Jinja2** filters and structural Ansible Roles.
  * Real-world playbooks for cross-platform fleet updates, automated storage provisioning, and OS tuning.
* **`2.terra_opentf/` (Declarative Cloud Provisioning)**
  * Terraform/OpenTFU philosophy, core workflows (`init`, `plan`, `apply`, `destroy`), and state engineering.
  * HCL syntax features: Advanced variable typing, local values, expressions (`for` loops), dynamic blocks, and explicit resource dependencies (`depends_on`).

### ☁️ [4.cloud](./4.cloud) & [5.git](./5.git) — Cloud Infrastructure & Version Control
* **`1.cloudinit/` & `2.yandex/`:** Infrastructure bootstrapping with **Cloud-Init** scripting. Cloud resource deployment via **Yandex Cloud CLI (YC)** and secure IAM/SSH key management.
* **`5.git/`:** Advanced branching git trees, merge-conflict troubleshooting, and clean `.gitignore` structural strategies.

---

## 🛠️ Automated Keyword Index (For Recruiters & ATS Systems)
`Linux (RHEL/Debian/Arch/Astra)` • `Distributed Systems (Raft/Paxos)` • `Kubernetes (k3s/Minikube)` • `Docker (BuildKit/Multi-stage)` • `Ansible Automation` • `Terraform (HCL)` • `Yandex Cloud (YC)` • `Cloud-Init` • `Bash Scripting (Sed/Awk)` • `PowerShell & SCCM` • `LVM & File Systems` • `Kernel (OOM Killer)` • `Network Security (SSH/RBAC)`

---
*Maintained by [@vslska](https://github.com) • Documented beautifully with Obsidian 🚀*
