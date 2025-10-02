---
title: "Using Ansible To Deploy a VM In ESXi"
date: 2021-10-21
toc: true
next: true
tags:
  - Homelab
---

## Introduction

After [configuring ESXi](https://brunoteixeira1996.github.io/publications/2021-07-22-My-ESXi-Server.html) and deploying some virtual machines, I felt the need to automate the process of creating VMs since most of the times I need to spin up a clean machine just for testing some stuff and making the same process over and over again is time consuming and boring. 

This is why I took some time to learn [Ansible](https://www.ansible.com/), so I could just create a [playbook](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html) and the VM would be up and running.

After some time and a lot of googling I found that since I am using ESXi free version I had to use an [iso](https://www.lifewire.com/iso-file-2625923) file instead of a [template](https://geek-university.com/vmware-esxi/what-is-a-virtual-machine-template/) for the VM creation. This minor detail was really important because using the template would never work without a [vCenter](https://www.vmware.com/products/vcenter-server.html) instance.

So the plan was to have an iso file in a [datastore](https://geek-university.com/vmware-esxi/what-is-a-datastore/) (in this example I will be using Ubuntu OS), configure ESXi host and a VM that runs Ansible.

For this to work the VM has to be running Ansible > 2.10 since previous versions don't support SATA.


## Configuration

### Configure ESXi

In the ESXi server the only thing I did was, reconfigure ssh to be able to root login, authenticate with a public key and disable the password authentication method. For that I edited the **/etc/ssh/sshd_config** file by adding the following commands:

```bash
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication no
```


### Configure Host Machine

The host machine used (this is where Ansible is running) was Ubuntu 20.04.2 LTS for the sake of simplicity.

Here I created a ssh key to authenticate in ESXI without the need of a password in the ssh connection.

```bash
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub | ssh ESXI_USER@ESXI_IP 'cat >> /etc/ssh/keys-root/authorized_keys'
```
Next step was to install Ansible and make sure the version was > 2.10.

```bash
pip3 install ansible-core
ansible --version
```

Before creating the playbook I had to configure some values in an **hosts** file. In this way Ansible knows what is the host used for the playbook and what configurations needs in run time. For that I had to create the **hosts** file inside **/etc/ansible/** directory with the following content:

```bash
[esxi]
<ip of the esxi host>

[esxi:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_connection=ssh
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

## Create The Playbook

*"Playbooks are expressed in YAML format with a minimum of syntax. A playbook is composed of one or more ‘plays’ in an ordered list. The terms ‘playbook’ and ‘play’ are sports analogies. Each play executes part of the overall goal of the playbook, running one or more tasks. Each task calls an Ansible module."*


To interact with an ESXi instance I used the [`community.vmware.vmware_guest`](https://docs.ansible.com/ansible/latest/collections/community/vmware/vmware_guest_module.html) module that is used to manage virtual machines in vCenter. Reading the documentation is pretty straight forward how the module works.

First I specified the host (**localhost** because it's already specified in the **hosts** file), then I needed to say what task I was going to run (**vmware_guest** module). Inside that task I identified every attribute I wanted to be present in the virtual machine. Finally I **delegated** the task to localhost and **registered** as **deploy_vm**.


```yaml
---
- hosts: localhost
  gather_facts: False
  tasks:
  - name: Create a VM in ESXi 
    vmware_guest:
      hostname: <ip of ESXi>
      username: <username of ESXi>
      password: <password of ESXi>
      validate_certs: False
      folder: /ha-datacenter/vm
      name: <vm-name>
      state: present
      guest_id: ubuntu64Guest
      esxi_hostname: <hostname of ESXi>
      disk:
      - size_gb: 100
        type: thin
        datastore: <relative path of datastore, like "[HDD1]">
      hardware:
        memory_mb: 8192
        num_cpus: 4
        scsi: lsilogicsas
      cdrom:
        - controller_number: 0
          unit_number: 0
          state: present
          type: iso
          iso_path: <relative path + iso name, like "[HDD1] debian-10.5.0-amd64-DVD-1.iso">
      networks:
      - name: VM Network
        type: static
        start_connected: true
        ip: <ip>
        netmask: <netmask>
        gateway: <gateway>
      customization:
        hostname: <vm-name>
        dns_servers:
          - 8.8.8.8
          - 8.8.4.4
    delegate_to: localhost
    register: deploy_vm
```

## Final Result

Finally the last thing was to execute the playbook using `ansible-playbook <playbook name>`.

<div style="text-align:center">
    <img style="width:900px" src="/deploy_vm_ansible_esxi/run_playbook_ansible.png">
</div>



Doing this was extremely helpful because now I can spin up virtual machines (either Windows or Linux) in a faster and easier way.


## References

* https://www.ansible.com/
* https://www.redhat.com/en/blog/ansible-101-ansible-beginners
* https://www.ansible.com/integrations/infrastructure/vmware
* https://docs.ansible.com/ansible/latest/collections/community/vmware/vmware_guest_module.html