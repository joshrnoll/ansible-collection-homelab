# Ansible Collection - joshrnoll.homelab

A collection of automation roles for my homelab.

[Install with Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/)

### [server_baseline](https://github.com/joshrnoll/ansible-collection-homelab/blob/main/roles/server_baseline/README.md)

Automates a baseline configuration for all linux servers (bare metal or VM).

### [docker_setup](https://github.com/joshrnoll/ansible-collection-homelab/blob/main/roles/docker_setup/README.md)

Installs [Docker](https://www.docker.com/), [Portainer](https://www.portainer.io/), and [Nautical Backup](https://minituff.github.io/nautical-backup/).

### [tailscale_container](https://github.com/joshrnoll/ansible-collection-homelab/blob/main/roles/tailscale_container/README.md)

Deploys a container with [Tailscale](https://tailscale.com) networking.

### [proxmox_template_vm](https://github.com/joshrnoll/ansible-collection-homelab/blob/main/roles/proxmox_template_vm/README.md)

Deploys clonable Proxmox template VMs with Cloud Init disks.

### [create_proxmox_vms](https://github.com/joshrnoll/ansible-collection-homelab/blob/main/roles/create_proxmox_vms/README.md)

Creates VMs on a Proxmox node/cluster.
