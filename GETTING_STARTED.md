# Prerequisites

## Scope

Repositories in this organisation are designed to support the following Proxmox VE topologies:

1. **Standalone (Single-Node):** Fully supported with any storage backend, including isolated local storage (`local-lvm`).
2. **Multi-Node Cluster:** Supported **ONLY** if backed by a replicated or Shared Storage subsystem (e.g., Ceph, NFS, ZFS over iSCSI).

*Warning: Multi-Node clusters utilizing isolated local storage (`local-lvm` without replication) are strictly unsupported due to global VMID collisions and OpenTofu Linked Clone constraints across physical nodes. Scaling to this specific topology requires manual refactoring of the automation code at your own risk.*

In addition, it is assumed that the operator workstation's OS is Linux (Ubuntu), thus all instructions are written for this OS.

*Note: All repositories are expected to be cloned to one root folder (e.g., `alcambic/`). This will ease the installation process.*

*Note: When executing commands from runbooks, pay attention to which folder you are in before executing them. It is expected that you execute commands from the root repository folder when working in a specific runbook (e.g., when you are in runbook 001 in `infra-core-iac`, we expect you to execute commands from the `infra-core-iac/` folder). If there are exceptions to this rule, it will be communicated explicitly.*

## 1. Operator Workstation

The local control plane used to trigger the deployment requires strict dependency management. Linux (Ubuntu/Debian) is the natively supported OS.

- **Make:** GNU Make is required for runbook orchestration.
    sudo apt update && sudo apt install make -y

- **Python Virtual Environment:** To prevent System Blast Radius and Dependency Hell we use python virtual environment. The system requires only the native Python virtual environment package to bootstrap the isolated toolchain. Then the project will automatically build venv for you when stepping forward to `make bootstrap-pve` command.
  ```bash
  sudo apt update && sudo apt install python3-venv -y
  ```

## 2. Proxmox VE Target

The physical server before automation begins:

- **OS:** Proxmox VE (preferably the newest for best security; older versions may raise errors but haven't been tested). The [Proxmox VE official installation guide](https://www.proxmox.com/en/products/proxmox-virtual-environment/get-started) covers the latest release.
- **Network:** Static IP assigned to `vmbr0`, reachable from your workstation.
- **Bootstrap Access:** Your workstation's public SSH key must be manually appended to `/root/.ssh/authorized_keys` on the Proxmox node for the initial bootstrapping run as outlined in the SSH Keypair point in the **Operator Workstation** module.
- **Repositories:** Ensure that you have changed the default `pve-enterprise` repository list to the free Proxmox repository source if you don't have a paid subscription. If you run `sudo apt update && sudo apt full-upgrade` and receive an error like *401 Unauthorized*, it's most likely because of an invalid repository configuration.

## 3. Next Steps - Building Infrastructure

Go to [INSTALLATION STEPS](INSTALLATION_STEPS.md) where runbooks are listed and explained.
