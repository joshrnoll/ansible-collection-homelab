server_baseline
=========

This role makes a baseline configuration for linux servers. The following configurations are made:

- Create a user with sudo privileges*
- Copy your SSH key to the server and disable password-based authentication
- Update apt/dnf and ensure a list of default packages are present
- Install any additional desired system packages
- Configure default deny on the firewall, allowing SSH connections* 
- Install and bring up [Tailscale](https://tailscale.com). For more info, see the tailscale [README](../../tailscale-info/README.md).

*Not executed on Proxmox hosts

Requirements
------------

#### Supported operating systems
- As of the latest update, the only tested and supported operating systems are **Ubuntu 22.04**, **Fedora 40** and **Proxmox 8**. Other versions may work but have not been tested.

#### Tailscale Oauth client secret
- You will need a [Tailscale](https://tailscale.com) account and an [Oauth client](https://tailscale.com/kb/1215/oauth-clients) secret. See the tailscale [README](../../tailscale-info/README.md) for more info.

Role Variables
--------------
#### Required Variables
Not providing these variables will cause the task to fail.
```YAML
server_baseline_server_baseline_created_username: <your-username> # Enter the user you wish to create with sudo privileges.
server_baseline_tailscale_oauth_client_secret: <your-tailscale-oauth-client-secret> # Your Tailscale Oauth client. It is recommended to store this in dictionary format like the example provided.
server_baseline_timezone: <server-timezone> # Timezone to set on the server 
```

#### Examples - Required Variables

```YAML
# Task will fail if these are not provided
server_baseline_server_baseline_created_username: josh
server_baseline_tailscale_oauth_client_secret: "{{ tailscale_servers_oauth_client['secret] }}" # From Ansible vault - must be stored as dict value!
server_baseline_timezone: America/New_York
```

#### Optional Variables

Use the below variables to install additional packages (beyond the below listed defaults) on the desired hosts.

```YAML
server_baseline_fedora_packages: # Any additional packages you would like installed beyond defaults. 
server_baseline_ubuntu_packages: # Any additional packages you would like installed beyond defaults.
server_baseline_proxmox_packages: # Any additional packages you would like installed beyond defaults.
```

#### Examples - Optional Variables
```YAML
server_baseline_fedora_packages: 
  - borgbackup
  - tmux
server_baseline_ubuntu_packages:
  - aptitude
  - borgbackup
  - tmux
server_baseline_proxmox_packages:
  - tmux
  - iperf3
```

#### Default Packages Installed by Distro

**Ubuntu:**
  - curl
  - nano
  - vim
  - git
  - ufw
  - cifs-utils

**Fedora:**
  - curl
  - nano
  - vim
  - git
  - dnf-plugins-core
  - firewalld
  - cifs-utils

  **Proxmox**
  - curl
  - nano
  - vim
  - git
  - python3-proxmoxer
  - python3-requests



Dependencies
------------

#### artis3n.tailscale
- This role relies on the [artis3n.tailscale](https://github.com/artis3n/ansible-role-tailscale) role. Because this role is a member of a collection, and artis3n.tailscale is a standalone role, it **will not be installed automatically as a dependency.** 
- Run the following command to install:

```bash
ansible-galaxy role install artis3n.tailscale
```

Example Playbook
----------------

 The following is an example playbook for Ubuntu servers:

```YAML
---
- name: Initial setup for servers
  hosts: all

  tasks:
    - name: Include variables from Ansible vault file
      include_vars: secrets.yml

    - name: Server baseline
      include_role:
        name: joshrnoll.homelab.server_baseline
      vars:
        server_baseline_created_username: josh
        server_baseline_tailscale_oauth_client_secret: "{{ tailscale_servers_oauth_key['key'] }}" # From Ansible Vault file
        server_baseline_ubuntu_packages:
          - borgbackup
          - aptitude
          - tmux
        server_baseline_fedora_packages:
          - borgbackup
          - tmux
        server_baseline_proxmox_packages:
          - aptitude
          - tmux
...
```
#### IMPORTANT!

It's recommended to store your secrets in Ansible Vault. To create a vault file called ```secrets.yml``` use the following command:

```bash
ansible-vault create secrets.yml
```

Put any sensitive variables in this file. An example of an Ansible Vault file:
```YAML
# Your Tailscale Oauth client secret in dict format
tailscale_servers_oauth_key:
  key: <your-oauth-client-secret>
```

#### Calling the Playbook

Ensure you use ```--ask-become-pass``` and ```--ask-vault-pass``` when calling your playbook. Example:

```bash
ansible-playbook playbook.yml -i hosts.yml --ask-become-pass --ask-vault-pass
```

You can also use the aliases ```-K``` for ```--ask-become-pass``` and ```-J``` for ```--ask-vault-pass```. Example:

```bash
ansible-playbook playbook.yml -i hosts.yml -K -J
```

License
-------

MIT

Author Information
------------------


Josh Noll 
https://joshrnoll.com
