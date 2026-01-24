# 7. Tratamento de Erros

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 7.1 Categorização de Erros

### Por Severidade

| Severidade | Descrição | Ação | Exemplo |
|------------|-----------|------|---------|
| **Critical** | Sistema indisponível | Alerta imediato + escalar | PostgreSQL down |
| **High** | Funcionalidade quebrada | Alerta + investigar | Evolution API falhou |
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

### Formato Padrão

```json
{
  "success": false,
  "error": "Mensagem de erro legível"
}
```

### Com Detalhes de Validação

```json
{
  "success": false,
  "error": "validation_error",
  "details": [
    {
      "field": "email",
      "code": "invalid_format",
      "message": "Email inválido"
    },
    {
      "field": "password",
      "code": "too_short",
      "message": "Mínimo 6 caracteres"
    }
  ]
}
```

### Códigos de Erro Padrão

| Código | Descrição |
|--------|-----------|
| `validation_error` | Dados não passaram validação Zod |
| `unauthorized` | Token inválido ou ausente |
| `forbidden` | Sem permissão para recurso |
| `not_found` | Recurso não encontrado |
| `conflict` | Email/telefone já cadastrado |
| `rate_limited` | Muitas requisições |
| `internal_error` | Erro interno do servidor |
| `external_service_error` | Evolution API / OpenAI falhou |

---

## 7.3 Estratégia de Retry

### Exponential Backoff

```typescript
const retryConfig = {
  maxRetries: 3,
  baseDelay: 1000, // 1 segundo
  maxDelay: 30000, // 30 segundos
  backoffMultiplier: 2,
  jitter: true // Adiciona variação aleatória
};

// Delays: ~1s, ~2s, ~4s (com jitter de ±20%)
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

### Configuração por Tipo de Endpoint

| Tier | Endpoints | Limite | Janela |
|------|-----------|--------|--------|
| **Auth** | `/auth/*` | 30 | 1 min |
| **Default** | `/api/*` | 100 | 1 min |
| **Heavy** | `/ai/analyze`, `/ai/onboarding/generate` | 30 | 1 min |
| **Webhook** | `/whatsapp/webhook` | 1000 | 1 min |

### Headers de Resposta

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
Retry-After: 60
```

### Resposta 429

```json
{
  "success": false,
  "error": "Too many requests. Please try again later."
}
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

### Configuração por Serviço Externo

| Serviço | Threshold | Timeout | Fallback |
|---------|-----------|---------|----------|
| Evolution API | 5 falhas | 30s | Fila de retry |
| OpenAI | 3 falhas | 60s | Resposta padrão |
| PostgreSQL | 3 falhas | 10s | Error crítico |
| Redis | 5 falhas | 15s | Fallback para DB |

### Implementação

```typescript
interface CircuitBreakerConfig {
  failureThreshold: number;
  successThreshold: number;
  timeout: number;
  monitorInterval: number;
}

const evolutionCircuitBreaker: CircuitBreakerConfig = {
  failureThreshold: 5,      // Falhas para abrir
  successThreshold: 3,      // Sucessos para fechar
  timeout: 30000,           // Tempo em OPEN (30s)
  monitorInterval: 10000    // Intervalo de check
};
```

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
  originalQueue: string;  // 'message-worker', 'followup-worker', etc.
  payload: unknown;
  error: {
    message: string;
    stack?: string;
    code?: string;
  };
  attempts: number;
  failedAt: Date;
  metadata: {
    companyId?: string;
    conversationId?: string;
    instanceId?: string;
  };
}
```

### Filas e DLQs

| Fila Original | DLQ | Retention |
|---------------|-----|-----------|
| message-worker | message-dlq | 7 dias |
| followup-worker | followup-dlq | 7 dias |
| embedding-worker | embedding-dlq | 3 dias |
| flow-worker | flow-dlq | 7 dias |

### Monitoramento

- Alerta se DLQ > 10 mensagens/hora
- Dashboard com mensagens pendentes
- Processo de reprocessamento manual via admin

---

## 7.7 Logging de Erros

### Estrutura do Log

```json
{
  "level": "error",
  "timestamp": "2026-01-19T10:00:00.000Z",
  "requestId": "uuid",
  "userId": "uuid",
  "companyId": "uuid",
  "error": {
    "code": "external_service_error",
    "message": "Evolution API connection failed",
    "stack": "Error: ECONNREFUSED..."
  },
  "context": {
    "endpoint": "POST /api/whatsapp/send",
    "instanceId": "uuid",
    "ip": "xxx.xxx.xxx.xxx",
    "userAgent": "Mozilla/5.0..."
  }
}
```

### Níveis de Log

| Nível | Uso | Exemplo |
|-------|-----|---------|
| `fatal` | Sistema down | PostgreSQL unreachable |
| `error` | Erros que precisam atenção | Falha ao enviar mensagem |
| `warn` | Situações problemáticas | Rate limit próximo |
| `info` | Eventos normais | Mensagem enviada |
| `debug` | Detalhes para debugging | Payload recebido |
| `trace` | Detalhes extremos | Query SQL executada |

### Campos Obrigatórios

- `timestamp`: ISO 8601
- `level`: Nível do log
- `message`: Descrição do evento
- `requestId`: UUID único por request (para correlação)

---

## 7.8 Erros por Domínio

### WhatsApp

| Erro | Código | Ação |
|------|--------|------|
| Instância não conectada | 400 | Solicitar reconexão |
| Número inválido | 400 | Validar formato |
| Evolution API down | 503 | Circuit breaker + retry |
| Mensagem muito longa | 400 | Truncar/dividir |

### Autenticação

| Erro | Código | Ação |
|------|--------|------|
| Token expirado | 401 | Refresh token |
| Token inválido | 401 | Redirecionar login |
| Sessão inválida | 401 | Forçar novo login |
| Empresa não encontrada | 404 | Verificar companyId |

### IA/OpenAI

| Erro | Código | Ação |
|------|--------|------|
| Rate limit OpenAI | 429 | Exponential backoff |
| Modelo indisponível | 503 | Fallback para modelo secundário |
| Context too long | 400 | Truncar histórico |
| API key inválida | 401 | Verificar configuração |

---

## 7.9 Idempotência

### Chave de Idempotência

Operações críticas suportam header de idempotência:

```
Header: X-Idempotency-Key: uuid
```

### Endpoints Idempotentes

| Método | Endpoint | Idempotente? | Mecanismo |
|--------|----------|--------------|-----------|
| GET | Todos | ✅ | Natural |
| POST | `/whatsapp/send` | ✅ | messageKeyId |
| POST | `/contacts` | ⚠️ | phoneNumber único |
| PUT | Todos | ✅ | Natural |
| DELETE | Todos | ✅ | Natural |

### Cache de Idempotência

```typescript
// Armazena resultado por 24h no Redis
const idempotencyCache = {
  key: `idempotency:${idempotencyKey}`,
  ttl: 86400, // 24 horas
  data: {
    response: object,
    statusCode: number,
    createdAt: Date
  }
};
```

---

## 7.10 Tratamento no Frontend

### Interceptor Axios

```typescript
axios.interceptors.response.use(
  response => response,
  async error => {
    const originalRequest = error.config;

    // Token expirado - tenta refresh
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      const newToken = await refreshToken();
      originalRequest.headers.Authorization = `Bearer ${newToken}`;
      return axios(originalRequest);
    }

    // Rate limit - aguarda e retry
    if (error.response?.status === 429) {
      const retryAfter = error.response.headers['retry-after'] || 60;
      await sleep(retryAfter * 1000);
      return axios(originalRequest);
    }

    return Promise.reject(error);
  }
);
```

### Toast de Erros

```typescript
const errorMessages: Record<string, string> = {
  'unauthorized': 'Sessão expirada. Faça login novamente.',
  'forbidden': 'Você não tem permissão para esta ação.',
  'not_found': 'Recurso não encontrado.',
  'rate_limited': 'Muitas requisições. Aguarde um momento.',
  'internal_error': 'Erro interno. Tente novamente.',
  'external_service_error': 'Serviço temporariamente indisponível.'
};
```

---

← [Voltar para SPEC](README.md) | [Próximo: Estratégia de Testes →](08-estrategia-testes.md)
