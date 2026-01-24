# Epic 08: Dashboard e Métricas

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Must Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Fornecer uma visão centralizada das métricas de negócio, atividades recentes e notificações para que gestores e vendedores acompanhem a performance em tempo real.

---

## User Stories

### US-08.1: Dashboard Principal

**Como** um usuário,
**Quero** ver um dashboard com métricas principais,
**Para que** eu tenha visão geral do negócio.

**Critérios de Aceitação:**
- [ ] Exibição de KPIs em cards
- [ ] Gráficos de evolução (últimos 7/30 dias)
- [ ] Carregamento rápido (< 2s)
- [ ] Refresh automático (opcional)
- [ ] Layout responsivo

**Prioridade:** Must Have

---

### US-08.2: Métricas de Mensagens

**Como** um gestor,
**Quero** ver métricas de mensagens,
**Para que** eu acompanhe o volume de atendimento.

**Critérios de Aceitação:**
- [ ] Total de mensagens recebidas (período)
- [ ] Total de mensagens enviadas (período)
- [ ] Mensagens respondidas por IA vs humano
- [ ] Taxa de resposta automática
- [ ] Gráfico de volume por dia/hora

**Prioridade:** Must Have

**Métricas Disponíveis:**

| Métrica | Descrição |
|---------|-----------|
| messages_received | Total de mensagens recebidas |
| messages_sent | Total de mensagens enviadas |
| ai_responses | Respostas geradas por IA |
| human_responses | Respostas de humanos |
| ai_response_rate | % de respostas automáticas |
| avg_response_time | Tempo médio de resposta |

---

### US-08.3: Atividades Recentes

**Como** um usuário,
**Quero** ver atividades recentes do sistema,
**Para que** eu saiba o que está acontecendo.

**Critérios de Aceitação:**
- [ ] Lista das últimas 20 atividades
- [ ] Tipos: nova mensagem, contato criado, deal movido, etc.
- [ ] Timestamp e autor
- [ ] Filtro por tipo de atividade
- [ ] Link para item relacionado

**Prioridade:** Should Have

---

### US-08.4: Conversas Recentes

**Como** um vendedor,
**Quero** ver conversas recentes no dashboard,
**Para que** eu acesse rapidamente atendimentos.

**Critérios de Aceitação:**
- [ ] Lista das 10 conversas mais recentes
- [ ] Indicador de não lidas
- [ ] Preview da última mensagem
- [ ] Acesso rápido à conversa
- [ ] Atualização em tempo real

**Prioridade:** Must Have

---

### US-08.5: Notificações de Equipe

**Como** um membro da equipe,
**Quero** receber notificações de eventos importantes,
**Para que** eu reaja rapidamente.

**Critérios de Aceitação:**
- [ ] Notificação em tempo real (WebSocket)
- [ ] Badge de notificações não lidas
- [ ] Lista de notificações com ações
- [ ] Marcar como lida individual/todas
- [ ] Tipos: lead quente, sentimento negativo, deal ganho

**Prioridade:** Should Have

**Tipos de Notificação:**

| Tipo | Descrição | Prioridade |
|------|-----------|------------|
| hot_lead | Lead demonstrou alto interesse | Alta |
| negative_sentiment | Sentimento negativo detectado | Alta |
| deal_won | Deal marcado como ganho | Normal |
| deal_lost | Deal marcado como perdido | Normal |
| followup_responded | Follow-up recebeu resposta | Normal |
| new_contact | Novo contato criado | Baixa |

---

### US-08.6: Métricas de Contatos

**Como** um gestor,
**Quero** ver métricas de contatos,
**Para que** eu acompanhe o crescimento da base.

**Critérios de Aceitação:**
- [ ] Total de contatos
- [ ] Novos contatos (período)
- [ ] Contatos por estágio
- [ ] Contatos ativos vs inativos
- [ ] Gráfico de crescimento

**Prioridade:** Should Have

---

### US-08.7: Analytics do Funil

**Como** um gestor,
**Quero** entender as métricas do funil com definições claras,
**Para que** eu tome decisões sem ambiguidade.

**Critérios de Aceitação:**
- [ ] Taxa de conversão calculada como ganhos / negociações criadas (com filtros)
- [ ] Ticket médio (total/ganhos/perdas) apenas com valores > 0
- [ ] Métrica "Em Risco" com definição por tooltip
- [ ] Tooltips com fórmula e base usada nos KPIs
- [ ] Evitar percentuais sem rótulo (ex.: "25%" precisa indicar o que representa)

**Prioridade:** Should Have

---

## Modelo de Dados

```prisma
model TeamNotification {
  id        String   @id @default(cuid())
  companyId String
  userId    String?  // null = toda equipe
  type      String
  title     String
  message   String
  metadata  Json?
  isRead    Boolean  @default(false)
  createdAt DateTime @default(now())

  company Company @relation(fields: [companyId], references: [id])
  user    User?   @relation(fields: [userId], references: [id])
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/dashboard/metrics` | Métricas principais |
| GET | `/api/dashboard/activities` | Atividades recentes |
| GET | `/api/dashboard/conversations/recent` | Conversas recentes |
| GET | `/api/notifications` | Listar notificações |
| PATCH | `/api/notifications/:id/read` | Marcar como lida |
| PATCH | `/api/notifications/mark-all-read` | Marcar todas como lidas |

---

## Layout do Dashboard

```
┌────────────────────────────────────────────────────────────────┐
│  Dashboard                                    [🔔 3] [👤 Ana]  │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ Contatos │ │ Mensagens│ │  Deals   │ │ Taxa IA  │          │
│  │   1.234  │ │   5.678  │ │ R$ 125K  │ │   78%    │          │
│  │  +12%    │ │  +8%     │ │ +15%     │ │  +5%     │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│                                                                │
│  ┌─────────────────────────────┬──────────────────────────────┤
│  │  Mensagens (7 dias)         │  Conversas Recentes          │
│  │  ┌─────────────────────┐    │  ┌────────────────────────┐  │
│  │  │     📈 Gráfico      │    │  │ João - "Olá, quero..." │  │
│  │  │                     │    │  │ Maria - "Ok, aguardo"  │  │
│  │  │                     │    │  │ Carlos - "Qual preço?" │  │
│  │  └─────────────────────┘    │  └────────────────────────┘  │
│  └─────────────────────────────┴──────────────────────────────┤
│                                                                │
│  Atividades Recentes                                          │
│  ┌────────────────────────────────────────────────────────────┤
│  │ 🟢 Nova mensagem de João - há 2 min                        │
│  │ 🔵 Contato Maria criado - há 15 min                        │
│  │ 🟡 Deal movido para Negociação - há 1 hora                 │
│  └────────────────────────────────────────────────────────────┘
└────────────────────────────────────────────────────────────────┘
```

---

## WebSocket Events

| Evento | Descrição | Payload |
|--------|-----------|---------|
| `notification:new` | Nova notificação | `{ id, type, title, message }` |
| `message:new` | Nova mensagem recebida | `{ conversationId, content }` |
| `metrics:update` | Atualização de métricas | `{ metric, value }` |
| `contact:updated` | Contato atualizado | `{ contactId, changes }` |

---

[← Voltar para User Stories](README.md)
