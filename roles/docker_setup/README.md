docker_setup
=========

This role installs Docker and, optionally, [Portainer](https://www.portainer.io/) and [Nautical Backup](https://github.com/Minituff/nautical-backup) on the target host. Nautical Backup leverages rsync to back up the bind mounts of active docker containers to an SMB share mounted on the host. Other share types are not currently supported. Containers that you wish to be backed up should be tagged with ```nautical.backups.enable: "true"```. See [Nautical Backup](https://github.com/Minituff/nautical-backup) for more details.

Portainer is installed with the [joshrnoll.homelab.tailscale_container](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/content/) role. See the tailscale [README](../../tailscale-info/README.md) for more info on using tailscale. 

Requirements
------------

#### Supported operating systems
- As of the latest update, the only tested and supported operating systems are **Ubuntu 22.04** and **Fedora 40**. Other versions of Ubuntu and Fedora may work but have not been tested.

#### Tailscale Oauth client secret
- You will need an [Oauth client](https://tailscale.com/kb/1215/oauth-clients). See the tailscale [README](../../tailscale-info/README.md) for more info.

#### cifs-utils
- Ensure **cifs-utils** is installed on the target host to support mounting of the SMB share for Nautical Backup. You can use the [joshrnoll.homelab.server_baseline](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/content/) role to do this. 

Role Variables
--------------
#### Required Variables
```YAML
# Task will fail if these are not provided
docker_setup_tailnet_name: # Your tailnet name. Ex. tailfe8c.ts.net (or cat-crocodile.ts.net, if you have a 'fun' name)
docker_setup_tailscale_oauth_client_secret: # Tailscale oauth client secret for deploying portainer. See tailscale README for more info. 
docker_setup_share_username: # Username with access to the share to be mounted
docker_setup_share_password: # Password of the user with access to the share to be mounted
```
#### Optional Variables
```YAML
# Both of these default to false!
docker_setup_install_portainer: # Whether to install portainer or not. Set to true or false.
docker_setup_install_nautical: # Whether to install nautical backup or not. Set to true or false.

# If docker_setup_install_nautical is set to true, you MUST define these variables as well:
docker_setup_nas_ip_or_hostname: # IP or Hostname of the NAS where you host your SMB share. Not the full path to the share
docker_setup_share_name: # The name of your SMB share.
docker_setup_mount_path: # Path on the host where you want your SMB share mounted

```
#### Examples
```YAML
docker_setup_install_portainer: true
docker_setup_install_nautical: true
docker_setup_tailnet_name: tailfe8c.ts.net
docker_setup_tailscale_oauth_client_secret: "{{ tailscale_containers_oauth_client['secret'] }}" # Stored in Ansible vault
docker_setup_nas_ip_or_hostname: 192.168.0.100
docker_setup_share_name: DockerBackups
docker_setup_mount_path: /home/user/dockerbackups
docker_setup_share_username: smbuser
docker_setup_share_password: "{{ password_stored_in_ansible_vault }}"
```

Dependencies
------------

#### community.docker
- Docker is installed with the [community.docker.docker_container](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html) module. This module is a member of the [community.docker](https://galaxy.ansible.com/ui/repo/published/community/docker/) collection and is installed as a dependency when installing the [joshrnoll.homelab](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/) collection.

#### joshrnoll.homelab.tailscale_container
- This role relies on **joshrnoll.homelab.tailscale_container** to deploy the Portainer container. This role is installed automatically when you install the [joshrnoll.homelab](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/) collection.

Example Playbook
----------------
```YAML
- hosts: docker
  tasks:
    - name: Include variables from Ansible Vault
      include_vars: secrets.yml
    
    - include_role:
        name: joshrnoll.homelab.docker_setup
      vars:
        docker_setup_install_portainer: true
        docker_setup_install_nautical: true
        docker_setup_tailscale_oauth_client_secret: "{{ tailscale_conatiners_oauth_client['secret'] }}"
        docker_setup_nas_ip_or_hostname: 192.168.0.100
        docker_setup_share_name: Backups
        docker_setup_share_username: nas_user01
        docker_setup_share_password: "{{ nas_user01_password }}"
```

#### IMPORTANT!

It's recommended to store your secrets in Ansible Vault. To create a vault file called ```secrets.yml``` use the following command:

```bash
ansible-vault create secrets.yml
```

Put any sensitive variables in this file. An example of an Ansible Vault file:

```YAML
# Your Tailscale Oauth client secret in dict format
tailscale_containers_oauth_key:
  key: <your-oauth-client-secret>

nas_user01_password: <password-for-user01>
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
