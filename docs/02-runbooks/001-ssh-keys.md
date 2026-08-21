# Runbook 001: Proxmox SSH Bootstrap & Agent Configuration

## 1. Objective

Establish an initial, secure SSH connection to the bare-metal Proxmox hypervisor using a passphrase-protected Ed25519 automation key. This key is temporarily injected into the default `root` account. During the automated Ansible Day-0 bootstrapping phase, this identity will be transferred to a dedicated operations account (`alcambic-admin`), and direct `root` SSH access will be permanently cryptographically severed.

## 2. Prerequisites

- Target Proxmox node static IP address accessible via management LAN (`vmbr0`).
- Target Proxmox initial `root` password.
- Operator workstation with `ssh-agent` running.

## 3. Execution Steps

### 3.1. Generate Hardened Automation Key

Create the persistent identity for the CI/CD works.
**CRITICAL:** You MUST provide a strong passphrase when prompted.

```bash
ssh-keygen -t ed25519 -a 100 -C "alcambic-ci-cd-key" -f ~/.ssh/id_ed25519_alcambic
```

### 3.2. Load Key into SSH Agent

To prevent deadlocks (waiting for standard input during parallel executions) during use of external applications (like Ansible), decrypt the key and load it into the local SSH agent memory.

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_alcambic
```

### 3.3. Inject Key to Hypervisor

Push the public key to the Proxmox `root` account temporarily. Enter the hypervisor's `root` password when prompted.

```bash
export PROXMOX_IP="<INSERT_PROXMOX_IP_HERE>"
ssh-copy-id -i ~/.ssh/id_ed25519_alcambic.pub root@${PROXMOX_IP}
```

### 3.4. Verify Passwordless Execution

Confirm the key is accepted via the agent without a password prompt.

```bash
ssh -i ~/.ssh/id_ed25519_alcambic root@${PROXMOX_IP} "echo 'SSH connection successful.'"
```

## 4. Next steps

- **Target Repository:** `infra-core-iac`
- **Next Action:** Will be pushed soon.
