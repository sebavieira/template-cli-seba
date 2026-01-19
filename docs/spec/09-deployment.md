# 9. Deployment e Infraestrutura

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

← [Voltar para SPEC](README.md)

---

## 9.1 Arquitetura de Deployment

```
┌─────────────────────────────────────────────────────────┐
│                    Cloud Provider                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌─────────────┐    ┌─────────────┐                   │
│   │   Frontend  │    │  Backend    │                   │
│   │   (Vercel)  │    │  (Railway)  │                   │
│   └─────────────┘    └─────────────┘                   │
│                            │                            │
│         ┌──────────────────┼──────────────────┐        │
│         ▼                  ▼                  ▼        │
│   ┌──────────┐      ┌──────────┐      ┌──────────┐    │
│   │ Postgres │      │  Redis   │      │  Storage │    │
│   │ (Managed)│      │ (Managed)│      │   (S3)   │    │
│   └──────────┘      └──────────┘      └──────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 9.2 Recursos Computacionais

### Ambiente de Produção

| Serviço | Recurso | Especificação | Custo Estimado |
|---------|---------|---------------|----------------|
| Backend | Container | 1 vCPU, 1GB RAM | ~$10-20/mês |
| Workers | Container | 1 vCPU, 1GB RAM | ~$10-20/mês |
| PostgreSQL | Managed DB | 1GB storage | ~$15-30/mês |
| Redis | Managed | 256MB | ~$5-10/mês |
| Storage | S3/GCS | 10GB | ~$1/mês |

**Total Estimado:** $40-80/mês

### Ambiente de Staging

| Serviço | Especificação | Custo |
|---------|---------------|-------|
| Backend | 0.5 vCPU, 512MB | ~$5/mês |
| PostgreSQL | 100MB | ~$5/mês |
| Redis | 128MB | ~$3/mês |

**Total Staging:** ~$15/mês

---

## 9.3 Variáveis de Ambiente

### Obrigatórias

```bash
# Aplicação
NODE_ENV=production
PORT=3000
API_URL=https://api.{{domain}}.com

# Banco de Dados
DATABASE_URL=postgresql://user:pass@host:5432/db?sslmode=require

# Redis
REDIS_URL=redis://user:pass@host:6379

# Autenticação
JWT_SECRET=your-super-secret-key-min-32-chars
JWT_EXPIRES_IN=1d
JWT_REFRESH_EXPIRES_IN=7d

# Integrações
{{INTEGRACAO_1}}_API_KEY=xxx
{{INTEGRACAO_2}}_API_KEY=xxx
```

### Opcionais

```bash
# Logging
LOG_LEVEL=info
LOG_FORMAT=json

# Rate Limiting
RATE_LIMIT_WINDOW=60000
RATE_LIMIT_MAX=100

# Cache
CACHE_TTL=300

# Feature Flags
FEATURE_X_ENABLED=true
```

---

## 9.4 Docker Configuration

### Dockerfile (Backend)

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

USER appuser
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### docker-compose.yml (Development)

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/app
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - .:/app
      - /app/node_modules

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## 9.5 Health Checks

### Endpoints

| Endpoint | Descrição | Resposta |
|----------|-----------|----------|
| `GET /health` | Health básico | 200 se app ok |
| `GET /health/ready` | Readiness | 200 se pode receber tráfego |
| `GET /health/live` | Liveness | 200 se processo vivo |

### Implementação

```typescript
// GET /health
app.get('/health', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    version: process.env.npm_package_version
  };
  res.json(health);
});

// GET /health/ready
app.get('/health/ready', async (req, res) => {
  try {
    await db.$queryRaw`SELECT 1`;
    await redis.ping();
    res.json({ status: 'ready' });
  } catch (error) {
    res.status(503).json({ status: 'not ready', error: error.message });
  }
});
```

---

## 9.6 Estratégia de Deploy

### Blue-Green Deployment

```
┌─────────────┐     ┌─────────────┐
│    Blue     │     │   Green     │
│  (Current)  │     │   (New)     │
└──────┬──────┘     └──────┬──────┘
       │                   │
       └───────┬───────────┘
               │
        ┌──────▼──────┐
        │ Load Balancer│
        │   (Switch)   │
        └─────────────┘
```

### Rollback

1. Detectar problema (métricas, alertas)
2. Switch load balancer para versão anterior
3. Investigar e corrigir
4. Redeploy com fix

---

## 9.7 Backup e Disaster Recovery

### Backup

| Componente | Frequência | Retenção | Destino |
|------------|------------|----------|---------|
| PostgreSQL | Diário | 30 dias | S3/GCS |
| Redis | Diário | 7 dias | S3/GCS |
| Uploads | Contínuo | Indefinido | S3/GCS |

### Objetivos

| Métrica | Alvo |
|---------|------|
| RTO (Recovery Time Objective) | < 4 horas |
| RPO (Recovery Point Objective) | < 1 hora |

### Processo de Recovery

1. Identificar escopo do problema
2. Provisionar nova infraestrutura (se necessário)
3. Restaurar banco de dados do backup
4. Verificar integridade dos dados
5. Reconfigurar DNS/Load Balancer
6. Validar funcionamento

---

## 9.8 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker Image
        run: docker build -t app:${{ github.sha }} .
      - name: Push to Registry
        run: |
          docker tag app:${{ github.sha }} registry/app:${{ github.sha }}
          docker push registry/app:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Production
        run: |
          # Trigger deployment in cloud provider
          curl -X POST ${{ secrets.DEPLOY_WEBHOOK }}
```

---

← [Voltar para SPEC](README.md) | [Próximo: Observabilidade →](10-observabilidade.md)
