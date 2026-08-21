# Prerequisites

## Scope

Repositories in this organisation are designed exclusively for a **Single-Node Proxmox VE** environment. Scaling to a multi-node cluster requires manual refactoring of the automation code.
In addition it is assumed that operator workstations OS is linux however on Windows with WSL2 and MacOs it should be also possible.

## 1. Operator Workstation

Your local machine used to trigger the deployment:

- OS: preferrably Linux (eventually MacOS or Windows with WSL2 enabled).
- Make command - make shure you have it by typing `make --version` or install it  running `sudo apt update && sudo apt install make` if you do not have it.
- Ansible - type `ansible --version` to check you have it or install it via the [Ansible installation documentation](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html), you will need python3 as well.
- OpenTofu - type (`tofu --version`) to check you have it or install it via the [OpenTofu installation documentation](https://opentofu.org/docs/intro/install/).
- SSH Keypair: Ed25519 (recommended), to make one and copy it to Proxmox see appriopriate [runbook](docs/02-runbooks/001-ssh-keys.md) (docs/02-runbooks/001-ssh-keys.md in .github repository).

## 2. Proxmox VE Target

The physical server before automation begins:

- OS: Proxmox VE (preferablly newest for best security) on older version may arrise some errors but it wasn't checked. The [Proxmox VE official installation guide](https://www.proxmox.com/en/products/proxmox-virtual-environment/get-started) covers the latest release.
- Network: Static IP assigned to `vmbr0`, reachable from your workstation.
- Bootstrap Access: Your workstation's public SSH key must be manually appended to `/root/.ssh/authorized_keys` on the Proxmox node for the initial bootstrapping run as said in SSH Keypair point in **Operator Workstation** module.

## 3. Next Steps - Building Infrastructure

Go to [INSTALLATION STEPS](INSTALLATION_STEPS.md) where runbooks are listed and explained.
