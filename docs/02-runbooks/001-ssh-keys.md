# Runbook 001: Proxmox SSH Bootstrap & Agent Configuration

## 1. Objective

Establish an initial, secure SSH connection to the bare-metal Proxmox hypervisor using a passphrase-protected Ed25519 automation key.

## 2. Prerequisites

- Operator workstation with `ssh-agent` running.

## 3. Execution Steps

### 3.1. Generate Hardened Automation Key

Create the persistent identity for the CI/CD works.

```bash
ssh-keygen -t ed25519 -C "operator-proxmox" -f ~/.ssh/proxmox_ve
```

### 3.2. Load Key into SSH Agent

To prevent deadlocks (waiting for standard input during parallel executions) during use of external applications (like Ansible), decrypt the key and load it into the local SSH agent memory.

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/proxmox_ve
```

### 3.3. Inject Key to Hypervisor

Push the public key to the Proxmox `root` account. Enter the hypervisor's `root` password when prompted.

```bash
export PROXMOX_IP="<INSERT_PROXMOX_IP_HERE>"
ssh-copy-id -i ~/.ssh/proxmox_ve.pub root@${PROXMOX_IP}
```

### 3.4. Verify Passwordless Execution

Confirm the key is accepted via the agent without a password prompt.

```bash
ssh -i ~/.ssh/proxmox_ve root@${PROXMOX_IP} "echo 'SSH connection successful.'"
```
