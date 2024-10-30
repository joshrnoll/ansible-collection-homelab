tailscale_container
=========

This role creates a docker container using the [community.docker.docker_container](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#ansible-collections-community-docker-docker-container-module) module along with a [tailscale container](https://tailscale.com/blog/docker-tailscale-guide) to serve as it's networking namespace. Containers will be automatically added to your tailnet using an [OAuth Client](https://tailscale.com/kb/1215/oauth-clients) secret.

Containers can receive valid HTTPS certificates with the FQDN of your [tailnet name](https://tailscale.com/kb/1217/tailnet-name) by using the [tailscale serve](https://tailscale.com/kb/1312/serve) feature. This will forward requests to the FQDN of the container/server over port 443 to a port of your choice running inside the container. Containers can also be optionally published to the internet using the [tailscale funnel](https://tailscale.com/kb/1223/funnel) feature. For more info on how this works, see the tailscale [README](/tailscale-info/README.md).

Requirements
------------

#### Tailscale Account and Oauth Client
- You will need a [tailscale account](https://tailscale.com/) and an [OAuth Client](https://tailscale.com/kb/1215/oauth-clients) secret for this role to run properly. For more info, see this tailscale [README](../../tailscale-info/README.md).

Role Variables
--------------

#### Required Variables (No Defaults)
Not providing these variables will cause the task to fail.
```YAML
# Failing to provide these variables will cause the task to fail.
tailscale_container_tailnet_name: # Required to use tailscale_container_ip_address in play
tailscale_container_oauth_client_secret: # Your Tailscale Oauth Client secret. See tailscale README for more info. 
tailscale_container_service_name: # The container's hostname as it will be added to your tailnet. This will also be used to create a sub-directory on the host for bind mounts. This sub-directory will be in the {{ ansible_user }}'s home directory.
tailscale_container_image: # Container's image (without tag appended)
tailscale_container_tag: # Container's tag
```

#### Examples: Required Variables (No Defaults)
```YAML
tailscale_container_tailnet_name: tailfe8c.ts.net # If you have generated a 'fun' tailnet name, you may have something like cat-crodocile.ts.net
tailscale_container_oauth_client_secret: "{{ tailscale_oauth_client_secret['secret'] }}" # Stored in Ansible vault
tailscale_container_service_name: uptime-kuma
tailscale_container_image: louislam/uptime-kuma
tailscale_container_tag: latest
```

#### Optional variables
Not providing these variables will not cause the task to fail. Use them according to your container's use case. 
```YAML
tailscale_container_serve_port: # The port to forward with tailscale serve. Do not use if you have a custom serve config file. 
tailscale_container_user: # User to run the container as -- advanced use only (generally not used)
tailscale_container_volumes: # Volumes in list format
tailscale_container_env_vars: # Environment variables in dictionary format
tailscale_container_labels: # Container labels in dictionary format
tailscale_container_commands: # Commands to run on container startup
```
#### Examples: Optional variables 
```YAML
tailscale_container_serve_port: 3001
tailscale_container_user: container_admin
tailscale_container_volumes: # It's recommended that all bind mounts exist in the /home/{{ ansible_user }}/{{ tailscale_container_service_name }} directory
  - /home/{{ ansible_user }}/{{ tailscale_container_service_name }}/app:/app/data
  - /home/{{ ansible_user }}/{{ tailscale_container_service_name }}/config:/config
tailscale_container_env_vars:
  PUID: "1000"
  PGID: "999"
  TZ: "America/New_York"
tailscale_container_labels:
  nautical.backups.enable: "true"
tailscale_container_commands: "echo 'startup commands for your container go here'"
```

#### Variables with Defaults
Not providing these variables will cause the play to run with the below defaults.
```YAML
# The following are the defaults and explanations of each variable.
tailscale_container_no_serve: false # Set to true if you do not want to use tailscale serve. Mutually exclusive with serve_port if true.
tailscale_container_https_container: false # Set to true if you the container you are serving uses HTTPS with a self-signed certificate. Mutually exclusive with no_serve.
tailscale_container_public: false # Set to true if you wish to publicly expose your container to the internet with the tailscale funnel feature.
tailscale_container_serve_config: serve-config.json # If you have a custom tailscale serve config file, you can pass it here. Otherwise, leave default.
tailscale_container_extra_args: "" # Use to pass additional arguments to the 'tailscale up' command.
tailscale_container_userspace_networking: "true" # Usually, you should leave this as default. See https://tailscale.com/kb/1112/userspace-networking
tailscale_container_pull_image: false # Set to true to re-pull the container's image every time. Useful for updating the container. 
```

#### Special Variable: tailscale_container_ip_address
The variable ```tailscale_container_ip_address``` is generated during the role's execution using the following command:

```dig @100.100.100.100 +short {{ tailscale_container_service_name }}.{{ tailscale_container_tailnet_name }}```

This stores the tailscale IP address of the container that was just deployed so that you can use it within your play. This is useful for passing to docker labels like the ones required by [traefik](https://traefik.io).

**Note that to for this variable to work, you must also provide your tailnet name via the ```tailscale_container_tailnet_name``` variable.**

Dependencies
------------
#### community.docker collection
- This role depends on the [community.docker](https://galaxy.ansible.com/ui/repo/published/community/docker/) ansible collection which is installed as a dependency when installing the joshrnoll.homelab collection. 

#### Docker installed on target node
- [Docker](https://docs.docker.com/engine/install/) is required to be installed on the target node. You can use [joshrnoll.homelab.docker_setup](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/content/) for this. 


Example Playbooks
----------------

#### Basic Example

This example installs Uptime Kuma as a non-public (not exposed to the internet) container. It is using the default of userspace networking and has no environment variables provided. The container will be reachable at ***https://uptime-kuma.tailnet-name.ts.net*** 
```YAML
---
- name: Install Uptime Kuma
  hosts: uptime-kuma

  tasks:  
    - name: Import variables from Ansible vault
      ansible.builtin.include_vars: secrets.yml
    
    - name: Install Uptime Kuma 
      ansible.builtin.include_role:
        name: joshrnoll.homelab.tailscale_container
      vars:
        tailscale_container_oauth_client_secret: "{{ tailscale_containers_oauth_client['secret'] }}"
        tailscale_container_service_name: uptime-kuma
        tailscale_container_image: louislam/uptime-kuma
        tailscale_container_tag: latest
        tailscale_container_serve_port: 3001
        tailscale_container_public: false
        tailscale_container_volumes:
          - /home/{{ ansible_user }}/{{ tailscale_container_service_name }}/app:/app/data
...
```

#### Example with Tailscale Funnel (Exposed to internet)

This example installs plex as a public (exposed to the internet) container. Notice that the ```tailscale_container_https_container``` is set to true because Plex uses https with a self-signed certificate by default. This playbook also provides an example of passing environment variables and labels to the container.

```YAML
---
- name: Install plex
  hosts: plex

  tasks:
    - name: Import variables from Ansible vault
      ansible.builtin.include_vars: secrets.yml

    - name: Install plex
      ansible.builtin.include_role:
        name: joshrnoll.homelab.tailscale_container
      vars:
        tailscale_container_tailnet_name: cat-crocodile.ts.net # Required to retrieve IP address for tailscale_container_ip_address
        tailscale_container_oauth_client_secret: "{{ tailscale_containers_oauth_client['secret'] }}"
        tailscale_container_service_name: plex
        tailscale_container_image: lscr.io/linuxserver/plex
        tailscale_container_tag: arm64v8-latest
        tailscale_container_https_container: true
        tailscale_container_serve_port: 32400
        tailscale_container_public: true
        tailscale_container_container_volumes:
          - /home/{{ ansible_user }}/{{ tailscale_container_service_name }}/config:/config
          - /home/{{ ansible_user }}/media:/media
        tailscale_container_env_vars:
          PUID: "1000"
          PGID: "999"
          TZ: "America/New_York"
          Version: "docker"
        tailscale_container_labels:
          nautical.backups.enable: "true"
...
```

#### Example using labels and tailscale_container_ip_address

This example shows the use of the special variable ```tailscale_container_ip_address``` with docker labels for traefik and [traefik-kop](https://github.com/jittering/traefik-kop).

```YAML
---
- name: Install nginx
  hosts: nginx

  tasks:
    - name: Import variables from Ansible vault
      ansible.builtin.include_vars: secrets.yml

    - name: Install nginx
      ansible.builtin.include_role:
        name: joshrnoll.homelab.tailscale_container
      vars:
        tailscale_container_oauth_client_secret: "{{ tailscale_containers_oauth_client['secret'] }}"
        tailscale_container_service_name: nginx
        tailscale_container_image: nginx
        tailscale_container_tag: latest
        tailscale_container_no_serve: true
        tailscale_container_userspace_networking: "false"
        tailscale_container_labels:
          traefik.enable: "true"
          traefik.http.routers.nginx.rule: "Host(`nginx.nollhome.casa`)"
          traefik.http.routers.nginx.entrypoints: "https"
          traefik.http.routers.nginx.tls: "true"
          traefik.http.routers.nginx.tls.certresolver: "cloudflare"
          traefik.http.services.nginx.loadbalancer.server.scheme: "http"
          traefik.http.services.nginx.loadbalancer.server.port: "80"
          kop.bind.ip: "{{ tailscale_container_ip_address }}" # Special variable created from within role
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
tailscale_containers_oauth_key:
  key: <your-oauth-client-secret>
```

#### Calling the Playbook

Ensure you use ```--ask-vault-pass``` when calling your playbook. Example:

```bash
ansible-playbook playbook.yml -i hosts.yml --ask-vault-pass
```

You can also use the aliases ```-J```. Example:

```bash
ansible-playbook playbook.yml -i hosts.yml  -J
```

License
-------

MIT

Author Information
------------------

Josh Noll
https://www.joshrnoll.com
