# 12. Performance e Otimizações

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

← [Voltar para SPEC](README.md)

---

## 12.1 Metas de Performance

| Métrica | Alvo | Crítico |
|---------|------|---------|
| Latência API (p50) | < 50ms | < 100ms |
| Latência API (p95) | < 200ms | < 500ms |
| Latência API (p99) | < 500ms | < 1s |
| Throughput | 1000 req/s | 500 req/s |
| Error Rate | < 0.1% | < 1% |
| Uptime | 99.9% | 99% |

---

## 12.2 Database Optimization

### Índices

```sql
-- Índices para queries frequentes
CREATE INDEX idx_{{tabela}}_user_id ON {{tabela}}(user_id);
CREATE INDEX idx_{{tabela}}_status ON {{tabela}}(status);
CREATE INDEX idx_{{tabela}}_created_at ON {{tabela}}(created_at DESC);

-- Índice composto para filtros combinados
CREATE INDEX idx_{{tabela}}_user_status ON {{tabela}}(user_id, status);

-- Índice parcial para queries específicas
CREATE INDEX idx_{{tabela}}_active ON {{tabela}}(id)
  WHERE status = 'active' AND deleted_at IS NULL;
```

### Query Optimization

```typescript
// ❌ N+1 Problem
const users = await prisma.user.findMany();
for (const user of users) {
  user.posts = await prisma.post.findMany({ where: { userId: user.id } });
}

// ✅ Eager Loading
const users = await prisma.user.findMany({
  include: { posts: true }
});

// ✅ Select only needed fields
const users = await prisma.user.findMany({
  select: { id: true, name: true, email: true }
});
```

### Connection Pooling

```typescript
// Prisma
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  }
});

// Pool configuration (connection string)
// ?connection_limit=10&pool_timeout=30
```

---

## 12.3 Caching Strategy

### Níveis de Cache

| Nível | TTL | Uso |
|-------|-----|-----|
| L1 (Memory) | 1-5 min | Hot data |
| L2 (Redis) | 5-60 min | Shared data |
| L3 (CDN) | 1-24h | Static assets |

### Implementação com Redis

```typescript
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getWithCache<T>(
  key: string,
  ttl: number,
  fetchFn: () => Promise<T>
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // Fetch and cache
  const data = await fetchFn();
  await redis.setex(key, ttl, JSON.stringify(data));
  return data;
}

// Uso
const user = await getWithCache(
  `user:${userId}`,
  300, // 5 minutos
  () => prisma.user.findUnique({ where: { id: userId } })
);
```

### Cache Invalidation

```typescript
// Invalidar após update
async function updateUser(id: string, data: UpdateUserDTO) {
  const user = await prisma.user.update({
    where: { id },
    data
  });

  // Invalidar cache
  await redis.del(`user:${id}`);
  await redis.del(`users:list:*`); // Pattern delete

  return user;
}
```

---

## 12.4 API Optimization

### Pagination

```typescript
// Cursor-based (recomendado para grandes datasets)
const items = await prisma.{{entidade}}.findMany({
  take: 20,
  skip: 1, // Skip cursor
  cursor: { id: lastId },
  orderBy: { createdAt: 'desc' }
});

// Offset-based (simpler)
const items = await prisma.{{entidade}}.findMany({
  skip: (page - 1) * limit,
  take: limit
});
```

### Response Compression

```typescript
import compression from 'compression';

app.use(compression({
  level: 6,
  threshold: 1024, // Compress responses > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

### Field Selection

```typescript
// Cliente especifica campos desejados
// GET /api/users?fields=id,name,email

app.get('/api/users', async (req, res) => {
  const fields = req.query.fields?.split(',');

  const select = fields
    ? Object.fromEntries(fields.map(f => [f, true]))
    : undefined;

  const users = await prisma.user.findMany({ select });
  res.json(users);
});
```

---

## 12.5 Async Processing

### Queue-based Processing

```typescript
import { Queue, Worker } from 'bullmq';

const queue = new Queue('{{queue_name}}');

// Adicionar job
await queue.add('process', { data }, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 1000
  }
});

// Worker
const worker = new Worker('{{queue_name}}', async (job) => {
  // Processamento pesado aqui
  await heavyProcessing(job.data);
}, {
  concurrency: 5
});
```

---

## 12.6 Frontend Performance

### Code Splitting

```typescript
// Next.js dynamic imports
import dynamic from 'next/dynamic';

const HeavyComponent = dynamic(
  () => import('../components/HeavyComponent'),
  { loading: () => <Skeleton /> }
);
```

### Image Optimization

```typescript
// Next.js Image
import Image from 'next/image';

<Image
  src="/image.jpg"
  width={800}
  height={600}
  loading="lazy"
  placeholder="blur"
/>
```

---

## 12.7 Monitoring Performance

### Key Metrics to Track

```typescript
// Request duration
const start = process.hrtime.bigint();
// ... process request
const end = process.hrtime.bigint();
const durationMs = Number(end - start) / 1e6;

// Memory usage
const memUsage = process.memoryUsage();
// { heapUsed, heapTotal, external, rss }

// Event loop lag
const start = Date.now();
setImmediate(() => {
  const lag = Date.now() - start;
  // Report lag
});
```

---

## 12.8 Performance Checklist

### Database
- [ ] Índices para queries frequentes
- [ ] Connection pooling configurado
- [ ] Queries otimizadas (no N+1)
- [ ] EXPLAIN em queries lentas

### Caching
- [ ] Redis para dados frequentes
- [ ] TTL apropriado
- [ ] Invalidação implementada
- [ ] CDN para assets

### API
- [ ] Pagination implementada
- [ ] Compression habilitada
- [ ] Gzip/Brotli
- [ ] Field selection

### Frontend
- [ ] Code splitting
- [ ] Image optimization
- [ ] Lazy loading
- [ ] Bundle analysis

---

← [Voltar para SPEC](README.md) | [Próximo: Rastreabilidade →](13-rastreabilidade.md)
