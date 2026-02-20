## Why

O projeto simple-matrix será utilizado pela equipe de TI de um hospital no Brasil. A documentação atual está apenas em inglês, o que cria uma barreira de entrada para o time técnico local que precisa implantar e operar o sistema Matrix. Traduzir o README para português brasileiro facilita a adoção e reduz erros operacionais.

## What Changes

- Criar novo `README.md` em **português brasileiro** (público primário)
- Mover conteúdo atual em inglês para `README.en.md`
- Adicionar links bidirecionais de alternância de idioma em ambos os arquivos
- Manter termos técnicos em inglês quando não há consenso de tradução (Ansible, playbook, Docker, etc.)
- Preservar blocos de código e comandos sem alteração

## Capabilities

### New Capabilities
- `readme-i18n`: Suporte a internacionalização da documentação principal do projeto, com estrutura para alternância entre idiomas

### Modified Capabilities
- (nenhum - esta mudança afeta apenas documentação, não requisitos funcionais)

## Impact

- **Documentação**: README.md reestruturado, novo arquivo README.en.md
- **Código**: Nenhuma alteração
- **APIs**: Nenhuma alteração
- **Dependências**: Nenhuma adição
- **Sistemas**: Nenhum impacto operacional
