# 13. Matriz de Rastreabilidade

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

← [Voltar para SPEC](README.md)

---

## 13.1 User Stories → Endpoints → Testes

| US | Descrição | Endpoints | Testes Unit | Testes Int | Testes E2E |
|----|-----------|-----------|-------------|------------|------------|
| US-001 | {{DESC}} | POST /{{e1}} | ✅ | ✅ | ✅ |
| US-002 | {{DESC}} | GET /{{e1}} | ✅ | ✅ | - |
| US-003 | {{DESC}} | PATCH /{{e1}}/:id | ✅ | ✅ | - |
| US-004 | {{DESC}} | DELETE /{{e1}}/:id | ✅ | ✅ | - |
| US-005 | {{DESC}} | POST /{{e1}}/:id/{{action}} | ✅ | ✅ | ✅ |

---

## 13.2 Requisitos Não Funcionais → Implementação

| RNF | Requisito | Implementação | Status | Validação |
|-----|-----------|---------------|--------|-----------|
| RNF-01 | Performance < 200ms | Caching, índices | 📋 | Teste de carga |
| RNF-02 | Escalabilidade | Arquitetura stateless | 📋 | Load test |
| RNF-03 | Disponibilidade 99.9% | Multi-AZ, health checks | 📋 | Monitoring |
| RNF-04 | Segurança | JWT, HTTPS, validação | 📋 | Pentest |
| RNF-05 | Compliance LGPD | Consentimento, exclusão | 📋 | Checklist |

---

## 13.3 Cobertura de Testes

### Por Módulo

| Módulo | Unitários | Integração | E2E | Coverage |
|--------|-----------|------------|-----|----------|
| Auth | 15 | 8 | 3 | 85% |
| {{Modulo1}} | 20 | 10 | 4 | 80% |
| {{Modulo2}} | 18 | 7 | 2 | 78% |
| Workers | 10 | 5 | 1 | 75% |
| **Total** | **63** | **30** | **10** | **80%** |

### Por Tipo

| Tipo | Quantidade | Meta | Status |
|------|------------|------|--------|
| Unitários | 63 | 60% dos testes | ✅ |
| Integração | 30 | 30% dos testes | ✅ |
| E2E | 10 | 10% dos testes | ✅ |

---

## 13.4 Fluxos Críticos

### Fluxo 1: Autenticação

| Etapa | Endpoint | Componentes | Testes |
|-------|----------|-------------|--------|
| 1. Register | POST /auth/register | AuthService, UserRepo | ✅ |
| 2. Login | POST /auth/login | AuthService, JWT | ✅ |
| 3. Refresh | POST /auth/refresh | AuthService, TokenRepo | ✅ |
| 4. Logout | POST /auth/logout | AuthService, TokenRepo | ✅ |

### Fluxo 2: {{Fluxo Principal}}

| Etapa | Endpoint | Componentes | Testes |
|-------|----------|-------------|--------|
| 1. Criar | POST /{{e}} | {{Service}}, {{Repo}} | ✅ |
| 2. Listar | GET /{{e}} | {{Service}}, {{Repo}}, Cache | ✅ |
| 3. Atualizar | PATCH /{{e}}/:id | {{Service}}, {{Repo}} | ✅ |
| 4. Processar | POST /{{e}}/:id/{{action}} | Worker, Queue | ✅ |

---

## 13.5 Dependências entre Módulos

```
┌──────────────┐
│     Auth     │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│  {{Modulo1}} │◄───►│  {{Modulo2}} │
└──────┬───────┘     └──────────────┘
       │
       ▼
┌──────────────┐
│   Workers    │
└──────────────┘
```

---

## 13.6 Checklist de Validação

### Antes do Deploy

- [ ] Todos os testes passando
- [ ] Coverage ≥ 80%
- [ ] Lint sem erros
- [ ] Types sem erros
- [ ] Security scan OK
- [ ] Performance benchmarks OK

### Pós Deploy

- [ ] Health checks passando
- [ ] Métricas dentro do esperado
- [ ] Logs sem erros críticos
- [ ] Alertas configurados

---

## 13.7 Mapa de Arquivos

| Funcionalidade | Arquivos Backend | Arquivos Frontend |
|----------------|------------------|-------------------|
| Auth | `auth/`, `middleware/auth.ts` | `pages/login.tsx` |
| {{Func1}} | `{{modulo}}/` | `pages/{{modulo}}/` |
| {{Func2}} | `{{modulo}}/` | `components/{{modulo}}/` |

---

## 13.8 APIs Externas

| Serviço | Endpoints Usados | Fallback | Monitoramento |
|---------|------------------|----------|---------------|
| {{API1}} | {{endpoints}} | Cache | ✅ |
| {{API2}} | {{endpoints}} | Retry | ✅ |

---

## 13.9 Atualização da Matriz

Esta matriz deve ser atualizada quando:

- Nova User Story é implementada
- Novo endpoint é criado
- Novos testes são adicionados
- RNF é implementado
- Integração externa é adicionada

**Responsável:** Tech Lead
**Frequência:** A cada sprint

---

← [Voltar para SPEC](README.md) | **Fim da Especificação Técnica**
