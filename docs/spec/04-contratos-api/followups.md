# 4.7 Follow-ups

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para Contratos de API](README.md)

---

## Regras de Follow-up

### POST /api/followups/rules

**Descrição:** Criar regra de follow-up automático.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Reengajamento 24h",
  "instanceId": "uuid (opcional)",
  "triggerType": "NO_RESPONSE",
  "triggerValue": 24,
  "triggerUnit": "hours",
  "message": "Olá! Notei que não tivemos retorno. Posso ajudar com algo?",
  "isActive": true,
  "maxAttempts": 3,
  "conditions": {
    "excludeTags": ["Não Incomodar"],
    "includeFunnelStages": ["Qualificação", "Negociação"]
  }
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Sim | Nome da regra |
| instanceId | string | Não | Instância específica (null = todas) |
| triggerType | string | Sim | Tipo de gatilho |
| triggerValue | number | Sim | Valor do gatilho |
| triggerUnit | string | Sim | Unidade (minutes, hours, days) |
| message | string | Sim | Mensagem a ser enviada |
| isActive | boolean | Não | Ativo (default: true) |
| maxAttempts | number | Não | Máximo de tentativas (default: 1) |
| conditions | object | Não | Condições de aplicação |

**Tipos de Gatilho:**

| Tipo | Descrição |
|------|-----------|
| NO_RESPONSE | Sem resposta após X tempo |
| STAGE_ENTERED | Entrou em estágio do funil |
| TAG_ADDED | Tag adicionada ao contato |
| DEAL_CREATED | Deal criado |
| CUSTOM | Gatilho personalizado |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Reengajamento 24h",
    "triggerType": "NO_RESPONSE",
    "triggerValue": 24,
    "triggerUnit": "hours",
    "isActive": true,
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Erro de validação: triggerValue: Required"
}
```

---

### GET /api/followups/rules

**Descrição:** Listar regras de follow-up.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| instanceId | string | Filtrar por instância |

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Reengajamento 24h",
      "triggerType": "NO_RESPONSE",
      "triggerValue": 24,
      "triggerUnit": "hours",
      "message": "Olá! Notei que não tivemos retorno...",
      "isActive": true,
      "maxAttempts": 3,
      "instance": {
        "id": "uuid",
        "instanceName": "vendas-01"
      },
      "stats": {
        "sent": 150,
        "pending": 25,
        "cancelled": 10
      },
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### GET /api/followups/rules/:id

**Descrição:** Obter regra de follow-up por ID.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Reengajamento 24h",
    "triggerType": "NO_RESPONSE",
    "triggerValue": 24,
    "triggerUnit": "hours",
    "message": "Olá! Notei que não tivemos retorno...",
    "isActive": true,
    "maxAttempts": 3,
    "conditions": {
      "excludeTags": ["Não Incomodar"]
    },
    "instance": {
      "id": "uuid",
      "instanceName": "vendas-01"
    },
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-01-15T10:00:00Z"
  }
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Regra não encontrada"
}
```

---

### PUT /api/followups/rules/:id

**Descrição:** Atualizar regra de follow-up.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Reengajamento 48h",
  "triggerValue": 48,
  "message": "Olá! Gostaria de saber se ainda posso ajudar?"
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Reengajamento 48h",
    "triggerValue": 48,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### DELETE /api/followups/rules/:id

**Descrição:** Excluir regra de follow-up.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Regra removida"
}
```

**Comportamento:**
- Follow-ups pendentes da regra são cancelados
- Histórico é mantido

---

### POST /api/followups/rules/:id/toggle

**Descrição:** Ativar ou desativar regra.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Reengajamento 24h",
    "isActive": false,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

## Fila de Follow-ups

### GET /api/followups/pending

**Descrição:** Listar follow-ups pendentes.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| conversationId | string | Filtrar por conversa |

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "rule": {
        "id": "uuid",
        "name": "Reengajamento 24h"
      },
      "conversation": {
        "id": "uuid",
        "contactNumber": "5511999999999",
        "contactName": "João Silva"
      },
      "contact": {
        "id": "uuid",
        "name": "João Silva"
      },
      "scheduledFor": "2026-01-20T10:00:00Z",
      "status": "PENDING",
      "attempt": 1,
      "createdAt": "2026-01-19T10:00:00Z"
    }
  ]
}
```

---

### POST /api/followups/schedule

**Descrição:** Agendar follow-up manualmente.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "ruleId": "uuid",
  "conversationId": "uuid",
  "contactId": "uuid (opcional)",
  "scheduledFor": "2026-01-20T14:00:00Z"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| ruleId | string | Sim | UUID da regra |
| conversationId | string | Sim | UUID da conversa |
| contactId | string | Não | UUID do contato |
| scheduledFor | string | Sim | Data/hora do envio (ISO 8601) |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "ruleId": "uuid",
    "conversationId": "uuid",
    "scheduledFor": "2026-01-20T14:00:00Z",
    "status": "PENDING",
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### POST /api/followups/cancel/:id

**Descrição:** Cancelar follow-up agendado.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Follow-up cancelado"
}
```

**Comportamento:**
- Muda status para CANCELLED
- Registra no histórico

---

### DELETE /api/followups/queue/:id

**Descrição:** Remover item da fila.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Item removido da fila"
}
```

---

### POST /api/followups/pending-by-conversations

**Descrição:** Obter follow-ups pendentes por múltiplas conversas.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "conversationIds": ["uuid-1", "uuid-2", "uuid-3"]
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "uuid-1": [
      {
        "id": "uuid",
        "ruleName": "Reengajamento 24h",
        "scheduledFor": "2026-01-20T10:00:00Z"
      }
    ],
    "uuid-2": [],
    "uuid-3": [
      {
        "id": "uuid",
        "ruleName": "Follow-up Proposta",
        "scheduledFor": "2026-01-21T14:00:00Z"
      }
    ]
  }
}
```

---

## Histórico e Estatísticas

### GET /api/followups/history

**Descrição:** Obter histórico de follow-ups executados.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Default | Descrição |
|-----------|------|---------|-----------|
| conversationId | string | - | Filtrar por conversa |
| limit | number | 50 | Limite de resultados |

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "rule": {
        "id": "uuid",
        "name": "Reengajamento 24h"
      },
      "conversation": {
        "id": "uuid",
        "contactNumber": "5511999999999",
        "contactName": "João Silva"
      },
      "status": "SENT",
      "scheduledFor": "2026-01-19T10:00:00Z",
      "sentAt": "2026-01-19T10:00:05Z",
      "messageId": "BAE5CF1234567890",
      "attempt": 1
    },
    {
      "id": "uuid",
      "rule": {
        "id": "uuid",
        "name": "Follow-up Proposta"
      },
      "conversation": {
        "id": "uuid",
        "contactNumber": "5511888888888",
        "contactName": "Maria Santos"
      },
      "status": "CANCELLED",
      "scheduledFor": "2026-01-19T14:00:00Z",
      "cancelledAt": "2026-01-19T12:00:00Z",
      "cancelReason": "Cliente respondeu antes"
    }
  ]
}
```

**Status Possíveis:**

| Status | Descrição |
|--------|-----------|
| PENDING | Aguardando envio |
| SENT | Enviado com sucesso |
| FAILED | Falha no envio |
| CANCELLED | Cancelado |
| RESPONDED | Cliente respondeu antes do envio |

---

### GET /api/followups/stats

**Descrição:** Obter estatísticas gerais de follow-ups.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "totalRules": 5,
    "activeRules": 4,
    "pendingFollowups": 45,
    "sentToday": 23,
    "sentThisWeek": 156,
    "sentThisMonth": 620,
    "successRate": 92.5,
    "averageResponseTime": 3600,
    "byRule": [
      {
        "ruleId": "uuid",
        "ruleName": "Reengajamento 24h",
        "sent": 300,
        "responded": 180,
        "responseRate": 60.0
      }
    ]
  }
}
```

---

### GET /api/followups/dashboard-metrics

**Descrição:** Obter métricas para o dashboard.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Default | Descrição |
|-----------|------|---------|-----------|
| days | number | 30 | Período em dias |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "period": {
      "start": "2025-12-20T00:00:00Z",
      "end": "2026-01-19T23:59:59Z",
      "days": 30
    },
    "summary": {
      "totalSent": 620,
      "totalResponded": 372,
      "totalCancelled": 48,
      "responseRate": 60.0,
      "cancelRate": 7.7
    },
    "daily": [
      {
        "date": "2026-01-19",
        "sent": 23,
        "responded": 14,
        "cancelled": 2
      }
    ],
    "byRule": [
      {
        "ruleId": "uuid",
        "ruleName": "Reengajamento 24h",
        "sent": 300,
        "responded": 180
      }
    ],
    "topPerformingRules": [
      {
        "ruleId": "uuid",
        "ruleName": "Follow-up Proposta",
        "responseRate": 75.0
      }
    ]
  }
}
```

---

### GET /api/followups/leads-in-automation

**Descrição:** Listar leads com follow-ups agendados.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| status | string | Filtrar por status (PENDING, SENT, etc.) |
| ruleId | string | Filtrar por regra |

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "contact": {
        "id": "uuid",
        "name": "João Silva",
        "phoneNumber": "5511999999999"
      },
      "conversation": {
        "id": "uuid",
        "lastMessageAt": "2026-01-18T10:00:00Z"
      },
      "followups": [
        {
          "id": "uuid",
          "ruleName": "Reengajamento 24h",
          "scheduledFor": "2026-01-19T10:00:00Z",
          "status": "PENDING",
          "attempt": 1
        }
      ],
      "totalPending": 1
    }
  ]
}
```

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Erro de validação | Campos obrigatórios ausentes |
| 400 | Erro ao agendar follow-up | Dados inválidos |
| 400 | Erro ao cancelar follow-up | Follow-up já enviado |
| 404 | Regra não encontrada | ID de regra inválido |
| 500 | Erro ao listar pendentes | Falha no banco de dados |

---

**Anterior:** [← IA/Prompts](ai.md) | **Próximo:** [Flows →](flows.md)
