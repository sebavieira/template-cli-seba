# 8. Estratégia de Testes

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

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

- **Runner:** Jest / Vitest
- **Mocks:** jest.mock / vi.mock
- **Coverage:** Istanbul / c8

### Exemplo de Teste Unitário

```typescript
describe('{{Service}}Service', () => {
  let service: {{Service}}Service;
  let mockRepository: jest.Mocked<{{Repository}}Repository>;

  beforeEach(() => {
    mockRepository = {
      findById: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
    };
    service = new {{Service}}Service(mockRepository);
  });

  describe('create', () => {
    it('should create {{item}} with valid data', async () => {
      const data = { name: 'Test', config: {} };
      const expected = { id: 'uuid', ...data };

      mockRepository.create.mockResolvedValue(expected);

      const result = await service.create(data);

      expect(mockRepository.create).toHaveBeenCalledWith(data);
      expect(result).toEqual(expected);
    });

    it('should throw validation error for invalid data', async () => {
      const data = { name: '' }; // inválido

      await expect(service.create(data))
        .rejects.toThrow(ValidationError);
    });
  });
});
```

### O que Testar

- [x] Lógica de negócio
- [x] Validações
- [x] Edge cases
- [x] Error handling
- [ ] Integração com DB (mock)
- [ ] APIs externas (mock)

---

## 8.3 Testes de Integração

### Ferramentas

- **Runner:** Jest / Vitest
- **HTTP:** Supertest
- **DB:** Testcontainers / SQLite in-memory

### Exemplo de Teste de Integração

```typescript
describe('POST /api/v1/{{entidade}}', () => {
  let app: Express;
  let token: string;

  beforeAll(async () => {
    app = await createTestApp();
    token = await getTestToken();
  });

  afterEach(async () => {
    await cleanDatabase();
  });

  it('should create {{item}} with valid data', async () => {
    const response = await request(app)
      .post('/api/v1/{{entidade}}')
      .set('Authorization', `Bearer ${token}`)
      .send({
        name: 'Test {{Item}}',
        config: { option: true }
      });

    expect(response.status).toBe(201);
    expect(response.body).toMatchObject({
      id: expect.any(String),
      name: 'Test {{Item}}',
      status: 'draft'
    });
  });

  it('should return 400 for invalid data', async () => {
    const response = await request(app)
      .post('/api/v1/{{entidade}}')
      .set('Authorization', `Bearer ${token}`)
      .send({ name: '' });

    expect(response.status).toBe(400);
    expect(response.body.error).toBe('validation_error');
  });

  it('should return 401 without auth', async () => {
    const response = await request(app)
      .post('/api/v1/{{entidade}}')
      .send({ name: 'Test' });

    expect(response.status).toBe(401);
  });
});
```

---

## 8.4 Testes E2E

### Ferramentas

- **Browser:** Playwright
- **API:** Supertest
- **Fixtures:** Factory functions

### Exemplo de Teste E2E

```typescript
import { test, expect } from '@playwright/test';

test.describe('{{Entidade}} Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
  });

  test('should create and activate {{item}}', async ({ page }) => {
    // Navigate to create
    await page.click('text=Nova {{Entidade}}');

    // Fill form
    await page.fill('[name="name"]', 'Test {{Item}}');
    await page.click('button:text("Salvar")');

    // Verify created
    await expect(page.locator('text=Test {{Item}}')).toBeVisible();
    await expect(page.locator('text=Rascunho')).toBeVisible();

    // Activate
    await page.click('button:text("Ativar")');
    await expect(page.locator('text=Ativo')).toBeVisible();
  });
});
```

---

## 8.5 Testes de Carga

### Ferramentas

- **k6** (recomendado)
- Artillery
- Locust

### Exemplo com k6

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },  // Ramp up
    { duration: '1m', target: 20 },   // Stay
    { duration: '10s', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<200'], // 95% < 200ms
    http_req_failed: ['rate<0.01'],   // <1% errors
  },
};

export default function () {
  const res = http.get('http://localhost:3000/api/v1/{{entidade}}', {
    headers: { Authorization: `Bearer ${__ENV.TOKEN}` },
  });

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(1);
}
```

---

## 8.6 Testes de Segurança

### OWASP Top 10

| Vulnerabilidade | Teste | Ferramenta |
|-----------------|-------|------------|
| Injection | SQL Injection, XSS | OWASP ZAP |
| Broken Auth | Token manipulation | Manual + Scripts |
| Data Exposure | Sensitive data leaks | OWASP ZAP |
| XXE | XML parsing | Manual |
| Broken Access | IDOR, privilege escalation | Manual |

### Checklist de Segurança

- [ ] SQL Injection
- [ ] XSS (Reflected, Stored, DOM)
- [ ] CSRF
- [ ] IDOR
- [ ] Rate Limiting
- [ ] Token Expiration
- [ ] Password Policy
- [ ] Headers de Segurança

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
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Unit Tests
        run: npm run test:unit

      - name: Integration Tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/test

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

---

← [Voltar para SPEC](README.md) | [Próximo: Deployment →](09-deployment.md)
