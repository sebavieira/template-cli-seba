# Refatoração: Validação Zod

**Escopo:** 127 route handlers sem validação
**Prioridade:** P0 - CRÍTICO (Segurança)
**Estimativa:** 2-3 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Meta | Status |
|---------|-------|------|--------|
| Routes com validação | 20 | 147 | 🔴 14% |
| Routes sem validação | 127 | 0 | 🔴 86% |

### Routes por Arquivo

| Arquivo | Handlers | Schemas | Gap |
|---------|----------|---------|-----|
| `whatsapp.routes.ts` | 23 | 0 | 23 |
| `contacts.routes.ts` | 19 | 0 | 19 |
| `funnels.routes.ts` | 16 | 0 | 16 |
| `followups.routes.ts` | 15 | 0 | 15 |
| `deals.routes.ts` | 13 | 0 | 13 |
| `flows.routes.ts` | 7 | 0 | 7 |
| `prompts.routes.ts` | 5 | 0 | 5 |
| `quick-replies.routes.ts` | 4 | 0 | 4 |
| `dashboard.routes.ts` | 3 | 0 | 3 |
| `assistant.routes.ts` | 4 | 0 | 4 |
| `onboarding.routes.ts` | 3 | 0 | 3 |
| Outros | 15 | 0 | 15 |

---

## Estrutura Proposta

```
server/src/
├── shared/
│   ├── schemas/
│   │   ├── index.ts              # Re-exports
│   │   ├── common.schemas.ts     # Schemas reutilizáveis
│   │   └── validation.middleware.ts  # Middleware Fastify
│   └── ...
└── modules/
    ├── whatsapp/
    │   ├── whatsapp.schemas.ts   # Schemas do módulo
    │   └── whatsapp.routes.ts
    ├── contacts/
    │   ├── contacts.schemas.ts
    │   └── contacts.routes.ts
    └── ... (cada módulo com seu .schemas.ts)
```

---

## Schemas Comuns

```typescript
// server/src/shared/schemas/common.schemas.ts

import { z } from 'zod';

// === IDs ===
export const uuidSchema = z.string().uuid('Invalid UUID format');

export const idParamSchema = z.object({
  id: uuidSchema,
});

// === Pagination ===
export const paginationSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  offset: z.coerce.number().int().min(0).optional(),
});

// === Sorting ===
export const sortSchema = z.object({
  sortBy: z.string().optional(),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
});

// === Search ===
export const searchSchema = z.object({
  search: z.string().min(1).max(100).optional(),
  q: z.string().min(1).max(100).optional(), // alias
});

// === Date Range ===
export const dateRangeSchema = z.object({
  startDate: z.coerce.date().optional(),
  endDate: z.coerce.date().optional(),
}).refine(
  data => !data.startDate || !data.endDate || data.startDate <= data.endDate,
  { message: 'startDate must be before endDate' }
);

// === Phone ===
export const phoneSchema = z.string()
  .min(10, 'Phone must have at least 10 digits')
  .max(15, 'Phone must have at most 15 digits')
  .regex(/^\+?[\d\s-]+$/, 'Invalid phone format');

// === Email ===
export const emailSchema = z.string().email('Invalid email format');

// === Common Filters ===
export const baseFilterSchema = paginationSchema
  .merge(sortSchema)
  .merge(searchSchema)
  .merge(dateRangeSchema);
```

---

## Middleware de Validação

```typescript
// server/src/shared/schemas/validation.middleware.ts

import { FastifyRequest, FastifyReply } from 'fastify';
import { z, ZodSchema, ZodError } from 'zod';

interface ValidationSchemas {
  body?: ZodSchema;
  params?: ZodSchema;
  query?: ZodSchema;
}

export function validate(schemas: ValidationSchemas) {
  return async (request: FastifyRequest, reply: FastifyReply) => {
    try {
      if (schemas.params) {
        request.params = schemas.params.parse(request.params);
      }
      if (schemas.query) {
        request.query = schemas.query.parse(request.query);
      }
      if (schemas.body) {
        request.body = schemas.body.parse(request.body);
      }
    } catch (error) {
      if (error instanceof ZodError) {
        return reply.status(400).send({
          success: false,
          error: 'Validation error',
          details: error.errors.map(e => ({
            field: e.path.join('.'),
            message: e.message,
          })),
        });
      }
      throw error;
    }
  };
}

// Helper para inferir tipos
export type InferBody<T extends ZodSchema> = z.infer<T>;
export type InferParams<T extends ZodSchema> = z.infer<T>;
export type InferQuery<T extends ZodSchema> = z.infer<T>;
```

---

## Schemas por Módulo

### WhatsApp Schemas

```typescript
// server/src/modules/whatsapp/whatsapp.schemas.ts

import { z } from 'zod';
import { uuidSchema, phoneSchema, paginationSchema } from '../../shared/schemas/common.schemas';

// === Params ===
export const instanceIdParamSchema = z.object({
  instanceId: uuidSchema,
});

export const conversationIdParamSchema = z.object({
  conversationId: uuidSchema,
});

// === Send Message ===
export const sendMessageBodySchema = z.object({
  to: phoneSchema,
  text: z.string().min(1).max(4096),
  quotedMessageId: z.string().optional(),
});

export const sendMediaBodySchema = z.object({
  to: phoneSchema,
  mediaUrl: z.string().url(),
  mediaType: z.enum(['image', 'video', 'audio', 'document']),
  caption: z.string().max(1024).optional(),
  fileName: z.string().optional(),
});

export const sendReactionBodySchema = z.object({
  to: phoneSchema,
  messageId: z.string().min(1),
  emoji: z.string().min(1).max(10),
});

// === Conversations ===
export const getConversationsQuerySchema = paginationSchema.extend({
  instanceId: uuidSchema.optional(),
  status: z.enum(['open', 'closed', 'pending']).optional(),
  assignedTo: uuidSchema.optional(),
  paused: z.coerce.boolean().optional(),
  archived: z.coerce.boolean().optional(),
  hasUnread: z.coerce.boolean().optional(),
  tags: z.string().transform(s => s.split(',')).optional(),
  funnelId: uuidSchema.optional(),
  stageId: uuidSchema.optional(),
  sentiment: z.enum(['positive', 'neutral', 'negative']).optional(),
});

export const togglePauseBodySchema = z.object({
  paused: z.boolean(),
});

// === Instances ===
export const createInstanceBodySchema = z.object({
  name: z.string().min(1).max(50),
  provider: z.enum(['evolution', 'uazapi']),
  credentialId: uuidSchema,
});

export const updateInstanceBodySchema = z.object({
  displayName: z.string().min(1).max(50).optional(),
});

// === Credentials ===
export const createCredentialBodySchema = z.object({
  provider: z.enum(['evolution', 'uazapi']),
  apiUrl: z.string().url(),
  apiKey: z.string().min(1),
  name: z.string().min(1).max(50).optional(),
});

// === Types (inferidos) ===
export type SendMessageBody = z.infer<typeof sendMessageBodySchema>;
export type SendMediaBody = z.infer<typeof sendMediaBodySchema>;
export type GetConversationsQuery = z.infer<typeof getConversationsQuerySchema>;
export type CreateInstanceBody = z.infer<typeof createInstanceBodySchema>;
```

### Contacts Schemas

```typescript
// server/src/modules/contacts/contacts.schemas.ts

import { z } from 'zod';
import { uuidSchema, phoneSchema, emailSchema, baseFilterSchema } from '../../shared/schemas/common.schemas';

// === Params ===
export const contactIdParamSchema = z.object({
  contactId: uuidSchema,
});

// === List/Search ===
export const listContactsQuerySchema = baseFilterSchema.extend({
  tags: z.string().transform(s => s.split(',')).optional(),
  funnelId: uuidSchema.optional(),
  stageId: uuidSchema.optional(),
  hasDeals: z.coerce.boolean().optional(),
  sentiment: z.enum(['positive', 'neutral', 'negative']).optional(),
});

// === Create ===
export const createContactBodySchema = z.object({
  name: z.string().min(1).max(100),
  phone: phoneSchema,
  email: emailSchema.optional(),
  notes: z.string().max(5000).optional(),
  customFields: z.record(z.string(), z.any()).optional(),
});

// === Update ===
export const updateContactBodySchema = z.object({
  name: z.string().min(1).max(100).optional(),
  email: emailSchema.optional().nullable(),
  notes: z.string().max(5000).optional().nullable(),
  customFields: z.record(z.string(), z.any()).optional(),
});

// === Tags ===
export const assignTagsBodySchema = z.object({
  tagIds: z.array(uuidSchema).min(1).max(20),
});

export const removeTagsBodySchema = z.object({
  tagIds: z.array(uuidSchema).min(1).max(20),
});

// === Notes ===
export const addNoteBodySchema = z.object({
  content: z.string().min(1).max(5000),
});

// === Types ===
export type ListContactsQuery = z.infer<typeof listContactsQuerySchema>;
export type CreateContactBody = z.infer<typeof createContactBodySchema>;
export type UpdateContactBody = z.infer<typeof updateContactBodySchema>;
```

### Deals Schemas

```typescript
// server/src/modules/deals/deals.schemas.ts

import { z } from 'zod';
import { uuidSchema, baseFilterSchema } from '../../shared/schemas/common.schemas';

// === Params ===
export const dealIdParamSchema = z.object({
  dealId: uuidSchema,
});

// === List ===
export const listDealsQuerySchema = baseFilterSchema.extend({
  funnelId: uuidSchema.optional(),
  stageId: uuidSchema.optional(),
  contactId: uuidSchema.optional(),
  minValue: z.coerce.number().min(0).optional(),
  maxValue: z.coerce.number().min(0).optional(),
  status: z.enum(['open', 'won', 'lost']).optional(),
});

// === Create ===
export const createDealBodySchema = z.object({
  contactId: uuidSchema,
  funnelId: uuidSchema,
  stageId: uuidSchema.optional(), // Se não informado, usa primeiro stage
  title: z.string().min(1).max(200).optional(),
  value: z.number().min(0).default(0),
  expectedCloseDate: z.coerce.date().optional(),
});

// === Update ===
export const updateDealBodySchema = z.object({
  title: z.string().min(1).max(200).optional(),
  value: z.number().min(0).optional(),
  expectedCloseDate: z.coerce.date().optional().nullable(),
  status: z.enum(['open', 'won', 'lost']).optional(),
});

// === Move Stage ===
export const moveDealStageBodySchema = z.object({
  stageId: uuidSchema,
  reason: z.string().max(500).optional(),
});

// === Notes ===
export const addDealNoteBodySchema = z.object({
  content: z.string().min(1).max(5000),
  type: z.enum(['note', 'call', 'meeting', 'email']).default('note'),
});

// === Attachments ===
export const addDealAttachmentBodySchema = z.object({
  fileName: z.string().min(1).max(255),
  fileUrl: z.string().url(),
  fileSize: z.number().int().min(0).optional(),
  mimeType: z.string().optional(),
});

// === Types ===
export type ListDealsQuery = z.infer<typeof listDealsQuerySchema>;
export type CreateDealBody = z.infer<typeof createDealBodySchema>;
export type UpdateDealBody = z.infer<typeof updateDealBodySchema>;
export type MoveDealStageBody = z.infer<typeof moveDealStageBodySchema>;
```

### Flows Schemas

```typescript
// server/src/modules/flows/flows.schemas.ts

import { z } from 'zod';
import { uuidSchema, paginationSchema } from '../../shared/schemas/common.schemas';

// === Params ===
export const flowIdParamSchema = z.object({
  flowId: uuidSchema,
});

// === List ===
export const listFlowsQuerySchema = paginationSchema.extend({
  status: z.enum(['active', 'inactive', 'draft']).optional(),
  triggerType: z.enum(['keyword', 'manual', 'api', 'schedule']).optional(),
});

// === Node Schema (recursivo) ===
const baseNodeSchema = z.object({
  id: z.string().min(1),
  type: z.enum([
    'trigger', 'message', 'condition', 'delay',
    'action', 'ai', 'webhook', 'end'
  ]),
  position: z.object({
    x: z.number(),
    y: z.number(),
  }),
  data: z.record(z.any()),
});

const edgeSchema = z.object({
  id: z.string().min(1),
  source: z.string().min(1),
  target: z.string().min(1),
  sourceHandle: z.string().optional(),
  targetHandle: z.string().optional(),
  label: z.string().optional(),
});

// === Create/Update ===
export const createFlowBodySchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),
  triggerType: z.enum(['keyword', 'manual', 'api', 'schedule']),
  triggerConfig: z.object({
    keywords: z.array(z.string()).optional(),
    schedule: z.string().optional(), // cron
  }).optional(),
  nodes: z.array(baseNodeSchema).min(1),
  edges: z.array(edgeSchema),
  status: z.enum(['active', 'inactive', 'draft']).default('draft'),
});

export const updateFlowBodySchema = createFlowBodySchema.partial().extend({
  id: uuidSchema,
});

// === Trigger ===
export const triggerFlowBodySchema = z.object({
  contactId: uuidSchema,
  conversationId: uuidSchema.optional(),
  variables: z.record(z.string(), z.any()).optional(),
});

// === Types ===
export type ListFlowsQuery = z.infer<typeof listFlowsQuerySchema>;
export type CreateFlowBody = z.infer<typeof createFlowBodySchema>;
export type TriggerFlowBody = z.infer<typeof triggerFlowBodySchema>;
```

---

## Aplicando nas Routes

### BEFORE (sem validação)

```typescript
// whatsapp.routes.ts - ANTES
app.post('/instances/:instanceId/messages', async (request, reply) => {
  const { instanceId } = request.params as { instanceId: string };
  const { to, text } = request.body as { to: string; text: string };

  // Sem validação - vulnerável!
  const result = await whatsappService.sendMessage(
    request.user!.companyId!,
    instanceId,
    to,
    text
  );

  return reply.send({ success: true, data: result });
});
```

### AFTER (com validação)

```typescript
// whatsapp.routes.ts - DEPOIS
import { validate, InferBody, InferParams } from '../../shared/schemas/validation.middleware';
import {
  instanceIdParamSchema,
  sendMessageBodySchema,
  type SendMessageBody,
} from './whatsapp.schemas';

app.post<{
  Params: z.infer<typeof instanceIdParamSchema>;
  Body: SendMessageBody;
}>(
  '/instances/:instanceId/messages',
  { preHandler: validate({ params: instanceIdParamSchema, body: sendMessageBodySchema }) },
  async (request, reply) => {
    const { instanceId } = request.params; // Validado e tipado!
    const { to, text, quotedMessageId } = request.body; // Validado e tipado!

    const result = await whatsappService.sendMessage(
      request.user.companyId, // Agora seguro
      instanceId,
      to,
      text,
      quotedMessageId
    );

    return reply.send({ success: true, data: result });
  }
);
```

---

## Ordem de Implementação

```
1. Criar shared/schemas/common.schemas.ts
2. Criar shared/schemas/validation.middleware.ts
3. whatsapp.schemas.ts + aplicar em whatsapp.routes.ts
4. contacts.schemas.ts + aplicar
5. deals.schemas.ts + aplicar
6. funnels.schemas.ts + aplicar
7. followups.schemas.ts + aplicar
8. flows.schemas.ts + aplicar
9. Demais módulos
10. Testes de validação
```

---

## Checklist

### Infraestrutura
- [ ] Criar `shared/schemas/common.schemas.ts`
- [ ] Criar `shared/schemas/validation.middleware.ts`
- [ ] Criar `shared/schemas/index.ts`

### Por Módulo
- [ ] `whatsapp.schemas.ts` (23 handlers)
- [ ] `contacts.schemas.ts` (19 handlers)
- [ ] `funnels.schemas.ts` (16 handlers)
- [ ] `followups.schemas.ts` (15 handlers)
- [ ] `deals.schemas.ts` (13 handlers)
- [ ] `flows.schemas.ts` (7 handlers)
- [ ] `prompts.schemas.ts` (5 handlers)
- [ ] `quick-replies.schemas.ts` (4 handlers)
- [ ] `dashboard.schemas.ts` (3 handlers)
- [ ] `assistant.schemas.ts` (4 handlers)
- [ ] `onboarding.schemas.ts` (3 handlers)
- [ ] Demais módulos (15 handlers)

### Aplicação
- [ ] Aplicar validação em todas as routes
- [ ] Atualizar tipos TypeScript
- [ ] Testes de validação (happy path)
- [ ] Testes de validação (error cases)

### Validação Final
- [ ] 0 handlers sem schema
- [ ] Build sem erros
- [ ] Testes passando

---

## Benefícios

| Aspecto | Antes | Depois |
|---------|-------|--------|
| Segurança | Vulnerável a injection | Validação estrita |
| Type Safety | `as any` espalhados | Tipos inferidos |
| Erros | Genéricos | Detalhados por campo |
| Documentação | Manual | Auto-gerada dos schemas |

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
