# Ansible Collection - joshrnoll.homelab

A collection of automation roles for my homelab.

[Install with Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/)

### [server_baseline](/roles/server_baseline/README.md)

Automates a baseline configuration for all linux servers (bare metal or VM).

### [docker_setup](/roles/docker_setup/README.md)

Installs [Docker](https://www.docker.com/), [Portainer](https://www.portainer.io/), and [Nautical Backup](https://minituff.github.io/nautical-backup/).

### [tailscale_container](/roles/tailscale_container/README.md)

Deploys a container with [Tailscale](https://tailscale.com) networking.

### [proxmox_template_vm](/roles/proxmox_template_vm/README.md)

Deploys clonable Proxmox template VMs with Cloud Init disks.

### [create_proxmox_vms](/roles/create_proxmox_vms/README.md)

Creates VMs on a Proxmox node/cluster.
