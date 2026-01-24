# Plano de Refatoração - Evo AI Connect

**Data de Análise:** 2026-01-22
**Versão:** 1.0.0
**Método:** Análise Multi-Agente com 4 agentes especializados

---

## Índice

1. [Resumo Executivo](#resumo-executivo)
2. [Arquivos Críticos](#arquivos-críticos)
3. [Análise Backend](#análise-backend)
4. [Análise Frontend](#análise-frontend)
5. [Code Smells Detectados](#code-smells-detectados)
6. [Recomendações de Divisão](#recomendações-de-divisão)
7. [Plano de Ação](#plano-de-ação)
8. [Métricas e KPIs](#métricas-e-kpis)
9. [Checklist de Execução](#checklist-de-execução)

---

## Resumo Executivo

### Visão Geral do Projeto

| Métrica | Backend | Frontend | Total |
|---------|---------|----------|-------|
| Linhas de código | 31.455 | 64.959 | **96.414** |
| Arquivos críticos (>800 linhas) | 10 | 5 | **15** |
| Arquivos em atenção (400-800) | 7 | 13 | **20** |
| Routes sem validação Zod | 127/147 | N/A | **86%** |

### Principais Problemas Identificados

1. **God Objects**: Services com 2.000+ linhas e múltiplas responsabilidades
2. **God Methods**: Métodos únicos com 2.100+ linhas
3. **Validação ausente**: 86% das routes sem schema Zod
4. **Duplicação de código**: Padrões repetidos em 200+ locais
5. **Acoplamento alto**: Services com 16+ importações

### Impacto Estimado

- **Tempo de refatoração**: 5-6 semanas (2-3 devs)
- **Redução de código**: 85% nos arquivos críticos
- **Melhoria em testabilidade**: 90%
- **Cobertura de validação**: de 18% para 100%

---

## Arquivos Críticos

### TOP 10 - Prioridade Máxima

| # | Arquivo | Linhas | Tipo | Severidade |
|---|---------|--------|------|-----------|
| 1 | `server/src/modules/ai/assistant.service.ts` | **2.612** | Service | 🔴 CRÍTICO |
| 2 | `server/src/modules/whatsapp/whatsapp.service.ts` | **2.482** | Service | 🔴 CRÍTICO |
| 3 | `src/lib/api.ts` | **2.037** | API Client | 🔴 CRÍTICO |
| 4 | `src/pages/Messages.tsx` | **1.699** | Page | 🔴 CRÍTICO |
| 5 | `src/components/editor/use-chat.ts` | **1.649** | Hook | 🔴 CRÍTICO |
| 6 | `src/pages/AdminCompanies.tsx` | **1.318** | Page | 🔴 CRÍTICO |
| 7 | `src/components/funnels/DealDetailsDrawer.tsx` | **1.147** | Component | 🟠 ALTO |
| 8 | `server/src/modules/followups/followups.service.ts` | **1.036** | Service | 🟠 ALTO |
| 9 | `server/src/modules/contacts/contacts.service.ts` | **995** | Service | 🟠 ALTO |
| 10 | `server/src/modules/deals/deals.service.ts` | **982** | Service | 🟠 ALTO |

### Classificação por Severidade

```
🔴 CRÍTICO (>800 linhas): Refatorar imediatamente
🟠 ALTO (400-800 linhas): Planejar refatoração
🟡 MÉDIO (200-400 linhas): Monitorar
✅ IDEAL (<200 linhas): Manter
```

---

## Análise Backend

### Arquivos Críticos (>800 linhas)

| Arquivo | Linhas | Métodos | Problema Principal |
|---------|--------|---------|-------------------|
| `assistant.service.ts` | 2.612 | 5 | God method `executeTool()` com 2.100+ linhas |
| `whatsapp.service.ts` | 2.482 | 28 | 7 responsabilidades distintas |
| `followups.service.ts` | 1.036 | ? | Múltiplas responsabilidades |
| `contacts.service.ts` | 995 | ? | Lógica de CRUD + enriquecimento |
| `deals.service.ts` | 982 | 53 | CRUD + notas + anexos |
| `chatbot-tool-executors.ts` | 970 | ? | Monolítico |
| `evolution.adapter.ts` | 901 | 12 | Parsing complexo |
| `uazapi.adapter.ts` | 899 | 14 | Duplicação com Evolution |
| `whatsapp.routes.ts` | 802 | 23 | Sem validação Zod |

### Arquivos em Atenção (400-800 linhas)

| Arquivo | Linhas | Notas |
|---------|--------|-------|
| `flow-engine.service.ts` | 674 | Motor de fluxo pode ser dividido |
| `dashboard.service.ts` | 624 | Múltiplos tipos de dashboards |
| `message.worker.ts` | 617 | Worker com lógica complexa |
| `followup.worker.ts` | 613 | Worker de followups |
| `admin.service.ts` | 584 | Operações administrativas variadas |
| `memory.service.ts` | 481 | Gerenciamento de memória |
| `chatbot-tools.ts` | 409 | Definições de ferramentas |

### Análise Detalhada: WhatsApp Service

**Arquivo**: `server/src/modules/whatsapp/whatsapp.service.ts`
**Linhas**: 2.482
**Métodos**: 28 async + 4 private = 32 total

**Métodos identificados:**
1. `processWebhook()` - Webhook processing
2. `markAsRead()` - Mark messages as read
3. `resolveContactForMessage()` - Contact resolution
4. `buildRemoteJid()` - Build JID format
5. `sendMessage()` - Send text messages
6. `sendMediaMessage()` - Send media
7. `sendReaction()` - Send reactions
8. `getCredentials()` - Get provider credentials
9. `getConversations()` - List conversations (12 filtros)
10. `getMessages()` - Get messages com paginação
11. `togglePause()` - Pause/unpause conversation
12. `archiveConversation()` - Archive conversation
13. `deleteConversation()` - Delete conversation
14. `listInstances()` - List WhatsApp instances
15. `checkInstanceConnection()` - Check single instance
16. `checkAllInstanceConnections()` - Check all instances
17. `getProviderCredentials()` - Get company credentials
18. `getUserProfile()` - Get user profile
19. `createInstance()` - Create new instance
20. `updateInstanceDisplayName()` - Update display name
21. `connectInstance()` - Connect instance
22. `deleteInstance()` - Delete instance
23. `createProviderCredential()` - Add provider cred
24. `deleteProviderCredential()` - Remove provider cred
25. `toggleProviderCredential()` - Toggle provider cred

**Responsabilidades identificadas:**

| Categoria | Responsabilidades |
|-----------|-------------------|
| Webhook Processing | Detectar provider, parsear payload, atualizar status |
| Contact Management | Resolver contatos, vincular LID, criar/atualizar |
| Message Operations | Enviar texto, media, reações, marcar como lido |
| Conversation Management | Listar, toggle pause, archive, delete |
| Instance Lifecycle | Create, connect, check status, delete, rename |
| Credential Management | Get/create/delete/toggle provider credentials |

### Análise Detalhada: Assistant Service

**Arquivo**: `server/src/modules/ai/assistant.service.ts`
**Linhas**: 2.612
**God Method**: `executeTool()` com 2.100+ linhas

**Problemas:**
- 82+ validações manuais `if (!x) throw`
- 40+ cases de ferramentas em um único switch
- 16 importações de diferentes módulos
- Múltiplos domínios: Funnels, Followups, Contacts, Deals, Admin, Auth

---

## Análise Frontend

### Arquivos Críticos (>300 linhas)

| Arquivo | Linhas | Problema Principal |
|---------|--------|-------------------|
| `src/lib/api.ts` | 2.037 | 74 exports + 57 tipos monolítico |
| `src/pages/Messages.tsx` | 1.699 | **81 hooks** em um arquivo |
| `src/components/editor/use-chat.ts` | 1.649 | Hook gigante (streaming + comentários) |
| `src/pages/AdminCompanies.tsx` | 1.318 | Múltiplas entidades |
| `src/components/funnels/DealDetailsDrawer.tsx` | 1.147 | Drawer com múltiplas abas |
| `src/components/followups/FollowupRuleWizard.tsx` | 1.078 | Wizard com lógica complexa |
| `src/components/ui/table-icons.tsx` | 862 | Todos ícones em um arquivo |
| `src/pages/Prompt.tsx` | 843 | Editor + settings + preview |
| `src/pages/ContactDetail.tsx` | 843 | Múltiplas abas |
| `src/components/ui/font-color-toolbar-button.tsx` | 830 | Botão com lógica complexa |
| `src/components/ui/ai-menu.tsx` | 723 | Menu AI com muitas opções |
| `src/components/messages/ChatInput.tsx` | 698 | Input com 14 hooks |
| `src/data/followupTemplates.ts` | 661 | Templates estáticos |
| `src/components/ui/table-node.tsx` | 658 | Nó de tabela complexo |
| `src/pages/Funnels.tsx` | 639 | Kanban + Analytics misturados |
| `src/components/ui/sidebar.tsx` | 637 | Sidebar com muita lógica |
| `src/components/ui/emoji-toolbar-button.tsx` | 627 | Picker embutido |

### Problema Específico: Messages.tsx

**81 hooks em um único componente!**

Hooks que precisam ser consolidados:
- useState (múltiplos)
- useEffect (múltiplos)
- useQuery/useMutation
- useWebSocket
- useCallback
- useMemo
- Custom hooks diversos

---

## Code Smells Detectados

### 1. Duplicação de Código

| Padrão | Ocorrências | Impacto |
|--------|-------------|---------|
| `request.user.profile.companyId` manual | 200+ | Alto |
| `if (!value) throw` validações | 200+ | Alto |
| try/catch em routes | 23 arquivos | Alto |
| `path.extname().toLowerCase()` | 4+ arquivos | Médio |

**Exemplo de duplicação:**
```typescript
// Repetido em 23 route files
try {
    const input = schema.parse(request.body);
    const result = await service.method(...);
    return reply.status(...).send({ success: true, data: result });
} catch (error) {
    return reply.status(...).send({
        success: false,
        error: error instanceof Error ? error.message : 'Erro...'
    });
}
```

**Solução**: Criar middleware de error handling reutilizável

### 2. Validação Ausente

| Route File | Handlers | Schemas | Gap |
|------------|----------|---------|-----|
| `whatsapp.routes.ts` | 23 | 0 | **100%** |
| `followups.routes.ts` | 15 | 0 | **100%** |
| `contacts.routes.ts` | 19 | 0 | **100%** |
| `deals.routes.ts` | 13 | 0 | **100%** |
| `flows.routes.ts` | 7 | 0 | **100%** |
| `funnels.routes.ts` | 16 | 0 | **100%** |
| `quick-replies.routes.ts` | 4 | 0 | **100%** |
| `dashboard.routes.ts` | 3 | 0 | **100%** |
| `prompts.routes.ts` | 5 | 0 | **100%** |

**Total: 127 handlers sem validação Zod (86%)**

### 3. God Objects

| Service | Linhas | Responsabilidades |
|---------|--------|-------------------|
| `assistant.service.ts` | 2.612 | Chat + Tools + Integrations |
| `whatsapp.service.ts` | 2.482 | Webhook + Messages + Instances + Credentials |
| `deals.service.ts` | 982 | CRUD + Stages + Notes + Attachments |

### 4. God Methods

| Método | Arquivo | Linhas |
|--------|---------|--------|
| `executeTool()` | assistant.service.ts | **2.100+** |
| `processNode()` | flow-engine.service.ts | 280 |
| `processMessage()` | message.worker.ts | 300+ |
| `parseWebhookPayload()` | evolution.adapter.ts | 200+ |

### 5. Error Handling Deficiente

- 134 ocorrências de `console.error`, `console.log` espalhadas
- Padrão `catch (error) { void error; }` em múltiplos lugares
- Sem logger centralizado/estruturado
- Erros silenciados sem logging ou retry

### 6. Type Safety Gaps

- Route handlers com `request.user!.companyId!` (força não-null)
- Sem validação que `request.user` existe
- Duplicação entre schemas Zod e tipagem manual

---

## Recomendações de Divisão

### 1. WhatsAppService (2.482 → 6 serviços)

```
server/src/modules/whatsapp/
├── whatsapp.service.ts (coordinator) → ~200 linhas
├── services/
│   ├── whatsapp-webhook.service.ts    → ~450 linhas
│   ├── whatsapp-messages.service.ts   → ~600 linhas
│   ├── whatsapp-conversations.service.ts → ~500 linhas
│   ├── whatsapp-instances.service.ts  → ~400 linhas
│   ├── whatsapp-credentials.service.ts → ~250 linhas
│   └── whatsapp-profile.service.ts    → ~100 linhas
└── index.ts (re-exports)
```

### 2. AssistantService (2.612 → 6 serviços)

```
server/src/modules/ai/
├── assistant.service.ts (core)        → ~500 linhas
├── services/
│   ├── assistant-tools.executor.ts    → ~800 linhas
│   ├── assistant-tools.definitions.ts → ~400 linhas
│   ├── assistant-memory.service.ts    → ~300 linhas
│   └── assistant-integrations.service.ts → ~400 linhas
└── assistant.types.ts                 → ~200 linhas
```

### 3. FlowEngineService (674 → 5 serviços)

```
server/src/modules/flows/
├── flow-engine.service.ts (coordinator) → ~150 linhas
├── services/
│   ├── flow-trigger.service.ts        → ~150 linhas
│   ├── flow-execution-state.service.ts → ~200 linhas
│   ├── node-processor.service.ts      → ~200 linhas
│   └── node-navigator.service.ts      → ~100 linhas
└── flow-metrics.service.ts            → ~50 linhas
```

### 4. Frontend api.ts (2.037 → módulos)

```
src/lib/api/
├── types.ts (tipos compartilhados)
├── client.ts (axios instance)
├── auth.ts
├── contacts.ts
├── messages.ts
├── conversations.ts
├── funnels.ts
├── followups.ts
├── flows.ts
├── deals.ts
├── admin.ts
└── index.ts (re-exports)
```

### 5. Messages.tsx (1.699 → hooks customizados)

```
src/pages/Messages/
├── index.tsx (container ~150 linhas)
├── hooks/
│   ├── useConversationList.ts
│   ├── useMessageThread.ts
│   ├── useMessageActions.ts
│   ├── useConversationFilters.ts
│   └── useWebSocketMessages.ts
└── components/
    ├── ConversationList.tsx
    ├── MessageThread.tsx
    ├── MessageInput.tsx
    └── ConversationHeader.tsx
```

### 6. use-chat.ts (1.649 → hooks especializados)

```
src/components/editor/hooks/
├── useChatBase.ts (~300 linhas)
├── useChatStreaming.ts (~400 linhas)
├── useChatComments.ts (~300 linhas)
├── useChatTableEdits.ts (~300 linhas)
├── chatChunks.ts (dados)
└── chatUtils.ts (helpers)
```

---

## Plano de Ação

### FASE 1 - CRÍTICO (Semanas 1-2)

#### Semana 1

| Dia | Tarefa | Responsável | Entregável |
|-----|--------|-------------|------------|
| 1-2 | Criar estrutura de pastas para WhatsApp | Dev 1 | Pastas criadas |
| 1-2 | Criar schemas Zod para routes críticas | Dev 2 | 50 schemas |
| 3-4 | Extrair WhatsApp webhook service | Dev 1 | `whatsapp-webhook.service.ts` |
| 3-4 | Adicionar validação em WhatsApp routes | Dev 2 | 23 routes validadas |
| 5 | Extrair WhatsApp messages service | Dev 1 | `whatsapp-messages.service.ts` |
| 5 | Code review + testes | Ambos | PR aprovado |

#### Semana 2

| Dia | Tarefa | Responsável | Entregável |
|-----|--------|-------------|------------|
| 1-2 | Extrair WhatsApp conversations service | Dev 1 | `whatsapp-conversations.service.ts` |
| 1-2 | Criar schemas para contacts/deals routes | Dev 2 | 32 schemas |
| 3-4 | Extrair WhatsApp instances/credentials | Dev 1 | 2 services |
| 3-4 | Validação em contacts/deals routes | Dev 2 | 32 routes |
| 5 | Integração + testes E2E | Ambos | WhatsApp refatorado |

### FASE 2 - ALTO (Semanas 3-4)

#### Semana 3

| Dia | Tarefa | Responsável | Entregável |
|-----|--------|-------------|------------|
| 1-2 | Dividir assistant.service.ts (tools executor) | Dev 1 | `assistant-tools.executor.ts` |
| 1-2 | Dividir api.ts em módulos | Dev 2 | `src/lib/api/` estruturado |
| 3-4 | Extrair assistant integrations | Dev 1 | `assistant-integrations.service.ts` |
| 3-4 | Criar hooks para Messages.tsx | Dev 2 | 5 hooks customizados |
| 5 | Code review + testes | Ambos | PR aprovado |

#### Semana 4

| Dia | Tarefa | Responsável | Entregável |
|-----|--------|-------------|------------|
| 1-2 | Finalizar assistant service refactoring | Dev 1 | Service dividido |
| 1-2 | Refatorar Messages.tsx | Dev 2 | Componente limpo |
| 3-4 | Refatorar followups.service.ts | Dev 1 | 3 services |
| 3-4 | Dividir use-chat.ts | Dev 2 | 5 hooks |
| 5 | Integração + testes E2E | Ambos | Fase 2 completa |

### FASE 3 - MÉDIO (Semanas 5-6)

#### Semana 5

| Dia | Tarefa | Responsável | Entregável |
|-----|--------|-------------|------------|
| 1-2 | Refatorar contacts.service.ts | Dev 1 | 3 services |
| 1-2 | Refatorar AdminCompanies.tsx | Dev 2 | Sub-páginas |
| 3-4 | Refatorar deals.service.ts | Dev 1 | 3 services |
| 3-4 | Refatorar ContactDetail.tsx | Dev 2 | Componentes por aba |
| 5 | Code review + testes | Ambos | PR aprovado |

#### Semana 6

| Dia | Tarefa | Responsável | Entregável |
|-----|--------|-------------|------------|
| 1-2 | Unificar adapters Evolution/UAZAPI | Dev 1 | Adapter base |
| 1-2 | Implementar error handling centralizado | Dev 2 | Middleware |
| 3-4 | Extrair webhook parsers | Dev 1 | Parser services |
| 3-4 | Adicionar logging estruturado | Dev 2 | Logger service |
| 5 | Testes finais + documentação | Ambos | Refatoração completa |

---

## Métricas e KPIs

### Métricas de Sucesso

| Métrica | Antes | Meta | Verificação |
|---------|-------|------|-------------|
| Maior arquivo backend | 2.612 | <400 | `wc -l` |
| Maior arquivo frontend | 2.037 | <300 | `wc -l` |
| Routes sem validação | 86% | 0% | grep schemas |
| Hooks por página (max) | 81 | <10 | Contagem manual |
| Arquivos >800 linhas | 15 | 0 | Script de análise |
| Cobertura de testes | ? | >80% | Coverage report |

### KPIs de Qualidade

| KPI | Definição | Meta |
|-----|-----------|------|
| Complexidade ciclomática | Máximo por função | <10 |
| Linhas por função | Máximo | <30 |
| Imports por arquivo | Máximo | <15 |
| Responsabilidades por service | Contagem | 1 (SRP) |

### Métricas de Performance

| Métrica | Antes | Esperado | Melhoria |
|---------|-------|----------|----------|
| Bundle size | ~800KB | ~680KB | 15%↓ |
| Time to Interactive | ~2.5s | ~1.8s | 28%↓ |
| Build time | ? | -20% | Otimização |

---

## Checklist de Execução

### Pré-Requisitos

- [ ] Backup do código atual
- [ ] Branch de feature criada: `refactor/code-quality-v1`
- [ ] CI/CD configurado para rodar testes
- [ ] Ambiente de staging disponível
- [ ] Documentação de APIs existentes

### FASE 1 Checklist

#### WhatsApp Service Refactoring
- [ ] Criar pasta `server/src/modules/whatsapp/services/`
- [ ] Extrair `whatsapp-webhook.service.ts`
- [ ] Extrair `whatsapp-messages.service.ts`
- [ ] Extrair `whatsapp-conversations.service.ts`
- [ ] Extrair `whatsapp-instances.service.ts`
- [ ] Extrair `whatsapp-credentials.service.ts`
- [ ] Atualizar imports em arquivos dependentes
- [ ] Testes unitários para cada service
- [ ] Testes de integração

#### Validação Zod
- [ ] Criar schemas para `whatsapp.routes.ts` (23)
- [ ] Criar schemas para `contacts.routes.ts` (19)
- [ ] Criar schemas para `deals.routes.ts` (13)
- [ ] Criar schemas para `followups.routes.ts` (15)
- [ ] Criar schemas para `flows.routes.ts` (7)
- [ ] Criar schemas para `funnels.routes.ts` (16)
- [ ] Criar schemas para demais routes (34)
- [ ] Middleware de validação reutilizável

### FASE 2 Checklist

#### Assistant Service Refactoring
- [ ] Criar pasta `server/src/modules/ai/services/`
- [ ] Extrair `assistant-tools.executor.ts`
- [ ] Extrair `assistant-tools.definitions.ts`
- [ ] Extrair `assistant-memory.service.ts`
- [ ] Extrair `assistant-integrations.service.ts`
- [ ] Criar Strategy Pattern para tools
- [ ] Atualizar imports
- [ ] Testes

#### Frontend Refactoring
- [ ] Criar estrutura `src/lib/api/`
- [ ] Dividir api.ts em módulos
- [ ] Criar hooks para Messages.tsx
- [ ] Refatorar Messages.tsx
- [ ] Dividir use-chat.ts
- [ ] Testes E2E

### FASE 3 Checklist

#### Services Adicionais
- [ ] Refatorar `followups.service.ts`
- [ ] Refatorar `contacts.service.ts`
- [ ] Refatorar `deals.service.ts`
- [ ] Unificar adapters

#### Páginas Frontend
- [ ] Refatorar `AdminCompanies.tsx`
- [ ] Refatorar `ContactDetail.tsx`
- [ ] Refatorar `Funnels.tsx`

#### Infraestrutura
- [ ] Implementar error handling centralizado
- [ ] Adicionar logging estruturado
- [ ] Remover console.log (134 ocorrências)
- [ ] Documentação atualizada

### Validação Final

- [ ] Todos os testes passando
- [ ] Build sem erros
- [ ] Lint sem warnings
- [ ] Code review aprovado
- [ ] Staging testado
- [ ] Performance validada
- [ ] Documentação completa

---

## Referências

### Documentação do Projeto
- [Skill de Refatoração](./.claude/skills/refactoring.md)
- [README](./README.md)
- [Changelog](./CHANGELOG.md)

### Padrões e Princípios
- SOLID Principles
- DRY (Don't Repeat Yourself)
- KISS (Keep It Simple, Stupid)
- Single Responsibility Principle

### Limites Estabelecidos

| Tipo | Ideal | Máximo | Crítico |
|------|-------|--------|---------|
| Linhas por arquivo | <200 | 400 | >800 |
| Linhas por função | <20 | 30 | >80 |
| Complexidade ciclomática | 1-5 | 10 | >20 |
| Parâmetros por função | 2-3 | 4 | >5 |
| Imports por arquivo | 5-7 | 10 | >15 |

---

**Documento gerado automaticamente por análise multi-agente**
**Última atualização**: 2026-01-22
