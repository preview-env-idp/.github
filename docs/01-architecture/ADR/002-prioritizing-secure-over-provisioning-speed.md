# ADR 002: Prioritizing "Secure by Default" and Blast Radius Isolation over Provisioning Speed

## Status

Accepted

## Context

The Alcambic platform provisions ephemeral environments (namespaces) triggered by Pull Requests. A fundamental architectural tension exists between two paradigms:

1. Optimizing for the lowest possible CI/CD feedback loop latency and minimal resource overhead.
2. Optimizing for Zero Trust, preventing container escapes, and minimizing the blast radius of a compromised ephemeral workload.

Since the Git repository acts as the Single Source of Truth in our GitOps pipeline (ArgoCD), it is inherently exposed as an attack surface. A malicious or negligent developer could inject a workload manifest with elevated privileges (e.g., `securityContext.privileged: true`, `hostNetwork: true`, or missing resource quotas). If the orchestrator blindly applies these manifests to prioritize speed, a compromised container leads directly to node-level (Hypervisor) compromise.

## Decision

We choose to enforce a strict "Secure by Default" and Policy-as-Code architecture, even at the cost of control plane latency and increased computational overhead.

Specifically, we will:

1. Implement Validating Admission Webhooks (via Kyverno) to intercept and evaluate every API request before it is persisted to the state store.
2. Reject any workload manifest that attempts to escalate privileges, mount host paths, or lacks explicitly defined CPU/Memory requests and limits.
3. Enforce L2/L3 network isolation assuming the workload is hostile.
4. Enforce `Rootless` execution for all tenant application containers, dropping all Linux capabilities by default to prevent privilege escalation.
5. Disable automatic mounting of API credentials (`automountServiceAccountToken: false`) across all tenant namespaces to prevent lateral movement and unauthorized Kubernetes API reconnaissance.

## Consequences

### Positive

* **Security:** Neutralizes the risk of container escape and lateral movement originating from untrusted PR manifests.
* **Stability:** Mandatory resource limits prevent a single ephemeral environment from causing a "Noisy Neighbor" effect or crashing the K3s node via OOM (Out Of Memory).
* **Compliance:** Aligns the platform with modern DevSecOps and SRE Zero Trust standards.

### Negative

* **Performance Penalty:** Evaluating policies adds synchronous network latency (HTTP POST to the webhook) to every Kubernetes API object creation.
* **Resource Overhead:** Running the Policy Engine (Kyverno) requires dedicated control plane resources (CPU/RAM).
* **Friction:** Legitimate workloads requiring specialized privileges will require explicit, documented exemptions (Policy Exceptions).
