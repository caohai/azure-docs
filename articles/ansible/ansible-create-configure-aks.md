---
title: Create and configure Azure Kubernetes Service instance with Ansible
description: Learn how to use Ansible to create and manage an Azure Kubernetes Service instance in Azure
ms.service: ansible
keywords: ansible, azure, devops, bash, cloudshell, playbook, aks
author: abc
manager: abc
ms.author: abc
ms.date: 06/29/2018
ms.topic: article
---

# Create and configure Azure Kubernetes Service instance with Ansible

Ansible allows you to automate the deployment and configuration of resources in your environment. You can use Ansible to manage your Azure Kubernetes Service instances. This article shows you how to create and configure an Azure Kubernetes Service instance with Ansible.

## Prerequisites

- **Azure subscription** 
  - If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) before you begin.

- **Azure credentials** 
  - [Create Azure credentials and configure Ansible](/azure/virtual-machines/linux/ansible-install-configure#create-azure-credentials)

- **Ansible and the Azure Python SDK modules installed on your host system** 
  - Install Ansible on [CentOS 7.4](ansible-install-configure.md#centos-74), [Ubuntu 16.04 LTS](ansible-install-configure.md#ubuntu-1604-lts), and [SLES 12 SP2](ansible-install-configure.md#sles-12-sp2)

- **Azure service principal**: 
  - Follow the directions in the section of the **Create the service principal** section in the article, [Create an Azure service principal with Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#create-the-service-principal). Take note of the values for the appId, displayName, password, and tenant.

**Note**: Ansible 2.6 is requried to run the following sample playbook for Azure Kubernetes Service. 

## Create a managed Azure Kubernetes Service instance

Ansible needs a resource group to deploy all your resources into. The first section in an Ansible playbook creates a resource group named *akstest1* in the *eastus* location:

The second section in the playbook creates a Azure Kubernetes Service instance named *acctestaks1* in the resource group we created above. Enter your own *client_id* and *client_secret* in the *service_principal* part:
```yaml
- name: Create AKS instance
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
        name: akstest1
        location: eastus
  - name: Create a managed Azure Container Services (AKS) instance
    azure_rm_aks:
        name: acctestaks1
        location: eastus
        resource_group: akstest1
        dns_prefix: akstest
        linux_profile:
          admin_username: azureuser
          ssh_key: ssh-rsa AAAAB3NzaC1yc...
        service_principal:
          client_id: "your_client_id"
          client_secret: "your_client_secret"
        agent_pool_profiles:
          - name: default
            count: 5
            vm_size: Standard_D2_v2
        tags:
          Environment: Production
```

To create the Azure Kubernetes Service instance with Ansible, run the playbook as follows:

```bash
ansible-playbook azure_create_aks.yml
```

The output looks similar to the following example that shows the Azure Kubernetes Service instance has been successfully created:

```bash
PLAY [Create AKS instance] ****************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [localhost]

TASK [Create resource group] **************************************************************************************
changed: [localhost]

TASK [Create a managed Azure Container Services (AKS) instance] ***************************************************
changed: [localhost]

PLAY RECAP *********************************************************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0
```

## Next steps

> [!div class="nextstepaction"] 
