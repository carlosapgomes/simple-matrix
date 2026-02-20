# readme-i18n Specification

## Purpose
TBD - created by archiving change translate-readme-pt-br. Update Purpose after archive.
## Requirements
### Requirement: Estrutura de arquivos de documentação multilíngue
O projeto SHALL manter arquivos README em português brasileiro (primário) e inglês (secundário), com nomenclatura padronizada.

#### Scenario: Estrutura correta de arquivos
- **WHEN** um usuário acessa o repositório
- **THEN** existe um arquivo `README.md` em português brasileiro na raiz
- **AND** existe um arquivo `README.en.md` com a versão em inglês

### Requirement: Navegação bidirecional entre idiomas
Ambos os arquivos README SHALL conter links no topo para alternar entre português e inglês.

#### Scenario: Link de PT para EN
- **WHEN** um usuário visualiza `README.md` (português)
- **THEN** o topo do arquivo contém um link para `README.en.md` indicando "English"

#### Scenario: Link de EN para PT
- **WHEN** um usuário visualiza `README.en.md` (inglês)
- **THEN** o topo do arquivo contém um link para `README.md` indicando "Português (Brasil)"

### Requirement: Tradução do conteúdo principal
O arquivo `README.md` em português SHALL conter todas as seções do README original traduzidas, mantendo precisão técnica.

#### Scenario: Seções traduzidas presentes
- **WHEN** um usuário lê o `README.md` em português
- **THEN** todas as seções originais estão presentes com títulos traduzidos:
  - "Pré-requisitos" (Prerequisites)
  - "Implantação" (Deploy)
  - "Configurar Variáveis de Inventário" (Configure Inventory Variables)
  - "Usando o Ansible Vault" (Using Ansible Vault)
  - "Configuração do Cloudflare Tunnel" (Cloudflare Tunnel Setup)
  - "O Que é Implantado" (What Gets Deployed)
  - "Notas Operacionais" (Operational Notes)
  - "Resetar Implantação de Dev/Teste" (Resetting a Dev/Test Deployment)
  - "Boas Práticas de Segurança" (Security Best Practices)
  - "Solução de Problemas" (Troubleshooting)

### Requirement: Preservação de termos técnicos
Termos técnicos específicos SHALL permanecer em inglês quando não há consenso de tradução na comunidade técnica brasileira.

#### Scenario: Termos técnicos mantidos em inglês
- **WHEN** o texto descreve conceitos técnicos
- **THEN** os seguintes termos permanecem em inglês:
  - Ansible, playbook, inventory, Vault
  - Docker, container, compose
  - Cloudflare Tunnel, token
  - Synapse, PostgreSQL, nginx
  - systemd, cron
  - FQDN, SSH, TLS, SSL

### Requirement: Preservação de blocos de código
Comandos, configurações YAML, e exemplos de código SHALL permanecer inalterados (em inglês).

#### Scenario: Códigos e comandos intactos
- **WHEN** o README contém blocos de código ou comandos shell
- **THEN** eles são idênticos aos do README original em inglês
- **AND** comentários dentro do código podem ser traduzidos quando apropriado

