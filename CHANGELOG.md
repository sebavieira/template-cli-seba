# Changelog - Evo AI Connect

Todas as mudanças notáveis neste projeto serão documentadas aqui.

O formato é baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/),
e este projeto adere ao [Versionamento Semântico](https://semver.org/lang/pt-BR/).

---

## [Unreleased]

### Adicionado
- Sistema de flows visuais para automação
- Integração de notas de deals com IA
- Editor Plate.js para notas enriquecidas

### Modificado
- Melhorias no modal de altura e scroll
- Otimizações no editor copilot

### Corrigido
- Correções de overflow e z-index em modais

---

## [0.2.0] - 2026-01-20

### Adicionado
- Documentação completa do projeto (PRD + SPEC)
- 9 Epics com user stories detalhadas
- 8 domínios de API documentados
- Especificações de segurança, performance e observabilidade
- Diagramas de sequência para fluxos principais
- Máquinas de estado para entidades do sistema
- Estratégia de testes completa
- Matriz de rastreabilidade

---

## [0.1.0] - 2026-01-19

### Adicionado
- Configuração inicial do projeto
- Estrutura de documentação (PRD + SPEC)
- Backend com Fastify + TypeScript + Prisma
- Frontend com React + Vite + TailwindCSS
- Integração com Evolution API/UAZAPI para WhatsApp
- Sistema de autenticação JWT com refresh tokens
- CRM básico com contatos e deals
- Sistema de funis e etapas
- Integração com OpenAI GPT-4
- Sistema de follow-ups agendados
- Dashboard com métricas básicas
- Sistema de filas com BullMQ

---

## Modelo de Entrada

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Adicionado
- Nova funcionalidade A
- Nova funcionalidade B

### Modificado
- Alteração na funcionalidade C

### Corrigido
- Bug X corrigido
- Bug Y corrigido

### Removido
- Funcionalidade depreciada D
```

---

## Convenções

### Tipos de Mudança

| Tipo | Descrição |
|------|-----------|
| **Adicionado** | Novas funcionalidades |
| **Modificado** | Mudanças em funcionalidades existentes |
| **Depreciado** | Funcionalidades que serão removidas |
| **Removido** | Funcionalidades removidas |
| **Corrigido** | Correções de bugs |
| **Segurança** | Correções de vulnerabilidades |

### Versionamento

- **MAJOR (X)**: Mudanças incompatíveis
- **MINOR (Y)**: Novas funcionalidades compatíveis
- **PATCH (Z)**: Correções de bugs compatíveis

---

[Unreleased]: https://github.com/evo-ai-connect/evo-ai-connect/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/evo-ai-connect/evo-ai-connect/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/evo-ai-connect/evo-ai-connect/releases/tag/v0.1.0
