## *Playbook de Ansible* para Instalação e Configuração do cliente *Certbot*

Como forma de promover a adopção da utilização do protocolo **ACME** na comunidade **RCTS**, a **FCCN** disponibiliza um pacote de *Ansible* que permite instalar e configurar um **cliente de ACME** para servidores *Linux*, especificamente o cliente [**certbot**](https://certbot.eff.org/).

### 4.1. Pré-requisitos - Antes de executar o playbook de *Ansible*

Um elemento essencial para a utilização do processo de gestão automática de certificados é a existência de contas ACME, particularmente da *Sectigo*.
A gestão destas contas, nomeadamente a sua criação, alteração ou remoção é da responsabilidade do grupo **SID** da **Área ASR**.

**Assim, e para a utilização do procedimento implementado no *playbook*, é essencial o pedido de uma conta ACME da *Sectigo*.**

Os passos a serem seguidos para o pedido de uma dessas contas encontram-se descrito abaixo:

#### 4.1.1. Envio da mensagem de pedido de criação de conta
A criação da conta **ACME** requer alguns pressupostos obrigatórios. 
1. **O primeiro é que as credenciais geradas sejam transmitidas de forma segura.**
Para isso, o requerente da conta deverá ter a utilização de certificados de assinatura de email ativa e a funcionar.
Caso necessite, por favor consulte a [informação sobre a emissão e utilização de certificados pessoais no email](https://share.fccn.pt/sites/rctscertificados/ManualUtilizador/#page-toc-3).
2. **O segundo pressuposto é que os domínios a serem usados devem estar já registados e validados na plataforma da Sectigo.**
Em caso de dúvida, devem validar esse registo com a equipa de **SID**.

Garantidos os pressupostos essenciais, o pedido de conta é iniciado com o envio de um e-mail para <noc@fccn.pt> indicando essa necessidade.
O pedido de conta **ACME** deve seguir o seguinte [*template*](LINKMISSING).

Importa destacar que devem ser fornecidas as informações presentes no template, nomeadamente:
* **Área FCCN**
* **Serviço**
* **Domínios a associar à conta ACME**

Relativamente aos domínios a associar, por questões de segurança, deve ser usado o mais específico possível dentro do serviço (por exemplo, deve ser pedido  *eduroam.fccn.pt* e não *fccn.pt*);
<!-- * Se necessário podem indicar um wild-card de um subdomínio, evitando um domínio de topo (ex: *.eduroam.fccn.pt e não*.fccn.pt); -->

#### 4.1.2. Criação da conta por parte da equipa de SID

Ao receber o pedido de criação de conta, a equipa de **SID** irá aceder à plataforma da *Sectigo* e realizar os passos necessários para a criação da conta, tendo em consideração a informação fornecida.

#### 4.1.3. Resposta com os dados da conta a serem usados (email cifrado e assinado)

Após a criação da conta, e com os dados já presentes, a equipa de **SID** irá responder à mensagem original, indicando as credenciais *EAB* associadas a uma conta **ACME** (``hmac_id`` e ``hmac_key``).
Recordamos que a mensagem a enviar será cifrada e portanto dirigida apenas ao requerente original.
Se possível a mensagem recebida deverá ser eliminada após salvaguarda das credenciais em local seguro e apropriado (por exemplo, *num inventário Ansible através de Ansible-Vault, como será detalhado na secção seguinte*).
Do lado da equipa **SID** a mensagem enviada será também eliminada, uma vez que caso seja necessário recuperar os dados de acesso à conta **ACME**, estes estão acessíveis dentro da plataforma da *Sectigo*.

### 4.2. *Playbook* de Ansible

#### 4.2.1. Inventário - Credenciais ACME

O cliente de ACME (``certbot``) necessita de credenciais *EAB* associadas a uma conta ACME (``hmac_id`` e ``hmac_key``), pelo que antes da execução do pacote de *Ansible*.

Depois de seguido o procedimento detalhado na [Secção anterior](https://share.fccn.pt/sites/rctscertificados/ACME/acme_internal_fccn/#page-toc-9), as credenciais recebidas devem ser colocadas num ficheiro *YAML*, por exemplo ``defaults/credentials.yml``, que deve ser construído seguindo a sintaxe exemplificada em ``defaults/template.yml``:

Ver abaixo exemplo do ficheiro ``defaults/template.yml``:

```yaml
contact_email: <contact_fccn> #exemplo: joao.guerreiro@fccn.pt

acme_accounts: 
  - name: <serviço> #exemplo: "CIENCIA ID"
    server: https://acme.sectigo.com/v2/OV
    mac_id: <credencial_eab_hmac_id>
    mac_key: <credencial_eab_hmac_key>
```

O pacote permite a existência de diferentes conjuntos de credenciais (por exemplo, associados a diferentes serviços/domínios), podendo ser colocados no ficheiro ``credentials.yml`` numa lista.
Cada conjunto de credenciais deve estar associado a um nome (por exemplo, *eduroam*, *rctsaai*, *etc.*) que será fornecido no momento de execução do pacote para escolha das credenciais a utilizar.

As credenciais podem ser encriptadas utilizando o [*Ansible Vault*](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

#### 4.2.2. Executar o *playbook*

O pacote fornecido consiste num *playbook* de *Ansible* que irá efetuar sempre as seguintes etapas:

1. Instalar o *certbot* (caso não esteja já instalado)
2. Registar o *certbot* como **cliente de ACME** na entidade certificadora da *Sectigo* com as credenciais *ACME* fornecidas (caso não esteja já registado)
3. Pedir o(s) certificado(s) desejados

**O *playbook* de Ansible permite que sejam fornecidos 7 argumentos:**

* **option_acme_account_name:**
    * **Obrigatório**
    * Nome das credenciais *ACME* a utilizar. Deve corresponder a um valor existente no ficheiro``defaults/credentials.yml``
    * *Exemplo: ``option_acme_account_name='eduroam'``*
* **option_common_names_list:**
    * **Obrigatório**
    * lista de *common-names*, separados por vírgulas. Para cada *common-name* será gerado um (ou vários, dependendo do valor de ``option_multi_domain``) certificado(s) *SSL* do tipo *OV*.
    * *Exemplo: ``option_common_names_list='teste1.eduroam.pt,teste2.eduroam.pt'``*
* **option_credentials_file:**
    * *Opcional (default: ``'defaults/credentials_sid.yml'``)*.
    * Caminho para o ficheiro com as credenciais **ACME**.
    * *Exemplo: ``option_credentials_file='defaults/credentials_sid.yml'``**
* **option_credentials_file:**
    * *Opcional (default: ``'single-domain'``)*.
    * Valores permitidos:
        * ``single-domain``
        * ``multi-domain``
    * Permite selecionar se se pretende um certificado **single-domain** por cada *common-name* ou um único certificado **multi-domain** com todos os *common-names* fornecidos.
    * *Exemplo: ``option_multi_domain='single-domain'``*
* **option_post_hook:**
    * *Opcional (default: ``''``)*.
    * Permite definir uma ação (comando, *script*, *etc.*) a executar sempre que existe uma nova versão de um certificado. Um exemplo pode ser reiniciar o servidor *web* (*e.g., apache, nginx, etc.*) após renovação de certificados.
    * *Exemplo: ``option_post_hook='service apache2 reload'``*
* **option_webserver:**
    * *Opcional (default: ``'standalone'``)*.
    * Permite definir o servidor *web* a ser usado para comunicar com a **CA**.
    * *Exemplo: ``option_webserver='standalone'``*
* **option_encryption_algorithm:**
    * *Opcional (default: ``'ecdsa'``)*.
    * Permite definir o algoritmo de encriptação usado.
    * *Exemplo: ``option_encryption_algorithm='ecdsa'``*
* **option_concatenate_certs:**
    * *Opcional (default: ``'False'``)*.
    * Permite gerar, para além dos ficheiros habituais, um ficheiro que agrega o certificado gerado e a chave privada. Util para algumas aplicações, como por exemplo *Haproxy.*
    * *Exemplo: ``option_concatenate_certs='False'``*

Uma vez clonado o repositório, e devidamente configurado o ficheiro ``defaults/credentials.yml``, o *playbook* de *Ansible* pode ser executado de uma de duas formas: 
1. **Localmente**.
2. **Remotamente** com inventário de *Ansible* devidamente configurado.

A execução local pode ser feita com o seguinte comando:

```bash
shell> ansible-playbook playbook_acme_ssl.yml --extra-vars "option_acme_account_name='eduroam' option_common_names_list='teste1.eduroam.pt,teste2.eduroam.pt' option_credentials_file='defaults/credentials_sid.yml' option_multi_domain='single-domain' option_post_hook='service apache2 reload'"
```

### 4.3 Estado Final

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
