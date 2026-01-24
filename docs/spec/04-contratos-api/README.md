# 4. Contratos de API

**Versão:** 1.0.0
**Última Atualização:** 2026-01-21

← [Voltar para SPEC](../README.md)

---

## Visão Geral

API REST com autenticação via JWT Bearer token.

### Base URL

```
Development: http://localhost:3000/api
Production:  https://api.evoaiconnect.com/api
```

### Autenticação

Todas as rotas (exceto auth públicas) requerem header:

```
Authorization: Bearer {jwt_token}
```

### Multi-Tenant

Todas as operações são isoladas por `companyId`. O sistema extrai automaticamente o `companyId` do token JWT.

---

## Índice de Endpoints

### Autenticação (6 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/auth/register` | Criar conta |
| POST | `/auth/login` | Fazer login |
| POST | `/auth/logout` | Fazer logout |
| POST | `/auth/refresh` | Renovar token |
| POST | `/auth/switch-company` | Trocar empresa ativa |
| GET | `/auth/me` | Obter perfil atual |

### WhatsApp (18 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/whatsapp/webhook` | Receber eventos (webhook) |
| POST | `/whatsapp/send` | Enviar mensagem |
| POST | `/whatsapp/send-media` | Enviar mídia |
| POST | `/whatsapp/send-reaction` | Enviar reação |
| GET | `/whatsapp/conversations` | Listar conversas |
| GET | `/whatsapp/conversations/:id/messages` | Listar mensagens |
| POST | `/whatsapp/conversations/:id/read` | Marcar como lida |
| POST | `/whatsapp/conversations/:id/toggle-pause` | Pausar/retomar IA |
| POST | `/whatsapp/conversations/:id/archive` | Arquivar conversa |
| DELETE | `/whatsapp/conversations/:id` | Excluir conversa |
| GET | `/whatsapp/instances` | Listar instâncias |
| POST | `/whatsapp/instances` | Criar instância |
| POST | `/whatsapp/instances/:id/connect` | Conectar (QR Code) |
| POST | `/whatsapp/instances/:id/check` | Verificar conexão |
| DELETE | `/whatsapp/instances/:id` | Excluir instância |
| GET | `/whatsapp/provider-credentials` | Listar credenciais |
| POST | `/whatsapp/provider-credentials` | Criar credencial |
| DELETE | `/whatsapp/provider-credentials/:id` | Excluir credencial |

### Contatos (15 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/contacts` | Criar contato |
| GET | `/contacts` | Listar contatos |
| GET | `/contacts/:id` | Obter contato |
| PUT | `/contacts/:id` | Atualizar contato |
| DELETE | `/contacts/:id` | Excluir contato |
| POST | `/contacts/:id/toggle-block` | Bloquear/desbloquear |
| POST | `/contacts/:id/notes` | Adicionar nota |
| GET | `/contacts/:id/notes` | Listar notas |
| DELETE | `/contacts/:id/notes/:noteId` | Excluir nota |
| GET | `/contacts/:id/history` | Histórico do contato |
| GET | `/contacts/tags/list` | Listar tags |
| POST | `/contacts/tags` | Criar tag |
| PUT | `/contacts/tags/:id` | Atualizar tag |
| DELETE | `/contacts/tags/:id` | Excluir tag |
| POST | `/contacts/:id/tags/:tagId` | Atribuir tag |

### Funis (13 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/funnels` | Criar funil |
| GET | `/funnels` | Listar funis |
| GET | `/funnels/:id` | Obter funil |
| PUT | `/funnels/:id` | Atualizar funil |
| DELETE | `/funnels/:id` | Excluir funil |
| GET | `/funnels/:id/stats` | Estatísticas do funil |
| GET | `/funnels/:id/search` | Buscar deals no funil |
| GET | `/funnels/stages` | Listar todos estágios |
| GET | `/funnels/:id/stages` | Listar estágios do funil |
| POST | `/funnels/:id/stages` | Criar estágio |
| PUT | `/funnels/stages/:stageId` | Atualizar estágio |
| DELETE | `/funnels/stages/:stageId` | Excluir estágio |
| POST | `/funnels/:id/stages/reorder` | Reordenar estágios |

### Deals (13 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/deals` | Criar deal |
| GET | `/deals/:id` | Obter deal |
| PUT | `/deals/:id` | Atualizar deal |
| DELETE | `/deals/:id` | Excluir deal |
| POST | `/deals/move` | Mover deal para estágio |
| POST | `/deals/duplicate` | Copiar deal para outro funil |
| GET | `/deals/:id/notes` | Listar notas |
| POST | `/deals/:id/notes` | Adicionar nota |
| PUT | `/deals/:id/notes/:noteId` | Atualizar nota |
| DELETE | `/deals/:id/notes/:noteId` | Excluir nota |
| GET | `/deals/:id/attachments` | Listar anexos |
| POST | `/deals/:id/attachments` | Upload de anexo |
| DELETE | `/deals/:id/attachments/:attachmentId` | Excluir anexo |

### IA/Prompts (12 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/prompts` | Listar prompts da instância |
| POST | `/prompts` | Criar/atualizar prompt |
| GET | `/prompts/onboarding` | Obter onboarding |
| PUT | `/prompts/onboarding` | Atualizar onboarding |
| POST | `/prompts/onboarding/generate` | Gerar prompt via IA |
| GET | `/ai/conversations/:id/state` | Estado da IA na conversa |
| POST | `/ai/analyze` | Analisar conversa |
| GET | `/ai/suggestions` | Obter sugestões |
| POST | `/assistant/conversations` | Criar conversa com assistente |
| GET | `/assistant/conversations` | Listar conversas |
| POST | `/assistant/conversations/:id/messages` | Enviar mensagem |
| GET | `/assistant/conversations/:id/messages` | Listar mensagens |

### Follow-ups (8 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/followups/rules` | Listar regras |
| POST | `/followups/rules` | Criar regra |
| GET | `/followups/rules/:id` | Obter regra |
| PUT | `/followups/rules/:id` | Atualizar regra |
| DELETE | `/followups/rules/:id` | Excluir regra |
| GET | `/followups/queue` | Listar fila |
| GET | `/followups/history` | Histórico |
| POST | `/followups/cancel/:id` | Cancelar follow-up |

### Flows (10 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/flows` | Listar fluxos |
| POST | `/flows` | Criar fluxo |
| GET | `/flows/:id` | Obter fluxo |
| PUT | `/flows/:id` | Atualizar fluxo |
| DELETE | `/flows/:id` | Excluir fluxo |
| POST | `/flows/:id/toggle` | Ativar/desativar |
| POST | `/flows/:id/duplicate` | Duplicar fluxo |
| GET | `/flows/:id/executions` | Listar execuções |
| POST | `/flows/test` | Testar fluxo |
| GET | `/flows/node-types` | Listar tipos de nó |

### Dashboard (4 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/dashboard/metrics` | Métricas gerais |
| GET | `/dashboard/conversations/stats` | Stats de conversas |
| GET | `/dashboard/ai/stats` | Stats da IA |
| GET | `/dashboard/funnels/stats` | Stats de funis |

### Notifications (3 endpoints)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/notifications` | Listar notificações |
| POST | `/notifications/:id/read` | Marcar como lida |
| POST | `/notifications/read-all` | Marcar todas como lidas |

---

## Documentação Detalhada

- [Autenticação](auth.md) - 6 endpoints
- [WhatsApp](whatsapp.md) - 18 endpoints
- [Contatos](contacts.md) - 15 endpoints
- [Funis](funnels.md) - 13 endpoints
- [Deals](deals.md) - 12 endpoints
- [IA/Prompts](ai.md) - 12 endpoints
- [Follow-ups](followups.md) - 8 endpoints
- [Flows](flows.md) - 10 endpoints

---

## Padrões de Resposta

### Sucesso (2xx)

```json
{
  "success": true,
  "data": { /* objeto ou array */ },
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "pages": 5
  }
}
```

### Erro (4xx/5xx)

```json
{
  "success": false,
  "error": "Mensagem de erro legível"
}
```

### Validação (400)

```json
{
  "success": false,
  "error": "validation_error",
  "details": [
    {
      "field": "email",
      "message": "Email inválido"
    }
  ]
}
```

---

## Códigos de Status

| Código | Significado | Uso |
|--------|-------------|-----|
| 200 | OK | Sucesso em GET/PUT |
| 201 | Created | Sucesso em POST |
| 204 | No Content | Sucesso em DELETE |
| 400 | Bad Request | Validação falhou |
| 401 | Unauthorized | Token inválido/ausente |
| 403 | Forbidden | Sem permissão |
| 404 | Not Found | Recurso não existe |
| 409 | Conflict | Duplicação |
| 422 | Unprocessable | Regra de negócio |
| 429 | Too Many Requests | Rate limit |
| 500 | Internal Error | Erro no servidor |

---

## Paginação

### Request

```
GET /contacts?page=2&limit=20&sort=-created_at
```

### Response

```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```

### Parâmetros Comuns

| Parâmetro | Tipo | Default | Descrição |
|-----------|------|---------|-----------|
| page | number | 1 | Página atual |
| limit | number | 20 | Itens por página (max: 100) |
| sort | string | -created_at | Campo de ordenação (- para DESC) |
| search | string | - | Busca textual |

---

## Filtros

### Exemplo

```
GET /contacts?search=maria&funnelId=abc&tagIds=tag1,tag2
```

### Operadores de Filtro

| Filtro | Exemplo | Descrição |
|--------|---------|-----------|
| Igual | `status=active` | Valor exato |
| Lista | `tagIds=a,b,c` | Qualquer valor |
| Range | `createdAfter=2024-01-01` | Data mínima |
| Busca | `search=texto` | Busca parcial |

---

## Rate Limiting

| Tier | Limite | Janela |
|------|--------|--------|
| Default | 100 | 1 minuto |
| Auth | 30 | 1 minuto |
| Heavy | 30 | 1 minuto |
| Webhook | 1000 | 1 minuto |

### Headers de Resposta

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

### Resposta 429

```json
{
  "success": false,
  "error": "Too many requests. Please try again later."
}
```

---

## Webhooks

### Configuração

Os webhooks são recebidos via Evolution API ou UAZAPI. Configure o URL de webhook no provedor:

```
https://api.seudominio.com/api/whatsapp/webhook/:instanceName
```

### Eventos Suportados

| Evento | Descrição |
|--------|-----------|
| `messages.upsert` | Nova mensagem recebida |
| `connection.update` | Status de conexão alterado |
| `qrcode.updated` | QR Code atualizado |
| `messages.update` | Status de mensagem alterado |

### Payload de Exemplo

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "ABC123"
    },
    "message": {
      "conversation": "Olá, tudo bem?"
    }
  }
}
```

---

## Permissões

### Roles de Empresa

| Role | Descrição |
|------|-----------|
| `admin` | Acesso total à empresa |
| `coordinator` | Gerencia equipe e configurações |
| `member` | Acesso básico operacional |

### Permissões Granulares

```json
{
  "canManageUsers": true,
  "canManageWhatsApp": true,
  "canViewReports": true,
  "canManageFlows": false,
  "canManageContacts": true,
  "canManagePrompt": true
}
```

---

← [Voltar para SPEC](../README.md)
