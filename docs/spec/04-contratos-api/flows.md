# 4.8 Flows (Automações)

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para Contratos de API](README.md)

---

## Visão Geral

Flows são fluxos de automação visual construídos com nós conectados. Permitem criar automações complexas sem código, respondendo a eventos e executando ações.

**Conceitos:**
- **Flow:** Fluxo de automação completo
- **Node:** Nó individual (trigger, action, condition)
- **Connection:** Conexão entre nós
- **Execution:** Instância de execução do fluxo

---

## Fluxos

### GET /api/flows

**Descrição:** Listar todos os fluxos da empresa.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Boas-vindas Automático",
      "description": "Envia mensagem de boas-vindas para novos contatos",
      "isActive": true,
      "triggerType": "NEW_CONTACT",
      "nodeCount": 5,
      "executionCount": 1250,
      "lastExecutedAt": "2026-01-19T10:00:00Z",
      "createdAt": "2026-01-01T00:00:00Z",
      "updatedAt": "2026-01-15T10:00:00Z"
    },
    {
      "id": "uuid",
      "name": "Qualificação de Lead",
      "description": "Fluxo de qualificação baseado em respostas",
      "isActive": false,
      "triggerType": "MESSAGE_RECEIVED",
      "nodeCount": 12,
      "executionCount": 450,
      "lastExecutedAt": "2026-01-18T15:00:00Z",
      "createdAt": "2026-01-05T00:00:00Z"
    }
  ]
}
```

---

### GET /api/flows/:id

**Descrição:** Obter fluxo completo com nós e conexões.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `id`: UUID do fluxo

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Boas-vindas Automático",
    "description": "Envia mensagem de boas-vindas para novos contatos",
    "isActive": true,
    "nodes": [
      {
        "id": "node-1",
        "type": "trigger",
        "subtype": "NEW_CONTACT",
        "label": "Novo Contato",
        "position": { "x": 100, "y": 100 },
        "data": {
          "instanceId": "uuid"
        }
      },
      {
        "id": "node-2",
        "type": "action",
        "subtype": "SEND_MESSAGE",
        "label": "Enviar Boas-vindas",
        "position": { "x": 100, "y": 250 },
        "data": {
          "message": "Olá! Bem-vindo à nossa empresa. Como posso ajudar?",
          "delay": 5
        }
      },
      {
        "id": "node-3",
        "type": "condition",
        "subtype": "WAIT_RESPONSE",
        "label": "Aguardar Resposta",
        "position": { "x": 100, "y": 400 },
        "data": {
          "timeout": 3600,
          "timeoutUnit": "seconds"
        }
      }
    ],
    "connections": [
      {
        "id": "conn-1",
        "sourceId": "node-1",
        "targetId": "node-2",
        "sourceHandle": "output",
        "targetHandle": "input"
      },
      {
        "id": "conn-2",
        "sourceId": "node-2",
        "targetId": "node-3",
        "sourceHandle": "output",
        "targetHandle": "input"
      }
    ],
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-01-15T10:00:00Z"
  }
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Fluxo não encontrado"
}
```

---

### POST /api/flows

**Descrição:** Criar novo fluxo.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageFlows`

**Request Body:**

```json
{
  "name": "Reengajamento Automático",
  "description": "Reengage leads inativos após 7 dias",
  "isActive": false,
  "nodes": [
    {
      "id": "node-1",
      "type": "trigger",
      "subtype": "SCHEDULE",
      "label": "Agendamento Diário",
      "position": { "x": 100, "y": 100 },
      "data": {
        "schedule": "0 9 * * *",
        "timezone": "America/Sao_Paulo"
      }
    },
    {
      "id": "node-2",
      "type": "action",
      "subtype": "FILTER_CONTACTS",
      "label": "Filtrar Inativos",
      "position": { "x": 100, "y": 250 },
      "data": {
        "filter": {
          "lastMessageDaysAgo": 7,
          "excludeTags": ["Não Incomodar"]
        }
      }
    }
  ],
  "connections": [
    {
      "sourceId": "node-1",
      "targetId": "node-2",
      "sourceHandle": "output",
      "targetHandle": "input"
    }
  ]
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Sim | Nome do fluxo |
| description | string | Não | Descrição |
| isActive | boolean | Não | Ativo (default: false) |
| nodes | array | Sim | Lista de nós |
| connections | array | Sim | Lista de conexões |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Reengajamento Automático",
    "isActive": false,
    "nodeCount": 2,
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### PUT /api/flows/:id

**Descrição:** Atualizar fluxo existente.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageFlows`

**Request Body:**

```json
{
  "id": "uuid",
  "name": "Reengajamento Automático (Atualizado)",
  "nodes": [...],
  "connections": [...]
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Reengajamento Automático (Atualizado)",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "ID no corpo da requisição difere do ID da URL"
}
```

---

### POST /api/flows/:id/toggle

**Descrição:** Ativar ou desativar fluxo.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageFlows`

**Request Body:**

```json
{
  "isActive": true
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| isActive | boolean | Sim | Novo estado |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Boas-vindas Automático",
    "isActive": true,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Campo isActive inválido"
}
```

---

### POST /api/flows/:id/duplicate

**Descrição:** Duplicar fluxo existente.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageFlows`

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid-novo",
    "name": "Boas-vindas Automático (Cópia)",
    "isActive": false,
    "nodeCount": 5,
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Comportamento:**
- Cria cópia com nome + " (Cópia)"
- Novo fluxo fica inativo por padrão
- Todos os nós e conexões são copiados

---

### DELETE /api/flows/:id

**Descrição:** Excluir fluxo.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageFlows`

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Fluxo removido com sucesso"
}
```

**Comportamento:**
- Remove fluxo, nós e conexões
- Histórico de execuções é mantido

---

## Tipos de Nós

### Triggers (Gatilhos)

| Subtipo | Descrição | Dados |
|---------|-----------|-------|
| NEW_CONTACT | Novo contato criado | instanceId |
| MESSAGE_RECEIVED | Mensagem recebida | instanceId, keywords |
| TAG_ADDED | Tag adicionada | tagId |
| STAGE_CHANGED | Estágio do funil alterado | funnelId, stageId |
| DEAL_CREATED | Deal criado | funnelId |
| SCHEDULE | Agendamento (cron) | schedule, timezone |
| WEBHOOK | Webhook externo | webhookUrl |

### Actions (Ações)

| Subtipo | Descrição | Dados |
|---------|-----------|-------|
| SEND_MESSAGE | Enviar mensagem | message, delay |
| SEND_MEDIA | Enviar mídia | mediaUrl, caption |
| ADD_TAG | Adicionar tag | tagId |
| REMOVE_TAG | Remover tag | tagId |
| MOVE_TO_STAGE | Mover para estágio | stageId |
| CREATE_DEAL | Criar deal | title, value, stageId |
| UPDATE_CONTACT | Atualizar contato | fields |
| SEND_EMAIL | Enviar email | to, subject, body |
| CALL_WEBHOOK | Chamar webhook | url, method, body |
| NOTIFY_TEAM | Notificar equipe | message, userIds |

### Conditions (Condições)

| Subtipo | Descrição | Dados |
|---------|-----------|-------|
| IF_CONDITION | Condição simples | field, operator, value |
| WAIT_RESPONSE | Aguardar resposta | timeout, timeoutUnit |
| WAIT_TIME | Aguardar tempo | delay, unit |
| SPLIT | Divisão A/B | percentages |
| FILTER_CONTACTS | Filtrar contatos | filter |

---

## Estrutura de Nó

```json
{
  "id": "string",
  "type": "trigger | action | condition",
  "subtype": "SEND_MESSAGE | IF_CONDITION | ...",
  "label": "Nome do Nó",
  "position": {
    "x": 100,
    "y": 200
  },
  "data": {
    "// dados específicos do subtipo"
  }
}
```

## Estrutura de Conexão

```json
{
  "id": "string (opcional)",
  "sourceId": "id-do-nó-origem",
  "targetId": "id-do-nó-destino",
  "sourceHandle": "output | true | false",
  "targetHandle": "input"
}
```

**Handles Especiais:**
- `true`: Saída quando condição é verdadeira
- `false`: Saída quando condição é falsa
- `timeout`: Saída quando timeout é atingido

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Campo isActive inválido | Boolean esperado |
| 400 | ID difere do URL | Inconsistência de IDs |
| 400 | Erro ao criar fluxo | Validação de nós/conexões falhou |
| 403 | Sem permissão | Usuário sem `canManageFlows` |
| 404 | Fluxo não encontrado | ID de fluxo inválido |

---

## Exemplos de Fluxos

### Boas-vindas Simples

```json
{
  "name": "Boas-vindas",
  "nodes": [
    {
      "id": "trigger",
      "type": "trigger",
      "subtype": "NEW_CONTACT",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "message",
      "type": "action",
      "subtype": "SEND_MESSAGE",
      "data": { "message": "Olá! Bem-vindo!" }
    }
  ],
  "connections": [
    { "sourceId": "trigger", "targetId": "message" }
  ]
}
```

### Qualificação com Condição

```json
{
  "name": "Qualificação",
  "nodes": [
    {
      "id": "trigger",
      "type": "trigger",
      "subtype": "MESSAGE_RECEIVED",
      "data": { "keywords": ["preço", "valor", "quanto"] }
    },
    {
      "id": "condition",
      "type": "condition",
      "subtype": "IF_CONDITION",
      "data": { "field": "contact.tags", "operator": "contains", "value": "VIP" }
    },
    {
      "id": "vip-response",
      "type": "action",
      "subtype": "SEND_MESSAGE",
      "data": { "message": "Olá! Como cliente VIP, tenho uma oferta especial..." }
    },
    {
      "id": "normal-response",
      "type": "action",
      "subtype": "SEND_MESSAGE",
      "data": { "message": "Olá! Nossos preços são..." }
    }
  ],
  "connections": [
    { "sourceId": "trigger", "targetId": "condition" },
    { "sourceId": "condition", "targetId": "vip-response", "sourceHandle": "true" },
    { "sourceId": "condition", "targetId": "normal-response", "sourceHandle": "false" }
  ]
}
```

---

**Anterior:** [← Follow-ups](followups.md) | **Próximo:** [Dashboard →](dashboard.md)
