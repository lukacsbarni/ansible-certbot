## *Ansible Role for Certbot Client Installation and Configuration*

### Prerequisites - Before running the Ansible playbook

An essential element for the use of the automatic certificate management process is the existence of ACME accounts, in this case the certification entity (CA) of Sectigo.

### 1. Ansible Playbook

#### 1.1. Ansible Requirements

Minumum ansible version: `2.13`
[Community General.Collection](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) - `ansible-galaxy collection install community.general --force`

ansible-vault encrypt secrets.yml
ansible-playbook playbook_acme_ssl.yml --ask-vault-pass



#### 1.2. Inventory - ACME Credentials

To communicate with the certification authority of Sectigo, the ACME client (`certbot`) requires EAB credentials associated with an ACME account (`hmac_id` and `hmac_key`).

After following the previously mentioned procedure, the received credentials should be placed in a YAML file, for example defaults/credentials.yml, which should be constructed following the syntax exemplified in defaults/template.yml:

See below for an example of the defaults/template.yml file:

```yaml
contact_email: <xy@cc.com>
option_acme_account_name: 'cc'
option_common_names_list: 'yourdomain.com'
option_multi_domain: 'single-domain'
option_post_hook: 'service nginx reload'

acme_accounts: 
  - name: "cc"
    server: https://acme.sectigo.com/v2/OV
    mac_id: "{{ mac_id }}"
    mac_key: "{{ mac_key }}"
```

The package allows for the existence of different sets of credentials (for example, associated with different services/domains), which can be placed in the credentials.yml file as a list. Each set of credentials should be associated with a name (for example, client1, client1, etc.) that will be provided at the time of package execution to choose the credentials to be used.

The credentials can be encrypted using Ansible Vault.

