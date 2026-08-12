# ADR 004: Enable Transparent Encryption (mTLS) via Cilium WireGuard for East-West Traffic

## Status

Accepted

## Context

While Edge Authentication (Cloudflare Tunnels) secures North-South traffic entering the cluster, East-West traffic (pod-to-pod communication) remains unencrypted by default. In a strict Zero Trust architecture, the internal network must be treated as hostile. 

If a tenant's ephemeral environment is compromised via a vulnerability (e.g., RCE) in the deployed application, an attacker could potentially escalate the attack by eavesdropping on internal network interfaces (packet sniffing). Unencrypted HTTP traffic between the `cloudflared` daemon and the Ingress Controller, or between internal microservices, could expose sensitive metadata or payload data.

Standard Mutual TLS (mTLS) implementations typically require complex Certificate Authority (CA) management and sidecar proxies (e.g., Istio, Linkerd) injected into every pod, increasing resource overhead and operational complexity.

## Decision

We will enforce transparent encryption for all East-West traffic at the kernel level by enabling Cilium's WireGuard integration (`encryption.type=wireguard`). 

Using eBPF, Cilium will automatically intercept and encrypt packets leaving the pod's virtual network interface before they traverse the host's network stack, and decrypt them immediately before delivery to the destination pod.

### Implementation Strategy

The activation of WireGuard encryption will be deferred until the post-bootstrap phase (Pre-MVP). This phased approach ensures that core L3/L4 network routing can be successfully verified and debugged using standard network tools, avoiding the observability penalties introduced by kernel-level encryption during the initial cluster setup.

## Consequences

* **Positive (Security):** Achieves Data-in-Transit encryption across the entire cluster, nullifying internal network eavesdropping vectors and heavily reducing the Blast Radius.
* **Positive (Operations):** Encryption is strictly transparent. It requires zero modifications to application code, Kubernetes manifests, or deployment pipelines.
* **Negative (Performance):** Introduces CPU overhead due to continuous cryptographic operations (encryption/decryption cycles), though partially mitigated by the host CPU's hardware acceleration (AES-NI).
* **Negative (Observability):** Complicates low-level network troubleshooting. Traditional packet analyzers (`tcpdump`, `wireshark`) on the host will only capture encrypted UDP cipher text. All L3/L4/L7 network visibility will strictly rely on Cilium's native observability tool (`hubble`).
