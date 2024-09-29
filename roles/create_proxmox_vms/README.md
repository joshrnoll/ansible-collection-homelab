create_proxmox_vms
=========

This role clones a template VM, makes virtual hardware updates based on provided parameters, and starts the VM.

Requirements
------------

#### IMPORTANT! - Hosts Group
If your Proxmox hosts are in a cluster, read carefully before using this role. 

Because of the way Ansible runs tasks in [parallel](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_strategies.html), this role will have issues if you attempt to run it against all hosts in the cluster. For example, if your hosts group looks like this:

```YAML
# DON'T DO THIS
proxmox_hosts:
  hosts:
    node-01:
      ansible_host: 192.168.0.200
      ansible_user: root
    node-02:
      ansible_host: 192.168.0.201
      ansible_user: root
    node-03:
      ansible_host: 192.168.0.202
      ansible_user: root
```
Ansible will end up calling the Proxmox API against all hosts at once, attempting to create the same VM. This results in multiple separate requests to the Proxmox API to create, for example, VM 100. Sometimes one will succeed and the others will fail, sometimes they all will fail. 

You can attempt to force Ansible to run the tasks against one host at a time by setting the ```serial``` option to 1. However, this doesn't always work well and can get messy when your playbook includes other tasks that you don't want to run in serial. 

The best practice is to choose a single host in your cluster, and add it it's own group like this:

```YAML
proxmox_api:
  hosts:
    node-01:
      ansible_host: 192.168.0.200
      ansible_user: root
```

Then, call your playbook against the ```proxmox_api``` host group. This will run the playbook against a single host, but since no configurations are being made on the host with this role (it's only calling the API), this isn't an issue. You can still create VMs on other nodes in the cluster by setting the desired ```node``` in the ```create_proxmox_vms``` variable.

#### Proxmox API Token
You will need to create a Proxmox API token to authenticate with your Proxmox node/cluster for VM creation. 

To create one go to **Datacenter** --> **Permissions** --> **API Tokens** in the Proxmox web UI.

#### Template VM
You must have a VM template to clone on your proxmox node or in your proxmox cluster. You can use [joshrnoll.homelab.proxmox_template_vm](https://galaxy.ansible.com/ui/repo/published/joshrnoll/homelab/content/) to deploy one if you don't have it. 

Role Variables
--------------

### Required Variables
The following are required variables and will cause the task to fail if not provided.
```YAML
create_proxmox_vms_proxmox_username: <your-proxmox-username> # Ex. root
create_proxmox_vms_proxmox_api_token_id: <your-proxmox-api-token-id>
create_proxmox_vms_proxmox_api_token_secret: <your-proxmox-api-token-secret>
```
VM information is provided in list format. See the below example for all possible options.

```YAML
create_proxmox_vms_list:
  - name: vm-01 # Name as displayed in Proxmox web UI. Name cannot have an UNDERSCORE _
    vmid: 100 # VMID of the new VM
    template: 5000 # VMID of the template to clone
    memory: 4096 # OPTIONAL - Memory in MB
    cores: 1 # OPTIONAL - CPU Cores
    node: node-01 # Hostname of the node that the VM will be created on
    ciuser: admin # OPTIONAL - Username to create with Cloud Init
    cipassword: "{{ cipassword }}" # OPTIONAL - Password to provide with Cloud Init
    sshkeys: "{{ lookup('file', lookup('env','HOME') + '/.ssh/id_rsa.pub') }}" # OPTIONAL - SSH Keys to provide with Cloud Init
    storage: local-lvm # Where the VM disk will be stored
    disk_size: 64G # Size of the VM boot disk
    vlan: 50 # OPTIONAL - Vlan tag 
    ip_address: 192.168.0.100/24 # OPTIONAL - IP address to configure with Cloud Init
    gateway: 192.168.0.1 # OPTIONAL - Gateway to configure with Cloud Init
    nameservers: # OPTIONAL - Nameservers to configure with Cloud Init
      - 192.168.0.25
```
If your template VM has your desired defaults baked in (cloud init configuration, vlan and network config, desired memory and CPU), then you may choose to provide minimal parameters. The following is an example of the minimum parameters required for VM creation.

```YAML
create_proxmox_vms_list:
  - name: vm-01 
    template: 5000
    vmid: 100
    node: node-01
    storage: local-lvm
    disk_size: 32G 
```

Dependencies
------------

#### community.general
[community.general](https://galaxy.ansible.com/ui/repo/published/community/general/) is a dependency of this role and is installed by default when installing the joshrnoll.homelab collection. The following modules are used:

- [community.general.proxmox_kvm](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_kvm_module.html)
- [community.general.proxmox_disk](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_disk_module.html)

Example Playbook
----------------
```YAML
---
- hosts: proxmox_api  
  tasks:
    - name: Import variables from Ansible vault
      ansible.builtin.include_vars: secrets.yml

    - name: Deploy VMs
      ansible.builtin.include_role: 
        name: joshrnoll.homelab.create_proxmox_vms
      vars:
        # Proxmox credentials
        create_proxmox_vms_proxmox_username: "{{ proxmox_username }}"
        create_proxmox_vms_proxmox_api_token_id: "{{ proxmox_token_id }}"
        create_proxmox_vms_proxmox_api_token_secret: "{{ proxmox_token_secret }}"

        # VMs to be created
        create_proxmox_vms_list:
          - name: vm-01 
            template: 5000
            vmid: 100
            memory: 8192
            cores: 2
            node: node-02 # The hostname of the node that the VM will be created on
            ciuser: josh
            cipassword: "{{ cipassword }}"
            sshkeys: "{{ lookup('file', lookup('env','HOME') + '/.ssh/id_rsa.pub') }}"
            storage: local-lvm
            disk_size: 32G 
            vlan: 50 
            ip_address: 192.168.0.100/24
            gateway: 192.168.0.1
            nameservers:
              - 192.168.0.25
              - 192.168.0.26
          
          - name: vm-02 
            vmid: 105
            template: 5500
            memory: 4096
            cores: 1
            node: node-03 # The hostname of the node that the VM will be created on
            ciuser: josh
            cipassword: "{{ cipassword }}"
            sshkeys: "{{ lookup('file', lookup('env','HOME') + '/.ssh/id_rsa.pub') }}"
            storage: local-lvm
            disk_size: 64G 
            vlan: 50 
            ip_address: 192.168.0.105/24
            gateway: 192.168.0.1
            nameservers:
              - 192.168.0.25
              - 192.168.0.26
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

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
