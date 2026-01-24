# 4. User Stories e Requisitos

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](../README.md) | [Anterior: Escopo](../03-escopo.md) | [Próximo: RNF →](../05-rnf.md)

---

## Visão Geral

Este documento indexa todas as User Stories do projeto Evo AI Connect, organizadas por Epics.

### Estatísticas

| Métrica | Valor |
|---------|-------|
| Total de Epics | 9 |
| Total de User Stories | 45+ |
| Must Have | 28 |
| Should Have | 12 |
| Could Have | 5+ |

---

## Índice de Epics

### [Epic 01: Autenticação e Gestão de Usuários](epic-01-autenticacao.md)
Gerenciamento de acesso, login, permissões e equipes.

**User Stories:**
- US-01.1: Registro de Usuário
- US-01.2: Login com Email/Senha
- US-01.3: Logout
- US-01.4: Refresh de Token
- US-01.5: Perfil do Usuário
- US-01.6: Gestão de Equipe
- US-01.7: Controle de Acesso (RBAC)

---

### [Epic 02: Integração WhatsApp](epic-02-whatsapp.md)
Conexão de instâncias, envio/recebimento de mensagens, webhooks.

**User Stories:**
- US-02.1: Conexão de Instância WhatsApp
- US-02.2: Listagem de Instâncias
- US-02.3: Recebimento de Mensagens (Webhook)
- US-02.4: Envio de Mensagens
- US-02.5: Visualização de Conversa
- US-02.6: Indicador de Mensagem Não Lida
- US-02.7: Suporte a Múltiplos Provedores
- US-02.8: Armazenamento de Mídia

---

### [Epic 03: CRM de Contatos](epic-03-contatos.md)
Gerenciamento de contatos, tags, notas e bloqueio.

**User Stories:**
- US-03.1: Listagem de Contatos
- US-03.2: Criação de Contato
- US-03.3: Edição de Contato
- US-03.4: Sistema de Tags
- US-03.5: Notas em Contatos
- US-03.6: Bloqueio de Contatos
- US-03.7: Busca e Filtros

---

### [Epic 04: Funis de Vendas (Kanban)](epic-04-funis.md)
Criação de funis, estágios e movimentação de contatos.

**User Stories:**
- US-04.1: Criação de Funil
- US-04.2: Gerenciamento de Estágios
- US-04.3: Movimentação de Contatos
- US-04.4: Marcação de Ganho/Perdido
- US-04.5: Limites de WIP

---

### [Epic 05: Chatbot com IA](epic-05-ia.md)
Configuração de prompts, respostas automáticas, níveis de autonomia.

**User Stories:**
- US-05.1: Configuração de Prompt
- US-05.2: Onboarding Guiado
- US-05.3: Níveis de Autonomia
- US-05.4: Respostas Automáticas
- US-05.5: Ações Autônomas
- US-05.6: Análise de Sentimento

---

### [Epic 06: Deals e Negócios](epic-06-deals.md)
Gerenciamento de oportunidades de venda, valores e tracking.

**User Stories:**
- US-06.1: Criação de Deal
- US-06.2: Movimentação de Deals
- US-06.3: Notas em Deals
- US-06.4: Atividades de Deal
- US-06.5: Anexos em Deals

---

### [Epic 07: Follow-ups Automáticos](epic-07-followups.md)
Regras condicionais, filas de mensagens, histórico.

**User Stories:**
- US-07.1: Criação de Regra de Follow-up
- US-07.2: Triggers Condicionais
- US-07.3: Fila de Execução
- US-07.4: Histórico de Follow-ups
- US-07.5: Post-Actions

---

### [Epic 08: Dashboard e Métricas](epic-08-dashboard.md)
Visualização de KPIs, atividades recentes, notificações.

**User Stories:**
- US-08.1: Dashboard Principal
- US-08.2: Métricas de Mensagens
- US-08.3: Atividades Recentes
- US-08.4: Notificações de Equipe

---

### [Epic 09: Fluxos de Automação](epic-09-flows.md)
Editor visual de fluxos, nodes, execuções.

**User Stories:**
- US-09.1: Criação de Flow
- US-09.2: Editor Visual
- US-09.3: Tipos de Nodes
- US-09.4: Execução de Flows
- US-09.5: Histórico de Execuções

---

## Matriz de Priorização por Epic

| Epic | Prioridade | Status | Fase |
|------|------------|--------|------|
| 01 - Autenticação | Must Have | ✅ Concluído | MVP |
| 02 - WhatsApp | Must Have | ✅ Concluído | MVP |
| 03 - Contatos | Must Have | ✅ Concluído | MVP |
| 04 - Funis | Must Have | ✅ Concluído | MVP |
| 05 - IA | Must Have | ✅ Concluído | MVP |
| 06 - Deals | Should Have | ✅ Concluído | V1.1 |
| 07 - Follow-ups | Should Have | ✅ Concluído | V1.1 |
| 08 - Dashboard | Must Have | ✅ Concluído | MVP |
| 09 - Flows | Could Have | 🚧 Em Desenvolvimento | V2.0 |

---

## Formato das User Stories

Cada User Story segue o formato:

```markdown
## US-XXX: Título da Story

**Como** [tipo de usuário]
**Quero** [ação/funcionalidade]
**Para que** [benefício/valor]

**Prioridade:** Must Have | Should Have | Could Have
**Estimativa:** P (pequena) | M (média) | G (grande)
**Epic:** [Nome do Epic]
**Dependências:** [US-XXX, US-YYY] ou Nenhuma

**Critérios de Aceitação:**
1. Dado [contexto], Quando [ação], Então [resultado]
2. ...

**Notas Técnicas:**
- [Nota 1]
- [Nota 2]
```

---

## Legenda de Prioridades

| Prioridade | Descrição | Cor |
|------------|-----------|-----|
| **Must Have** | Essencial para MVP | 🔴 |
| **Should Have** | Importante, não bloqueante | 🟡 |
| **Could Have** | Desejável se houver tempo | 🟢 |
| **Won't Have** | Fora do escopo atual | ⚪ |

## Legenda de Estimativas

| Tamanho | Descrição | Pontos |
|---------|-----------|--------|
| **P (Pequena)** | Tarefa simples, < 1 dia | 1-3 |
| **M (Média)** | Complexidade moderada, 1-3 dias | 5-8 |
| **G (Grande)** | Tarefa complexa, > 3 dias | 13+ |

---

[← Voltar para Índice PRD](../README.md) | [Próximo: RNF →](../05-rnf.md)
