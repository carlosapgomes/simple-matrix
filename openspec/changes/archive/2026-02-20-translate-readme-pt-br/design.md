## Context

O projeto simple-matrix possui atualmente um único arquivo `README.md` em inglês (~250 linhas). Este documento cobre:
- Pré-requisitos e deploy Ansible
- Variáveis de inventário e configurações
- Setup do Cloudflare Tunnel
- Descrição dos serviços implantados
- Notas operacionais e troubleshooting

A equipe de TI do hospital no Brasil, usuária principal deste projeto, precisa de documentação em português brasileiro para facilitar a implantação e operação do sistema Matrix.

## Goals / Non-Goals

**Goals:**
- Criar README.md em português brasileiro como documentação primária
- Mover conteúdo atual em inglês para README.en.md
- Implementar navegação bidirecional entre idiomas
- Manter precisão técnica na tradução
- Preservar todos os exemplos de código e comandos

**Non-Goals:**
- Traduzir código-fonte do projeto
- Traduzir comentários dentro dos playbooks Ansible
- Criar versões em outros idiomas além de PT-BR e EN
- Modificar o conteúdo técnico (apenas traduzir)
- Adicionar novas seções além das existentes

## Decisions

### Decisão: Estrutura de arquivos (README.md + README.en.md)
**Escolha:** README.md em português (primário), README.en.md em inglês (secundário)

**Racional:**
- O público primário é a equipe brasileira do hospital
- Segue o padrão já estabelecido pelo usuário em outros projetos (augmented-triage-system)
- Mantém compatibilidade com GitHub (que renderiza README.md por padrão)

**Alternativas consideradas:**
- README.pt-br.md + README.md (inglês): Rejeitado porque dificulta o público primário
- Diretório docs/ com traduções: Rejeitado por adicionar complexidade desnecessária

### Decisão: Formato dos links de idioma
**Escolha:** Banner simples no topo com emoji 🌐 e links bidirecionais

**Exemplo:**
```markdown
🌐 **Português (Brasil)** | [English](README.en.md)
```

**Racional:**
- Visualmente discreto mas visível
- Padrão consistente em ambos os arquivos
- Funciona corretamente no renderizador Markdown do GitHub

### Decisão: O que traduzir vs. manter em inglês
**Traduções:**
- Todos os títulos de seções
- Todo o texto explicativo e descritivo
- Instruções passo-a-passo
- Notas e avisos

**Manter em inglês:**
- Nomes de variáveis (ansible_host, matrix_fqdn, etc.)
- Comandos shell
- Arquivos de configuração (YAML, JSON)
- Paths de diretórios (/opt/matrix, etc.)
- Termos técnicos sem tradução consensual

**Racional:**
- Evita confusão ao copiar comandos
- Mantém compatibilidade com logs de erro em inglês
- Segue prática padrão da documentação técnica brasileira

## Risks / Trade-offs

| Risco | Impacto | Mitigação |
|-------|---------|-----------|
| [Risco] Divergência entre versões PT e EN no futuro | Médio | Documentar que PT é a fonte da verdade; sincronizar mudanças futuras |
| [Risco] Links quebrados entre arquivos | Baixo | Verificar links após criação; usar paths relativos simples |
| [Risco] Termos técnicos traduzidos incorretamente | Baixo | Manter termos técnicos em inglês quando não houver consenso |
| [Risco] Formatação Markdown corrompida | Baixo | Renderizar preview antes de commit; manter blocos de código intactos |

## Migration Plan

Não há migração necessária - esta é uma mudança de documentação pura que não afeta:
- Código em execução
- Banco de dados
- Configurações de deploy
- APIs ou interfaces

**Passos de implantação:**
1. Criar README.en.md com conteúdo atual
2. Criar novo README.md em português
3. Adicionar links de idioma em ambos
4. Verificar renderização Markdown no GitHub
5. Commit em uma única operação para manter histórico limpo

**Rollback:**
- Reverter para versão anterior do README.md (inglês) se necessário
- README.en.md permanece como backup

## Open Questions

- (nenhum - escopo bem definido)
