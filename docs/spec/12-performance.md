# 12. Performance e Otimizações

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 12.1 Metas de Performance

### API Backend

| Métrica | Alvo | Crítico | Atual |
|---------|------|---------|-------|
| Latência (p50) | < 50ms | < 100ms | TBD |
| Latência (p95) | < 200ms | < 500ms | TBD |
| Latência (p99) | < 500ms | < 1s | TBD |
| Throughput | 500 req/s | 200 req/s | TBD |
| Error Rate | < 0.1% | < 1% | TBD |

### WhatsApp Webhook

| Métrica | Alvo | Crítico |
|---------|------|---------|
| Tempo de resposta | < 200ms | < 1s |
| Processamento de mensagem | < 500ms | < 2s |
| Filas (backlog máximo) | 100 jobs | 500 jobs |

### Frontend

| Métrica | Alvo | Crítico |
|---------|------|---------|
| LCP (Largest Contentful Paint) | < 2.5s | < 4s |
| FID (First Input Delay) | < 100ms | < 300ms |
| CLS (Cumulative Layout Shift) | < 0.1 | < 0.25 |
| TTI (Time to Interactive) | < 3s | < 5s |
| Bundle Size (initial) | < 300KB | < 500KB |

---

## 12.2 Database Optimization

### Índices Essenciais

```sql
-- Contatos
CREATE INDEX idx_contacts_company_id ON contacts(company_id);
CREATE INDEX idx_contacts_phone ON contacts(phone_number);
CREATE INDEX idx_contacts_company_phone ON contacts(company_id, phone_number);
CREATE INDEX idx_contacts_created_at ON contacts(created_at DESC);

-- Conversas
CREATE INDEX idx_conversations_company_id ON conversations(company_id);
CREATE INDEX idx_conversations_instance_id ON conversations(instance_id);
CREATE INDEX idx_conversations_remote_jid ON conversations(remote_jid);
CREATE INDEX idx_conversations_last_message ON conversations(last_message_at DESC);
CREATE INDEX idx_conversations_is_paused ON conversations(is_paused) WHERE is_paused = true;

-- Mensagens
CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_messages_created_at ON messages(created_at DESC);
CREATE INDEX idx_messages_key_id ON messages(message_key_id);

-- Deals
CREATE INDEX idx_deals_company_id ON deals(company_id);
CREATE INDEX idx_deals_stage_id ON deals(stage_id);
CREATE INDEX idx_deals_contact_id ON deals(contact_id);
CREATE INDEX idx_deals_status ON deals(status);
CREATE INDEX idx_deals_company_status ON deals(company_id, status);

-- Follow-ups
CREATE INDEX idx_followup_queue_status ON followup_queue(status);
CREATE INDEX idx_followup_queue_scheduled ON followup_queue(scheduled_for)
  WHERE status = 'PENDING';

-- pgvector para embeddings
CREATE INDEX idx_embeddings_vector ON conversation_embeddings
  USING ivfflat (embedding vector_cosine_ops);
```

### Query Optimization

```typescript
// ❌ N+1 Problem
const deals = await prisma.deal.findMany({ where: { companyId } });
for (const deal of deals) {
  deal.contact = await prisma.contact.findUnique({ where: { id: deal.contactId } });
  deal.stage = await prisma.stage.findUnique({ where: { id: deal.stageId } });
}

// ✅ Eager Loading
const deals = await prisma.deal.findMany({
  where: { companyId },
  include: {
    contact: {
      select: { id: true, name: true, phoneNumber: true }
    },
    stage: {
      select: { id: true, name: true, color: true }
    }
  }
});

// ✅ Select apenas campos necessários
const contacts = await prisma.contact.findMany({
  where: { companyId },
  select: {
    id: true,
    name: true,
    phoneNumber: true,
    email: true
    // Não seleciona campos grandes como notes, metadata
  }
});

// ✅ Paginação cursor-based para grandes datasets
const messages = await prisma.message.findMany({
  where: { conversationId },
  take: 50,
  skip: cursor ? 1 : 0,
  cursor: cursor ? { id: cursor } : undefined,
  orderBy: { createdAt: 'desc' }
});
```

### Connection Pooling

```typescript
// Prisma com pool configurado
const prisma = new PrismaClient({
  log: process.env.NODE_ENV === 'development'
    ? ['query', 'warn', 'error']
    : ['error']
});

// Connection string com pool
// DATABASE_URL=postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=30
```

### Query Analysis

```sql
-- Analisar query lenta
EXPLAIN ANALYZE
SELECT * FROM messages
WHERE conversation_id = 'uuid'
ORDER BY created_at DESC
LIMIT 50;

-- Identificar índices faltando
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
WHERE idx_scan = 0;

-- Queries mais lentas
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

---

## 12.3 Caching Strategy

### Arquitetura de Cache

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Request    │────▶│  L1 Cache   │────▶│  L2 Cache   │
│             │     │  (Memory)   │     │   (Redis)   │
└─────────────┘     └──────┬──────┘     └──────┬──────┘
                           │                    │
                           ▼                    ▼
                    ┌─────────────┐     ┌─────────────┐
                    │  Hit? Return│     │  Hit? Return│
                    │  Miss? Next │     │  Miss? DB   │
                    └─────────────┘     └─────────────┘
```

### Níveis de Cache

| Nível | TTL | Uso | Tamanho |
|-------|-----|-----|---------|
| L1 (Memory) | 1-5 min | Hot data, sessões | ~100MB |
| L2 (Redis) | 5-60 min | Dados compartilhados | ~1GB |
| L3 (CDN) | 1-24h | Assets estáticos | Ilimitado |

### Implementação com Redis

```typescript
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Cache genérico com TTL
async function cacheGet<T>(
  key: string,
  ttl: number,
  fetchFn: () => Promise<T>
): Promise<T> {
  // Tenta cache primeiro
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // Busca e cacheia
  const data = await fetchFn();
  await redis.setex(key, ttl, JSON.stringify(data));
  return data;
}

// Exemplos de uso
const contact = await cacheGet(
  `contact:${contactId}`,
  300, // 5 minutos
  () => prisma.contact.findUnique({ where: { id: contactId } })
);

const funnel = await cacheGet(
  `funnel:${funnelId}:stages`,
  600, // 10 minutos
  () => prisma.stage.findMany({ where: { funnelId }, orderBy: { position: 'asc' } })
);

// Prompt de IA (cache mais longo - raramente muda)
const prompt = await cacheGet(
  `company:${companyId}:ai-prompt`,
  3600, // 1 hora
  () => prisma.company.findUnique({
    where: { id: companyId },
    select: { aiPrompt: true }
  })
);
```

### Cache Invalidation

```typescript
// Padrões de invalidação
class CacheService {
  private redis: Redis;

  // Invalidar específico
  async invalidate(key: string) {
    await this.redis.del(key);
  }

  // Invalidar por padrão
  async invalidatePattern(pattern: string) {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }

  // Invalidar após update de contato
  async onContactUpdated(contactId: string) {
    await this.invalidate(`contact:${contactId}`);
    // Invalida também listas que podem conter este contato
    await this.invalidatePattern(`contacts:list:*`);
  }

  // Invalidar após mudança no funil
  async onFunnelUpdated(funnelId: string) {
    await this.invalidatePattern(`funnel:${funnelId}:*`);
  }
}

// Integrar com hooks do Prisma
prisma.$use(async (params, next) => {
  const result = await next(params);

  // Invalidar cache após mutations
  if (['create', 'update', 'delete'].includes(params.action)) {
    if (params.model === 'Contact') {
      await cacheService.onContactUpdated(result.id);
    }
    if (params.model === 'Stage' || params.model === 'Funnel') {
      await cacheService.onFunnelUpdated(params.args.where.funnelId || result.funnelId);
    }
  }

  return result;
});
```

---

## 12.4 API Optimization

### Response Compression

```typescript
import fastifyCompress from '@fastify/compress';

app.register(fastifyCompress, {
  global: true,
  encodings: ['gzip', 'deflate', 'br'], // Brotli preferido
  threshold: 1024, // Comprimir respostas > 1KB
  customTypes: /application\/json/
});
```

### Pagination

```typescript
// Cursor-based (recomendado para listas grandes)
interface PaginatedResponse<T> {
  data: T[];
  nextCursor: string | null;
  hasMore: boolean;
}

async function getMessages(
  conversationId: string,
  cursor?: string,
  limit: number = 50
): Promise<PaginatedResponse<Message>> {
  const messages = await prisma.message.findMany({
    where: { conversationId },
    take: limit + 1, // Busca 1 a mais para saber se tem mais
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' }
  });

  const hasMore = messages.length > limit;
  const data = hasMore ? messages.slice(0, -1) : messages;

  return {
    data,
    nextCursor: hasMore ? data[data.length - 1].id : null,
    hasMore
  };
}

// Offset-based (para tabelas com filtros complexos)
async function getContacts(
  companyId: string,
  page: number = 1,
  limit: number = 20
): Promise<{ data: Contact[]; total: number; pages: number }> {
  const [data, total] = await Promise.all([
    prisma.contact.findMany({
      where: { companyId },
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: 'desc' }
    }),
    prisma.contact.count({ where: { companyId } })
  ]);

  return {
    data,
    total,
    pages: Math.ceil(total / limit)
  };
}
```

### Field Selection (Sparse Fieldsets)

```typescript
// GET /api/contacts?fields=id,name,phoneNumber
app.get('/contacts', async (request, reply) => {
  const { fields } = request.query as { fields?: string };

  const select = fields
    ? Object.fromEntries(fields.split(',').map(f => [f.trim(), true]))
    : undefined;

  const contacts = await prisma.contact.findMany({
    where: { companyId: request.user.companyId },
    select
  });

  return { success: true, data: contacts };
});
```

---

## 12.5 Async Processing (BullMQ)

### Configuração dos Workers

```typescript
import { Queue, Worker, QueueScheduler } from 'bullmq';

const connection = { host: 'localhost', port: 6379 };

// Filas
const messageQueue = new Queue('message-worker', { connection });
const followupQueue = new Queue('followup-worker', { connection });
const embeddingQueue = new Queue('embedding-worker', { connection });
const flowQueue = new Queue('flow-worker', { connection });

// Workers com concorrência otimizada
const messageWorker = new Worker('message-worker', processMessage, {
  connection,
  concurrency: parseInt(process.env.MESSAGE_WORKER_CONCURRENCY || '10'),
  limiter: {
    max: 100,
    duration: 1000 // 100 jobs/segundo
  }
});

const embeddingWorker = new Worker('embedding-worker', processEmbedding, {
  connection,
  concurrency: parseInt(process.env.EMBEDDING_WORKER_CONCURRENCY || '3'),
  limiter: {
    max: 20,
    duration: 1000 // Rate limit OpenAI
  }
});
```

### Job Options Otimizadas

```typescript
// Adicionar job com configurações de retry
await messageQueue.add('process-message', {
  conversationId,
  messageId
}, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 1000 // 1s, 2s, 4s
  },
  removeOnComplete: {
    age: 3600, // Remove após 1 hora
    count: 1000 // Mantém últimos 1000
  },
  removeOnFail: {
    age: 86400 // Remove falhos após 24h
  }
});

// Job com prioridade
await messageQueue.add('urgent-message', data, {
  priority: 1 // Menor = maior prioridade
});

// Job agendado
await followupQueue.add('send-followup', data, {
  delay: 3600000 // 1 hora
});
```

### Monitoramento de Filas

```typescript
// Health check das filas
async function getQueueHealth() {
  const [messageStats, followupStats, embeddingStats] = await Promise.all([
    messageQueue.getJobCounts(),
    followupQueue.getJobCounts(),
    embeddingQueue.getJobCounts()
  ]);

  return {
    messageWorker: {
      waiting: messageStats.waiting,
      active: messageStats.active,
      completed: messageStats.completed,
      failed: messageStats.failed
    },
    followupWorker: followupStats,
    embeddingWorker: embeddingStats
  };
}

// Alertar se fila crescendo
if (messageStats.waiting > 500) {
  logger.warn({ queueSize: messageStats.waiting }, 'Message queue backlog');
}
```

---

## 12.6 Frontend Performance

### Code Splitting com Vite

```typescript
// Lazy loading de rotas
import { lazy, Suspense } from 'react';

const Conversations = lazy(() => import('./pages/Conversations'));
const CRM = lazy(() => import('./pages/CRM'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/conversations" element={<Conversations />} />
        <Route path="/crm/*" element={<CRM />} />
        <Route path="/settings/*" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### Otimização de Bundle

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendors separados
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          'vendor-query': ['@tanstack/react-query'],
          'vendor-editor': ['@tiptap/react', '@tiptap/starter-kit'],
        }
      }
    },
    chunkSizeWarningLimit: 500
  }
});
```

### TanStack Query Optimization

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minuto
      gcTime: 1000 * 60 * 5, // 5 minutos
      refetchOnWindowFocus: false,
      retry: 1
    }
  }
});

// Query com cache otimizado
const { data: contacts } = useQuery({
  queryKey: ['contacts', { page, search }],
  queryFn: () => api.getContacts({ page, search }),
  placeholderData: keepPreviousData, // Mantém dados anteriores durante refetch
  staleTime: 30000 // 30 segundos
});

// Prefetch para navegação
const prefetchConversation = async (conversationId: string) => {
  await queryClient.prefetchQuery({
    queryKey: ['conversation', conversationId],
    queryFn: () => api.getConversation(conversationId)
  });
};
```

### Virtualization para Listas Grandes

```typescript
import { useVirtualizer } from '@tanstack/react-virtual';

function MessageList({ messages }: { messages: Message[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: messages.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 80, // Altura estimada de cada mensagem
    overscan: 5 // Renderiza 5 itens extras fora da viewport
  });

  return (
    <div ref={parentRef} className="h-full overflow-auto">
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              transform: `translateY(${virtualItem.start}px)`
            }}
          >
            <MessageItem message={messages[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 12.7 WebSocket Optimization

### Socket.io Performance

```typescript
import { Server } from 'socket.io';

const io = new Server(server, {
  cors: { origin: process.env.FRONTEND_URL },
  // Otimizações
  pingTimeout: 60000,
  pingInterval: 25000,
  transports: ['websocket'], // Sem polling fallback
  perMessageDeflate: {
    threshold: 1024 // Comprimir mensagens > 1KB
  }
});

// Namespaces por empresa (isolamento)
io.of(/^\/company\/[\w-]+$/).on('connection', (socket) => {
  const companyId = socket.nsp.name.split('/')[2];

  // Autenticação
  const token = socket.handshake.auth.token;
  const user = verifyToken(token);

  if (user.companyId !== companyId) {
    socket.disconnect();
    return;
  }

  // Rooms por conversa
  socket.on('join:conversation', (conversationId) => {
    socket.join(`conversation:${conversationId}`);
  });

  socket.on('leave:conversation', (conversationId) => {
    socket.leave(`conversation:${conversationId}`);
  });
});

// Emit otimizado (apenas para rooms relevantes)
function emitNewMessage(message: Message) {
  io.of(`/company/${message.companyId}`)
    .to(`conversation:${message.conversationId}`)
    .emit('message:new', message);
}
```

---

## 12.8 Performance Monitoring

### Métricas de Performance

```typescript
// Middleware de métricas
app.addHook('onResponse', (request, reply, done) => {
  const responseTime = reply.getResponseTime();

  // Log requests lentas
  if (responseTime > 500) {
    logger.warn({
      method: request.method,
      url: request.url,
      responseTime,
      statusCode: reply.statusCode
    }, 'Slow request detected');
  }

  // Métricas para Prometheus
  httpRequestDuration.observe(
    { method: request.method, path: request.routerPath },
    responseTime / 1000
  );

  done();
});

// Monitor de memória
setInterval(() => {
  const usage = process.memoryUsage();

  if (usage.heapUsed > 500 * 1024 * 1024) { // 500MB
    logger.warn({ memoryUsage: usage }, 'High memory usage');
  }

  memoryGauge.set({ type: 'heapUsed' }, usage.heapUsed);
  memoryGauge.set({ type: 'heapTotal' }, usage.heapTotal);
}, 30000);

// Event loop lag
const lagMonitor = require('event-loop-lag')(1000);
setInterval(() => {
  const lag = lagMonitor();
  if (lag > 100) {
    logger.warn({ eventLoopLag: lag }, 'High event loop lag');
  }
  eventLoopLagGauge.set(lag);
}, 5000);
```

---

## 12.9 Performance Checklist

### Database
- [x] Índices para queries frequentes
- [x] Connection pooling configurado (20 conexões)
- [x] Queries otimizadas (no N+1)
- [ ] EXPLAIN em queries lentas
- [x] Paginação cursor-based para grandes datasets

### Caching
- [x] Redis para dados frequentes
- [x] TTL apropriado por tipo de dado
- [x] Invalidação implementada
- [ ] CDN para assets estáticos

### API
- [x] Compression habilitada (gzip/brotli)
- [x] Paginação em todas as listas
- [x] Field selection disponível
- [x] Rate limiting por endpoint

### Workers
- [x] Concorrência configurada por worker
- [x] Retry com exponential backoff
- [x] DLQ para falhas persistentes
- [x] Monitoramento de filas

### Frontend
- [x] Code splitting por rota
- [x] Virtualization para listas longas
- [x] TanStack Query com cache
- [x] Bundle size < 500KB

### WebSocket
- [x] Namespaces por empresa
- [x] Rooms por conversa
- [x] Compressão habilitada
- [x] Apenas transporte websocket

---

← [Voltar para SPEC](README.md) | [Próximo: Rastreabilidade →](13-rastreabilidade.md)
