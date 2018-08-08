---
title: Use Ansible to create a complete Linux VM in Azure | Microsoft Docs
description: Learn how to use Ansible to create and manage a complete Linux virtual machine environment in Azure
services: virtual-machines-linux
documentationcenter: virtual-machines
author: iainfoulds
manager: jeconnoc
editor: na
tags: azure-resource-manager

ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/30/2018
ms.author: iainfou
---

# Create a complete Linux virtual machine environment in Azure with Ansible
Ansible allows you to automate the deployment and configuration of resources in your environment. You can use Ansible to manage your virtual machines (VMs) in Azure, the same as you would any other resource. This article shows you how to create a complete Linux environment and supporting resources with Ansible. You can also learn how to [Create a basic VM with Ansible](ansible-create-vm.md).


## Prerequisites
To manage Azure resources with Ansible, you need the following:

- Ansible and the Azure Python SDK modules installed on your host system.
    - Install Ansible on [CentOS 7.4](ansible-install-configure.md#centos-74), [Ubuntu 16.04 LTS](ansible-install-configure.md#ubuntu-1604-lts), and [SLES 12 SP2](ansible-install-configure.md#sles-12-sp2)
- Azure credentials, and Ansible configured to use them.
    - [Create Azure credentials and configure Ansible](ansible-install-configure.md#create-azure-credentials)

## Create resource group
Let's look at each section of an Ansible playbook and create the individual Azure resources. For the complete playbook, see [this section of the article](#complete-ansible-playbook).

Ansible needs a resource group to deploy all your resources into. The following section in an Ansible playbook creates a resource group named *myResourceGroup* in the *eastus* location:

```yaml
- name: Create resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: eastus
```

## Create virtual network

The following section in an Ansible playbook creates a virtual network named *myVnet* in the *10.0.0.0/16* address space:

```yaml
- name: Create virtual network
  azure_rm_virtualnetwork:
    resource_group: myResourceGroup
    name: myVnet
    address_prefixes: "10.0.0.0/16"
```

To add a subnet, the following section creates a subnet named *mySubnet* in the *myVnet* virtual network:

```yaml
- name: Add subnet
  azure_rm_subnet:
    resource_group: myResourceGroup
    name: mySubnet
    address_prefix: "10.0.1.0/24"
    virtual_network: myVnet
```


## Create public IP address
To access resources across the Internet, create and assign a public IP address to your VM. The following section in an Ansible playbook creates a public IP address named *myPublicIP*:

```yaml
- name: Create public IP address
  azure_rm_publicipaddress:
    resource_group: myResourceGroup
    allocation_method: Static
    name: myPublicIP
```


## Create Network Security Group
Network Security Groups control the flow of network traffic in and out of your VM. The following section in an Ansible playbook creates a network security group named *myNetworkSecurityGroup* and defines a rule to allow SSH traffic on TCP port 22:

```yaml
- name: Create Network Security Group that allows SSH
  azure_rm_securitygroup:
    resource_group: myResourceGroup
    name: myNetworkSecurityGroup
    rules:
      - name: SSH
        protocol: Tcp
        destination_port_range: 22
        access: Allow
        priority: 1001
        direction: Inbound
```


## Create virtual network interface card
A virtual network interface card (NIC) connects your VM to a given virtual network, public IP address, and network security group. The following section in an Ansible playbook creates a virtual NIC named *myNIC* connected to the virtual networking resources you have created:

```yaml
- name: Create virtual network inteface card
  azure_rm_networkinterface:
    resource_group: myResourceGroup
    name: myNIC
    virtual_network: myVnet
    subnet: mySubnet
    public_ip_name: myPublicIP
    security_group: myNetworkSecurityGroup
```


## Create virtual machine
The final step is to create a VM and use all the resources created. The following section in an Ansible playbook creates a VM named *myVM* and attaches the virtual NIC named *myNIC*. Enter your own complete public key data in the *key_data* pair as follows:

```yaml
- name: Create VM
  azure_rm_virtualmachine:
    resource_group: myResourceGroup
    name: myVM
    vm_size: Standard_DS1_v2
    admin_username: azureuser
    ssh_password_enabled: false
    ssh_public_keys:
      - path: /home/azureuser/.ssh/authorized_keys
        key_data: "ssh-rsa AAAAB3Nz{snip}hwhqT9h"
    network_interfaces: myNIC
    image:
      offer: CentOS
      publisher: OpenLogic
      sku: '7.5'
      version: latest
```

## Complete Ansible playbook
To bring all these sections together, create an Ansible playbook named *azure_create_complete_vm.yml* and paste the following contents. Enter your own complete public key data in the *key_data* pair:

```yaml
- name: Create Azure VM
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: eastus
  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: myResourceGroup
      name: myVnet
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: myResourceGroup
      name: mySubnet
      address_prefix: "10.0.1.0/24"
      virtual_network: myVnet
  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: myResourceGroup
      allocation_method: Static
      name: myPublicIP
  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: myResourceGroup
      name: myNetworkSecurityGroup
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound
  - name: Create virtual network inteface card
    azure_rm_networkinterface:
      resource_group: myResourceGroup
      name: myNIC
      virtual_network: myVnet
      subnet: mySubnet
      public_ip_name: myPublicIP
      security_group: myNetworkSecurityGroup
  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: myResourceGroup
      name: myVM
      vm_size: Standard_DS1_v2
      admin_username: azureuser
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/azureuser/.ssh/authorized_keys
          key_data: "ssh-rsa AAAAB3Nz{snip}hwhqT9h"
      network_interfaces: myNIC
      image:
        offer: CentOS
        publisher: OpenLogic
        sku: '7.5'
        version: latest
```


To create the complete VM environment with Ansible, run the playbook as follows:

```bash
ansible-playbook azure_create_complete_vm.yml
```

The output looks similar to the following example that shows the VM has been successfully created:

```bash
PLAY [Create Azure VM] ****************************************************

TASK [Gathering Facts] ****************************************************
ok: [localhost]

TASK [Create resource group] *********************************************
changed: [localhost]

TASK [Create virtual network] *********************************************
changed: [localhost]

TASK [Add subnet] *********************************************************
changed: [localhost]

TASK [Create public IP address] *******************************************
changed: [localhost]

TASK [Create Network Security Group that allows SSH] **********************
changed: [localhost]

TASK [Create virtual network inteface card] *******************************
changed: [localhost]

TASK [Create VM] **********************************************************
changed: [localhost]

PLAY RECAP ****************************************************************
localhost                  : ok=8    changed=7    unreachable=0    failed=0
```

## Stop created virtual machine
Ansible allows you to stop the created running virtual machine. The following playbook deallocates(stops) a running virtual machine.
```yaml
- name: Stop Azure VM
  hosts: localhost
  connection: local
  tasks:
  - name: Deallocate the virtual machine
    azure_rm_virtualmachine:
        resource_group: myResourceGroup
        name: myVM
        allocated: no 
```
To stop the running virtual machine with Ansible, run the playbook as follows:

```bash
ansible-playbook azure_vm_stop.yml
```

The output looks similar to the following example that shows the VM has been successfully stopped:

```bash
PLAY [Stop Azure VM] ***********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************
ok: [localhost]

TASK [Deallocate the Virtual Machine] ******************************************************************************************
changed: [localhost]

PLAY RECAP *********************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0```
```

## Start the stopped virtual machine
Ansible allows you to start the deallocated(stopped) virtual machine. The following playbook starts a stopped virtual machine.
```yaml
- name: Start Azure VM
  hosts: localhost
  connection: local
  tasks:
  - name: Start the virtual machine
    azure_rm_virtualmachine:
        resource_group: myResourceGroup
        name: myVM
```
To start the stopped virtual machine with Ansible, run the playbook as follows:

```bash
ansible-playbook azure_vm_start.yml
```

The output looks similar to the following example that shows the VM has been successfully started:

```bash
PLAY [Stop Azure VM] ***********************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************
ok: [localhost]

TASK [Start the Virtual Machine] ******************************************************************************************
changed: [localhost]

PLAY RECAP *********************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0```
```

## Next steps
This example creates a complete VM environment including the required virtual networking resources. For a more direct example to create a VM into existing network resources with default options, see [Create a VM](ansible-create-vm.md).
