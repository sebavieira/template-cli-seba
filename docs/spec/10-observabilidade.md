# 10. Observabilidade e Monitoramento

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

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
  "timestamp": "2024-01-15T10:30:00.000Z",
  "service": "{{service_name}}",
  "request_id": "uuid",
  "user_id": "uuid",
  "message": "{{Entidade}} created",
  "data": {
    "{{entidade}}_id": "uuid",
    "action": "create"
  },
  "duration_ms": 45
}
```

### Níveis de Log

| Nível | Uso | Exemplo |
|-------|-----|---------|
| `error` | Erros que precisam ação | DB connection failed |
| `warn` | Situações problemáticas | Rate limit próximo |
| `info` | Eventos normais | Request processado |
| `debug` | Detalhes para debugging | Query executada |
| `trace` | Muito detalhado | Entrada/saída de funções |

### Configuração

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    service: '{{service_name}}',
    env: process.env.NODE_ENV,
  },
});

// Uso
logger.info({ userId, action: 'login' }, 'User logged in');
```

---

## 10.3 Métricas

### Métricas de Aplicação

| Métrica | Tipo | Descrição |
|---------|------|-----------|
| `http_requests_total` | Counter | Total de requisições |
| `http_request_duration_seconds` | Histogram | Latência das requisições |
| `http_request_size_bytes` | Histogram | Tamanho das requisições |
| `active_connections` | Gauge | Conexões ativas |
| `queue_size` | Gauge | Tamanho da fila |
| `job_duration_seconds` | Histogram | Duração dos jobs |

### Métricas de Negócio

| Métrica | Tipo | Descrição |
|---------|------|-----------|
| `{{entidade}}_created_total` | Counter | Total criados |
| `{{entidade}}_active` | Gauge | Ativos no momento |
| `{{entidade}}_processed_total` | Counter | Processados |
| `errors_by_type` | Counter | Erros por tipo |

### Exemplo com Prometheus

```typescript
import { Counter, Histogram, Gauge } from 'prom-client';

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status']
});

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 5]
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestsTotal.inc({ method: req.method, path: req.path, status: res.statusCode });
    httpRequestDuration.observe({ method: req.method, path: req.path }, duration);
  });
  next();
});
```

---

## 10.4 Distributed Tracing

### Implementação com OpenTelemetry

```typescript
import { trace, context } from '@opentelemetry/api';

const tracer = trace.getTracer('{{service_name}}');

async function processRequest(data) {
  const span = tracer.startSpan('process-request');

  try {
    span.setAttributes({
      'request.type': data.type,
      'user.id': data.userId
    });

    const result = await doWork(data);

    span.setStatus({ code: SpanStatusCode.OK });
    return result;
  } catch (error) {
    span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
    throw error;
  } finally {
    span.end();
  }
}
```

---

## 10.5 Alertas

### Regras de Alerta

| Alerta | Condição | Severidade | Ação |
|--------|----------|------------|------|
| High Error Rate | error_rate > 5% por 5min | Critical | PagerDuty |
| High Latency | p95 > 500ms por 5min | Warning | Slack |
| Low Disk Space | disk_usage > 85% | Warning | Slack |
| Service Down | up == 0 por 1min | Critical | PagerDuty |
| Queue Backlog | queue_size > 1000 | Warning | Slack |

### Exemplo de Alerta (Prometheus)

```yaml
groups:
  - name: {{service_name}}_alerts
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
```

---

## 10.6 Dashboards

### Dashboard Principal

```
┌────────────────────────────────────────────────────────────┐
│                    {{SERVICE_NAME}} Dashboard              │
├────────────────────────────────────────────────────────────┤
│  Request Rate    │  Error Rate     │  Latency (p95)       │
│  [ 1.5k/min ]   │  [ 0.3% ]       │  [ 45ms ]            │
├────────────────────────────────────────────────────────────┤
│  Active Users    │  Queue Size     │  CPU Usage           │
│  [ 234 ]        │  [ 12 ]         │  [ 35% ]             │
├────────────────────────────────────────────────────────────┤
│                  Request Rate Over Time                    │
│  ▄▄█▄▄▄▄▄█▄▄▄▄▄▄▄▄▄█▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄█▄▄▄▄▄█▄         │
└────────────────────────────────────────────────────────────┘
```

### Métricas do Dashboard

- Requests por minuto
- Taxa de erro (%)
- Latência p50, p95, p99
- Usuários ativos
- Tamanho da fila
- CPU e memória
- Uptime

---

## 10.7 Stack de Observabilidade

### Opção 1: Open Source

| Componente | Ferramenta |
|------------|------------|
| Métricas | Prometheus + Grafana |
| Logs | Loki + Grafana |
| Traces | Jaeger |
| Alertas | Alertmanager |

### Opção 2: Managed

| Componente | Ferramenta |
|------------|------------|
| APM | Datadog / New Relic |
| Logs | CloudWatch / Stackdriver |
| Alertas | PagerDuty / Opsgenie |

---

← [Voltar para SPEC](README.md) | [Próximo: Segurança →](11-seguranca.md)
