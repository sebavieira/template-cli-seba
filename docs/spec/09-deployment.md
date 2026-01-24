# 9. Deployment e Infraestrutura

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 9.1 Arquitetura de Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│                    Cloud Infrastructure                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│   │   Frontend  │    │  Backend    │    │  Workers    │        │
│   │   (Vercel)  │    │  (Railway)  │    │  (Railway)  │        │
│   └─────────────┘    └─────────────┘    └─────────────┘        │
│          │                  │                  │                 │
│          │    ┌─────────────┼─────────────────┼──────┐          │
│          │    │             │                  │      │          │
│          ▼    ▼             ▼                  ▼      ▼          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│   │  PostgreSQL  │  │    Redis     │  │   Storage    │         │
│   │   + pgvector │  │   (BullMQ)   │  │    (S3)      │         │
│   └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                  │
│   ┌──────────────────────────────────────────────────┐          │
│   │              External Services                    │          │
│   │   Evolution API  │  OpenAI  │  UAZAPI            │          │
│   └──────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9.2 Recursos Computacionais

### Ambiente de Produção

| Serviço | Recurso | Especificação | Custo Estimado |
|---------|---------|---------------|----------------|
| Frontend | Vercel Pro | Serverless Edge | ~$20/mês |
| Backend API | Railway | 2 vCPU, 2GB RAM | ~$20-40/mês |
| Workers (4x) | Railway | 1 vCPU, 1GB RAM cada | ~$40-80/mês |
| PostgreSQL 16 | Managed DB | 10GB storage + pgvector | ~$30-50/mês |
| Redis 7 | Managed | 512MB | ~$10-20/mês |
| Storage (S3) | AWS/Cloudflare R2 | 50GB | ~$5/mês |

**Total Estimado:** $125-215/mês

### Ambiente de Staging

| Serviço | Especificação | Custo |
|---------|---------------|-------|
| Frontend | Vercel Hobby | Gratuito |
| Backend | Railway Starter | ~$5/mês |
| Workers | Railway (1 instância) | ~$5/mês |
| PostgreSQL | Railway PostgreSQL | ~$5/mês |
| Redis | Railway Redis | ~$5/mês |

**Total Staging:** ~$20/mês

---

## 9.3 Variáveis de Ambiente

### Backend - Obrigatórias

```bash
# Aplicação
NODE_ENV=production
PORT=3000
API_URL=https://api.evoaiconnect.com
FRONTEND_URL=https://app.evoaiconnect.com

# Banco de Dados
DATABASE_URL=postgresql://user:pass@host:5432/evoai?sslmode=require

# Redis
REDIS_URL=redis://user:pass@host:6379

# Autenticação
JWT_SECRET=your-super-secret-key-min-32-chars
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# OpenAI
OPENAI_API_KEY=sk-xxx
OPENAI_MODEL=gpt-4o-mini

# Storage
STORAGE_PROVIDER=s3
S3_BUCKET=evoai-uploads
S3_REGION=us-east-1
S3_ACCESS_KEY=xxx
S3_SECRET_KEY=xxx
```

### Backend - Opcionais

```bash
# Logging
LOG_LEVEL=info

# Rate Limiting
RATE_LIMIT_WINDOW=60000
RATE_LIMIT_MAX=100

# Workers
MESSAGE_WORKER_CONCURRENCY=10
FOLLOWUP_WORKER_CONCURRENCY=5
EMBEDDING_WORKER_CONCURRENCY=3

# Feature Flags
ENABLE_AI_ANALYSIS=true
ENABLE_EMBEDDINGS=true
```

### Frontend

```bash
VITE_API_URL=https://api.evoaiconnect.com
VITE_WS_URL=wss://api.evoaiconnect.com
VITE_STORAGE_URL=https://storage.evoaiconnect.com
```

---

## 9.4 Docker Configuration

### Dockerfile (Backend)

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build

# Production stage
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
COPY --from=builder /app/prisma ./prisma

USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### docker-compose.yml (Development)

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./server
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/evoai
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=dev-secret-key-32-characters-min
    depends_on:
      - db
      - redis
    volumes:
      - ./server:/app
      - /app/node_modules

  frontend:
    build:
      context: ./client
      dockerfile: Dockerfile.dev
    ports:
      - "5173:5173"
    environment:
      - VITE_API_URL=http://localhost:3000
    volumes:
      - ./client:/app
      - /app/node_modules

  db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: evoai
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

  evolution:
    image: atendai/evolution-api:latest
    ports:
      - "8080:8080"
    environment:
      - SERVER_URL=http://localhost:8080

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
  try {
    // Verifica PostgreSQL
    await prisma.$queryRaw`SELECT 1`;

    // Verifica Redis
    await redis.ping();

    return { status: 'ready' };
  } catch (error) {
    reply.status(503);
    return { status: 'not ready', error: error.message };
  }
});

// GET /health/live
app.get('/health/live', async (request, reply) => {
  return { status: 'alive', pid: process.pid };
});
```

---

## 9.6 Estratégia de Deploy

### Fluxo de Deploy

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Push   │────▶│  Tests  │────▶│  Build  │────▶│ Deploy  │
│  main   │     │  (CI)   │     │ (Docker)│     │ (Prod)  │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                     │                              │
                     ▼                              ▼
                ┌─────────┐                   ┌─────────┐
                │  Fail   │                   │ Monitor │
                │ (Block) │                   │ (Alert) │
                └─────────┘                   └─────────┘
```

### Zero-Downtime Deployment

1. **Nova versão** é deployada em novas instâncias
2. **Health check** verifica se está pronta
3. **Load balancer** redireciona tráfego gradualmente
4. **Instâncias antigas** são terminadas após drenagem

### Rollback

1. Detectar problema (métricas, alertas, erros)
2. Reverter para versão anterior no Railway/Vercel
3. Investigar logs e corrigir
4. Redeploy com fix

---

## 9.7 Backup e Disaster Recovery

### Backup

| Componente | Frequência | Retenção | Destino |
|------------|------------|----------|---------|
| PostgreSQL | Diário (00:00 UTC) | 30 dias | S3 |
| PostgreSQL | A cada 6h (PITR) | 7 dias | Railway |
| Redis | Diário | 7 dias | RDB snapshots |
| Uploads (S3) | Contínuo (versioning) | Indefinido | S3 Cross-region |

### Objetivos

| Métrica | Alvo |
|---------|------|
| RTO (Recovery Time Objective) | < 2 horas |
| RPO (Recovery Point Objective) | < 1 hora |

### Processo de Recovery

1. **Identificar** escopo do problema
2. **Comunicar** equipe e stakeholders
3. **Provisionar** nova infraestrutura (se necessário)
4. **Restaurar** banco de dados do backup mais recente
5. **Verificar** integridade dos dados
6. **Reconfigurar** DNS se necessário
7. **Validar** funcionamento completo
8. **Documentar** incidente e RCA

---

## 9.8 CI/CD Pipeline

### GitHub Actions - Deploy

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install & Test Server
        working-directory: ./server
        run: |
          npm ci
          npm run lint
          npm run type-check
          npm run test

      - name: Install & Test Client
        working-directory: ./client
        run: |
          npm ci
          npm run lint
          npm run type-check
          npm run build

  deploy-backend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Railway
        uses: railwayapp/railway-deploy@v1
        with:
          token: ${{ secrets.RAILWAY_TOKEN }}
          service: backend

  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'

  migrate:
    needs: deploy-backend
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Migrations
        run: |
          cd server
          npm ci
          npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## 9.9 Scaling

### Horizontal Scaling

| Componente | Trigger | Ação |
|------------|---------|------|
| Backend API | CPU > 70% por 5min | +1 instância (max 4) |
| Workers | Queue depth > 1000 | +1 worker (max 8) |
| PostgreSQL | Connections > 80% | Aumentar pool |

### Limites por Plano

| Plano | Instâncias WhatsApp | Contatos | Mensagens/mês |
|-------|--------------------:|----------:|---------------:|
| Free | 1 | 500 | 5.000 |
| Pro | 3 | 10.000 | 50.000 |
| Business | 10 | 50.000 | 200.000 |
| Enterprise | Ilimitado | Ilimitado | Ilimitado |

---

← [Voltar para SPEC](README.md) | [Próximo: Observabilidade →](10-observabilidade.md)
