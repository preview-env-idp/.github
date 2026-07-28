# ADR-001: Multi-Repository Taxonomy and Platform Evolution Roadmap

## Status
Accepted

## Context
The **Alcambic** Internal Developer Platform (IDP) is designed to automatically provision ephemeral preview environments per Pull Request. Operating on dedicated bare-metal hardware requires an architectural foundation that balances aggressive FinOps resource optimization, strict security boundaries, GitOps automation, and open-source licensing safety.

While the platform is engineered and maintained as a solo initiative, building a monolithic repository where application code, Kubernetes desired state, and hypervisor provisioning scripts coexist would violate systemic design principles. Furthermore, running ephemeral staging environments in the public cloud (e.g., AWS EKS) introduces prohibitive recurring operating expenses (OpEx) for workloads that only exist for a limited time.

## Decision: Platform Evolution Roadmap
To prevent architectural drift and ensure long-term scalability without constantly rewriting core documentation, the platform is executed across a structured, multi-phase roadmap:

### Phase 1: Core Platform Bootstrap & Containerized Environments
* **Step 1 (Containerized MVP):** Implementing foundational Layer 1 virtualization on bare-metal Proxmox VE. The minimum viable product strictly delivers: isolated DMZ network topologies, host-level declarative firewalls, and a lightweight K3s cluster managed via ArgoCD. Preview environments at this stage focus exclusively on containerized microservice stacks (web applications, ephemeral databases, and routing tunnels triggered by PR events).
* **Step 2 (Roadmap — Full Virtualization & Heavy IaC):** Expanding platform automation beyond lightweight containers to provision complete, ephemeral Virtual Machines (VMs) via **OpenTofu**. This allows testing complex infrastructure-as-code changes, custom Linux provisioning, and Ansible automation scripts inside isolated sandbox VMs per PR, which are automatically destroyed upon merge.

### Phase 2: Control Plane Refactoring (Hexagonal Architecture)
* **Architectural Evolution:** As the custom platform controller (`platform-reaper-bot`) scales to support both K3s namespaces and full Proxmox VMs, its core logic will be refactored into a strict **Hexagonal Architecture (Ports & Adapters)**.
* **Why Hexagonal?** This completely decouples the core domain logic (FinOps TTL rules, environment state machines, Separation of Duties checks) from external infrastructure drivers. Incoming webhooks (GitHub VCS) and outgoing execution adapters (Proxmox API, ArgoCD, Cloudflare DNS) will become interchangeable plugins around an immutable core.

## Decision: Multi-Repository Taxonomy & Blast Radius Containment
To enforce strict boundaries across automated CI/CD loops and prevent privilege escalation, the platform is decomposed into five specialized repositories. Their scope evolves cleanly across the roadmap without violating their core security boundaries:

1. **`.github` (Organization Governance):** Acts as the central control plane for documentation, public profile rendering, and global security policies (`SECURITY.md`).
2. **`infra-core-iac` (Layer 1 - Underlay & VM Provisioning):** Strictly dedicated to physical host configuration, virtualized DMZ network bridges, declarative firewalls, and infrastructure-as-code definitions. 
   * *Phase 1:* Governs Proxmox host hardening and base networking.
   * *Phase 1.2 & Phase 2:* Expands to house **OpenTofu and Ansible** modules for automated, ephemeral Virtual Machine provisioning. Application CI/CD pipelines have zero write access to this tier.
3. **`cluster-state-gitops` (Layer 2 - Desired State Overlay):** The single source of truth for declarative environment configurations and SOPS-encrypted secrets. Completely isolated from physical hypervisor storage pools.
   * *Phase 1:* Houses Kubernetes desired state (ArgoCD manifests and Helm charts).
   * *Phase 2:* Acts as the central GitOps repository for *all* environment types (both container namespaces and OpenTofu-driven VMs), serving as the declarative state target for hexagonal control plane adapters.
4. **`platform-reaper-bot` (Platform Controller & Orchestrator):** Contains the software engineering codebase for the custom lifecycle operator responsible for PR webhook ingestion, TTL calculations, and automated environment decommissioning.
   * *Phase 1:* Operates as a lightweight reconciler for K3s namespaces.
   * *Phase 2:* Refactored into a **Hexagonal Architecture (Ports & Adapters)**, where incoming webhooks (GitHub VCS) drive interchangeable execution adapters (ArgoCD for K8s, Proxmox API / OpenTofu for VMs, Cloudflare DNS).
5. **`sample-tenant-app` (Tenant Reference Workload):** A representative microservice demonstrating the target Developer Experience (DevEx), where developers trigger ephemeral preview environments via standard PR workflows without knowledge of whether the underlying target is a container namespace or a dedicated VM.
## Consequences

### Positive
* **Aggressive FinOps Optimization:** By dynamically provisioning and reclaiming ephemeral resources on owned bare-metal hardware, platform hosting costs are reduced to near zero operational expenditure (OpEx) compared to public cloud equivalents (e.g., AWS EKS or commercial PaaS staging tiers).
* **Architectural Rigor & State Traceability:** Enforcing GitOps workflows and managing infrastructure modifications through structured pull requests establishes strict engineering discipline, reproducible environments, and an immutable audit trail.
* **Future-Proofing:** The explicit decoupling of Layer 1 (OpenTofu underlay) from Layer 2 (ArgoCD overlay) guarantees a seamless transition from Phase 1 container workloads to Phase 2 Hexagonal control plane routing without breaking existing infrastructure.

### Trade-offs
* **Bare-Metal Responsibility:** Unlike managed cloud services, the platform engineer is directly responsible for Linux kernel updates, disk sanitization, storage pool maintenance, and physical resource monitoring.