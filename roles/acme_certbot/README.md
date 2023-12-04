Role Name
=========

A brief description of the role goes here.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).


# Instalação e Configuração do Cliente de ACME com a Sectigo - Debian only

### Credenciais ACME

O cliente de ACME (``certbot``) necessita de credenciais de uma conta ACME (``hmac_id`` e ``hmac_key``), pelo que antes da execução do pacote de *Ansible*, o Administrador da Instituição responsável pelos certificados SSL deve ser criar/configurar uma conta *ACME* no portal da *Sectigo* associada aos domínios desejados.

Uma vez criada a conta ACME, as credenciais devem ser colocadas num ficheiro em ``defaults/credentials.yml`` (ver exemplo ``defaults/template.yml``).
O pacote permite a existência de diferentes conjuntos de credenciais (por exemplo, associados a diferentes serviços/domínios), podendo ser colocados no ficheiro ``credentials.yml`` numa lista.
Cada conjunto de credenciais deve estar associado a um nome (por exemplo, *eduroam*, *rctsaai*, *etc.*) que será fornecido no momento de execução do pacote para escolha das credenciais a utilizar.

As credenciais podem ser encriptadas utilizando o [*Ansible Vault*](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

### Executar o pacote

O pacote fornecido consiste num *playbook* de *Ansible* que irá efetuar sempre as seguintes etapas:
1. Instalar o Certbot (caso não esteja já instalado)
2. Registar o Certbot como Cliente de ACME no servidor da *Sectigo* com as credenciais *ACME* fornecidas (caso não esteja já registado)
3. Pedir o(s) certificado(s) desejados

O pacote de Ansible requer que sejam fornecidos 2 argumentos:
* ``option_acme_account_name``: nome das credenciais *ACME* a utilizar. Deve corresponder a um valor existente no ficheiro``defaults/credentials.yml``. *Exemplo: ``option_acme_account_name='eduroam'``*
* ``option_common_names_list``: lista de *common-names*, separados por vírgulas. Para cada *common-name* será gerado um certificado *SSL* do tipo *OV*. *Exemplo: ``option_common_names_list='teste1.eduroam.pt,teste2.eduroam.pt'``*

Uma vez clonado o repositório, e devidamente configurado o ficheiro ``defaults/credentials.yml``, o *playbook* de *Ansible* pode ser executado de uma de duas formas: 1) localmente, ou 2) remotamente com inventário de *Ansible* devidamente configurado.

A execução local pode ser feita com o seguinte comando:

```bash
ansible-playbook acme.yml -i "localhost," --connection=local --extra-vars "option_acme_account_name='eduroam' option_common_names_list='teste1.eduroam.pt,teste2.eduroam.pt'"
```
#### Configuração dependente de software



### Estado Final

Após execução do pacote, deverá ser criada a directoria ``/etc/letsencrypt/`` com a seguinte estrutura:

```shell
/etc/letsencrypt/
├── accounts/
│   └── acme.sectigo.com/
│       └── v2/
│           └── OV/
│               └── 37894fa3c8e8c29f299959b2f81c6fe8/
├── archive/
│   ├── teste1.eduroam.pt/
│   │   ├── cert1.pem*
│   │   ├── chain1.pem*
│   │   ├── fullchain1.pem*
│   │   └── privkey1.pem*
│   └── teste2.eduroam.pt/
│       ├── cert1.pem*
│       ├── chain1.pem*
│       ├── fullchain1.pem*
│       └── privkey1.pem*
├── live/
│   ├── teste1.eduroam.pt/
│   │   ├── cert.pem -> ../../archive/teste1.eduroam.pt/cert1.pem*
│   │   ├── chain.pem -> ../../archive/teste1.eduroam.pt/chain1.pem*
│   │   ├── fullchain.pem -> ../../archive/teste1.eduroam.pt/fullchain1.pem*
│   │   ├── privkey.pem -> ../../archive/teste1.eduroam.pt/privkey1.pem*
│   │   └── README*
│   ├── teste2.eduroam.pt/
│   │   ├── cert.pem -> ../../archive/teste2.eduroam.pt/cert1.pem*
│   │   ├── chain.pem -> ../../archive/teste2.ciencia-id.pt/chain1.pem*
│   │   ├── fullchain.pem -> ../../archive/teste2.ciencia-id.pt/fullchain1.pem*
│   │   ├── privkey.pem -> ../../archive/teste2.ciencia-id.pt/privkey1.pem*
│   │   └── README*
│   └── README*
├── options-ssl-apache.conf*
├── renewal/
│   ├── teste1.ciencia-id.pt.conf*
│   └── teste2.ciencia-id.pt.conf*
└── renewal-hooks/
    ├── deploy/
    ├── post/
    └── pre/
...
```

#### Renovação automática

