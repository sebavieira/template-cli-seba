# 4.5 Deals

**Versão:** 1.0.0
**Última Atualização:** 2026-01-21

← [Voltar para Contratos de API](README.md)

---

## Deals (Negociações)

### POST /api/deals

**Descrição:** Criar nova negociação.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "title": "Proposta Comercial - Empresa XYZ",
  "value": 15000.00,
  "contactId": "uuid",
  "stageId": "uuid",
  "expectedCloseDate": "2026-02-15",
  "description": "Proposta para implementação de sistema"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| title | string | Sim | Título da negociação |
| value | number | Não | Valor monetário (default: 0) |
| contactId | string | Sim | UUID do contato associado |
| stageId | string | Sim | UUID do estágio inicial |
| expectedCloseDate | string | Não | Data prevista de fechamento (ISO 8601) |
| description | string | Não | Descrição detalhada |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "publicId": 1001,
    "title": "Proposta Comercial - Empresa XYZ",
    "value": 15000.00,
    "status": "OPEN",
    "contact": {
      "id": "uuid",
      "name": "João Silva",
      "phoneNumber": "5511999999999"
    },
    "stage": {
      "id": "uuid",
      "name": "Novo",
      "funnelId": "uuid"
    },
    "expectedCloseDate": "2026-02-15T00:00:00Z",
    "createdBy": {
      "id": "uuid",
      "fullName": "Vendedor"
    },
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Contato ou estágio não encontrado"
}
```

---

### GET /api/deals/:id

**Descrição:** Obter deal por ID com detalhes completos.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `id`: UUID do deal

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "publicId": 1001,
    "title": "Proposta Comercial - Empresa XYZ",
    "value": 15000.00,
    "status": "OPEN",
    "description": "Proposta para implementação de sistema",
    "clonedFromDealId": null,
    "contact": {
      "id": "uuid",
      "name": "João Silva",
      "phoneNumber": "5511999999999",
      "email": "joao@empresa.com"
    },
    "stage": {
      "id": "uuid",
      "name": "Negociação",
      "order": 3,
      "funnel": {
        "id": "uuid",
        "name": "Vendas B2B"
      }
    },
    "expectedCloseDate": "2026-02-15T00:00:00Z",
    "closedAt": null,
    "lostReason": null,
    "createdBy": {
      "id": "uuid",
      "fullName": "Vendedor"
    },
    "activities": [
      {
        "id": "uuid",
        "type": "STAGE_CHANGED",
        "description": "Movido de 'Novo' para 'Negociação'",
        "createdAt": "2026-01-18T10:00:00Z"
      }
    ],
    "noteCount": 3,
    "attachmentCount": 2,
    "createdAt": "2026-01-15T10:00:00Z",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Negocio nao encontrado"
}
```

---

### PUT /api/deals/:id

**Descrição:** Atualizar deal.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "title": "Proposta Comercial - Empresa XYZ (Revisada)",
  "value": 18000.00,
  "expectedCloseDate": "2026-02-28",
  "status": "WON",
  "lostReason": null
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| title | string | Não | Título da negociação |
| value | number | Não | Valor monetário |
| expectedCloseDate | string | Não | Data prevista de fechamento |
| status | string | Não | OPEN, WON, LOST |
| lostReason | string | Não | Motivo da perda (se status = LOST) |
| description | string | Não | Descrição detalhada |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Proposta Comercial - Empresa XYZ (Revisada)",
    "value": 18000.00,
    "status": "WON",
    "closedAt": "2026-01-19T10:00:00Z",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Status Possíveis:**

| Status | Descrição |
|--------|-----------|
| OPEN | Negociação em andamento |
| WON | Negociação ganha/fechada |
| LOST | Negociação perdida |

**Comportamento:**
- Ao mudar para WON ou LOST, `closedAt` é preenchido automaticamente
- Registra atividade de mudança de status

---

### DELETE /api/deals/:id

**Descrição:** Excluir deal.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Negociacao removida"
}
```

**Regras:**
- Remove deal e todas as notas/atividades associadas
- Anexos são removidos do storage
- Não afeta o contato associado

---

### POST /api/deals/move

**Descrição:** Mover deal para outro estágio.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "dealId": "uuid",
  "stageId": "uuid"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| dealId | string | Sim | UUID do deal |
| stageId | string | Sim | UUID do estágio destino |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Proposta Comercial",
    "stage": {
      "id": "uuid",
      "name": "Fechamento",
      "order": 4
    },
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

**Comportamento:**
- Registra atividade STAGE_CHANGED
- Dispara eventos para automações
- Estágio deve pertencer ao mesmo funil do deal

---

### POST /api/deals/duplicate

**Descrição:** Copiar deal para outro funil/etapa sem alterar o original.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "dealId": "uuid",
  "funnelId": "uuid",
  "stageId": "uuid"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| dealId | string | Sim | UUID do deal de origem |
| funnelId | string | Sim | UUID do funil destino |
| stageId | string | Sim | UUID do estágio destino |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Proposta Comercial",
    "status": "OPEN",
    "funnelId": "uuid",
    "stageId": "uuid",
    "clonedFromDealId": "uuid",
    "createdAt": "2026-01-21T10:00:00Z"
  }
}
```

**Comportamento:**
- O deal original permanece inalterado
- Registra atividade de clonagem nos dois deals
- Dispara eventos para automações do deal novo

---

## Notas do Deal

### GET /api/deals/:id/notes

**Descrição:** Listar notas do deal.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Reunião de Alinhamento",
      "content": "Cliente solicitou prazo de 30 dias para pagamento.",
      "contentHtml": "<p>Cliente solicitou prazo de 30 dias para pagamento.</p>",
      "createdBy": {
        "id": "uuid",
        "fullName": "Vendedor"
      },
      "createdAt": "2026-01-18T10:00:00Z",
      "updatedAt": "2026-01-18T10:00:00Z"
    }
  ]
}
```

---

### POST /api/deals/:id/notes

**Descrição:** Adicionar nota ao deal.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "title": "Follow-up pós-reunião",
  "content": "Enviar proposta revisada até sexta-feira.",
  "contentHtml": "<p>Enviar proposta revisada até <strong>sexta-feira</strong>.</p>"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| title | string | Não | Título da nota |
| content | string | Condicional | Conteúdo em texto plano |
| contentHtml | string | Condicional | Conteúdo em HTML (rich text) |

**Regras:**
- Pelo menos `content`, `contentHtml` ou `title` deve ser fornecido

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Follow-up pós-reunião",
    "content": "Enviar proposta revisada até sexta-feira.",
    "contentHtml": "<p>Enviar proposta revisada até <strong>sexta-feira</strong>.</p>",
    "createdBy": {
      "id": "uuid",
      "fullName": "Vendedor"
    },
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### PUT /api/deals/:id/notes/:noteId

**Descrição:** Atualizar nota do deal.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "title": "Follow-up pós-reunião (Atualizado)",
  "content": "Proposta enviada. Aguardando retorno.",
  "contentHtml": "<p>Proposta enviada. <em>Aguardando retorno.</em></p>"
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Follow-up pós-reunião (Atualizado)",
    "content": "Proposta enviada. Aguardando retorno.",
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### DELETE /api/deals/:id/notes/:noteId

**Descrição:** Excluir nota do deal.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Nota removida"
}
```

---

## Anexos do Deal

### GET /api/deals/:id/attachments

**Descrição:** Listar anexos do deal.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "fileName": "proposta-comercial.pdf",
      "fileUrl": "/uploads/deals/uuid/1705665600-proposta-comercial.pdf",
      "fileSize": 245000,
      "mimeType": "application/pdf",
      "storageProvider": "local",
      "createdBy": {
        "id": "uuid",
        "fullName": "Vendedor"
      },
      "createdAt": "2026-01-19T10:00:00Z"
    }
  ]
}
```

---

### POST /api/deals/:id/attachments

**Descrição:** Upload de anexo para o deal.

**Autenticação:** Bearer token (JWT)

**Content-Type:** `multipart/form-data`

**Request Body (FormData):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| file | File | Sim | Arquivo a ser enviado |

**Tipos Suportados:**
- Documentos: PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX
- Imagens: PNG, JPG, JPEG, GIF, WEBP
- Outros: TXT, CSV, ZIP

**Limite de Tamanho:** 50MB por arquivo

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "fileName": "contrato-assinado.pdf",
    "fileUrl": "/uploads/deals/uuid/1705665600-contrato-assinado.pdf",
    "fileKey": "deals/uuid/1705665600-contrato-assinado.pdf",
    "fileSize": 512000,
    "mimeType": "application/pdf",
    "storageProvider": "local",
    "createdBy": {
      "id": "uuid",
      "fullName": "Vendedor"
    },
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Arquivo obrigatorio"
}
```

---

### DELETE /api/deals/:id/attachments/:attachmentId

**Descrição:** Excluir anexo do deal.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Anexo removido"
}
```

**Comportamento:**
- Remove registro do banco
- Remove arquivo do storage (local ou S3)

---

## Atividades do Deal

As atividades são geradas automaticamente pelo sistema quando ocorrem eventos relevantes.

**Tipos de Atividade:**

| Tipo | Descrição |
|------|-----------|
| CREATED | Deal foi criado |
| UPDATED | Deal foi atualizado |
| STAGE_CHANGED | Deal mudou de estágio |
| STATUS_CHANGED | Status mudou (OPEN → WON/LOST) |
| NOTE_ADDED | Nota foi adicionada |
| ATTACHMENT_ADDED | Anexo foi adicionado |
| VALUE_CHANGED | Valor foi alterado |

**Estrutura da Atividade:**

```json
{
  "id": "uuid",
  "dealId": "uuid",
  "type": "STAGE_CHANGED",
  "description": "Movido de 'Qualificação' para 'Negociação'",
  "metadata": {
    "fromStageId": "uuid-1",
    "fromStageName": "Qualificação",
    "toStageId": "uuid-2",
    "toStageName": "Negociação"
  },
  "createdBy": {
    "id": "uuid",
    "fullName": "Vendedor"
  },
  "createdAt": "2026-01-19T10:00:00Z"
}
```

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Contato não encontrado | contactId inválido |
| 400 | Estágio não encontrado | stageId inválido |
| 400 | Arquivo obrigatório | Upload sem arquivo |
| 400 | Nada para atualizar | PUT sem dados válidos |
| 404 | Negócio não encontrado | ID de deal inválido |
| 404 | Nota não encontrada | ID de nota inválido |
| 404 | Anexo não encontrado | ID de anexo inválido |

---

**Anterior:** [← Funis](funnels.md) | **Próximo:** [IA/Prompts →](ai.md)
