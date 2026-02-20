🌐 **Português (Brasil)** | [English](README.en.md)

# Implantação Interna de Matrix em Hospital (Ansible)

Este projeto implanta uma stack Matrix single-node, não-federada, para uso interno em hospitais no Ubuntu 24.04. Utiliza Docker rootless, Cloudflare Tunnel e exposição via nginx na porta `8080` protegida por política de firewall do host.

## Pré-requisitos

1. Ansible instalado na sua máquina de controle.
2. Host de destino é Ubuntu 24.04 LTS com acesso SSH.
3. Cloudflare Tunnel já criado no dashboard da Cloudflare.
4. Instale a coleção Ansible necessária:

```bash
ansible-galaxy collection install community.general
```

## Implantação

Terminologia:

- Nó de controle: a máquina onde você executa o Ansible.
- Nó gerenciado: o servidor de destino onde o Matrix será instalado.

Passos (fluxo comum):

1. No nó de controle, clone este repositório:

```bash
git clone https://github.com/carlosapgomes/simple-matrix.git
cd simple-matrix
```

2. Atualize o `inventory.yml` para apontar para seu nó gerenciado (ou adicione seu hostname/IP).

Exemplo de `inventory.yml`:

```yaml
all:
  hosts:
    matrix1:
      ansible_host: 203.0.113.10
      ansible_user: ubuntu
```

3. Preencha o `group_vars/all.yml` com valores reais (use Vault se desejar).

4. Execute o playbook do nó de controle.

Usuário sudo (recomendado):

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -K
```

A flag `-K` solicita a senha sudo no nó gerenciado.

Se você autentica com senha SSH (sem chaves SSH), use:

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -k -K
```

A flag `-k` solicita a senha de login SSH.

Conta root:

```bash
ansible-playbook -i inventory.yml playbook.yml -u root
```

Se você criptografou o `group_vars/all.yml`, adicione `--ask-vault-pass`:

```bash
ansible-playbook -i inventory.yml playbook.yml -u ubuntu -K --ask-vault-pass
```

Notas:

- `-K` solicita a senha sudo. Se seu usuário sudo tem sudo sem senha, você pode omiti-la.
- Substitua `ubuntu` pelo seu usuário SSH real.

## Configurar Variáveis de Inventário

Edite o `group_vars/all.yml` e defina os valores obrigatórios:

- `matrix_fqdn`
- `matrix_instance_name`
- `matrix_admin_user`
- `matrix_admin_password`
- `postgres_password`
- `cloudflare_tunnel_token`
- `matrix_retention_days`
- `backup_retention_days`

Configurações opcionais de assets do cliente web:

- `matrix_web_assets_url_path` (padrão: `/matrix-assets`; prefixo URL de assets estáticos servidos pelo nginx)
- `matrix_web_assets_host_path` (padrão: `/opt/matrix/web-assets`; pasta de origem montada como read-only no nginx)
- `matrix_web_assets_sync_from_controller` (padrão: `true`; copia assets do nó de controle se a pasta local existir)
- `matrix_web_assets_local_path` (padrão: `{{ playbook_dir }}/assets/matrix-web`; pasta do nó de controle copiada para `matrix_web_assets_host_path`)

Implantação do Element Classic:

- `element_classic_image` (padrão: `docker.io/vectorim/element-web:latest`)
- `element_classic_upstream_port` (padrão: `80`)
- `element_classic_config_container_path` (padrão: `/app/config.json`)
- `element_classic_config_json` (padrão define `default_server_config` para `matrix_fqdn` e desabilita URLs customizadas)

Sincronização de assets controlada pelo controlador (opcional):

- Coloque arquivos no nó de controle em `assets/matrix-web/` neste repositório (ou sobrescreva `matrix_web_assets_local_path`).
- Na execução do playbook, se essa pasta local existir, seu conteúdo é copiado para `matrix_web_assets_host_path` no host gerenciado.

Ajustes opcionais do Docker rootless:

- `docker_packages_state` (padrão: `latest`)
- `docker_min_server_version` (padrão: `28.0.1`)
- `docker_rootlesskit_port_driver` (padrão: `builtin`)
- `matrix_network_internal` (padrão: `false`)

Ajustes opcionais do Synapse Admin:

- `synapse_admin_build_enabled` (padrão: `true`; compila Synapse Admin do código-fonte durante `docker compose up`)
- `synapse_admin_build_context` (padrão: `https://github.com/carlosapgomes/etkecc-synapse-admin.git`)
- `synapse_admin_build_dockerfile` (padrão: `Dockerfile`)
- `synapse_admin_image` (padrão: `matrix-synapse-admin:local`; tag a ser atribuída à imagem compilada)
- `synapse_admin_upstream_port` (padrão: `8080`)
- `synapse_admin_restrict_baseurl` (padrão: `https://{{ matrix_fqdn }}`; publicado via `/.well-known/matrix/client` como `cc.etke.synapse-admin.restrictBaseUrl`)
- `synapse_admin_well_known_homeserver` (padrão: `https://{{ matrix_fqdn }}`; publicado via `/.well-known/matrix/client` como `io.famedly.login.homeserver`)
- `synapse_admin_config_container_path` (padrão: `/var/public/config.json`)
- `synapse_admin_config_json` (padrão define `restrictBaseUrl`; renderizado para `/opt/matrix/synapse-admin/config.json` e montado no container)

Política de registro opcional:

- `synapse_enable_registration` (padrão: `false`)
- `synapse_enable_registration_without_verification` (padrão: `false`)

Implantação opcional de script de reset:

- `matrix_reset_script_enabled` (padrão: `true`)
- `matrix_reset_script_path` (padrão: `/opt/matrix/reset-matrix.sh`)
- `matrix_reset_remove_backups` (padrão: `true`)

Melhor prática de segurança: armazene secrets no Ansible Vault ao invés de texto plano.

## Usando o Ansible Vault (Recomendado)

Este projeto inclui secrets de longa duração (senha de admin, senha do BD, token do tunnel). Mesmo para um deploy de uso único, é mais seguro armazená-los criptografados.

Criptografe suas variáveis de inventário:

```bash
ansible-vault encrypt group_vars/all.yml
```

Execute o playbook:

```bash
ansible-playbook -i inventory.yml playbook.yml --ask-vault-pass
```

## Configuração do Cloudflare Tunnel (Token)

Este projeto espera um Cloudflare Tunnel pré-criado e usa apenas o token do tunnel.
Ele não cria ou gerencia recursos da Cloudflare automaticamente.

Passos para obter o token:

1. Faça login no dashboard da Cloudflare.
2. Vá em `Zero Trust` → `Access` → `Tunnels`.
3. Crie um novo tunnel (tipo: Cloudflared).
4. Nos detalhes do tunnel, copie o token do seu tunnel.
5. Defina `cloudflare_tunnel_token` no `group_vars/all.yml` (preferencialmente via Ansible Vault).

Requisito de roteamento:

- O hostname público `https://<matrix_fqdn>` deve apontar para `http://localhost:8080` via tunnel.

Melhor prática de segurança:

- Trate o token do tunnel como um secret. Rotacione-o se vazado.


## O Que é Implantado

- Usuário de sistema dedicado `matrix`
- Docker rootless + Docker Compose
- Synapse + PostgreSQL
- Synapse Admin UI em `/admin` (compilado de `https://github.com/carlosapgomes/etkecc-synapse-admin.git` por padrão)
- Cliente web Matrix (Element Classic) em `/`
- Proxy reverso nginx publicado na porta `8080` do host
- Serviço systemd do Cloudflare Tunnel
- Firewall do host via `ufw`
- Container cron de backup (DB + mídia)
 - Criação automática inicial de usuário admin (idempotente)

## Notas Operacionais

- Todos os serviços rodam em `/opt/matrix` e são de propriedade do usuário `matrix`.
- Apenas o nginx é publicado na porta `8080` do host; a política de firewall do host bloqueia acesso externo.
- O daemon Docker rootless escuta em um socket Unix (`/run/user/<uid>/docker.sock`), não em porta TCP.
- A Cloudflare gerencia TLS; o nginx roda sem SSL localmente.
- O Cloudflared roda como um serviço systemd e encaminha `https://chat.hospital.example` para `http://localhost:8080`.
- Backups são armazenados em `/opt/matrix/backups`.
 - O Docker rootless roda como um serviço de usuário systemd para `matrix`.

## Resetar Implantação de Dev/Teste

A role implanta um script helper de reset no nó gerenciado (caminho padrão: `/opt/matrix/reset-matrix.sh`).

Execute no nó gerenciado como root (ou via sudo):

```bash
sudo /opt/matrix/reset-matrix.sh
```

Este script irá:

- Parar a stack (`docker compose down`) se o arquivo compose existir
- Remover `/opt/matrix/data/postgres`
- Remover `/opt/matrix/data/synapse`
- Remover `/opt/matrix/data/media_store`
- Remover `/opt/matrix/.secrets`
- Remover opcionalmente `/opt/matrix/backups` quando `matrix_reset_remove_backups=true`

Após executá-lo, execute o playbook Ansible novamente do nó de controle para provisionar uma instância limpa.

## Boas Práticas de Segurança

- Mantenha `matrix_admin_password` e `postgres_password` no Vault.
- Não exponha portas além do localhost `8080`.
- Restrinja acesso SSH e verifique as regras de firewall antes de habilitar o ufw.
- Rotacione tokens do Cloudflare tunnel se vazados.

## Solução de Problemas

- Verifique status do systemd: `systemctl status cloudflared-matrix`
- Verifique Docker rootless: `sudo -u matrix systemctl --user status docker`
- Verifique containers: `sudo -u matrix docker compose -f /opt/matrix/docker-compose.yml ps`
- Verifique porta publicada do nginx: `sudo -u matrix XDG_RUNTIME_DIR=/run/user/$(id -u matrix) DOCKER_HOST=unix:///run/user/$(id -u matrix)/docker.sock docker compose -f /opt/matrix/docker-compose.yml -p matrix port nginx 80`
