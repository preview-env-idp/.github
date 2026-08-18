# ADR 006: Data Protection via Secrets Encryption at Rest in K3s

## Status

Accepted

## Context

The Alcambic platform utilizes K3s as its Kubernetes distribution, which by default uses an embedded SQLite database instead of etcd to store cluster state.

By default, Kubernetes (and K3s) stores `Secret` objects in this database as plaintext (Base64 encoded, which provides no cryptographic security). If a malicious actor gains unauthorized access to the underlying hypervisor storage (e.g., via a compromised LXC container escape, physical theft of the NVMe drive, or unauthorized snapshotting), they can trivially extract all database passwords, TLS certificates, and API tokens belonging to all tenant environments.

## Decision

We will explicitly enable the Secrets Encryption at Rest feature during the K3s cluster bootstrap phase by passing the `--secrets-encryption` flag to the K3s server.

This configures the Kubernetes API Server to perform symmetric encryption (e.g., AES-CBC or AES-GCM) on all `Secret` resources *before* persisting them to the SQLite datastore.

## Consequences

* **Positive (Security):** Mitigates the risk of sensitive data exfiltration in the event of bare-metal host storage compromise or offline data extraction.
* **Negative (Performance):** Introduces minor CPU overhead on the API Server due to cryptographic operations during Secret creation/retrieval (though heavily mitigated by AES-NI hardware acceleration on the host CPU).
* **Negative (Operations):** Introduces key management overhead. In a disaster recovery scenario, the cluster state cannot be restored from a SQLite backup without the original encryption keys stored on the host filesystem.
