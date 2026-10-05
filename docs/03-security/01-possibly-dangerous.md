# Possibly Dangerous

## Alcmabic Account

If your Proxmox VE has an account named *alcambic* note that this project will be using it for it's automated operetions - you can change it by manually going through the files and changing necessary things.

## Proxmox Datacenter Firewall

This project after running the ansible scripts will set also the Datacenter firewall so it not only deny by default all incoming traffic but also dany all outgoing traffic if it is not on the list to allow which is being configured in [pve_firewall](../../../infra-core-iac/src/ansible/roles/pve_firewall/tasks/main.yaml) file. If you need custom ports to be allowed in/out or NFS (port 111) **check this file and make apriopriate changes** so it's suits you. 

*Note:* only IPs from internal network (specified by `management_subnet_cidr` variable) will be able to access proxox cluster (by SSH, GUI, SPICE).

## SSH Access

By default after running ansible scripts you won't be able to log in using ssh by password. Olny keys will grant access. Visit [secure_ssh](../../../infra-core-iac/src/ansible/roles/secure_ssh/tasks/main.yaml) role to rewrite it if it's bad for you.
