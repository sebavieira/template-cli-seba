# 7. Tratamento de Erros

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

← [Voltar para SPEC](README.md)

---

## 7.1 Categorização de Erros

### Por Severidade

| Severidade | Descrição | Ação | Exemplo |
|------------|-----------|------|---------|
| **Critical** | Sistema indisponível | Alerta imediato + escalar | DB down |
| **High** | Funcionalidade quebrada | Alerta + investigar | API externa falhou |
| **Medium** | Degradação parcial | Monitorar + retry | Timeout ocasional |
| **Low** | Problema menor | Log + revisar depois | Validação falhou |

### Por Tipo

| Tipo | Código HTTP | Descrição |
|------|-------------|-----------|
| **Validation** | 400 | Dados inválidos |
| **Authentication** | 401 | Token inválido/ausente |
| **Authorization** | 403 | Sem permissão |
| **NotFound** | 404 | Recurso não existe |
| **Conflict** | 409 | Conflito de dados |
| **RateLimit** | 429 | Limite excedido |
| **Internal** | 500 | Erro no servidor |
| **External** | 502/503 | Serviço externo falhou |

---

## 7.2 Formato de Resposta de Erro

```json
{
  "error": "error_code",
  "message": "Human readable message",
  "details": [
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Email must be valid"
    }
  ],
  "request_id": "uuid",
  "timestamp": "ISO 8601"
}
```

### Códigos de Erro Padrão

| Código | Descrição |
|--------|-----------|
| `validation_error` | Dados não passaram validação |
| `unauthorized` | Token inválido ou ausente |
| `forbidden` | Sem permissão para recurso |
| `not_found` | Recurso não encontrado |
| `conflict` | Duplicação ou conflito |
| `rate_limited` | Muitas requisições |
| `internal_error` | Erro interno do servidor |
| `external_service_error` | Serviço externo falhou |

---

## 7.3 Estratégia de Retry

### Exponential Backoff

```typescript
const retryConfig = {
  maxRetries: 3,
  baseDelay: 1000, // 1 segundo
  maxDelay: 30000, // 30 segundos
  backoffMultiplier: 2
};

// Delays: 1s, 2s, 4s (com jitter)
```

### Erros Retryable

| Erro | Retry? | Motivo |
|------|--------|--------|
| 400 Bad Request | ❌ | Dados inválidos, não vai mudar |
| 401 Unauthorized | ❌ | Token inválido |
| 403 Forbidden | ❌ | Sem permissão |
| 404 Not Found | ❌ | Recurso não existe |
| 429 Too Many Requests | ✅ | Aguardar e tentar |
| 500 Internal Error | ✅ | Pode ser temporário |
| 502/503 Service Error | ✅ | Serviço pode voltar |
| Timeout | ✅ | Pode ser temporário |
| Connection Error | ✅ | Rede pode estabilizar |

---

## 7.4 Rate Limiting

### Configuração por Endpoint

| Endpoint | Limite | Janela | Comportamento |
|----------|--------|--------|---------------|
| `/auth/*` | 10 | 1 min | Block |
| `/api/v1/*` | 100 | 1 min | Queue |
| `/webhooks/*` | 1000 | 1 min | Block |

### Headers de Resposta

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
Retry-After: 60
```

---

## 7.5 Circuit Breaker

### Estados

```
CLOSED ─(falhas > threshold)─> OPEN
   ↑                            │
   │                            │
   └──(sucesso)── HALF_OPEN <───┘
                  (timeout)
```

### Configuração

```typescript
const circuitBreakerConfig = {
  failureThreshold: 5,      // Falhas para abrir
  successThreshold: 3,      // Sucessos para fechar
  timeout: 30000,           // Tempo em OPEN (30s)
  monitorInterval: 10000    // Intervalo de check
};
```

### Por Serviço Externo

| Serviço | Threshold | Timeout | Fallback |
|---------|-----------|---------|----------|
| {{SERVICO_1}} | 5 falhas | 30s | Cache |
| {{SERVICO_2}} | 3 falhas | 60s | Retry later |
| Database | 3 falhas | 10s | Error |

---

## 7.6 Dead Letter Queue (DLQ)

### Quando Usar DLQ

1. Job falhou todas as tentativas de retry
2. Erro de parsing/validação
3. Timeout excessivo
4. Erro desconhecido

### Estrutura da DLQ

```typescript
interface DLQMessage {
  id: string;
  originalQueue: string;
  payload: unknown;
  error: {
    message: string;
    stack?: string;
    code?: string;
  };
  attempts: number;
  failedAt: Date;
  metadata: Record<string, unknown>;
}
```

### Monitoramento

- Alerta se DLQ > 10 mensagens/hora
- Dashboard com mensagens pendentes
- Processo de reprocessamento manual

---

## 7.7 Logging de Erros

### Estrutura do Log

```json
{
  "level": "error",
  "timestamp": "ISO 8601",
  "request_id": "uuid",
  "user_id": "uuid",
  "error": {
    "code": "internal_error",
    "message": "Database connection failed",
    "stack": "..."
  },
  "context": {
    "endpoint": "POST /api/v1/{{entidade}}",
    "ip": "xxx.xxx.xxx.xxx",
    "user_agent": "..."
  }
}
```

### Níveis de Log

| Nível | Uso | Exemplo |
|-------|-----|---------|
| `error` | Erros que precisam atenção | Falha de DB |
| `warn` | Situações problemáticas | Rate limit próximo |
| `info` | Eventos normais | Request processado |
| `debug` | Detalhes para debugging | Payload recebido |

---

## 7.8 Idempotência

### Chave de Idempotência

```
Header: Idempotency-Key: uuid
```

### Implementação

```typescript
// Armazenar resultado por 24h
const idempotencyCache = new Map<string, {
  response: unknown;
  statusCode: number;
  createdAt: Date;
}>();
```

### Endpoints Idempotentes

| Método | Endpoint | Idempotente? |
|--------|----------|--------------|
| GET | Todos | ✅ Natural |
| POST | Criar | ✅ Com header |
| PUT | Atualizar | ✅ Natural |
| PATCH | Atualizar | ⚠️ Depende |
| DELETE | Excluir | ✅ Natural |

---

← [Voltar para SPEC](README.md) | [Próximo: Estratégia de Testes →](08-estrategia-testes.md)
