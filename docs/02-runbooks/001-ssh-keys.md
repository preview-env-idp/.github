# Runbook 001: Proxmox SSH Bootstrap

## 1. Objective

Establish initial SSH connection to Proxmox using a dedicated automation key. This key is temporarily injected into the `root` account. During the automated IaC bootstrapping phase, this key will be migrated to a dedicated service account, and the `root` SSH access will be permanently disabled.

## 2. Prerequisites

- Target Proxmox node static IP address.
- Target Proxmox initial `root` password.

## 3. Execution Steps

### 3.1. Generate Automation Key

Create the persistent identity for the Ansible bot on the Operator's workstation:

```bash
ssh-keygen -t ed25519 -C "alcambic-automation-bot" -f ~/.ssh/id_ed25519_alcambic_bot
```

### 3.2. Inject Key to Hypervisor

Push the public key to the Proxmox `root` account temporarily. Enter the `root` password when prompted.

```bash
export PROXMOX_IP="<INSERT_PROXMOX_IP_HERE>"
ssh-copy-id -i ~/.ssh/id_ed25519_alcambic_bot.pub root@${PROXMOX_IP}
```

### 3.3. Verify Connection

Confirm the key is accepted without a password prompt.

```bash
ssh -i ~/.ssh/id_ed25519_alcambic_bot root@${PROXMOX_IP}
```

## 4. Next steps

- **Target Repository:** `infra-core-iac`
- **Next Runbook:** `docs/02-runbooks/001-ansible-hypervisor-setup.md`
