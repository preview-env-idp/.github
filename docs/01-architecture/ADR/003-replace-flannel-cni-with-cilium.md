# ADR 003: Replace Default Flannel CNI with Cilium (eBPF) for Kernel-Level Network Isolation

## Status

Accepted

## Context

The project utilizes K3s as the underlying lightweight Kubernetes distribution. By default, K3s bundles Flannel as its Container Network Interface (CNI). Flannel operates as a simple Layer 3 overlay network (typically via VXLAN) and critically lacks native support for standard Kubernetes `NetworkPolicy` resources.

In the context of our ephemeral preview environments, multiple tenant workloads (originating from different Pull Requests) coexist on the same cluster. Using a flat network model implies that a compromised pod in one environment (e.g., via SSRF or RCE) has unrestricted lateral network access to all other environments and internal cluster services. This results in a 100% Blast Radius, invalidating the core security premise of the platform.

## Decision

We will explicitly disable the default Flannel CNI and the built-in network policy controller during K3s initialization by passing the `--flannel-backend=none` and `--disable-network-policy` flags.

In its place, we will deploy Cilium, an eBPF-based CNI. Cilium will be utilized to enforce strict micro-segmentation and a "Default Deny" network posture at the kernel level across all dynamically provisioned namespaces.

## Consequences

* **Positive:** Enables eBPF-based packet filtering, bypassing the inefficient iptables stack.
* **Positive:** Drastically reduces the Blast Radius, directly proving the core thesis of the project regarding Attack Surface minimization.
* **Negative:** Increases the complexity of the cluster bootstrapping process, requiring Cilium to be injected via Helm immediately after K3s starts before nodes can transition to the `Ready` state.
