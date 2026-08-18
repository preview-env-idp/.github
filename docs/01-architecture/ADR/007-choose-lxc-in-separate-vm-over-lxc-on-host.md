# ADR 008: Enforcing Hardware-Level Isolation (KVM) over Shared-Kernel Containers (LXC) for Kubernetes Nodes

## Status

Accepted

## Context

The Alcambic Internal Developer Platform (IDP) provisions ephemeral preview environments using K3s. A fundamental premise of this project's thesis is the aggressive reduction of the Attack Surface and Blast Radius in a Zero Trust architecture.

Initially, Linux Containers (LXC) were considered for hosting the K3s cluster nodes to minimize RAM overhead on the bare-metal Proxmox hypervisor. However, LXC relies on OS-level virtualization (cgroups and namespaces). This means the K3s pods, the Kubelet, and the LXC container itself all share the exact same underlying Linux kernel (Ring 0) with the bare-metal hypervisor.

If a malicious tenant payload or a compromised dependency successfully executes a Container Escape exploit leveraging a kernel vulnerability (e.g., eBPF flaws, Dirty Pipe, or network stack bugs), the attacker does not just break out of the Kubernetes pod—they break out into the Proxmox host's kernel. This results in an immediate root-level compromise of the physical machine, exposing all other isolated tenant environments (a 100% Blast Radius).

## Decision

We mandate the use of full hardware virtualization (KVM/QEMU) leveraging Intel VT-x/AMD-V instructions for all K3s cluster nodes. **Thus use of LXC for Kubernetes nodes is strictly forbidden.**

To implement this without excessive operational overhead (e.g., manual OS installations or maintaining heavy HashiCorp Packer pipelines for the MVP), the `infra-core-iac` layer will utilize OpenTofu to provision these KVM instances directly from official Cloud-Init images.

## Consequences

### Positive (Security & Architecture)

* **Hardware Boundary:** Radically reduces the Blast Radius. A successful kernel-level container escape from a tenant pod now only compromises the virtualized guest OS (the K3s node). To compromise the hypervisor, an attacker would have to chain the container escape with a significantly more complex Virtual Machine Escape (e.g., exploiting QEMU/VirtIO drivers), which elevates the platform's security posture to enterprise cloud standards.
* **Kernel Independence:** Allows the K3s guest OS to run a different kernel version or specific kernel hardening parameters independent of the Proxmox host.

### Negative (Resource Overhead)

* **Memory Tax:** Introduces a strict memory overhead for the guest Operating System. Unlike LXC, which shares the host's memory pages dynamically, each KVM instance requires a dedicated RAM allocation (approximately 250MB–500MB baseline overhead per node just for the guest OS kernel and systemd), marginally reducing the maximum density of ephemeral environments on the single bare-metal node.
* **Boot Latency:** KVM instances have slightly longer boot times compared to LXC, marginally increasing the initial Day-1 cluster bootstrap time (though irrelevant for Day-2 ephemeral pod provisioning).
