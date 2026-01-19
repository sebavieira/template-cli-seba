# 4. Contratos de API

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

← [Voltar para SPEC](../README.md)

---

## Visão Geral

API REST com autenticação via JWT Bearer token.

### Base URL

```
Development: http://localhost:3000/api/v1
Production:  https://api.{{domain}}.com/v1
```

### Autenticação

Todas as rotas (exceto auth) requerem header:

```
Authorization: Bearer {jwt_token}
```

---

## Índice de Endpoints

### Autenticação
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/auth/register` | Criar conta |
| POST | `/auth/login` | Fazer login |
| POST | `/auth/logout` | Fazer logout |
| POST | `/auth/refresh` | Renovar token |
| POST | `/auth/forgot-password` | Solicitar reset |
| POST | `/auth/reset-password` | Resetar senha |

### {{DOMINIO_1}}
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/{{dominio_1}}` | Listar todos |
| POST | `/{{dominio_1}}` | Criar novo |
| GET | `/{{dominio_1}}/:id` | Obter por ID |
| PATCH | `/{{dominio_1}}/:id` | Atualizar |
| DELETE | `/{{dominio_1}}/:id` | Excluir |

### {{DOMINIO_2}}
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/{{dominio_1}}/:id/{{dominio_2}}` | Listar relacionados |
| POST | `/{{dominio_1}}/:id/{{dominio_2}}` | Criar relacionado |
| PATCH | `/{{dominio_2}}/:id` | Atualizar |
| DELETE | `/{{dominio_2}}/:id` | Excluir |

---

## Documentação Detalhada

- [{{DOMINIO_1}}]({{dominio_1}}.md) - {{QTD_1}} endpoints
- [{{DOMINIO_2}}]({{dominio_2}}.md) - {{QTD_2}} endpoints
- (Adicione mais conforme necessário)

---

## Padrões de Resposta

### Sucesso (2xx)

```json
{
  "data": { /* objeto ou array */ },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

### Erro (4xx/5xx)

```json
{
  "error": "error_code",
  "message": "Human readable message",
  "details": [
    {
      "field": "email",
      "error": "Invalid format"
    }
  ]
}
```

---

## Códigos de Status

| Código | Significado | Uso |
|--------|-------------|-----|
| 200 | OK | Sucesso em GET/PATCH |
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
GET /{{dominio_1}}?page=2&limit=20&sort=-created_at
```

### Response

```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```

---

## Filtros

```
GET /{{dominio_1}}?status=active&created_after=2024-01-01
```

---

## Rate Limiting

| Tier | Limite | Janela |
|------|--------|--------|
| Default | 100 | 1 minuto |
| Auth | 10 | 1 minuto |
| Heavy | 30 | 1 minuto |

Headers de resposta:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

---

← [Voltar para SPEC](../README.md)
