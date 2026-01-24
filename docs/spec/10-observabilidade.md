# 10. Observabilidade e Monitoramento

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 10.1 Pilares da Observabilidade

### 1. Logs
Registros de eventos para debugging e auditoria.

### 2. Métricas
Dados numéricos agregados para monitoramento.

### 3. Traces
Rastreamento de requisições através do sistema.

---

## 10.2 Logging

### Formato Estruturado (JSON)

```json
{
  "level": "info",
  "timestamp": "2026-01-19T10:30:00.000Z",
  "service": "evo-ai-connect-api",
  "request_id": "uuid",
  "user_id": "uuid",
  "company_id": "uuid",
  "message": "Message sent to WhatsApp",
  "data": {
    "conversation_id": "uuid",
    "instance_id": "uuid",
    "action": "send_message"
  },
  "duration_ms": 45
}
```

### Níveis de Log

| Nível | Uso | Exemplo |
|-------|-----|---------|
| `fatal` | Sistema down | PostgreSQL unreachable |
| `error` | Erros que precisam ação | Evolution API connection failed |
| `warn` | Situações problemáticas | Rate limit próximo (80%) |
| `info` | Eventos normais | Mensagem enviada, Deal movido |
| `debug` | Detalhes para debugging | Payload recebido, Query executada |
| `trace` | Muito detalhado | Entrada/saída de funções |

### Configuração com Pino (Fastify)

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    service: 'evo-ai-connect-api',
    env: process.env.NODE_ENV,
  },
  redact: ['password', 'token', 'apiKey', 'refreshToken'], // Nunca logar dados sensíveis
});

// Uso
logger.info({ userId, companyId, action: 'send_message' }, 'WhatsApp message sent');
logger.error({ err, conversationId }, 'Failed to process incoming message');
```

### Logs por Domínio

| Domínio | Eventos Logados | Nível |
|---------|-----------------|-------|
| Auth | Login, logout, refresh, invalid token | info/warn |
| WhatsApp | Mensagens, conexão, webhook | info/error |
| AI | Análise, geração de prompt, tokens usados | info |
| CRM | Deal criado/movido, follow-up enviado | info |
| Workers | Job started/completed/failed | info/error |

---

## 10.3 Métricas

### Métricas de Aplicação

| Métrica | Tipo | Descrição |
|---------|------|-----------|
| `http_requests_total` | Counter | Total de requisições HTTP |
| `http_request_duration_seconds` | Histogram | Latência das requisições |
| `http_request_size_bytes` | Histogram | Tamanho das requisições |
| `active_websocket_connections` | Gauge | Conexões Socket.io ativas |
| `bullmq_queue_size` | Gauge | Tamanho de cada fila BullMQ |
| `bullmq_job_duration_seconds` | Histogram | Duração dos jobs |
| `bullmq_job_failed_total` | Counter | Jobs que falharam |

### Métricas de Negócio - WhatsApp

| Métrica | Tipo | Descrição |
|---------|------|-----------|
| `whatsapp_instances_connected` | Gauge | Instâncias conectadas |
| `whatsapp_messages_received_total` | Counter | Mensagens recebidas |
| `whatsapp_messages_sent_total` | Counter | Mensagens enviadas |
| `whatsapp_ai_responses_total` | Counter | Respostas geradas por IA |
| `whatsapp_webhook_latency_seconds` | Histogram | Latência do webhook |

### Métricas de Negócio - CRM

| Métrica | Tipo | Descrição |
|---------|------|-----------|
| `crm_contacts_total` | Gauge | Total de contatos |
| `crm_deals_by_status` | Gauge | Deals por status (OPEN/WON/LOST) |
| `crm_followups_pending` | Gauge | Follow-ups pendentes |
| `crm_followups_sent_total` | Counter | Follow-ups enviados |
| `crm_deal_conversion_rate` | Gauge | Taxa de conversão |

### Métricas de IA

| Métrica | Tipo | Descrição |
|---------|------|-----------|
| `openai_requests_total` | Counter | Total de chamadas à OpenAI |
| `openai_tokens_used_total` | Counter | Tokens consumidos (input + output) |
| `openai_latency_seconds` | Histogram | Latência da API OpenAI |
| `openai_errors_total` | Counter | Erros da API OpenAI |
| `embeddings_generated_total` | Counter | Embeddings gerados |

### Exemplo com Prometheus

```typescript
import { Counter, Histogram, Gauge, Registry } from 'prom-client';

const register = new Registry();

// HTTP Metrics
const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
  registers: [register]
});

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path'],
  buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5],
  registers: [register]
});

// WhatsApp Metrics
const whatsappInstancesConnected = new Gauge({
  name: 'whatsapp_instances_connected',
  help: 'Number of connected WhatsApp instances',
  registers: [register]
});

const whatsappMessagesSent = new Counter({
  name: 'whatsapp_messages_sent_total',
  help: 'Total WhatsApp messages sent',
  labelNames: ['instance_id', 'type'],
  registers: [register]
});

// BullMQ Metrics
const queueSize = new Gauge({
  name: 'bullmq_queue_size',
  help: 'Current queue size',
  labelNames: ['queue_name'],
  registers: [register]
});

// Fastify Hook
app.addHook('onResponse', (request, reply, done) => {
  const duration = reply.getResponseTime() / 1000;
  httpRequestsTotal.inc({
    method: request.method,
    path: request.routerPath || request.url,
    status: reply.statusCode
  });
  httpRequestDuration.observe({
    method: request.method,
    path: request.routerPath || request.url
  }, duration);
  done();
});

// Endpoint de métricas
app.get('/metrics', async (request, reply) => {
  reply.header('Content-Type', register.contentType);
  return register.metrics();
});
```

---

## 10.4 Distributed Tracing

### Implementação com OpenTelemetry

```typescript
import { trace, context, SpanStatusCode } from '@opentelemetry/api';
import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node';
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';

// Setup
const provider = new NodeTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(
  new JaegerExporter({ endpoint: process.env.JAEGER_ENDPOINT })
));
provider.register();

const tracer = trace.getTracer('evo-ai-connect-api');

// Tracing de requisição WhatsApp
async function processIncomingMessage(data: WebhookPayload) {
  const span = tracer.startSpan('process-whatsapp-message');

  try {
    span.setAttributes({
      'whatsapp.instance_id': data.instanceId,
      'whatsapp.message_type': data.messageType,
      'company.id': data.companyId
    });

    // Span filho para busca de contexto
    const contextSpan = tracer.startSpan('fetch-conversation-context', {
      parent: span
    });
    const conversation = await getConversationContext(data);
    contextSpan.end();

    // Span filho para IA
    const aiSpan = tracer.startSpan('generate-ai-response', {
      parent: span
    });
    aiSpan.setAttributes({
      'ai.model': 'gpt-4o-mini',
      'ai.prompt_tokens': conversation.tokenCount
    });
    const response = await generateAIResponse(conversation);
    aiSpan.end();

    // Span filho para envio
    const sendSpan = tracer.startSpan('send-whatsapp-message', {
      parent: span
    });
    await sendMessage(response);
    sendSpan.end();

    span.setStatus({ code: SpanStatusCode.OK });
    return response;
  } catch (error) {
    span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
    span.recordException(error);
    throw error;
  } finally {
    span.end();
  }
}
```

### Correlação de Traces

```typescript
// Request ID propagation
app.addHook('onRequest', (request, reply, done) => {
  const requestId = request.headers['x-request-id'] || crypto.randomUUID();
  request.requestId = requestId;
  reply.header('x-request-id', requestId);
  done();
});

// Incluir requestId em todos os logs
logger.child({ requestId: request.requestId }).info('Processing request');
```

---

## 10.5 Alertas

### Regras de Alerta

| Alerta | Condição | Severidade | Ação |
|--------|----------|------------|------|
| High Error Rate | error_rate > 5% por 5min | Critical | PagerDuty |
| High Latency | p95 > 500ms por 5min | Warning | Slack |
| WhatsApp Disconnected | instances_connected < expected | Critical | PagerDuty |
| Queue Backlog | queue_size > 500 por 5min | Warning | Slack |
| OpenAI Errors | openai_errors > 10/min | Warning | Slack |
| Low Disk Space | disk_usage > 85% | Warning | Slack |
| Service Down | up == 0 por 1min | Critical | PagerDuty |
| DLQ Growing | dlq_size > 10/hour | Warning | Slack |
| Memory High | memory_usage > 85% | Warning | Slack |

### Alertas Específicos de Negócio

| Alerta | Condição | Severidade | Ação |
|--------|----------|------------|------|
| No Messages | messages_sent = 0 por 30min (horário comercial) | Warning | Slack |
| High AI Token Usage | tokens_used > 1M/day | Warning | Slack |
| Follow-up Failures | followup_failures > 10% | Warning | Slack |
| Webhook Timeout | webhook_latency > 5s | Warning | Slack |

### Configuração Prometheus Alertmanager

```yaml
# prometheus/alerts.yml
groups:
  - name: evo_ai_connect_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
          > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
          > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }}s"

      - alert: WhatsAppDisconnected
        expr: whatsapp_instances_connected < 1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "WhatsApp instance disconnected"
          description: "No WhatsApp instances connected"

      - alert: QueueBacklog
        expr: bullmq_queue_size{queue_name="message-worker"} > 500
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Message queue backlog"
          description: "Queue size is {{ $value }}"

      - alert: OpenAIErrors
        expr: rate(openai_errors_total[5m]) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "OpenAI API errors increasing"
          description: "Error rate: {{ $value }}/s"

      - alert: DLQGrowing
        expr: increase(bullmq_dlq_size[1h]) > 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Dead Letter Queue growing"
          description: "{{ $value }} messages added to DLQ in last hour"
```

---

## 10.6 Dashboards

### Dashboard Principal - Evo AI Connect

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Evo AI Connect - Dashboard                            │
├─────────────────────────────────────────────────────────────────────────┤
│  Request Rate    │  Error Rate     │  Latency (p95)  │  Uptime          │
│  [ 1.2k/min ]   │  [ 0.3% ]       │  [ 85ms ]       │  [ 99.95% ]      │
├─────────────────────────────────────────────────────────────────────────┤
│                         WhatsApp Metrics                                 │
│  Instances       │  Msgs Received  │  Msgs Sent      │  AI Responses    │
│  [ 5/5 ✅ ]      │  [ 3.2k/hour ] │  [ 2.8k/hour ] │  [ 2.5k/hour ]  │
├─────────────────────────────────────────────────────────────────────────┤
│                         CRM Metrics                                      │
│  Active Deals    │  Conversion     │  Follow-ups     │  Contacts        │
│  [ 234 ]        │  [ 23.5% ]      │  [ 45 pending ] │  [ 8.2k ]        │
├─────────────────────────────────────────────────────────────────────────┤
│                         Queue Status                                     │
│  message-worker  │  followup-worker │ embedding-worker │ flow-worker    │
│  [ 12 jobs ]    │  [ 3 jobs ]      │  [ 5 jobs ]      │  [ 0 jobs ]    │
├─────────────────────────────────────────────────────────────────────────┤
│                    Request Rate & Errors Over Time                       │
│  ▄▄█▄▄▄▄▄█▄▄▄▄▄▄▄▄▄█▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄█▄▄▄▄▄█▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄       │
└─────────────────────────────────────────────────────────────────────────┘
```

### Dashboard de Workers

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Workers Dashboard                                     │
├─────────────────────────────────────────────────────────────────────────┤
│  Worker              │  Active │  Completed │  Failed │  Avg Time       │
├─────────────────────────────────────────────────────────────────────────┤
│  message-worker      │  10     │  45.2k     │  23     │  125ms          │
│  followup-worker     │  5      │  12.8k     │  5      │  89ms           │
│  embedding-worker    │  3      │  8.5k      │  12     │  450ms          │
│  flow-worker         │  3      │  3.2k      │  2      │  210ms          │
├─────────────────────────────────────────────────────────────────────────┤
│                    Dead Letter Queues                                    │
│  message-dlq: 3    │  followup-dlq: 1  │  embedding-dlq: 2             │
└─────────────────────────────────────────────────────────────────────────┘
```

### Métricas do Dashboard

**Painel Principal:**
- Requests por minuto
- Taxa de erro (%)
- Latência p50, p95, p99
- Uptime

**Painel WhatsApp:**
- Instâncias conectadas vs total
- Mensagens recebidas/hora
- Mensagens enviadas/hora
- Respostas de IA/hora

**Painel CRM:**
- Deals ativos
- Taxa de conversão (30 dias)
- Follow-ups pendentes
- Total de contatos

**Painel Workers:**
- Jobs por fila
- Jobs processados/hora
- Taxa de erro por worker
- Tempo médio de processamento

---

## 10.7 Stack de Observabilidade

### Recomendação: Open Source

| Componente | Ferramenta | Descrição |
|------------|------------|-----------|
| Métricas | Prometheus + Grafana | Coleta e visualização de métricas |
| Logs | Loki + Grafana | Agregação de logs |
| Traces | Jaeger | Distributed tracing |
| Alertas | Alertmanager | Gestão de alertas |

### Alternativa: Managed Services

| Componente | Ferramenta | Descrição |
|------------|------------|-----------|
| APM | Datadog / New Relic | All-in-one monitoring |
| Logs | CloudWatch / Stackdriver | Cloud-native logs |
| Alertas | PagerDuty / Opsgenie | Incident management |
| Uptime | Better Uptime / Pingdom | External monitoring |

### Configuração Docker Compose (Observabilidade)

```yaml
# docker-compose.observability.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"  # UI
      - "14268:14268"  # HTTP collector

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

volumes:
  grafana_data:
```

---

## 10.8 Health Checks Detalhados

### Endpoints de Health

```typescript
// GET /health
app.get('/health', async (request, reply) => {
  return {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    version: process.env.npm_package_version
  };
});

// GET /health/ready
app.get('/health/ready', async (request, reply) => {
  const checks = {
    database: false,
    redis: false,
    evolutionApi: false
  };

  try {
    // PostgreSQL
    await prisma.$queryRaw`SELECT 1`;
    checks.database = true;
  } catch (e) { /* */ }

  try {
    // Redis
    await redis.ping();
    checks.redis = true;
  } catch (e) { /* */ }

  try {
    // Evolution API
    const response = await fetch(`${process.env.EVOLUTION_API_URL}/health`);
    checks.evolutionApi = response.ok;
  } catch (e) { /* */ }

  const allHealthy = Object.values(checks).every(v => v);

  reply.status(allHealthy ? 200 : 503);
  return {
    status: allHealthy ? 'ready' : 'not ready',
    checks
  };
});

// GET /health/live
app.get('/health/live', async (request, reply) => {
  return {
    status: 'alive',
    pid: process.pid,
    memory: process.memoryUsage()
  };
});
```

---

← [Voltar para SPEC](README.md) | [Próximo: Segurança →](11-seguranca.md)
