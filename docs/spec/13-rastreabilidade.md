# 13. Matriz de Rastreabilidade

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 13.1 User Stories → Endpoints → Testes

### Epic 1: Autenticação e Multitenancy

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-001 | Cadastro de empresa | POST /auth/register | ✅ | ✅ | ✅ |
| US-002 | Login com email/senha | POST /auth/login | ✅ | ✅ | ✅ |
| US-003 | Logout e invalidar token | POST /auth/logout | ✅ | ✅ | - |
| US-004 | Refresh de access token | POST /auth/refresh | ✅ | ✅ | - |
| US-005 | Recuperar dados do usuário | GET /auth/me | ✅ | ✅ | - |

### Epic 2: WhatsApp Multi-Instance

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-010 | Criar instância WhatsApp | POST /whatsapp/instances | ✅ | ✅ | ✅ |
| US-011 | Conectar via QR Code | POST /whatsapp/instances/:id/connect | ✅ | ✅ | ✅ |
| US-012 | Desconectar instância | POST /whatsapp/instances/:id/disconnect | ✅ | ✅ | - |
| US-013 | Receber mensagens (webhook) | POST /whatsapp/webhook/:instance | ✅ | ✅ | ✅ |
| US-014 | Enviar mensagem texto | POST /whatsapp/send/text | ✅ | ✅ | ✅ |
| US-015 | Enviar mídia | POST /whatsapp/send/media | ✅ | ✅ | - |

### Epic 3: Chat em Tempo Real

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-020 | Listar conversas | GET /conversations | ✅ | ✅ | ✅ |
| US-021 | Ver histórico de mensagens | GET /conversations/:id/messages | ✅ | ✅ | ✅ |
| US-022 | Pausar IA na conversa | POST /conversations/:id/pause | ✅ | ✅ | ✅ |
| US-023 | Retomar IA na conversa | POST /conversations/:id/resume | ✅ | ✅ | - |
| US-024 | Arquivar conversa | POST /conversations/:id/archive | ✅ | ✅ | - |
| US-025 | Atualização em tempo real | WebSocket /company/:id | ✅ | ✅ | ✅ |

### Epic 4: IA e Atendimento Automático

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-030 | Configurar prompt da empresa | GET/PUT /companies/:id/prompt | ✅ | ✅ | ✅ |
| US-031 | Resposta automática por IA | Worker message-worker | ✅ | ✅ | ✅ |
| US-032 | Onboarding assistido por IA | POST /ai/onboarding/generate | ✅ | ✅ | ✅ |
| US-033 | Análise de sentimento | POST /ai/analyze | ✅ | ✅ | - |
| US-034 | Action prompts por situação | GET/POST /ai/prompts | ✅ | ✅ | - |
| US-035 | Chat com assistente IA | POST /ai/assistant/chat | ✅ | ✅ | - |

### Epic 5: CRM - Contatos

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-040 | Criar contato | POST /contacts | ✅ | ✅ | ✅ |
| US-041 | Listar contatos | GET /contacts | ✅ | ✅ | ✅ |
| US-042 | Editar contato | PATCH /contacts/:id | ✅ | ✅ | - |
| US-043 | Excluir contato | DELETE /contacts/:id | ✅ | ✅ | - |
| US-044 | Gerenciar tags | POST/DELETE /contacts/:id/tags | ✅ | ✅ | - |
| US-045 | Importar contatos | POST /contacts/import | ✅ | ✅ | - |

### Epic 6: CRM - Funil e Deals

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-050 | Criar funil | POST /funnels | ✅ | ✅ | ✅ |
| US-051 | Gerenciar estágios | POST/PATCH/DELETE /stages | ✅ | ✅ | - |
| US-052 | Criar deal | POST /deals | ✅ | ✅ | ✅ |
| US-053 | Mover deal entre estágios | POST /deals/:id/move | ✅ | ✅ | ✅ |
| US-054 | Marcar deal ganho/perdido | POST /deals/:id/won, /lost | ✅ | ✅ | - |
| US-055 | Adicionar notas ao deal | POST /deals/:id/notes | ✅ | ✅ | - |
| US-056 | Kanban view | GET /funnels/:id/deals | ✅ | ✅ | ✅ |

### Epic 7: Follow-ups Automáticos

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-060 | Criar regra de follow-up | POST /followups/rules | ✅ | ✅ | ✅ |
| US-061 | Listar follow-ups pendentes | GET /followups/queue | ✅ | ✅ | - |
| US-062 | Executar follow-up | Worker followup-worker | ✅ | ✅ | ✅ |
| US-063 | Cancelar follow-up | POST /followups/queue/:id/cancel | ✅ | ✅ | - |
| US-064 | Histórico de follow-ups | GET /followups/history | ✅ | ✅ | - |

### Epic 8: Automações Visuais (Flows)

| US | Descrição | Endpoints | Unit | Int | E2E |
|----|-----------|-----------|------|-----|-----|
| US-070 | Criar flow | POST /flows | ✅ | ✅ | ✅ |
| US-071 | Editor visual de nós | Frontend only | - | - | ✅ |
| US-072 | Ativar/desativar flow | POST /flows/:id/toggle | ✅ | ✅ | - |
| US-073 | Executar flow | Worker flow-worker | ✅ | ✅ | ✅ |
| US-074 | Duplicar flow | POST /flows/:id/duplicate | ✅ | ✅ | - |

---

## 13.2 Requisitos Não Funcionais → Implementação

| RNF | Requisito | Implementação | Status | Validação |
|-----|-----------|---------------|--------|-----------|
| RNF-01 | Latência API < 200ms (p95) | Caching Redis, índices, query optimization | 📋 | Load test k6 |
| RNF-02 | Throughput 500 req/s | Fastify, connection pooling, workers | 📋 | Load test k6 |
| RNF-03 | Disponibilidade 99.9% | Multi-instance, health checks, Railway | 📋 | Monitoring |
| RNF-04 | Segurança OWASP Top 10 | JWT, bcrypt, Zod, helmet, rate limiting | ✅ | Pentest |
| RNF-05 | Multi-tenancy isolation | companyId em todas queries, middleware | ✅ | Teste IDOR |
| RNF-06 | Real-time < 500ms | Socket.io, Redis pub/sub | 📋 | E2E test |
| RNF-07 | Webhook response < 200ms | Async processing com BullMQ | ✅ | Load test |
| RNF-08 | Escalabilidade horizontal | Stateless API, Redis sessions | 📋 | Staging |
| RNF-09 | Backup diário | PostgreSQL PITR, S3 | 📋 | DR test |
| RNF-10 | LGPD Compliance | Soft delete, export data, consent | 📋 | Checklist |

---

## 13.3 Cobertura de Testes

### Por Módulo

| Módulo | Unitários | Integração | E2E | Coverage |
|--------|-----------|------------|-----|----------|
| Auth | 15 | 8 | 3 | 85% |
| WhatsApp | 25 | 12 | 5 | 82% |
| Conversations | 18 | 10 | 4 | 80% |
| AI | 20 | 8 | 2 | 78% |
| Contacts | 15 | 8 | 3 | 82% |
| Funnels | 12 | 6 | 2 | 80% |
| Deals | 18 | 10 | 4 | 81% |
| Follow-ups | 15 | 7 | 2 | 79% |
| Flows | 12 | 6 | 2 | 77% |
| Workers | 20 | 10 | 3 | 75% |
| **Total** | **170** | **85** | **30** | **80%** |

### Por Tipo

| Tipo | Quantidade | Percentual | Meta | Status |
|------|------------|------------|------|--------|
| Unitários | 170 | 60% | 60% | ✅ |
| Integração | 85 | 30% | 30% | ✅ |
| E2E | 30 | 10% | 10% | ✅ |

### Cobertura Alvo

| Métrica | Meta | Bloqueante? |
|---------|------|-------------|
| Coverage Unit | ≥ 80% | Sim |
| Coverage Integration | ≥ 70% | Sim |
| Coverage Total | ≥ 75% | Sim |
| Tests Passing | 100% | Sim |

---

## 13.4 Fluxos Críticos

### Fluxo 1: Autenticação Completa

| Etapa | Endpoint | Componentes | Testes |
|-------|----------|-------------|--------|
| 1. Register | POST /auth/register | AuthService, UserRepo, CompanyRepo | ✅ |
| 2. Login | POST /auth/login | AuthService, JWT, SessionRepo | ✅ |
| 3. Access API | GET /api/* (Bearer) | AuthMiddleware, JWTVerify | ✅ |
| 4. Refresh | POST /auth/refresh | AuthService, SessionRepo | ✅ |
| 5. Logout | POST /auth/logout | AuthService, SessionRepo | ✅ |

### Fluxo 2: Mensagem WhatsApp (Recebida)

| Etapa | Endpoint | Componentes | Testes |
|-------|----------|-------------|--------|
| 1. Webhook | POST /whatsapp/webhook/:instance | WebhookHandler, InstanceRepo | ✅ |
| 2. Criar/Buscar Conversa | - | ConversationService | ✅ |
| 3. Salvar Mensagem | - | MessageService, Prisma | ✅ |
| 4. Enfileirar Job | - | BullMQ, message-queue | ✅ |
| 5. Processar IA | Worker | AIService, OpenAI | ✅ |
| 6. Enviar Resposta | - | EvolutionAPI, MessageService | ✅ |
| 7. Notificar Real-time | - | Socket.io | ✅ |

### Fluxo 3: Deal no Funil

| Etapa | Endpoint | Componentes | Testes |
|-------|----------|-------------|--------|
| 1. Criar Deal | POST /deals | DealService, DealRepo | ✅ |
| 2. Listar Kanban | GET /funnels/:id/deals | FunnelService, StageRepo | ✅ |
| 3. Mover Deal | POST /deals/:id/move | DealService, ActivityRepo | ✅ |
| 4. Trigger Flow | - | FlowTrigger, flow-queue | ✅ |
| 5. Marcar Ganho | POST /deals/:id/won | DealService, ActivityRepo | ✅ |

### Fluxo 4: Follow-up Automático

| Etapa | Endpoint | Componentes | Testes |
|-------|----------|-------------|--------|
| 1. Criar Regra | POST /followups/rules | FollowupService | ✅ |
| 2. Agendar | POST /followups/queue/schedule | FollowupService, BullMQ | ✅ |
| 3. Verificar Resposta | Worker | FollowupWorker, ConversationRepo | ✅ |
| 4. Enviar Mensagem | - | WhatsAppService, EvolutionAPI | ✅ |
| 5. Registrar Histórico | - | FollowupHistoryRepo | ✅ |

---

## 13.5 Dependências entre Módulos

```
┌──────────────┐
│     Auth     │ ← Todos os módulos dependem
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│   WhatsApp   │◄───►│ Conversations│
└──────┬───────┘     └──────┬───────┘
       │                    │
       │    ┌───────────────┘
       │    │
       ▼    ▼
┌──────────────┐     ┌──────────────┐
│      AI      │◄───►│   Contacts   │
└──────┬───────┘     └──────┬───────┘
       │                    │
       │    ┌───────────────┘
       │    │
       ▼    ▼
┌──────────────┐     ┌──────────────┐
│    Deals     │◄───►│   Funnels    │
└──────┬───────┘     └──────────────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│  Follow-ups  │◄───►│    Flows     │
└──────────────┘     └──────────────┘
       │                    │
       │    ┌───────────────┘
       │    │
       ▼    ▼
┌──────────────────────────────────┐
│            Workers               │
│  (message, followup, embedding,  │
│           flow)                  │
└──────────────────────────────────┘
```

---

## 13.6 Checklist de Validação

### Antes do Deploy

- [ ] Todos os testes passando (100%)
- [ ] Coverage ≥ 80%
- [ ] Lint sem erros (`npm run lint`)
- [ ] Types sem erros (`npm run type-check`)
- [ ] Security scan OK (`npm audit`)
- [ ] Performance benchmarks OK (k6)
- [ ] Migrations aplicadas (`npx prisma migrate deploy`)
- [ ] Variáveis de ambiente configuradas
- [ ] Build sem erros (`npm run build`)

### Pós Deploy

- [ ] Health checks passando (`/health`, `/health/ready`)
- [ ] Métricas dentro do esperado (Grafana)
- [ ] Logs sem erros críticos (Loki)
- [ ] Alertas configurados (Alertmanager)
- [ ] WebSocket funcionando
- [ ] Webhook recebendo mensagens
- [ ] Workers processando filas

---

## 13.7 Mapa de Arquivos

### Backend

| Funcionalidade | Arquivos |
|----------------|----------|
| Auth | `server/src/modules/auth/` |
| WhatsApp | `server/src/modules/whatsapp/` |
| Conversations | `server/src/modules/conversations/` |
| AI | `server/src/modules/ai/` |
| Contacts | `server/src/modules/contacts/` |
| Funnels | `server/src/modules/funnels/` |
| Deals | `server/src/modules/deals/` |
| Follow-ups | `server/src/modules/followups/` |
| Flows | `server/src/modules/flows/` |
| Workers | `server/src/workers/` |
| Middleware | `server/src/middleware/` |
| Config | `server/src/config/`, `server/src/lib/` |

### Frontend

| Funcionalidade | Arquivos |
|----------------|----------|
| Auth | `client/src/pages/auth/`, `client/src/features/auth/` |
| Chat | `client/src/pages/conversations/`, `client/src/features/chat/` |
| CRM | `client/src/pages/crm/`, `client/src/features/crm/` |
| Settings | `client/src/pages/settings/` |
| Components | `client/src/components/` |
| Hooks | `client/src/hooks/` |
| Store | `client/src/store/` |
| API | `client/src/lib/api/` |

---

## 13.8 APIs Externas

| Serviço | Endpoints Usados | Fallback | Monitoramento |
|---------|------------------|----------|---------------|
| Evolution API | `/instance/*`, `/message/*` | Retry + DLQ | ✅ Circuit Breaker |
| UAZAPI | `/send/*`, `/status/*` | Evolution API | ✅ Circuit Breaker |
| OpenAI | `/chat/completions`, `/embeddings` | Resposta padrão | ✅ Rate limit |

### Limites e Quotas

| Serviço | Limite | Período | Ação se Exceder |
|---------|--------|---------|-----------------|
| OpenAI | 90 RPM | Por minuto | Queue backpressure |
| OpenAI | 200K TPM | Por minuto | Truncar contexto |
| Evolution API | 1000 msgs | Por minuto | Queue delay |

---

## 13.9 Atualização da Matriz

Esta matriz deve ser atualizada quando:

- Nova User Story é implementada
- Novo endpoint é criado
- Novos testes são adicionados
- RNF é implementado
- Integração externa é adicionada
- Módulo é criado ou modificado

**Responsável:** Tech Lead
**Frequência:** A cada sprint
**Revisão:** Antes de cada release

---

← [Voltar para SPEC](README.md) | **Fim da Especificação Técnica**
