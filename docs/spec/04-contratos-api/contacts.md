# 4.3 Contatos

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para Contratos de API](README.md)

---

## Contatos

### POST /api/contacts

**Descrição:** Criar novo contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Request Body:**

```json
{
  "name": "João Silva",
  "phoneNumber": "5511999999999",
  "email": "joao@email.com",
  "notes": "Cliente VIP",
  "customFields": {
    "empresa": "Acme Corp",
    "cargo": "Gerente"
  }
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Não | Nome do contato |
| phoneNumber | string | Sim | Número de telefone (formato internacional) |
| email | string | Não | Email do contato |
| notes | string | Não | Observações gerais |
| customFields | object | Não | Campos personalizados (JSONB) |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "publicId": 123,
    "name": "João Silva",
    "phoneNumber": "5511999999999",
    "email": "joao@email.com",
    "isBlocked": false,
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Número de telefone já cadastrado"
}
```

---

### GET /api/contacts

**Descrição:** Listar contatos com filtros.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Default | Descrição |
|-----------|------|---------|-----------|
| search | string | - | Busca por nome, email ou telefone |
| funnelId | string | - | Filtrar por funil |
| stageId | string | - | Filtrar por estágio |
| tagIds | string | - | IDs de tags (separados por vírgula) |
| page | number | 1 | Página |
| limit | number | 50 | Itens por página |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "contacts": [
      {
        "id": "uuid",
        "publicId": 123,
        "name": "João Silva",
        "phoneNumber": "5511999999999",
        "email": "joao@email.com",
        "isBlocked": false,
        "tags": [
          { "id": "uuid", "name": "VIP", "color": "#FF0000" }
        ],
        "currentStage": {
          "id": "uuid",
          "name": "Qualificação",
          "funnelId": "uuid"
        },
        "aiProfile": {
          "sentiment": "positive",
          "urgency": "low",
          "interests": ["produto-x", "suporte"]
        },
        "createdAt": "2026-01-19T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 150
    }
  }
}
```

---

### GET /api/contacts/:id

**Descrição:** Obter contato por ID.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `id`: UUID do contato

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "publicId": 123,
    "name": "João Silva",
    "phoneNumber": "5511999999999",
    "email": "joao@email.com",
    "notes": "Cliente VIP",
    "isBlocked": false,
    "customFields": {
      "empresa": "Acme Corp"
    },
    "tags": [],
    "conversations": [
      {
        "id": "uuid",
        "instanceId": "uuid",
        "lastMessageAt": "2026-01-19T10:00:00Z"
      }
    ],
    "deals": [
      {
        "id": "uuid",
        "title": "Proposta Comercial",
        "value": 5000.00,
        "status": "OPEN"
      }
    ],
    "createdAt": "2026-01-19T10:00:00Z",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Contato não encontrado"
}
```

---

### PUT /api/contacts/:id

**Descrição:** Atualizar contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Request Body:**

```json
{
  "name": "João Silva Atualizado",
  "email": "joao.novo@email.com",
  "notes": "Atualizado em 2026"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Não | Nome do contato |
| email | string | Não | Email do contato |
| notes | string | Não | Observações |
| customFields | object | Não | Campos personalizados |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "João Silva Atualizado",
    "email": "joao.novo@email.com",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### DELETE /api/contacts/:id

**Descrição:** Excluir contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Contato removido"
}
```

**Regras:**
- Remove contato do banco
- Conversas e mensagens são mantidas (contactId = null)
- Notas do contato são removidas

---

### POST /api/contacts/:id/toggle-block

**Descrição:** Bloquear ou desbloquear contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "isBlocked": true,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Comportamento:**
- Contato bloqueado não recebe respostas da IA
- Mensagens recebidas são armazenadas mas ignoradas

---

## Notas do Contato

### POST /api/contacts/:id/notes

**Descrição:** Adicionar nota ao contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Request Body:**

```json
{
  "content": "Cliente solicitou desconto de 10% na próxima compra."
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| content | string | Sim | Conteúdo da nota |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "content": "Cliente solicitou desconto de 10% na próxima compra.",
    "createdBy": {
      "id": "uuid",
      "fullName": "Atendente"
    },
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### GET /api/contacts/:id/notes

**Descrição:** Listar notas do contato.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "content": "Cliente solicitou desconto...",
      "createdBy": {
        "id": "uuid",
        "fullName": "Atendente"
      },
      "createdAt": "2026-01-19T10:00:00Z"
    }
  ]
}
```

---

### DELETE /api/contacts/:id/notes/:noteId

**Descrição:** Excluir nota do contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Nota removida"
}
```

---

### GET /api/contacts/:id/history

**Descrição:** Obter histórico completo do contato.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "conversations": [
      {
        "id": "uuid",
        "instanceName": "vendas-01",
        "messageCount": 45,
        "lastMessageAt": "2026-01-19T10:00:00Z"
      }
    ],
    "deals": [
      {
        "id": "uuid",
        "title": "Proposta Comercial",
        "value": 5000.00,
        "status": "WON",
        "closedAt": "2026-01-15T10:00:00Z"
      }
    ],
    "stageHistory": [
      {
        "stage": "Qualificação",
        "funnel": "Vendas B2B",
        "enteredAt": "2026-01-10T10:00:00Z",
        "exitedAt": "2026-01-12T10:00:00Z"
      }
    ],
    "followups": [
      {
        "id": "uuid",
        "rule": "Reengajamento 7 dias",
        "status": "SENT",
        "sentAt": "2026-01-18T10:00:00Z"
      }
    ]
  }
}
```

---

## Tags

### GET /api/contacts/tags/list

**Descrição:** Listar todas as tags da empresa.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "VIP",
      "color": "#FF0000",
      "description": "Clientes premium",
      "contactCount": 25,
      "createdAt": "2026-01-01T00:00:00Z"
    },
    {
      "id": "uuid",
      "name": "Prospect",
      "color": "#00FF00",
      "description": "Leads em prospecção",
      "contactCount": 150,
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### POST /api/contacts/tags

**Descrição:** Criar nova tag.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Request Body:**

```json
{
  "name": "Lead Quente",
  "color": "#FF5500",
  "description": "Leads com alta probabilidade de conversão"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Sim | Nome da tag (único por empresa) |
| color | string | Não | Cor em hexadecimal (default: #6B7280) |
| description | string | Não | Descrição da tag |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Lead Quente",
    "color": "#FF5500",
    "description": "Leads com alta probabilidade de conversão",
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Tag com este nome já existe"
}
```

---

### PUT /api/contacts/tags/:id

**Descrição:** Atualizar tag.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Request Body:**

```json
{
  "name": "Lead Quente Atualizado",
  "color": "#FF6600"
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Lead Quente Atualizado",
    "color": "#FF6600",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### POST /api/contacts/tags/merge

**Descrição:** Mesclar múltiplas tags em uma.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Request Body:**

```json
{
  "sourceTagIds": ["uuid-1", "uuid-2"],
  "targetTagId": "uuid-3"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| sourceTagIds | string[] | Sim | IDs das tags a serem mescladas |
| targetTagId | string | Sim | ID da tag destino |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "merged": 2,
    "contactsUpdated": 45,
    "targetTag": {
      "id": "uuid-3",
      "name": "Tag Consolidada",
      "contactCount": 70
    }
  }
}
```

**Comportamento:**
- Contatos das tags fonte recebem a tag destino
- Tags fonte são removidas
- Não duplica atribuições de tag

---

### DELETE /api/contacts/tags/:id

**Descrição:** Excluir tag.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Tag removida"
}
```

**Regras:**
- Remove tag do banco
- Remove atribuições aos contatos
- Não afeta os contatos em si

---

### POST /api/contacts/:id/tags/:tagId

**Descrição:** Atribuir tag a um contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Tag atribuída"
}
```

**Regras:**
- Ignora se já atribuída
- Cria registro em `contact_tag_assignments`

---

### DELETE /api/contacts/:id/tags/:tagId

**Descrição:** Remover tag de um contato.

**Autenticação:** Bearer token (JWT)

**Permissão:** `canManageContacts`

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Tag removida do contato"
}
```

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Número já cadastrado | Tentativa de criar contato com telefone existente |
| 400 | Tag já existe | Nome de tag duplicado na empresa |
| 404 | Contato não encontrado | ID de contato inválido |
| 404 | Tag não encontrada | ID de tag inválido |
| 403 | Sem permissão | Usuário sem `canManageContacts` |

---

**Anterior:** [← WhatsApp](whatsapp.md) | **Próximo:** [Funis →](funnels.md)
