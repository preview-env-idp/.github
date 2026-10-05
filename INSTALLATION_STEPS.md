# Installation Guide

## Scope

To ensure best experience this file redirects to other files which have precise scope each.

## 1. Generating SSH Keypair And Loading It To SSH Agent

Go to [runbook-001](docs/02-runbooks/001-ssh-keys.md) and create SSH keypair (ed25519 mechanism recomemded) and load it to terminal memory.

## 2. Familiarizing with Delicate Settings

Read the [possibly-dangerous](docs/03-security/01-possibly-dangerous.md) file to know what should be eventualy changed in this project to don't stop your work.
You don't need to read it if proxmox is freshly installed and has more lass nothing more than out of the box settings, hovewer I recomend to read it either ways.

## 3. Provision of Python Virtual Environment

Go to *infra-core-iac* repository folder and run `make deps` to provision ansible in Python venv.

## 4. Filling in the .env File

Go to [infra-core-iac/runbook-001](../infra-core-iac/docs/02-runbooks/001-filling-.env-file.md), create and fill the .env file in infra-core-iac.

## 5. Filling in the src/ansible/group_vars/proxmox_cluster/vault.yaml File

Go to [infra-core-iac/runbook-002](../infra-core-iac/docs/02-runbooks/002-filling-ansible-pve-cluster-vault-file.md), create and fill the vault for Ansible.

## 6. Filling in the src/ansible/group_vars/proxmox_cluster/vars.yaml File

Go to [infra-core-iac/runbook-004](../infra-core-iac/docs/02-runbooks/004-filling-ansible-pve-cluster-file.md) and fill the nacessary viariables for Ansible.

## 7. Filling in the src/ansible/group_vars/all/vars.yaml File

Go to [infra-core-iac/runbook-005](../infra-core-iac/docs/02-runbooks/005-filling-ansible-all-vars-file.md) and fill the nacessary viariables for Ansible.

## 8. Filling in the infrastructure_matrix.yaml File

Go to [infra-core-iac/runbook-006](../infra-core-iac/docs/02-runbooks/006-filling-infrasctructure-matrix-file.md) and fill the nacessary viariables for Ansible and OpenTofu.

## 9. Filling in the src/ansible/group_vars/management_plane/vault.yaml File

Go to [infra-core-iac/runbook-007](../infra-core-iac/docs/02-runbooks/007-filling-ansible-management-plane-vault-file.md) and fill the nacessary viariables for Ansible.
