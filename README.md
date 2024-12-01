# Ansible Playbook to deploy Berachain 

This Ansible playbook automates the deployment and configuration of Berachain execution (beacond) and consensus (nethermind) client. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator and node keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves private validator keys and node keys from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`
For example:
`testnet/berachain/encapsulate/validator/priv_validator_key.json`

This structure ensures easy organization and secure retrieval of secrets.

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

Example for `testnet.yml`

```yaml
---
all:
  vars:
    env: testnet
  children:
    berachain:
      hosts:
        validator.berachain.testnet.encapsulate.xyz:
          type: validator
        fullnode.berachain.testnet.encapsulate.xyz:
          type: fullnode
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port, source url configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version specific variables.
- **`group_vars/vault.yml`**: Store secret variables, such as `jwtsecret`, in this file.
- There are role specific variables defined in each roles `default/main.yml` and `vars/main.yml`.

**Note**: Make sure to set the `VAULT_TOKEN` environment variable, as it enables logging in and fetching secrets from HashiCorp Vault.

Create a `group_vars/vault.yml` with your preferred ansible-vault password:

```
ansible-vault create group_vars/vault.yml
```

Example `group_vars/vault.yml`:
```
vault:
  default:
    hcp:
      vault:
        url: "https://your_hashicorp_vault_url"
  mainnet:
    berachain:
      validator:
        jwt_secret: "yout_json_web_token_secret"
      fullnode:
        jwt_secret: "yout_json_web_token_secret"
  testnet:
    berachain:
      validator:
        jwt_secret: "yout_json_web_token_secret"
      fullnode:
        jwt_secret: "yout_json_web_token_secret"
```

### Usage

1. First, install the dependencies:

  ```
  ansible-galaxy role install -r roles/requirements.yml
  ansible-galaxy collection install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

4. Then run the playbook:

- To deploy consensus client:

```
ansible-playbook setup_consensus_client.yml -l validator.berachain.testnet.encapsulate.xyz -e "fetch_validator_keys=true"
```

**Note**: The default value for `fetch_validator_keys` is false, which disables fetching keys from Hashicorp Vault.

- To deploy execution client:

```bash
ansible-playbook setup_execution_client.yml -l validator.berachain.testnet.encapsulate.xyz
```

- To sync consensus client from a lz4 compressed snapshot url:

```bash
ansible-playbook snapshot_sync.yml -l validator.berachain.testnet.encapsulate.xyz
```

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [validator.berachain.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.berachain.testnet.encapsulate.xyz",
        "Groups: ['berachain']",
        "Project: berachain",
        "Environment: testnet",
        "Type: validator",
        "Version: v0.2.0-alpha.8",
        "Username: berachain-consensus",
        "Service Name: berachain-consensus",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Set all port variables dynamically] ******************************************************************************************************
ok: [validator.berachain.testnet.encapsulate.xyz]

TASK [Display all port variables] **************************************************************************************************************
ok: [validator.berachain.testnet.encapsulate.xyz] => {
    "msg": [
        "P2P Port: 20356",
        "RPC Port: 20357",
        "ABCI Port: 20358",
        "Prometheus Port: 20360",
        "Pprof Port: 20361",
        "REST API Port: 20317",
        "Rosetta Port: 20380",
        "gRPC Port: 20390",
        "gRPC-Web Port: 20391"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.berachain.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.berachain.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.berachain.testnet.encapsulate.xyz",
        "Project: berachain",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [validator.berachain.testnet.encapsulate.xyz]
```
