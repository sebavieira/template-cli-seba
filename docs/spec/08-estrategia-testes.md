# 8. Estratégia de Testes

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 8.1 Pirâmide de Testes

```
        /\
       /  \
      / E2E \      10% - Testes de ponta a ponta
     /______\
    /        \
   / Integração\   30% - Testes de integração
  /______________\
 /                \
/    Unitários    \  60% - Testes unitários
\__________________/
```

### Distribuição Alvo

| Tipo | Percentual | Cobertura Mínima |
|------|------------|------------------|
| Unitários | 60% | ≥ 80% |
| Integração | 30% | ≥ 70% |
| E2E | 10% | Fluxos críticos |

---

## 8.2 Testes Unitários

### Ferramentas

- **Runner:** Vitest
- **Mocks:** vi.mock
- **Coverage:** c8 / Istanbul

### Exemplo de Teste Unitário - Backend

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { contactsService } from './contacts.service';

describe('ContactsService', () => {
  let mockPrisma: any;

  beforeEach(() => {
    mockPrisma = {
      contact: {
        findFirst: vi.fn(),
        create: vi.fn(),
        update: vi.fn(),
      },
    };
    vi.mock('../../lib/prisma', () => ({ prisma: mockPrisma }));
  });

  describe('createContact', () => {
    it('should create contact with valid data', async () => {
      const data = {
        phoneNumber: '5511999999999',
        name: 'Test Contact'
      };
      const expected = { id: 'uuid', ...data };

      mockPrisma.contact.findFirst.mockResolvedValue(null);
      mockPrisma.contact.create.mockResolvedValue(expected);

      const result = await contactsService.createContact('company-id', 'user-id', data);

      expect(mockPrisma.contact.create).toHaveBeenCalled();
      expect(result).toEqual(expected);
    });

    it('should throw error for duplicate phone number', async () => {
      const data = { phoneNumber: '5511999999999' };
      mockPrisma.contact.findFirst.mockResolvedValue({ id: 'existing' });

      await expect(
        contactsService.createContact('company-id', 'user-id', data)
      ).rejects.toThrow('Número já cadastrado');
    });
  });
});
```

### Exemplo de Teste Unitário - Frontend

```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { ConversationItem } from './ConversationItem';

describe('ConversationItem', () => {
  const mockConversation = {
    id: 'uuid',
    contactName: 'João Silva',
    contactNumber: '5511999999999',
    lastMessage: { messageText: 'Olá!' },
    unreadCount: 3,
  };

  it('should render contact name', () => {
    render(<ConversationItem conversation={mockConversation} />);
    expect(screen.getByText('João Silva')).toBeInTheDocument();
  });

  it('should show unread badge when unreadCount > 0', () => {
    render(<ConversationItem conversation={mockConversation} />);
    expect(screen.getByText('3')).toBeInTheDocument();
  });

  it('should not show unread badge when unreadCount is 0', () => {
    render(<ConversationItem conversation={{ ...mockConversation, unreadCount: 0 }} />);
    expect(screen.queryByText('0')).not.toBeInTheDocument();
  });
});
```

### O que Testar

- [x] Lógica de negócio (services)
- [x] Validações Zod
- [x] Edge cases
- [x] Error handling
- [x] Formatação de dados
- [x] Componentes React isolados
- [ ] Integração com DB (mock)
- [ ] APIs externas (mock)

---

## 8.3 Testes de Integração

### Ferramentas

- **Runner:** Vitest
- **HTTP:** Supertest / @fastify/inject
- **DB:** Testcontainers / PostgreSQL de teste

### Exemplo de Teste de Integração - API

```typescript
import { describe, it, expect, beforeAll, afterEach } from 'vitest';
import { buildApp } from '../app';
import { prisma } from '../lib/prisma';

describe('POST /api/contacts', () => {
  let app: FastifyInstance;
  let token: string;

  beforeAll(async () => {
    app = await buildApp();
    // Cria usuário de teste e obtém token
    token = await getTestToken(app);
  });

  afterEach(async () => {
    await prisma.contact.deleteMany({ where: { name: { startsWith: 'Test' } } });
  });

  it('should create contact with valid data', async () => {
    const response = await app.inject({
      method: 'POST',
      url: '/api/contacts',
      headers: { Authorization: `Bearer ${token}` },
      payload: {
        name: 'Test Contact',
        phoneNumber: '5511999999999',
        email: 'test@example.com'
      }
    });

    expect(response.statusCode).toBe(201);
    expect(response.json()).toMatchObject({
      success: true,
      data: {
        id: expect.any(String),
        name: 'Test Contact',
        phoneNumber: '5511999999999'
      }
    });
  });

  it('should return 400 for duplicate phone number', async () => {
    // Cria primeiro contato
    await app.inject({
      method: 'POST',
      url: '/api/contacts',
      headers: { Authorization: `Bearer ${token}` },
      payload: { name: 'Test 1', phoneNumber: '5511888888888' }
    });

    // Tenta criar com mesmo número
    const response = await app.inject({
      method: 'POST',
      url: '/api/contacts',
      headers: { Authorization: `Bearer ${token}` },
      payload: { name: 'Test 2', phoneNumber: '5511888888888' }
    });

    expect(response.statusCode).toBe(400);
    expect(response.json().error).toContain('Número');
  });

  it('should return 401 without auth', async () => {
    const response = await app.inject({
      method: 'POST',
      url: '/api/contacts',
      payload: { name: 'Test' }
    });

    expect(response.statusCode).toBe(401);
  });
});
```

### Testes de Worker (BullMQ)

```typescript
import { describe, it, expect, vi } from 'vitest';
import { messageWorker } from './message.worker';

describe('MessageWorker', () => {
  it('should process message and call AI', async () => {
    const mockJob = {
      data: {
        conversationId: 'uuid',
        messageId: 'uuid',
        instanceId: 'uuid'
      }
    };

    const mockAiService = vi.spyOn(aiService, 'generateResponse');
    mockAiService.mockResolvedValue('Resposta da IA');

    await messageWorker.process(mockJob);

    expect(mockAiService).toHaveBeenCalled();
  });
});
```

---

## 8.4 Testes E2E

### Ferramentas

- **Browser:** Playwright
- **API:** @playwright/test
- **Fixtures:** Factory functions

### Exemplo de Teste E2E - WhatsApp Chat

```typescript
import { test, expect } from '@playwright/test';

test.describe('WhatsApp Chat Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    await page.waitForURL('/conversations');
  });

  test('should send message and see it in chat', async ({ page }) => {
    // Seleciona primeira conversa
    await page.click('[data-testid="conversation-item"]');

    // Digita mensagem
    await page.fill('[data-testid="message-input"]', 'Olá, teste E2E!');
    await page.click('[data-testid="send-button"]');

    // Verifica que mensagem aparece
    await expect(page.locator('text=Olá, teste E2E!')).toBeVisible();
    await expect(page.locator('[data-testid="message-status-sent"]')).toBeVisible();
  });

  test('should pause and resume AI', async ({ page }) => {
    await page.click('[data-testid="conversation-item"]');

    // Pausa IA
    await page.click('[data-testid="pause-ai-button"]');
    await expect(page.locator('text=IA Pausada')).toBeVisible();

    // Retoma IA
    await page.click('[data-testid="resume-ai-button"]');
    await expect(page.locator('text=IA Pausada')).not.toBeVisible();
  });
});
```

### Exemplo de Teste E2E - CRM Funnel

```typescript
test.describe('CRM Funnel Flow', () => {
  test('should create deal and move through stages', async ({ page }) => {
    await page.goto('/crm');

    // Cria novo deal
    await page.click('button:text("Novo Deal")');
    await page.fill('[name="title"]', 'Proposta E2E Test');
    await page.fill('[name="value"]', '5000');
    await page.selectOption('[name="contactId"]', { index: 1 });
    await page.click('button:text("Salvar")');

    // Verifica criação
    await expect(page.locator('text=Proposta E2E Test')).toBeVisible();

    // Arrasta para próximo estágio (drag and drop)
    const deal = page.locator('[data-testid="deal-card"]:has-text("E2E Test")');
    const targetStage = page.locator('[data-testid="stage-column"]:nth-child(2)');
    await deal.dragTo(targetStage);

    // Verifica movimentação
    await expect(
      page.locator('[data-testid="stage-column"]:nth-child(2) >> text=Proposta E2E Test')
    ).toBeVisible();
  });
});
```

---

## 8.5 Testes de Carga

### Ferramentas

- **k6** (recomendado)
- Artillery
- Locust

### Exemplo com k6 - WhatsApp Webhook

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },   // Ramp up
    { duration: '2m', target: 100 },   // Peak (simula 100 msgs/s)
    { duration: '30s', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% < 500ms
    http_req_failed: ['rate<0.01'],    // <1% errors
  },
};

const payload = JSON.stringify({
  event: 'messages.upsert',
  instance: 'test-instance',
  data: {
    key: { remoteJid: '5511999999999@s.whatsapp.net', fromMe: false, id: 'test' },
    message: { conversation: 'Test message' },
    pushName: 'Test User'
  }
});

export default function () {
  const res = http.post(
    'http://localhost:3000/api/whatsapp/webhook/test-instance',
    payload,
    { headers: { 'Content-Type': 'application/json' } }
  );

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(0.01); // 100 req/s
}
```

### Cenários de Carga

| Cenário | Descrição | Alvo |
|---------|-----------|------|
| Normal | Uso diário típico | 50 req/s |
| Pico | Horário de maior uso | 100 req/s |
| Estresse | Teste de limite | 200 req/s |

---

## 8.6 Testes de Segurança

### OWASP Top 10

| Vulnerabilidade | Teste | Ferramenta |
|-----------------|-------|------------|
| Injection | SQL Injection, XSS | OWASP ZAP |
| Broken Auth | Token manipulation | Manual + Scripts |
| Data Exposure | Sensitive data leaks | OWASP ZAP |
| XXE | XML parsing | N/A (JSON only) |
| Broken Access | IDOR, privilege escalation | Manual |
| Rate Limiting | Brute force | k6 |

### Checklist de Segurança

- [x] SQL Injection (Prisma ORM previne)
- [x] XSS (React escapa por padrão)
- [x] CSRF (JWT stateless)
- [ ] IDOR - Testar acesso entre empresas
- [x] Rate Limiting
- [x] Token Expiration
- [x] Password Policy (mínimo 6 caracteres)
- [ ] Headers de Segurança (helmet.js)

### Teste de IDOR

```typescript
test('should not access other company data', async () => {
  const userA = await createTestUser({ companyId: 'company-a' });
  const userB = await createTestUser({ companyId: 'company-b' });

  // Cria contato como userA
  const contact = await createContact(userA.token, { name: 'Secret' });

  // Tenta acessar como userB
  const response = await app.inject({
    method: 'GET',
    url: `/api/contacts/${contact.id}`,
    headers: { Authorization: `Bearer ${userB.token}` }
  });

  expect(response.statusCode).toBe(404); // Não encontra (isolado por company)
});
```

---

## 8.7 CI/CD Integration

### Pipeline de Testes

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci
        working-directory: ./server

      - name: Run Migrations
        run: npx prisma migrate deploy
        working-directory: ./server
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test

      - name: Lint
        run: npm run lint
        working-directory: ./server

      - name: Type Check
        run: npm run type-check
        working-directory: ./server

      - name: Unit Tests
        run: npm run test:unit
        working-directory: ./server

      - name: Integration Tests
        run: npm run test:integration
        working-directory: ./server
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test
          REDIS_URL: redis://localhost:6379

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
```

---

## 8.8 Métricas de Qualidade

| Métrica | Meta | Bloqueante? |
|---------|------|-------------|
| Coverage (Unit) | ≥ 80% | Sim |
| Coverage (Integration) | ≥ 70% | Sim |
| Coverage (Total) | ≥ 75% | Sim |
| Tests Passing | 100% | Sim |
| Lint Errors | 0 | Sim |
| Type Errors | 0 | Sim |
| Security Vulnerabilities | 0 Critical | Sim |

---

← [Voltar para SPEC](README.md) | [Próximo: Deployment →](09-deployment.md)
