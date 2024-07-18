## *Ansible Role for Certbot Client Installation and Configuration*

### Prerequisites - Before running the Ansible playbook

An essential element for the use of the automatic certificate management process is the existence of ACME accounts, in this case the certification entity (CA) of Sectigo.

### 1. Ansible Playbook

#### 1.1. Ansible Requirements

Minumum ansible version: `2.13`

[Community General.Collection](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) - `ansible-galaxy collection install community.general --force`


#### 1.2. Inventory - ACME Credentials

To communicate with the certification authority of Sectigo, the ACME client (`certbot`) requires EAB credentials associated with an ACME account (`hmac_id` and `hmac_key`).

After following the previously mentioned procedure, the received credentials should be placed in a YAML file, for example defaults/credentials.yml, which should be constructed following the syntax exemplified in defaults/template.yml:

See below for an example of the defaults/template.yml file:

```yaml
contact_email: <xy@cc.com>
option_acme_account_name: 'cc'
option_common_names_list: 'test.yourdomain.com'
option_multi_domain: 'single-domain'
option_post_hook: 'service nginx reload'

acme_accounts: 
  - name: "cc"
    server: https://acme.sectigo.com/v2/OV
    mac_id: "{{ mac_id }}"
    mac_key: "{{ mac_key }}"
```

See below for an example of the defaults/credentials.yml file:

```yaml
mac_id: aPxxxxxxxxxxxxxxxxxxxxxx
mac_key: Kmxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

The credentials must be encrypted using Ansible Vault.

`ansible-vault encrypt defaults/secrets.yml`

The package allows for the existence of different sets of credentials (for example, associated with different services/domains), which can be placed in the credentials.yml file as a list. Each set of credentials should be associated with a name (for example, client1, client1, etc.) that will be provided at the time of package execution to choose the credentials to be used.

#### 1.3. Execute the playbook

The provided package consists of an Ansible playbook that will always perform the following steps:

1. Install the certbot using snapd (if not already installed);
2. Register certbot as an ACME client with the Sectigo certification authority using the provided ACME credentials (if not already registered);
3. Request the desired certificate(s).

**The Ansible playbook allows 7 arguments to be provided:**

* **option_acme_account_name:**
    * **Mandatory**
    * Name of the ACME credentials to use. It should correspond to an existing value in the file ``defaults/credentials.yml``.
    * *Example: ``option_acme_account_name='cc'``*
* **option_common_names_list:**
    * **Required**
    * List of common names, separated by commas. For each common name, one or more (depending on the value of option_multi_domain) OV SSL certificates will be generated.
    * *Example: ``option_common_names_list='test.yourdomain.com,test1.yourdomain.com'``*
* **option_credentials_file:**
    * *Optional  (default: ``'defaults/credentials_sid.yml'``)*.
    * Path to the file with the ACME credentials.
    * *Example: ``option_credentials_file='defaults/credentials_sid.yml'``**
* **option_credentials_file:**
    * *Optional  (default: ``'single-domain'``)*.
    * Allowed values:
        * ``single-domain``
        * ``multi-domain``
    * Allows you to select whether you want a single-domain certificate for each common name or a single multi-domain certificate with all provided common names.
    * *Example: ``option_multi_domain='single-domain'``*
* **option_post_hook:**
    * *Optional  (default: ``''``)*.
    * Allows you to define an action (command, script, etc.) to execute whenever there is a new version of a certificate. An example can be restarting the web server (e.g., Apache, Nginx, etc.) after certificate renewal.
    * *Example: ``option_post_hook='service apache2 reload'``*
* **option_webserver:**
    * *Optional  (default: ``'standalone'``)*.
    * Allows you to define the web server to be used to communicate with the CA.
    * *Example: ``option_webserver='standalone'``*
* **option_encryption_algorithm:**
    * *Optional (default: ``'ecdsa'``)*.
    * Allows you to define the encryption algorithm used.
    * *Example: ``option_encryption_algorithm='ecdsa'``*
* **option_concatenate_certs:**
    * *Optional (default: ``'False'``)*.


