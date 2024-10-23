# Ansible Playbook to deploy Berachain 

This Ansible playbook automates the deployment and configuration of Berachain execution (beacond) and consensus (nethermind) client. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

**Note**: The following ansible playbook dynamically fetches private validator and node keys from hashicorp vault. 

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)


### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/berachain-ansible.git
cd berachain-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for testnet.yml

```yaml
---
all:
  vars:
    env: testnet
  children:
    bera
      hosts:
        validator.bera.testnet.encapsulate.xyz:
          type: validator
        fullnode.bera.testnet.encapsulate.xyz:
          type: fullnode
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.
- **`group_vars/vault.yml`**: Store secret variables, such as `jwtsecret`, in this file.



### Usage

1. First, install the dependencies:

   ```bash
   ansible-galaxy install -r requirements.yml

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Then run the playbook:

- To deploy consensus client:

```bash
ansible-playbook setup_consensus_client.yml -l validator.bera.testnet.encapsulate.xyz
```
- To deploy execution client:

```bash
ansible-playbook setup_execution_client.yml -l validator.bera.testnet.encapsulate.xyz
```
After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [validator.bera.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.bera.testnet.encapsulate.xyz",
        "Groups: ['bera']",
        "Project: bera",
        "Environment: testnet",
        "Type: validator",
        "Version: v0.11.0",
        "Username: bera-consensus",
        "Service Name: bera-consensus",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Set all port variables dynamically] ******************************************************************************************************
ok: [validator.bera.testnet.encapsulate.xyz]

TASK [Display all port variables] **************************************************************************************************************
ok: [validator.bera.testnet.encapsulate.xyz] => {
    "msg": [
        "P2P Port: 10156",
        "RPC Port: 10157",
        "ABCI Port: 10158",
        "Prometheus Port: 10160",
        "Pprof Port: 10161",
        "REST API Port: 10117",
        "Rosetta Port: 10180",
        "gRPC Port: 10190",
        "gRPC-Web Port: 10191"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.bera.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.bera.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.bera.testnet.encapsulate.xyz",
        "Project: bera",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.bera.testnet.encapsulate.xyz]
```
