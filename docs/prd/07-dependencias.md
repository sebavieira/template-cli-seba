# 7. Dependências e Integrações

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## 7.1 Dependências Externas Críticas

### APIs de Terceiros

| Serviço | Finalidade | Custo Estimado | Status |
|---------|------------|----------------|--------|
| Evolution API | Integração WhatsApp | Gratuito (self-hosted) | ✅ Integrado |
| UAZAPI | Integração WhatsApp (backup) | Variável por volume | ✅ Integrado |
| OpenAI API | IA Conversacional | ~$0.01-0.03/1K tokens | ✅ Integrado |
| AWS S3 / Cloudflare R2 | Armazenamento de mídia | ~$0.023/GB | ✅ Integrado |

### Detalhamento de Integrações

#### Evolution API

- **Finalidade:** Conexão e comunicação com WhatsApp
- **Tipo:** API REST + Webhook
- **Autenticação:** API Key
- **Rate Limits:** Definido pela instância
- **Custo:** Gratuito (self-hosted) ou plano comercial
- **Documentação:** [Evolution API Docs](https://doc.evolution-api.com)

**Funcionalidades:**
- Criação de instâncias WhatsApp
- Geração de QR Code
- Envio/recebimento de mensagens
- Suporte a mídia (imagem, áudio, vídeo, documento)
- Webhooks para eventos

#### UAZAPI

- **Finalidade:** Provedor alternativo de WhatsApp
- **Tipo:** API REST + Webhook
- **Autenticação:** API Token
- **Rate Limits:** Por plano contratado
- **Custo:** Variável por volume de mensagens
- **Documentação:** [UAZAPI Docs](https://app.uazapi.com)

#### OpenAI API

- **Finalidade:** Geração de respostas com IA
- **Tipo:** API REST
- **Autenticação:** API Key (Bearer Token)
- **Rate Limits:** 10.000 RPM (GPT-4)
- **Custo:** ~$0.01/1K tokens input, ~$0.03/1K tokens output (GPT-4-turbo-mini)
- **Documentação:** [OpenAI API Docs](https://platform.openai.com/docs)

**Modelos Utilizados:**
- `gpt-4-turbo-mini`: Chatbot principal (custo-benefício)
- `gpt-4o`: Geração de prompts (onboarding)
- `text-embedding-ada-002`: Embeddings para busca semântica

---

## 7.2 Stack Tecnológico

### Backend

| Tecnologia | Versão | Finalidade |
|------------|--------|------------|
| Node.js | 20.x LTS | Runtime JavaScript |
| TypeScript | 5.6.3 | Tipagem estática |
| Fastify | 4.28.1 | Framework web de alta performance |
| Prisma | 5.22.0 | ORM e migrations |
| BullMQ | 5.x | Filas de jobs |
| Socket.io | 4.8.3 | WebSocket para real-time |

### Frontend

| Tecnologia | Versão | Finalidade |
|------------|--------|------------|
| React | 18.3.1 | Framework UI |
| TypeScript | 5.8.3 | Tipagem estática |
| Vite | 5.4.19 | Build tool |
| Tailwind CSS | 3.4.17 | Estilização utility-first |
| Shadcn/UI | Latest | Componentes UI |
| TanStack Query | 5.x | Data fetching |
| React Hook Form | 7.x | Formulários |

### Banco de Dados

| Tecnologia | Versão | Finalidade |
|------------|--------|------------|
| PostgreSQL | 16.x | Banco de dados principal |
| pgvector | 0.5.x | Extensão para embeddings |
| Redis | 7.x | Cache e filas |

### Infraestrutura

| Serviço | Provider | Finalidade |
|---------|----------|------------|
| Containers | Docker | Deployment consistente |
| Orchestration | Docker Compose | Gerenciamento de serviços |
| Reverse Proxy | Nginx | Load balancing, HTTPS |
| Storage | S3/R2/MinIO | Armazenamento de arquivos |

---

## 7.3 Custo Estimado Mensal

### Cenário: 1000 usuários ativos

| Categoria | Serviço | Custo |
|-----------|---------|-------|
| Hospedagem | VPS 4GB RAM | R$ 100-200 |
| Banco de Dados | PostgreSQL managed | R$ 50-150 |
| Redis | Cache managed | R$ 30-80 |
| OpenAI | ~500K tokens/dia | R$ 200-500 |
| Storage | 50GB S3/R2 | R$ 5-20 |
| **Total** | | **R$ 385-950** |

### Cenário: 10.000 usuários ativos

| Categoria | Serviço | Custo |
|-----------|---------|-------|
| Hospedagem | 3x VPS 8GB | R$ 600-1200 |
| Banco de Dados | PostgreSQL HA | R$ 400-800 |
| Redis | Cluster | R$ 200-400 |
| OpenAI | ~5M tokens/dia | R$ 2000-5000 |
| Storage | 500GB | R$ 50-100 |
| **Total** | | **R$ 3250-7500** |

---

## 7.4 Matriz de Risco de Dependências

| Dependência | Criticidade | Risco | Mitigação |
|-------------|-------------|-------|-----------|
| Evolution API | Alta | Instabilidade | UAZAPI como backup |
| OpenAI API | Alta | Custos, rate limits | Caching, modelo configurável |
| PostgreSQL | Alta | Perda de dados | Backups diários |
| Redis | Média | Perda de cache | Persistência opcional |
| S3/R2 | Média | Indisponibilidade | MinIO local como fallback |

---

## 7.5 SLAs e Garantias

| Serviço | SLA Esperado | Ação se Falhar |
|---------|--------------|----------------|
| Evolution API | 99% | Fallback para UAZAPI |
| OpenAI | 99.9% | Fila de retry |
| PostgreSQL | 99.9% | Réplica read-only |
| Redis | 99% | Fallback para DB |

---

## 7.6 Variáveis de Ambiente

```env
# Database
DATABASE_URL=postgresql://user:pass@host:5432/db

# Redis
REDIS_URL=redis://localhost:6379

# Auth
JWT_SECRET=your-secret-key

# OpenAI
OPENAI_API_KEY=sk-your-key

# WhatsApp - Evolution
EVOLUTION_API_URL=https://api.evolution.example.com
EVOLUTION_API_KEY=your-key

# WhatsApp - UAZAPI
UAZAPI_URL=https://app.uazapi.com
UAZAPI_TOKEN=your-token

# Storage
STORAGE_PROVIDER=s3|r2|local
STORAGE_BUCKET=bucket-name
STORAGE_REGION=us-east-1
STORAGE_ACCESS_KEY=key
STORAGE_SECRET_KEY=secret

# Server
PORT=3001
NODE_ENV=development|production
FRONTEND_URL=http://localhost:5173
```

---

[← Voltar para Índice PRD](README.md)
