# 4.4 Funis

**Versão:** 1.0.0
**Última Atualização:** 2026-01-21

← [Voltar para Contratos de API](README.md)

---

## Funis

### POST /api/funnels

**Descrição:** Criar novo funil de vendas/atendimento.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Vendas B2B",
  "description": "Funil para vendas corporativas",
  "color": "#4F46E5"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Sim | Nome do funil (2-100 caracteres) |
| description | string | Não | Descrição do funil |
| color | string | Não | Cor em hexadecimal |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "publicId": 1,
    "name": "Vendas B2B",
    "description": "Funil para vendas corporativas",
    "color": "#4F46E5",
    "isDefault": false,
    "stages": [
      {
        "id": "uuid",
        "name": "Novo",
        "order": 1,
        "color": "#6B7280"
      }
    ],
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Comportamento:**
- Estágio inicial "Novo" é criado automaticamente
- Primeiro funil da empresa se torna `isDefault = true`

---

### GET /api/funnels

**Descrição:** Listar todos os funis da empresa.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "publicId": 1,
      "name": "Vendas B2B",
      "description": "Funil para vendas corporativas",
      "color": "#4F46E5",
      "isDefault": true,
      "stageCount": 5,
      "contactCount": 150,
      "dealCount": 45,
      "totalValue": 250000.00,
      "stages": [
        {
          "id": "uuid",
          "name": "Novo",
          "order": 1,
          "color": "#6B7280",
          "contactCount": 30,
          "dealCount": 10
        }
      ],
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### GET /api/funnels/:id

**Descrição:** Obter funil por ID com estágios e estatísticas.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `id`: UUID do funil

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "publicId": 1,
    "name": "Vendas B2B",
    "description": "Funil para vendas corporativas",
    "color": "#4F46E5",
    "isDefault": true,
    "stages": [
      {
        "id": "uuid",
        "name": "Novo",
        "order": 1,
        "color": "#6B7280",
        "contacts": [
          {
            "id": "uuid",
            "name": "João Silva",
            "phoneNumber": "5511999999999"
          }
        ],
        "deals": [
          {
            "id": "uuid",
            "title": "Proposta Comercial",
            "value": 5000.00,
            "contact": {
              "id": "uuid",
              "name": "João Silva"
            }
          }
        ]
      }
    ],
    "createdAt": "2026-01-01T00:00:00Z"
  }
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Funil não encontrado"
}
```

---

### GET /api/funnels/:id/search

**Descrição:** Buscar deals dentro de um funil.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Descrição |
|-----------|------|-----------|
| q | string | Termo de busca (mínimo 2 caracteres) |

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Proposta Comercial",
      "value": 5000.00,
      "status": "OPEN",
      "contact": {
        "id": "uuid",
        "name": "João Silva"
      },
      "stage": {
        "id": "uuid",
        "name": "Negociação"
      }
    }
  ]
}
```

---

### GET /api/funnels/:id/stats

**Descrição:** Obter estatísticas do funil.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "totalContacts": 150,
    "totalDeals": 45,
    "totalValue": 250000.00,
    "wonDeals": 20,
    "wonValue": 100000.00,
    "lostDeals": 10,
    "lostValue": 50000.00,
    "openDeals": 15,
    "openValue": 100000.00,
    "conversionRate": 66.67,
    "averageDealValue": 5555.56,
    "averageCycleTime": 14.5,
    "stageStats": [
      {
        "stageId": "uuid",
        "stageName": "Novo",
        "contactCount": 30,
        "dealCount": 10,
        "dealValue": 50000.00
      }
    ]
  }
}
```

**Observações de cálculo:**
- `conversionRate` deve representar ganhos / negociações criadas no período filtrado.
- Para UI, exibir a base usada (ex.: "12 / 100") e tooltip com a fórmula.

---

### PUT /api/funnels/:id

**Descrição:** Atualizar funil.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Vendas B2B Atualizado",
  "description": "Nova descrição",
  "isDefault": true
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Vendas B2B Atualizado",
    "description": "Nova descrição",
    "isDefault": true,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Regras:**
- Se `isDefault = true`, outros funis perdem o flag

---

### DELETE /api/funnels/:id

**Descrição:** Excluir funil.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Funil removido"
}
```

**Regras:**
- Não pode excluir funil com `isDefault = true`
- Remove todos os estágios associados
- Contatos no funil perdem a associação (stageId = null)

---

## Estágios

### GET /api/funnels/stages

**Descrição:** Listar todos os estágios de todos os funis.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Novo",
      "order": 1,
      "color": "#6B7280",
      "funnel": {
        "id": "uuid",
        "name": "Vendas B2B"
      }
    },
    {
      "id": "uuid",
      "name": "Qualificação",
      "order": 2,
      "color": "#3B82F6",
      "funnel": {
        "id": "uuid",
        "name": "Vendas B2B"
      }
    }
  ]
}
```

---

### GET /api/funnels/:id/stages

**Descrição:** Listar estágios de um funil específico.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Novo",
      "order": 1,
      "color": "#6B7280",
      "contactCount": 30,
      "dealCount": 10
    },
    {
      "id": "uuid",
      "name": "Qualificação",
      "order": 2,
      "color": "#3B82F6",
      "contactCount": 25,
      "dealCount": 8
    }
  ]
}
```

---

### POST /api/funnels/:id/stages

**Descrição:** Criar novo estágio no funil.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Proposta Enviada",
  "description": "Etapa para proposta enviada e aguardando resposta",
  "color": "#10B981",
  "order": 3
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Sim | Nome do estágio |
| description | string | Não | Descrição do estágio (opcional) |
| color | string | Não | Cor em hexadecimal |
| order | number | Não | Posição no funil (auto-incrementa) |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Proposta Enviada",
    "color": "#10B981",
    "order": 3,
    "funnelId": "uuid",
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### PUT /api/funnels/stages/:stageId

**Descrição:** Atualizar estágio.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Proposta Enviada (Atualizado)",
  "description": "Etapa com proposta revisada",
  "color": "#059669"
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Proposta Enviada (Atualizado)",
    "color": "#059669",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### DELETE /api/funnels/stages/:stageId

**Descrição:** Excluir estágio.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Etapa removida"
}
```

**Regras:**
- Não pode excluir se for o único estágio do funil
- Contatos e deals são movidos para o estágio anterior ou primeiro

---

### POST /api/funnels/:id/stages/reorder

**Descrição:** Reordenar estágios do funil.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "stageIds": ["uuid-3", "uuid-1", "uuid-2", "uuid-4"]
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| stageIds | string[] | Sim | IDs dos estágios na nova ordem |

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Etapas reordenadas"
}
```

---

## Movimentação

### POST /api/funnels/move-to-stage

**Descrição:** Mover contato para outro estágio.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "contactId": "uuid",
  "stageId": "uuid"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| contactId | string | Sim | UUID do contato |
| stageId | string | Sim | UUID do estágio destino |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "João Silva",
    "currentStage": {
      "id": "uuid",
      "name": "Qualificação",
      "funnelId": "uuid"
    },
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Comportamento:**
- Registra histórico de movimentação
- Dispara eventos para automações

---

### POST /api/funnels/move-to-funnel

**Descrição:** Mover contato para outro funil.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "contactId": "uuid",
  "funnelId": "uuid",
  "stageId": "uuid (opcional)"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| contactId | string | Sim | UUID do contato |
| funnelId | string | Sim | UUID do funil destino |
| stageId | string | Não | UUID do estágio destino (default: primeiro) |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "João Silva",
    "currentStage": {
      "id": "uuid",
      "name": "Novo",
      "funnelId": "uuid"
    },
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### GET /api/funnels/:id/available-contacts

**Descrição:** Listar contatos disponíveis para adicionar ao funil.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Maria Santos",
      "phoneNumber": "5511888888888",
      "currentStage": null
    },
    {
      "id": "uuid",
      "name": "Carlos Lima",
      "phoneNumber": "5511777777777",
      "currentStage": {
        "id": "uuid",
        "name": "Outro Funil",
        "funnelId": "outro-uuid"
      }
    }
  ]
}
```

**Comportamento:**
- Retorna contatos sem estágio ou em outros funis
- Útil para adicionar contatos manualmente ao funil

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Nome já existe | Funil com mesmo nome na empresa |
| 400 | Não pode excluir funil padrão | Tentativa de excluir funil default |
| 400 | Estágio único | Tentativa de excluir único estágio |
| 404 | Funil não encontrado | ID de funil inválido |
| 404 | Estágio não encontrado | ID de estágio inválido |
| 404 | Contato não encontrado | ID de contato inválido |

---

**Anterior:** [← Contatos](contacts.md) | **Próximo:** [Deals →](deals.md)
