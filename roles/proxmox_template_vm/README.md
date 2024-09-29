proxmox_template_vm
=========

This role deploys template VMs to your Proxmox host/cluster with a cloud init disk. These templates can be copied to create VMs that are pre-configured with your user, password, and SSH keys. You can even pre-configure the network settings, allowing you to simply clone the template and SSH into the VM as soon as it boots up and finishes its configuration.

A template VM will be created on each host in the group. A randomized 4 to 5 digit VMID will be generated based on the host's machine ID. 

Requirements
------------

#### Proxmox API Token
- You will need to create a Proxmox API token to authenticate with your Proxmox node/cluster for VM creation. 

- To create one go to **Datacenter** --> **Permissions** --> **API Tokens** in the Proxmox web UI. 


Role Variables
--------------

#### Required Variables

Not providing any of the following variables will cause the task to fail.

```YAML
proxmox_template_vm_proxmox_username: <your-proxmox-username> # User you use to log into Proxmox with. Ex. root.
proxmox_template_vm_proxmox_api_token_id: <your-proxmox-api-token-id> # ID for token.
proxmox_template_vm_proxmox_api_token_secret: <your-proxmox-api-token-secret> # Your token secret

proxmox_template_vm_distros: # A list of distros you would like templates created for. Currently only fedora and ubuntu are valid options.
```

#### Examples - Required Variables

```YAML
proxmox_template_vm_proxmox_username: "{{ proxmox_username }}" # From Ansible vault
proxmox_template_vm_proxmox_api_token_id: "{{ proxmox_api_token_id }}" # From Ansible vault
proxmox_template_vm_proxmox_api_token_secret: "{{ proxmox_api_token_secret }}" # From Ansible vault

# Required to provide at least one - this should be in list format as shown
proxmox_template_vm_distros:
  - fedora
  - ubuntu
```

#### Template Configuration Variables

The following are the variables used to customize the VM templates. Each has a default value as shown below. Ensure you customize these to your use case. If you do not provide one of these variables, the task will run with the default value.

```YAML
proxmox_template_vm_ubuntu_template_name: ubuntu-template
proxmox_template_vm_ubuntu_template_memory: 8192
proxmox_template_vm_ubuntu_template_cores: 2
proxmox_template_vm_ubuntu_template_storage: local-lvm

proxmox_template_vm_fedora_template_name: fedora-template
proxmox_template_vm_fedora_template_memory: 8192
proxmox_template_vm_fedora_template_cores: 2
proxmox_template_vm_fedora_template_storage: local-lvm
```
The following variables do not have templates and configuration of these features is omitted if they are not provided.

```YAML
proxmox_template_vm_ubuntu_ciuser: # Username for cloud init configuration
proxmox_template_vm_ubuntu_cipassword: # Password for cloud init configuration
proxmox_template_vm_ubuntu_sshkeys: # SSH Key for cloud init configuration
proxmox_template_vm_ubuntu_vlan: # Add a VLAN tag to the machine's network interface
```
#### Adjusting For Slow Storage
If your cluster uses storage that is particularly slow and prone to file locking, set the following variable to true to add pause breaks at key points in the role. The default is **false**.

If a task fails due to a file lock error, it is safe to simply re-run the playbook. 

```YAML
proxmox_template_vm_slow_storage: true
```

For more info see this [discussion board](https://forum.proxmox.com/threads/error-with-cfs-lock-unable-to-create-image-got-lock-timeout-aborting-command.65786/
). 

Dependencies
------------

#### community.general
[community.general](https://galaxy.ansible.com/ui/repo/published/community/general/) is a dependency of this role and is installed by default when installing the joshrnoll.homelab collection. The following modules are used:

- [community.general.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html)
- [community.general.proxmox_disk](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_disk_module.html)

Example Playbook
----------------

```YAML
- name: Deploy VM templates
  hosts: proxmox_hosts

  tasks:
    - name: Import variables from Ansible vault
      ansible.builtin.include_vars: secrets.yml
    
    - name: Deploy templates with proxmox_template_vm
      ansible.builtin.include_role:
        name: joshrnoll.homelab.proxmox_template_vm
      vars: 
        # Required to provide at least one
        proxmox_template_vm_distros:
          - fedora
          - ubuntu

        # Required proxmox credentials
        proxmox_template_vm_proxmox_username: "{{ proxmox_username }}" # From Ansible vault
        proxmox_template_vm_proxmox_api_token_id: "{{ proxmox_api_token_id }}" # From Ansible vault
        proxmox_template_vm_proxmox_api_token_secret: "{{ proxmox_api_token_secret }}" # From Ansible vault
        
        # Optional customizations for ubuntu
        proxmox_template_vm_ubuntu_storage: "zfs-pool-01"
        proxmox_template_vm_ubuntu_name: ubuntu-2204-template
        proxmox_template_vm_ubuntu_memory: 4096
        proxmox_template_vm_ubuntu_cores: 1
        proxmox_template_vm_ubuntu_ciuser: josh
        proxmox_template_vm_ubuntu_cipassword: "{{ cipassword }}" # From Ansible vault
        proxmox_template_vm_ubuntu_sshkeys: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_rsa.pub') }}" # gets your ssh key from /home/user/.ssh/id_rsa.pub -- customize this to your needs
        proxmox_template_vm_ubuntu_vlan: 50

        # Optional customizations for fedora
        proxmox_template_vm_fedora_storage: "zfs-pool-01"        
        proxmox_template_vm_fedora_name: fedora-40-template
        proxmox_template_vm_fedora_memory: 4096
        proxmox_template_vm_fedora_cores: 1
        proxmox_template_vm_fedora_ciuser: josh
        proxmox_template_vm_fedora_cipassword: "{{ cipassword }}" # From Ansible vault
        proxmox_template_vm_fedora_sshkeys: "{{ lookup('file', lookup('env', 'HOME') + '/.ssh/id_rsa.pub') }}" # gets your ssh key from /home/user/.ssh/id_rsa.pub -- customize this to your needs
        proxmox_template_vm_fedora_vlan: 50

        # Set to true if you have slow storage to avoid file locks
        proxmox_template_vm_slow_storage: true
```

#### IMPORTANT!

It's recommended to store your secrets in Ansible Vault. To create a vault file called ```secrets.yml``` use the following command:

```bash
ansible-vault create secrets.yml
```

Put any sensitive variables in this file. An example of an Ansible Vault file:
```YAML
# Your Proxmox Credentials
proxmox_username: <username-for-your-proxmox-node-or-cluster>
proxmox_api_token_id: <proxmox-api-token-id>
proxmox_api_token_secret: <proxmox-api-token-secret>

# Cloud init password
cipassword: <your-password>
```

#### Calling the Playbook

When calling your playbook, ensure you pass the ```--ask-vault-pass``` or ```-J``` flag. Assuming your playbook is in a file named ```playbook.yml``` and your hosts file is in a file named ```hosts.yml```, your command would look like this:

```bash
ansible-playbook playbook.yml -i hosts.yml --ask-vault-pass
```
or
```bash
ansible-playbook playbook.yml -i hosts.yml -J
```

License
-------

MIT

Author Information
------------------

Josh Noll 
https://joshrnoll.com