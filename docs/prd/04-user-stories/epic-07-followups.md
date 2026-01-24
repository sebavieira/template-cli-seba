# Epic 07: Follow-ups Automáticos

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Should Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que empresas configurem regras de follow-up automático que enviam mensagens condicionais baseadas em eventos como ausência de resposta, mudança de estágio ou detecção de sentimento.

---

## User Stories

### US-07.1: Criação de Regra de Follow-up

**Como** um admin,
**Quero** criar regras de follow-up automático,
**Para que** leads recebam acompanhamento sem intervenção manual.

**Critérios de Aceitação:**
- [ ] Nome e descrição da regra
- [ ] Seleção de trigger (evento gatilho)
- [ ] Configuração de condições
- [ ] Template de mensagem ou geração por IA
- [ ] Ativar/desativar regra

**Prioridade:** Should Have

---

### US-07.2: Triggers Condicionais

**Como** um admin,
**Quero** definir diferentes gatilhos para follow-ups,
**Para que** mensagens sejam enviadas no momento certo.

**Critérios de Aceitação:**
- [ ] **NO_RESPONSE**: Sem resposta após X horas
- [ ] **TAG_ADDED**: Tag específica adicionada
- [ ] **STAGE_CHANGED**: Contato movido para estágio X
- [ ] **SENTIMENT_DETECTED**: Sentimento negativo detectado
- [ ] Combinação de condições (AND/OR)

**Prioridade:** Should Have

**Triggers Disponíveis:**

| Trigger | Parâmetros | Exemplo |
|---------|------------|---------|
| NO_RESPONSE | hours: number | "Sem resposta há 24h" |
| TAG_ADDED | tagId: string | "Tag 'interessado' adicionada" |
| STAGE_CHANGED | stageId: string | "Movido para 'Proposta'" |
| SENTIMENT_DETECTED | sentiment: string | "Sentimento negativo" |

---

### US-07.3: Fila de Execução

**Como** sistema,
**Quero** gerenciar uma fila de follow-ups pendentes,
**Para que** mensagens sejam enviadas de forma controlada.

**Critérios de Aceitação:**
- [ ] Enfileiramento com data/hora agendada
- [ ] Processamento por worker background
- [ ] Respeito a horário comercial (se configurado)
- [ ] Limite máximo de follow-ups por contato
- [ ] Intervalo mínimo entre follow-ups

**Prioridade:** Must Have (para funcionamento do epic)

---

### US-07.4: Histórico de Follow-ups

**Como** um vendedor,
**Quero** ver histórico de follow-ups enviados,
**Para que** eu saiba o que foi comunicado.

**Critérios de Aceitação:**
- [ ] Lista de follow-ups por contato
- [ ] Status: PENDING, SENT, RESPONDED, FAILED, CANCELLED
- [ ] Data de agendamento e envio
- [ ] Conteúdo da mensagem enviada
- [ ] Filtros por regra, status, período

**Prioridade:** Should Have

---

### US-07.5: Post-Actions

**Como** um admin,
**Quero** definir ações após envio do follow-up,
**Para que** o sistema reaja automaticamente.

**Critérios de Aceitação:**
- [ ] Adicionar tag após envio
- [ ] Mover para outro estágio
- [ ] Notificar equipe
- [ ] Agendar próximo follow-up (cadência)
- [ ] Configuração por regra

**Prioridade:** Could Have

---

### US-07.6: Cancelamento Automático

**Como** sistema,
**Quero** cancelar follow-ups quando contato responder,
**Para que** não enviemos mensagens desnecessárias.

**Critérios de Aceitação:**
- [ ] Cancelar follow-ups pendentes ao receber resposta
- [ ] Marcar como RESPONDED no histórico
- [ ] Reativar cadência se configurado
- [ ] Log de cancelamento com motivo

**Prioridade:** Must Have

---

## Modelo de Dados

```prisma
model FollowupRule {
  id            String              @id @default(cuid())
  companyId     String
  name          String
  description   String?
  triggerType   FollowupTriggerType
  triggerConfig Json                // Configuração específica do trigger
  messageType   String              @default("ai") // ai, template
  messageTemplate String?           // Template se messageType = template
  isActive      Boolean             @default(true)
  businessHoursOnly Boolean         @default(true)
  maxFollowups  Int                 @default(3)
  minInterval   Int                 @default(24) // horas
  postActions   Json?               // Ações pós-envio
  createdAt     DateTime            @default(now())
  updatedAt     DateTime            @updatedAt

  company  Company          @relation(fields: [companyId], references: [id])
  queue    FollowupQueue[]
  history  FollowupHistory[]
}

model FollowupQueue {
  id          String         @id @default(cuid())
  ruleId      String
  contactId   String
  scheduledAt DateTime
  status      FollowupStatus @default(PENDING)
  attempts    Int            @default(0)
  createdAt   DateTime       @default(now())

  rule    FollowupRule @relation(fields: [ruleId], references: [id])
  contact Contact      @relation(fields: [contactId], references: [id])
}

model FollowupHistory {
  id          String         @id @default(cuid())
  ruleId      String
  contactId   String
  status      FollowupStatus
  messageContent String?
  sentAt      DateTime?
  respondedAt DateTime?
  errorMessage String?
  createdAt   DateTime       @default(now())

  rule    FollowupRule @relation(fields: [ruleId], references: [id])
  contact Contact      @relation(fields: [contactId], references: [id])
}

enum FollowupTriggerType {
  NO_RESPONSE
  TAG_ADDED
  STAGE_CHANGED
  SENTIMENT_DETECTED
}

enum FollowupStatus {
  PENDING
  SENT
  RESPONDED
  FAILED
  CANCELLED
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/followups/rules` | Listar regras |
| POST | `/api/followups/rules` | Criar regra |
| GET | `/api/followups/rules/:id` | Detalhes da regra |
| PUT | `/api/followups/rules/:id` | Atualizar regra |
| DELETE | `/api/followups/rules/:id` | Excluir regra |
| PATCH | `/api/followups/rules/:id/toggle` | Ativar/desativar |
| GET | `/api/followups/queue` | Ver fila pendente |
| GET | `/api/followups/history` | Histórico de execuções |
| POST | `/api/followups/cancel/:queueId` | Cancelar follow-up |

---

## Fluxo de Processamento

```
Evento Gatilho (ex: mensagem recebida sem resposta)
    ↓
Avaliação de Regras Ativas
    ↓
Verificar Condições da Regra
    ↓
Se Match:
    ↓
    Verificar Limites (max follow-ups, intervalo)
        ↓
        Se Permitido:
            ↓
            Criar entrada na FollowupQueue
            ↓
            Agendar para scheduledAt
    ↓
Worker de Follow-up (cron)
    ↓
    Busca items com scheduledAt <= now()
    ↓
    Verifica horário comercial (se configurado)
    ↓
    Gera mensagem (IA ou template)
    ↓
    Envia via WhatsApp
    ↓
    Atualiza status e cria FollowupHistory
    ↓
    Executa post-actions (se configurado)
```

---

## Exemplo de Regra

```json
{
  "name": "Follow-up 24h sem resposta",
  "triggerType": "NO_RESPONSE",
  "triggerConfig": {
    "hours": 24
  },
  "messageType": "ai",
  "businessHoursOnly": true,
  "maxFollowups": 3,
  "minInterval": 24,
  "postActions": {
    "addTag": "follow-up-enviado",
    "notifyTeam": true
  }
}
```

---

[← Voltar para User Stories](README.md)
