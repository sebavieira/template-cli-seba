# 4.2 WhatsApp

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para Contratos de API](README.md)

---

## Mensagens

### POST /api/whatsapp/send

**Descrição:** Enviar mensagem de texto via WhatsApp.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "instanceId": "uuid",
  "to": "5511999999999",
  "text": "Olá! Como posso ajudar?",
  "contactId": "uuid (opcional)",
  "contactName": "João Silva (opcional)",
  "replyToMessageId": "message-key (opcional)",
  "replyToMessageText": "texto original (opcional)"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| instanceId | string | Sim | UUID da instância WhatsApp |
| to | string | Sim | Número do destinatário (sem @s.whatsapp.net) |
| text | string | Sim | Texto da mensagem |
| contactId | string | Não | ID do contato (para vincular) |
| contactName | string | Não | Nome do contato |
| replyToMessageId | string | Não | ID da mensagem sendo respondida |
| replyToMessageText | string | Não | Texto da mensagem original |

**Response 200 OK:**

```json
{
  "success": true,
  "messageId": "BAE5CF1234567890",
  "message": {
    "id": "uuid",
    "publicId": 123,
    "conversationId": "uuid",
    "messageText": "Olá! Como posso ajudar?",
    "messageType": "SENT",
    "status": "SENT",
    "isBot": false,
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Missing required fields: instanceId, to, text"
}
```

---

### POST /api/whatsapp/send-media

**Descrição:** Enviar mídia (imagem, vídeo, áudio, documento) via WhatsApp.

**Autenticação:** Bearer token (JWT)

**Content-Type:** `multipart/form-data`

**Request Body (FormData):**

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| file | File | Sim | Arquivo de mídia |
| instanceId | string | Sim | UUID da instância |
| to | string | Sim | Número do destinatário |
| caption | string | Não | Legenda da mídia |
| contactId | string | Não | ID do contato |
| contactName | string | Não | Nome do contato |
| mediaType | string | Não | Tipo: image, video, audio, document |

**Tipos de Mídia Suportados:**

| Tipo | Extensões | MIME Types |
|------|-----------|------------|
| image | .png, .jpg, .jpeg, .gif, .webp | image/* |
| video | .mp4, .mov, .avi, .mkv, .webm | video/* |
| audio | .mp3, .wav, .ogg, .m4a | audio/* |
| document | * | application/* |

**Response 200 OK:**

```json
{
  "success": true,
  "messageId": "BAE5CF1234567890",
  "message": {
    "id": "uuid",
    "mediaUrl": "https://storage.../file.jpg",
    "mediaType": "image",
    "mediaMimeType": "image/jpeg",
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### POST /api/whatsapp/send-reaction

**Descrição:** Enviar reação a uma mensagem.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "instanceId": "uuid",
  "messageId": "BAE5CF1234567890",
  "reaction": "👍",
  "conversationId": "uuid (opcional)"
}
```

**Response 200 OK:**

```json
{
  "success": true
}
```

---

## Conversas

### GET /api/whatsapp/conversations

**Descrição:** Listar conversas do WhatsApp.

**Autenticação:** Bearer token (JWT)

**Query Parameters:**

| Parâmetro | Tipo | Default | Descrição |
|-----------|------|---------|-----------|
| page | number | 1 | Página |
| limit | number | 20 | Itens por página |
| search | string | - | Busca por nome/número |
| isArchived | boolean | false | Listar arquivadas |
| status | string | - | 'active' ou 'paused' |
| unreadOnly | boolean | false | Apenas não lidas |
| tagIds | string | - | IDs de tags (separados por vírgula) |
| funnelId | string | - | Filtrar por funil |
| stageId | string | - | Filtrar por estágio |
| instanceId | string | - | Filtrar por instância |
| sentiments | string | - | Sentimentos (separados por vírgula) |
| urgencies | string | - | Urgências (separados por vírgula) |
| match | string | all | 'all' ou 'any' para filtros |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "conversations": [
      {
        "id": "uuid",
        "publicId": 123,
        "contactNumber": "5511999999999",
        "contactName": "João Silva",
        "isPaused": false,
        "isArchived": false,
        "lastMessageAt": "2026-01-19T10:00:00Z",
        "contact": {
          "id": "uuid",
          "name": "João Silva",
          "email": "joao@email.com",
          "tags": [
            { "id": "uuid", "name": "VIP", "color": "#FF0000" }
          ]
        },
        "lastMessage": {
          "messageText": "Última mensagem...",
          "messageType": "RECEIVED",
          "createdAt": "2026-01-19T10:00:00Z"
        },
        "aiState": {
          "sentiment": "positive",
          "urgency": "low"
        },
        "unreadCount": 3
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150
    }
  }
}
```

---

### GET /api/whatsapp/conversations/:id/messages

**Descrição:** Listar mensagens de uma conversa.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `id`: UUID da conversa

**Query Parameters:**

| Parâmetro | Tipo | Default | Descrição |
|-----------|------|---------|-----------|
| page | number | 1 | Página |
| limit | number | 50 | Mensagens por página |
| before | string | - | Cursor para paginação |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "uuid",
        "publicId": 456,
        "messageText": "Olá!",
        "messageType": "RECEIVED",
        "status": "RECEIVED",
        "isBot": false,
        "messageKeyId": "BAE5CF1234567890",
        "mediaUrl": null,
        "mediaType": null,
        "reactions": { "👍": 1 },
        "aiMetadata": {
          "actions_performed": []
        },
        "createdAt": "2026-01-19T10:00:00Z"
      }
    ],
    "hasMore": true
  }
}
```

---

### POST /api/whatsapp/conversations/:id/read

**Descrição:** Marcar mensagens da conversa como lidas.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "updated": true
  }
}
```

---

### POST /api/whatsapp/conversations/:id/toggle-pause

**Descrição:** Pausar ou retomar IA na conversa.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "isPaused": true
  }
}
```

---

### POST /api/whatsapp/conversations/:id/archive

**Descrição:** Arquivar conversa.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "isArchived": true
  }
}
```

---

### DELETE /api/whatsapp/conversations/:id

**Descrição:** Excluir conversa e todas as mensagens.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "deleted": true
  }
}
```

**Regras:**
- Exclui conversa do banco
- Exclui todas as mensagens associadas
- Não é possível restaurar

---

## Instâncias

### GET /api/whatsapp/instances

**Descrição:** Listar instâncias WhatsApp da empresa.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "publicId": 1,
      "instanceName": "vendas-principal",
      "provider": "EVOLUTION",
      "isConnected": true,
      "connectionStatus": "open",
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### POST /api/whatsapp/instances

**Descrição:** Criar nova instância WhatsApp.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "instanceName": "atendimento-01",
  "provider": "EVOLUTION"
}
```

| Campo | Tipo | Obrigatório | Valores |
|-------|------|-------------|---------|
| instanceName | string | Sim | 3-50 caracteres |
| provider | string | Não | EVOLUTION (default), UAZAPI |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "instanceName": "atendimento-01",
    "provider": "EVOLUTION",
    "isConnected": false,
    "connectionStatus": "close"
  }
}
```

**Regras:**
- Nome deve ser único por usuário
- Limite de instâncias conforme plano (`maxWhatsappInstances`)
- Credencial do provedor deve existir

---

### POST /api/whatsapp/instances/:id/connect

**Descrição:** Conectar instância e obter QR Code.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "qrcode": "data:image/png;base64,iVBORw0KGgoAAAANSUh...",
    "status": "qrcode"
  }
}
```

**Estados de Conexão:**

| Status | Descrição |
|--------|-----------|
| qrcode | Aguardando escaneamento |
| connecting | Conectando... |
| open | Conectado |
| close | Desconectado |

---

### POST /api/whatsapp/instances/:id/check

**Descrição:** Verificar status de conexão.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "isConnected": true,
    "connectionStatus": "open",
    "phoneNumber": "5511999999999"
  }
}
```

---

### DELETE /api/whatsapp/instances/:id

**Descrição:** Excluir instância WhatsApp.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "deleted": true
  }
}
```

**Regras:**
- Desconecta da Evolution/UAZAPI
- Mantém conversas e mensagens no histórico
- Remove instância do banco

---

## Credenciais do Provedor

### GET /api/whatsapp/provider-credentials

**Descrição:** Listar credenciais de provedores WhatsApp.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "provider": "EVOLUTION",
      "apiUrl": "https://evolution.exemplo.com",
      "isActive": true,
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### POST /api/whatsapp/provider-credentials

**Descrição:** Criar credencial de provedor.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "provider": "EVOLUTION",
  "apiUrl": "https://evolution.exemplo.com",
  "apiKey": "sua-api-key-secreta"
}
```

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| provider | string | Sim |
| apiUrl | string | Sim |
| apiKey | string | Sim |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "provider": "EVOLUTION",
    "apiUrl": "https://evolution.exemplo.com",
    "isActive": true
  }
}
```

**Regras:**
- `apiKey` é criptografado no banco
- Apenas uma credencial ativa por provedor por empresa
- Validação de conexão realizada na criação

---

### DELETE /api/whatsapp/provider-credentials/:id

**Descrição:** Excluir credencial de provedor.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "deleted": true
}
```

---

## Webhooks

### POST /api/whatsapp/webhook

**Descrição:** Receber eventos do provedor WhatsApp (Evolution/UAZAPI).

**Autenticação:** Não requerida (webhook público)

**Rate Limit:** 1000 req/min

**Body Limit:** 50MB

**Request Body (Evolution API):**

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5CF1234567890"
    },
    "pushName": "João Silva",
    "message": {
      "conversation": "Olá, tudo bem?"
    }
  }
}
```

**Eventos Processados:**

| Evento | Ação |
|--------|------|
| messages.upsert | Cria/atualiza mensagem, dispara IA |
| connection.update | Atualiza status de conexão |
| qrcode.updated | Armazena novo QR Code |
| messages.update | Atualiza status de entrega |

**Response 200 OK:**

```json
{
  "success": true,
  "processed": true,
  "reason": "message_created"
}
```

**Regras:**
- Sempre retorna 200 para evitar retries
- Mensagens duplicadas são ignoradas (por `messageKeyId`)
- IA é disparada assincronamente via BullMQ

---

**Anterior:** [← Autenticação](auth.md) | **Próximo:** [Contatos →](contacts.md)
