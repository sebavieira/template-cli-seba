# Guia de Refatoração - Evo AI Connect

**Navegação modular para refatoração do projeto**

---

## Como Usar Este Guia

Cada documento é **autocontido** e pode ser lido independentemente. Carregue apenas o documento relevante para a tarefa atual.

```
# Para refatorar WhatsApp:
Read: doc/refactoring/01-whatsapp-service.md

# Para refatorar Assistant:
Read: doc/refactoring/02-assistant-service.md
```

---

## Índice de Refatorações

### Backend - Crítico (Prioridade P0)

| # | Documento | Arquivo Original | Linhas | Status |
|---|-----------|------------------|--------|--------|
| 01 | [WhatsApp Service](./01-whatsapp-service.md) | `whatsapp.service.ts` | 2.482 | ✅ **100%** |
| 02 | [Assistant Service](./02-assistant-service.md) | `assistant.service.ts` | 2.612 | ✅ **100%** |
| 03 | [Validação Zod](./03-validation-schemas.md) | Múltiplos routes | 127 handlers | ✅ **100%** |

#### 📊 Detalhes: 03-Validação Zod (Completo ✅)
**Branch:** `refactor`
**Resultado:** Todas as 23 rotas agora têm validação Zod aplicada

**✅ Arquivos de Schema Criados:**
| Módulo | Arquivo | Handlers | Status |
|--------|---------|----------|--------|
| **Shared** | `shared/schemas/common.schemas.ts` | - | ✅ |
| **Shared** | `shared/schemas/validation.middleware.ts` | - | ✅ |
| **Shared** | `shared/schemas/index.ts` | - | ✅ |
| **WhatsApp** | `whatsapp/whatsapp.schemas.ts` | 23 | ✅ |
| **Contacts** | `contacts/contacts.schemas.ts` | 19 | ✅ |
| **Deals** | `deals/deals.schemas.ts` | 13 | ✅ |
| **Funnels** | `funnels/funnels.schemas.ts` | 16 | ✅ |
| **Followups** | `followups/followups.schemas.ts` | 15 | ✅ |
| **Flows** | `flows/flows.schemas.ts` | 7 | ✅ |
| **AI** | `ai/ai.schemas.ts` | 14 | ✅ |
| **Dashboard** | `dashboard/dashboard.schemas.ts` | 3 | ✅ |

**Rotas com validação inline (já existentes):**
- auth.routes.ts, admin.routes.ts, api-keys.routes.ts, account.routes.ts
- notifications.routes.ts, company-users.routes.ts, playground.routes.ts
- prompts.routes.ts, quick-replies.routes.ts, sentiment-rules.routes.ts, user-prompts.routes.ts

**Build:** ✅ Passou

#### 📊 Detalhes: 02-Assistant Service (Completo ✅)
**Branch:** `refactor`
**Resultado:** assistant.service.ts agora tem **242 linhas** (era 2.612, meta ~300) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `assistant.types.ts` | 113 | ✅ |
| **Interface** | `tools/tool.interface.ts` | 79 | ✅ |
| **Tools** | `tools/contact.tools.ts` | 157 | ✅ |
| **Tools** | `tools/tag.tools.ts` | 89 | ✅ |
| **Tools** | `tools/funnel.tools.ts` | 166 | ✅ |
| **Tools** | `tools/flow.tools.ts` | 640 | ✅ |
| **Tools** | `tools/followup.tools.ts` | 404 | ✅ |
| **Tools** | `tools/whatsapp.tools.ts` | 206 | ✅ |
| **Tools** | `tools/prompt.tools.ts` | 364 | ✅ |
| **Tools** | `tools/settings.tools.ts` | 79 | ✅ |
| **Registry** | `tools/index.ts` | 84 | ✅ |
| **Services** | `services/tool-executor.service.ts` | 65 | ✅ |
| **Services** | `services/index.ts` | 2 | ✅ |
| **Definitions** | `definitions/openai-tools.ts` | 887 | ✅ |
| **Definitions** | `definitions/system-prompt.ts` | 33 | ✅ |
| **Definitions** | `definitions/index.ts` | 5 | ✅ |
| **Helpers** | `helpers/flow-helpers.ts` | 313 | ✅ |
| **Helpers** | `helpers/index.ts` | 25 | ✅ |

**Total:** 3.711 linhas em 18 arquivos modulares
**Maior arquivo:** 887 linhas (openai-tools.ts - são apenas definições JSON)
**Build:** ✅ Passou

#### 📊 Detalhes: 01-WhatsApp Service (Completo ✅)
**Branch:** `refactor/whatsapp-service`
**Arquivos Criados:** 23 arquivos modulares

| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `whatsapp.types.ts` | 184 | ✅ |
| **Helpers** | `helpers/jid-builder.ts` | 57 | ✅ |
| **Helpers** | `helpers/contact-resolver.ts` | 129 | ✅ |
| **Helpers** | `helpers/index.ts` | 6 | ✅ |
| **Services** | `services/credential.service.ts` | 130 | ✅ |
| **Services** | `services/instance.service.ts` | 339 | ✅ |
| **Services** | `services/message.service.ts` | 521 | ✅ |
| **Services** | `services/conversation.service.ts` | 546 | ✅ |
| **Services** | `services/index.ts` | 10 | ✅ |
| **Webhook** | `services/webhook/webhook.service.ts` | 227 | ✅ |
| **Webhook** | `services/webhook/webhook-helpers.ts` | 102 | ✅ |
| **Webhook** | `services/webhook/index.ts` | 18 | ✅ |
| **Handlers** | `services/webhook/handlers/status.handler.ts` | 62 | ✅ |
| **Handlers** | `services/webhook/handlers/reaction.handler.ts` | 85 | ✅ |
| **Handlers** | `services/webhook/handlers/contact.handler.ts` | 187 | ✅ |
| **Handlers** | `services/webhook/handlers/conversation.handler.ts` | 120 | ✅ |
| **Handlers** | `services/webhook/handlers/index.ts` | 16 | ✅ |
| **Processors** | `services/webhook/processors/audio.processor.ts` | 88 | ✅ |
| **Processors** | `services/webhook/processors/media.processor.ts` | 157 | ✅ |
| **Processors** | `services/webhook/processors/index.ts` | 10 | ✅ |
| **Storage** | `services/webhook/storage/message.storage.ts` | 206 | ✅ |
| **Storage** | `services/webhook/storage/index.ts` | 7 | ✅ |
| **Facade** | `whatsapp.facade.ts` | 174 | ✅ |

**Total:** 3.381 linhas em 23 arquivos
**Maior arquivo:** 546 linhas (conversation.service.ts) - dentro do limite de 600
**Build:** ✅ Passou

### Frontend - Crítico (Prioridade P0)

| # | Documento | Arquivo Original | Linhas | Status |
|---|-----------|------------------|--------|--------|
| 04 | [API Client](./04-api-frontend.md) | `src/lib/api.ts` | 2.037 | ✅ **100%** |
| 05 | [Messages Page](./05-messages-page.md) | `Messages.tsx` | 1.699 | ✅ **100%** |
| 06 | [Use Chat Hook](./06-use-chat-hook.md) | `use-chat.ts` | 1.649 | ✅ **100%** |

#### 📊 Detalhes: 06-Use Chat Hook (Completo ✅)
**Branch:** `refactor`
**Resultado:** use-chat.ts agora tem **227 linhas** (era 1.649) ✅

**✅ Estrutura Criada:**
```
src/components/editor/
├── use-chat.ts                  # Hook principal (227 linhas)
├── types/
│   ├── index.ts                 # Re-exports (1 linha)
│   └── chat.types.ts            # Tipos (36 linhas)
└── __mocks__/
    ├── index.ts                 # Re-exports (4 linhas)
    ├── markdown-chunks.ts       # Chunks markdown (142 linhas)
    ├── mdx-chunks.ts            # Chunks MDX (231 linhas)
    ├── chunk-creators.ts        # Criadores de chunks (102 linhas)
    └── fake-stream.ts           # Stream falso para testes (155 linhas)
```

**Total:** 898 linhas em 8 arquivos modulares
**Maior arquivo:** 231 linhas (mdx-chunks.ts) - dados de mock
**Build:** ✅ Passou

#### 📊 Detalhes: 05-Messages Page (Completo ✅)
**Branch:** `refactor`
**Resultado:** Messages.tsx refatorado em 10 arquivos modulares com hooks customizados

**✅ Estrutura Criada:**
```
src/pages/Messages/
├── index.tsx                         # Re-export (2 linhas)
├── MessagesPage.tsx                  # Container principal (508 linhas)
├── types.ts                          # Tipos locais (61 linhas)
└── hooks/
    ├── index.ts                      # Re-exports (7 linhas)
    ├── useConversationFilters.ts     # Filtros de conversa (186 linhas)
    ├── useConversations.ts           # Lista de conversas (206 linhas)
    ├── useMessages.ts                # Mensagens da conversa (269 linhas)
    ├── useMessageActions.ts          # Envio de mensagens (388 linhas)
    ├── useWebSocketMessages.ts       # Eventos WebSocket (191 linhas)
    └── useConversationActions.ts     # Ações de conversa (195 linhas)
```

**Total:** 2.013 linhas em 10 arquivos modulares
**Maior arquivo:** 508 linhas (MessagesPage.tsx) - dentro do limite de 600
**Build:** ✅ Passou

#### 📊 Detalhes: 04-API Client (Completo ✅)
**Branch:** `refactor`
**Resultado:** api.ts agora é um re-export de 25 arquivos modulares

**✅ Estrutura Criada:**
```
src/lib/api/
├── index.ts              # Re-exports (69 linhas)
├── client.ts             # Fetch base + token (131 linhas)
├── auth.api.ts           # Auth API (42 linhas)
├── account.api.ts        # Account + Company API (64 linhas)
├── contacts.api.ts       # Contacts API (134 linhas)
├── funnels.api.ts        # Funnels API (93 linhas)
├── deals.api.ts          # Deals API (93 linhas)
├── followups.api.ts      # Followups + Sentiment API (125 linhas)
├── ai.api.ts             # AI + Prompts + Onboarding API (140 linhas)
├── whatsapp.api.ts       # WhatsApp API (229 linhas)
├── flows.api.ts          # Flows API (42 linhas)
├── admin.api.ts          # Admin + Quick Replies + Keys API (175 linhas)
├── dashboard.api.ts      # Dashboard API (22 linhas)
└── types/
    ├── index.ts          # Re-exports (15 linhas)
    ├── common.types.ts   # Types comuns (4 linhas)
    ├── auth.types.ts     # Types auth (28 linhas)
    ├── account.types.ts  # Types account (28 linhas)
    ├── contact.types.ts  # Types contact (69 linhas)
    ├── funnel.types.ts   # Types funnel (37 linhas)
    ├── deal.types.ts     # Types deal (80 linhas)
    ├── followup.types.ts # Types followup (92 linhas)
    ├── ai.types.ts       # Types AI (56 linhas)
    ├── whatsapp.types.ts # Types WhatsApp (107 linhas)
    ├── flow.types.ts     # Types flow (62 linhas)
    ├── admin.types.ts    # Types admin (33 linhas)
    ├── dashboard.types.ts# Types dashboard (53 linhas)
    └── notification.types.ts # Types notification (16 linhas)
```

**Total:** 2.096 linhas em 25 arquivos modulares
**Maior arquivo:** 229 linhas (whatsapp.api.ts) - dentro do limite de 300
**Build:** ✅ Passou

### Backend - Alto (Prioridade P1)

| # | Documento | Arquivo Original | Linhas | Status |
|---|-----------|------------------|--------|--------|
| 07 | [Followups Service](./07-followups-service.md) | `followups.service.ts` | 1.036 | ✅ **100%** |
| 08 | [Contacts Service](./08-contacts-service.md) | `contacts.service.ts` | 995 | ✅ **100%** |
| 09 | [Deals Service](./09-deals-service.md) | `deals.service.ts` | 982 | ✅ **100%** |
| 10 | [Flow Engine](./10-flow-engine.md) | `flow-engine.service.ts` | 674 | ✅ **100%** |
| 11 | [Adapters](./11-adapters.md) | `evolution/uazapi.adapter.ts` | ~1.800 | ✅ **100%** |

#### 📊 Detalhes: 11-Adapters (Completo ✅)
**Branch:** `refactor`
**Resultado:** Adapters refatorados com parsers extraídos e helpers compartilhados

**✅ Arquivos Criados/Modificados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Adapter** | `evolution.adapter.ts` | 506 | ✅ (era 930) |
| **Adapter** | `uazapi.adapter.ts` | 587 | ✅ (era 899) |
| **Parser** | `evolution/evolution.parser.ts` | 508 | ✅ (novo) |
| **Parser** | `uazapi/uazapi.parser.ts` | 409 | ✅ (novo) |
| **Helper** | `helpers/status-mapper.ts` | 63 | ✅ (novo) |
| **Helper** | `helpers/message-extractor.ts` | 151 | ✅ (novo) |
| **Index** | `helpers/index.ts` | 13 | ✅ (novo) |
| **Index** | `evolution/index.ts` | 3 | ✅ (novo) |
| **Index** | `uazapi/index.ts` | 3 | ✅ (novo) |

**Métricas:**
- evolution.adapter.ts: 930 → 506 linhas (-45%)
- uazapi.adapter.ts: 899 → 587 linhas (-35%)
- Código duplicado: Extraído para helpers compartilhados
- Parsers: Extraídos para arquivos separados
**Build:** ✅ Passou

#### 📊 Detalhes: 10-Flow Engine (Completo ✅)
**Branch:** `refactor`
**Resultado:** flow-engine.service.ts agora tem **343 linhas** (era 674) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `flows.types.ts` | 186 | ✅ |
| **Helpers** | `helpers/time-utils.ts` | 85 | ✅ |
| **Helpers** | `helpers/variable-resolver.ts` | 102 | ✅ |
| **Helpers** | `helpers/condition-evaluator.ts` | 187 | ✅ |
| **Helpers** | `helpers/index.ts` | 8 | ✅ |
| **Processors** | `processors/processor.interface.ts` | 87 | ✅ |
| **Processors** | `processors/trigger.processor.ts` | 26 | ✅ |
| **Processors** | `processors/action.processor.ts` | 259 | ✅ |
| **Processors** | `processors/condition.processor.ts` | 60 | ✅ |
| **Processors** | `processors/control.processor.ts` | 152 | ✅ |
| **Processors** | `processors/index.ts` | 53 | ✅ |
| **Services** | `services/flow-trigger.service.ts` | 139 | ✅ |
| **Services** | `services/flow-state.service.ts` | 265 | ✅ |
| **Services** | `services/flow-navigator.service.ts` | 155 | ✅ |
| **Services** | `services/flow-metrics.service.ts` | 153 | ✅ |
| **Services** | `services/index.ts` | 9 | ✅ |

**Total:** 2.269 linhas em 17 arquivos modulares
**Maior arquivo:** 343 linhas (flow-engine.service.ts) - dentro do limite de 400
**Padrão:** Strategy Pattern para processors de nós
**Build:** ✅ Passou

#### 📊 Detalhes: 09-Deals Service (Completo ✅)
**Branch:** `refactor`
**Resultado:** deals.service.ts agora tem **122 linhas** (era 982) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `deals.types.ts` | 198 | ✅ |
| **Helpers** | `helpers/deal-utils.ts` | 96 | ✅ |
| **Helpers** | `helpers/index.ts` | 2 | ✅ |
| **Services** | `services/deal-crud.service.ts` | 459 | ✅ |
| **Services** | `services/deal-stage.service.ts` | 238 | ✅ |
| **Services** | `services/deal-notes.service.ts` | 139 | ✅ |
| **Services** | `services/deal-attachments.service.ts` | 108 | ✅ |
| **Services** | `services/index.ts` | 5 | ✅ |

**Total:** 1.367 linhas em 9 arquivos modulares
**Maior arquivo:** 459 linhas (deal-crud.service.ts) - dentro do limite de 600
**Build:** ✅ Passou

#### 📊 Detalhes: 08-Contacts Service (Completo ✅)
**Branch:** `refactor`
**Resultado:** contacts.service.ts agora tem **201 linhas** (era 995) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `contacts.types.ts` | 160 | ✅ |
| **Helpers** | `helpers/string-utils.ts` | 35 | ✅ |
| **Helpers** | `helpers/phone-normalizer.ts` | 53 | ✅ |
| **Helpers** | `helpers/tag-utils.ts` | 78 | ✅ |
| **Helpers** | `helpers/index.ts` | 5 | ✅ |
| **Services** | `services/contact-crud.service.ts` | 328 | ✅ |
| **Services** | `services/contact-tags.service.ts` | 333 | ✅ |
| **Services** | `services/contact-notes.service.ts` | 62 | ✅ |
| **Services** | `services/contact-merge.service.ts` | 321 | ✅ |
| **Services** | `services/index.ts` | 6 | ✅ |

**Total:** 1.582 linhas em 11 arquivos modulares
**Maior arquivo:** 333 linhas (contact-tags.service.ts) - dentro do limite de 400
**Build:** ✅ Passou

#### 📊 Detalhes: 07-Followups Service (Completo ✅)
**Branch:** `refactor`
**Resultado:** followups.service.ts agora tem **148 linhas** (era 1.036) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `followups.types.ts` | 119 | ✅ |
| **Helpers** | `helpers/index.ts` | 54 | ✅ |
| **Services** | `services/followup-rules.service.ts` | 126 | ✅ |
| **Services** | `services/followup-scheduler.service.ts` | 264 | ✅ |
| **Services** | `services/followup-trigger.service.ts` | 159 | ✅ |
| **Services** | `services/followup-analytics.service.ts` | 156 | ✅ |
| **Services** | `services/index.ts` | 5 | ✅ |
| **Schemas** | `followups.schemas.ts` | 127 | ✅ (atualizado) |

**Total:** ~1.158 linhas em 8 arquivos modulares
**Maior arquivo:** 264 linhas (followup-scheduler.service.ts) - dentro do limite de 300
**Build:** ✅ Passou

### Frontend - Alto (Prioridade P1)

| # | Documento | Arquivo Original | Linhas | Status |
|---|-----------|------------------|--------|--------|
| 12 | [Admin Companies](./12-admin-companies.md) | `AdminCompanies.tsx` | 1.318 | ✅ **100%** |
| 13 | [Contact Detail](./13-contact-detail.md) | `ContactDetail.tsx` | 843 | ✅ **100%** |

#### 📊 Detalhes: 13-Contact Detail (Completo ✅)
**Branch:** `refactor`
**Resultado:** ContactDetail.tsx refatorado em 15 arquivos modulares

**✅ Estrutura Criada:**
```
src/pages/ContactDetail/
├── index.tsx                     # Container principal (195 linhas)
├── types.ts                      # Tipos locais (93 linhas)
├── PageHeader.tsx                # Header com breadcrumb (63 linhas)
├── ContactHeader.tsx             # Card do contato com avatar (56 linhas)
├── ContactTabs.tsx               # Tabs laterais (89 linhas)
├── hooks/
│   ├── index.ts                  # Re-exports (6 linhas)
│   ├── useContact.ts             # Queries (55 linhas)
│   ├── useContactMutations.ts    # Mutations (151 linhas)
│   └── useContactForm.ts         # Form state (51 linhas)
└── components/
    ├── index.ts                  # Re-exports (8 linhas)
    ├── TagsManager.tsx           # Gerenciador de tags (145 linhas)
    ├── ContactForm.tsx           # Formulário de edição (197 linhas)
    ├── DeleteConfirmDialog.tsx   # Dialog de exclusão (49 linhas)
    ├── NewChatDialog.tsx         # Dialog nova conversa (58 linhas)
    └── DangerZone.tsx            # Zona de perigo (28 linhas)
```

**Total:** 1.244 linhas em 15 arquivos modulares
**Maior arquivo:** 197 linhas (ContactForm.tsx) - dentro do limite de 200
**Build:** ✅ Passou

#### 📊 Detalhes: 12-Admin Companies (Completo ✅)
**Branch:** `refactor`
**Resultado:** AdminCompanies.tsx refatorado em 24 arquivos modulares

**✅ Estrutura Criada:**
```
src/pages/Admin/
├── index.tsx                         # Re-export (1 linha)
└── AdminCompanies/
    ├── index.tsx                     # Container principal (73 linhas)
    ├── types.ts                      # Tipos locais (112 linhas)
    ├── utils/
    │   ├── index.ts                  # Re-exports (1 linha)
    │   └── helpers.ts                # validatePassword, getCompanyTrialStatus (68 linhas)
    ├── hooks/
    │   ├── index.ts                  # Re-exports (3 linhas)
    │   ├── useCompanies.ts           # Queries (36 linhas)
    │   ├── useCompanyMutations.ts    # Mutations (172 linhas)
    │   └── useCompanyFilters.ts      # Filtros (55 linhas)
    ├── components/
    │   ├── index.ts                  # Re-exports (4 linhas)
    │   ├── CompanyFilters.tsx        # Filtros de busca (58 linhas)
    │   ├── CompanyList.tsx           # Tabela de empresas (104 linhas)
    │   ├── CompanyDetailsDialog.tsx  # Dialog principal (120 linhas)
    │   └── tabs/
    │       ├── index.ts              # Re-exports (3 linhas)
    │       ├── OverviewTab.tsx       # Tab visão geral (162 linhas)
    │       ├── UsersTab.tsx          # Tab usuários (102 linhas)
    │       └── SettingsTab.tsx       # Tab configurações (181 linhas)
    └── modals/
        ├── index.ts                  # Re-exports (6 linhas)
        ├── CreateCompanyModal.tsx    # Criar empresa (138 linhas)
        ├── InviteUserModal.tsx       # Convidar usuário (102 linhas)
        ├── EditUserModal.tsx         # Editar usuário (156 linhas)
        ├── DeleteUserDialog.tsx      # Confirmar deletar usuário (54 linhas)
        ├── DeleteCompanyDialog.tsx   # Confirmar deletar empresa (82 linhas)
        └── ExtendTrialDialog.tsx     # Estender trial (55 linhas)
```

**Total:** 1.848 linhas em 24 arquivos modulares
**Maior arquivo:** 181 linhas (SettingsTab.tsx) - dentro do limite de 200
**Build:** ✅ Passou

### Backend - Médio (Prioridade P2)

| # | Documento | Arquivo Original | Linhas | Status |
|---|-----------|------------------|--------|--------|
| 14 | [Message Worker](./14-message-worker.md) | `message.worker.ts` | 617 | ✅ **100%** |
| 15 | [Followup Worker](./15-followup-worker.md) | `followup.worker.ts` | 613 | ✅ **100%** |
| 16 | [Dashboard Service](./16-dashboard-service.md) | `dashboard.service.ts` | 624 | ✅ **100%** |

#### 📊 Detalhes: 16-Dashboard Service (Completo ✅)
**Branch:** `refactor`
**Resultado:** dashboard.service.ts agora tem **41 linhas** (era 624) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `dashboard.types.ts` | 70 | ✅ |
| **Helpers** | `helpers/trend.helper.ts` | 52 | ✅ |
| **Helpers** | `helpers/format.helper.ts` | 37 | ✅ |
| **Helpers** | `helpers/index.ts` | 3 | ✅ |
| **Services** | `services/metrics.service.ts` | 147 | ✅ |
| **Services** | `services/ai-summary.service.ts` | 126 | ✅ |
| **Services** | `services/activities.service.ts` | 237 | ✅ |
| **Services** | `services/index.ts` | 4 | ✅ |

**Total:** 717 linhas em 9 arquivos modulares
**Maior arquivo:** 237 linhas (activities.service.ts) - dentro do limite de 250
**Build:** ✅ Passou

#### 📊 Detalhes: 15-Followup Worker (Completo ✅)
**Branch:** `refactor`
**Resultado:** followup.worker.ts agora tem **27 linhas** (era 613) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `workers/followup/types.ts` | 52 | ✅ |
| **Processor** | `workers/followup/followup-processor.ts` | 254 | ✅ |
| **Utils** | `workers/followup/time-utils.ts` | 101 | ✅ |
| **Builder** | `workers/followup/message-builder.ts` | 119 | ✅ |
| **Validator** | `workers/followup/ai-validator.ts` | 79 | ✅ |
| **Queue** | `workers/followup/queue-manager.ts` | 204 | ✅ |
| **Sender** | `workers/followup/message-sender.ts` | 195 | ✅ |
| **Index** | `workers/followup/index.ts` | 60 | ✅ |

**Total:** 1.091 linhas em 9 arquivos modulares
**Maior arquivo:** 254 linhas (followup-processor.ts) - dentro do limite de 300
**Build:** ✅ Passou

#### 📊 Detalhes: 14-Message Worker (Completo ✅)
**Branch:** `refactor`
**Resultado:** message.worker.ts agora tem **32 linhas** (era 617) ✅

**✅ Arquivos Criados:**
| Categoria | Arquivo | Linhas | Status |
|-----------|---------|--------|--------|
| **Types** | `workers/message/types.ts` | 96 | ✅ |
| **Services** | `workers/message/message-processor.ts` | 289 | ✅ |
| **Services** | `workers/message/ai-completion.service.ts` | 200 | ✅ |
| **Services** | `workers/message/message-sender.service.ts` | 192 | ✅ |
| **Services** | `workers/message/audio-transcriber.service.ts` | 136 | ✅ |
| **Services** | `workers/message/metrics-recorder.service.ts` | 145 | ✅ |
| **Index** | `workers/message/index.ts` | 21 | ✅ |
| **Helpers** | `helpers/text-splitter.ts` | 80 | ✅ |
| **Helpers** | `helpers/ai-text-splitter.ts` | 146 | ✅ (usa IA para split natural) |
| **Helpers** | `helpers/index.ts` | 4 | ✅ |

**Melhoria Adicional:** Split de mensagens agora usa IA para quebrar textos de forma natural e humana (não apenas por contagem de caracteres).

**Total:** 1.341 linhas em 11 arquivos modulares
**Maior arquivo:** 289 linhas (message-processor.ts) - dentro do limite de 300
**Build:** ✅ Passou

### Frontend - Médio (Prioridade P2)

| # | Documento | Arquivo Original | Linhas | Status |
|---|-----------|------------------|--------|--------|
| 17 | [Deal Details Drawer](./17-deal-details-drawer.md) | `DealDetailsDrawer.tsx` | 1.147 | ✅ **100%** |
| 18 | [Followup Wizard](./18-followup-wizard.md) | `FollowupRuleWizard.tsx` | 1.078 | ✅ **100%** |

#### 📊 Detalhes: 18-Followup Wizard (Completo ✅)
**Branch:** `refactor`
**Resultado:** FollowupRuleWizard.tsx agora é um re-export de módulo com **18 arquivos** modulares

**✅ Estrutura Criada:**
```
src/components/followups/FollowupWizard/
├── index.tsx                     # Re-export (2 linhas)
├── FollowupWizard.tsx            # Container principal (119 linhas)
├── types.ts                      # Tipos e constantes (136 linhas)
├── hooks/
│   ├── index.ts                  # Re-exports (3 linhas)
│   ├── useWizardData.ts          # Queries de dados (68 linhas)
│   ├── useRuleForm.ts            # Estado do formulário (181 linhas)
│   └── useRuleMutations.ts       # Mutations (97 linhas)
├── steps/
│   ├── index.ts                  # Re-exports (5 linhas)
│   ├── InfoStep.tsx              # Step 1 - Identificação (42 linhas)
│   ├── TriggerStep.tsx           # Step 2 - Gatilhos (172 linhas)
│   ├── ActionStep.tsx            # Step 3 - Ação (78 linhas)
│   ├── LimitsStep.tsx            # Step 4 - Limites (98 linhas)
│   └── PostActionsStep.tsx       # Step 5 - Pós-ações (130 linhas)
└── components/
    ├── index.ts                  # Re-exports (2 linhas)
    ├── WizardStepper.tsx         # Navegação de passos (41 linhas)
    └── WizardNavigation.tsx      # Botões anterior/próximo (53 linhas)
```

**Total:** ~1.227 linhas em 18 arquivos modulares
**Maior arquivo:** 181 linhas (useRuleForm.ts) - dentro do limite de 200
**Build:** ✅ Passou

#### 📊 Detalhes: 17-Deal Details Drawer (Completo ✅)
**Branch:** `refactor`
**Resultado:** DealDetailsDrawer.tsx agora tem **333 linhas** (era 1.147) ✅

**✅ Estrutura Criada:**
```
src/components/funnels/DealDetails/
├── index.tsx                         # Re-export (2 linhas)
├── DealDetailsDrawer.tsx             # Container principal (333 linhas)
├── types.ts                          # Tipos e utilitários (81 linhas)
├── hooks/
│   ├── index.ts                      # Re-exports (4 linhas)
│   ├── useDealDetails.ts             # Query do deal (39 linhas)
│   ├── useDealMutations.ts           # Mutations (160 linhas)
│   ├── useContactPicker.ts           # Picker de contato (45 linhas)
│   └── useTagManager.ts              # Gerenciamento de tags (110 linhas)
├── dialogs/
│   ├── index.ts                      # Re-exports (2 linhas)
│   ├── StatusConfirmDialog.tsx       # Confirmar WON/LOST (82 linhas)
│   └── DeleteConfirmDialog.tsx       # Confirmar exclusão (47 linhas)
└── components/
    ├── index.ts                      # Re-exports (7 linhas)
    ├── StatusSection.tsx             # Status + botões (61 linhas)
    ├── CloneSection.tsx              # Cópias do deal (68 linhas)
    ├── FormSection.tsx               # Formulário de edição (151 linhas)
    ├── ContactSection.tsx            # Contato vinculado (98 linhas)
    ├── TagsSection.tsx               # Tags do contato (101 linhas)
    ├── ActivitySection.tsx           # Histórico de atividades (151 linhas)
    └── LossReasonSection.tsx         # Motivo da perda (20 linhas)
```

**Total:** 1.562 linhas em 18 arquivos modulares
**Maior arquivo:** 333 linhas (DealDetailsDrawer.tsx) - dentro do limite de 400
**Build:** ✅ Passou

---

## Ordem de Execução Recomendada

### Semana 1-2: Backend Core
```
1. 01-whatsapp-service.md (maior impacto)
2. 03-validation-schemas.md (segurança)
```

### Semana 3-4: AI + Frontend Core
```
3. 02-assistant-service.md (complexidade AI)
4. 04-api-frontend.md (dependência de todo frontend)
5. 05-messages-page.md (página principal)
```

### Semana 5-6: P1 Services
```
6. 06-use-chat-hook.md
7. 07-followups-service.md
8. 08-contacts-service.md
9. 09-deals-service.md
10. 10-flow-engine.md
11. 11-adapters.md
```

### Semana 7-8: P1 Frontend + P2
```
12. 12-admin-companies.md
13. 13-contact-detail.md
14. 14-message-worker.md
15. 15-followup-worker.md
16. 16-dashboard-service.md
17. 17-deal-details-drawer.md
18. 18-followup-wizard.md
```

---

## Protocolo de Refatoração

### Antes de Começar
```bash
# 1. Criar branch
git checkout -b refactor/[nome-modulo]

# 2. Verificar testes existentes
npm test -- --grep "[modulo]"

# 3. Backup do estado atual
cp arquivo.ts arquivo.ts.backup
```

### Durante a Refatoração
```
1. Ler documento específico
2. Criar estrutura de pastas
3. Extrair UM service por vez
4. Rodar testes após cada extração
5. Commit atômico
```

### Após Completar
```bash
# 1. Rodar todos os testes
npm test

# 2. Build
npm run build

# 3. Atualizar status neste README
# 4. Marcar documento como ✅ Completo
```

---

## Template de Documento

Cada documento segue esta estrutura (~300-400 linhas):

```markdown
# Refatoração: [Nome]

## Estado Atual
- Linhas, métodos, problemas

## Estrutura Proposta
- Árvore de arquivos novos

## Mapeamento de Métodos
- Qual método vai para qual arquivo

## Código: Before/After
- Exemplos concretos

## Dependências
- Arquivos que importam este

## Checklist
- Lista de tarefas específicas
```

---

## Progresso Geral

| Fase | Documentos | Completos | Em Progresso | Progresso |
|------|------------|-----------|--------------|-----------|
| P0 Backend | 3 | 3 | 0 | ✅ 100% |
| P0 Frontend | 3 | 3 | 0 | ✅ 100% |
| P1 Backend | 5 | 5 | 0 | ✅ 100% |
| P1 Frontend | 2 | 2 | 0 | ✅ 100% |
| P2 Backend | 3 | 3 | 0 | ✅ 100% |
| P2 Frontend | 2 | 2 | 0 | ✅ 100% |
| **Total** | **18** | **18** | **0** | **✅ 100%** |

### 📈 Histórico de Progresso

| Data | Refatoração | Ação | Branch |
|------|-------------|------|--------|
| 2026-01-22 | 01-WhatsApp Service | ✅ Completa - 23 arquivos, 3.381 linhas | `refactor/whatsapp-service` |
| 2026-01-22 | 02-Assistant Service | ✅ Completa - 18 arquivos, 3.711 linhas, 242 linhas no arquivo principal | `refactor` |
| 2026-01-22 | 03-Validação Zod | 🔄 Iniciada - WhatsApp (23), Contacts (19), Deals (13) validados | `refactor` |
| 2026-01-23 | 03-Validação Zod | 🔄 Progresso - Funnels (16), Followups (15), Flows (7) validados | `refactor` |
| 2026-01-23 | 03-Validação Zod | ✅ Completa - AI (14), Dashboard (3) + 11 rotas inline validadas | `refactor` |
| 2026-01-23 | 04-API Client | ✅ Completa - 25 arquivos modulares, 2.096 linhas | `refactor` |
| 2026-01-23 | 05-Messages Page | ✅ Completa - 10 arquivos modulares, 2.013 linhas, 508 linhas no arquivo principal | `refactor` |
| 2026-01-23 | 06-Use Chat Hook | ✅ Completa - 8 arquivos modulares, 898 linhas, 227 linhas no arquivo principal | `refactor` |
| 2026-01-23 | 07-Followups Service | ✅ Completa - 8 arquivos modulares, ~1.158 linhas, 148 linhas no arquivo principal | `refactor` |
| 2026-01-23 | 08-Contacts Service | ✅ Completa - 11 arquivos modulares, 1.582 linhas, 201 linhas no arquivo principal | `refactor` |
| 2026-01-23 | 09-Deals Service | ✅ Completa - 9 arquivos modulares, 1.367 linhas, 122 linhas no arquivo principal | `refactor` |
| 2026-01-23 | 10-Flow Engine | ✅ Completa - 17 arquivos modulares, 2.269 linhas, 343 linhas no arquivo principal, Strategy Pattern | `refactor` |
| 2026-01-23 | 11-Adapters | ✅ Completa - Parsers extraídos, helpers compartilhados, evolution -45%, uazapi -35% | `refactor` |
| 2026-01-23 | 12-Admin Companies | ✅ Completa - 24 arquivos modulares, 1.848 linhas, 73 linhas no container principal | `refactor` |
| 2026-01-23 | 13-Contact Detail | ✅ Completa - 15 arquivos modulares, 1.244 linhas, 195 linhas no container principal | `refactor` |
| 2026-01-23 | 14-Message Worker | ✅ Completa - 11 arquivos modulares, 1.341 linhas, 32 linhas no worker principal + AI text splitter | `refactor` |
| 2026-01-23 | 15-Followup Worker | ✅ Completa - 9 arquivos modulares, 1.091 linhas, 27 linhas no worker principal | `refactor` |
| 2026-01-23 | 16-Dashboard Service | ✅ Completa - 9 arquivos modulares, 717 linhas, 41 linhas no service principal | `refactor` |
| 2026-01-23 | 17-Deal Details Drawer | ✅ Completa - 18 arquivos modulares, 1.562 linhas, 333 linhas no container principal | `refactor` |
| 2026-01-23 | 18-Followup Wizard | ✅ Completa - 18 arquivos modulares, ~1.227 linhas, 119 linhas no container principal | `refactor` |

---

## Links

- [Plano Master](../REFACTORING-PLAN.md)
- [Skill de Refatoração](../../.claude/skills/refactoring.md)
- [STATUS do Projeto](../docs/STATUS.md)

---

**Última Atualização:** 2026-01-23 (Followup Wizard ✅ 100%, **REFATORAÇÃO COMPLETA!** 🎉)
