# Refatoração: WhatsApp Service

**Arquivo:** `server/src/modules/whatsapp/whatsapp.service.ts`
**Linhas:** 2.482
**Prioridade:** P0 - CRÍTICO
**Estimativa:** 2-3 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 2.482 | 400 | 🔴 CRÍTICO |
| Métodos | 32 | 10 | 🔴 CRÍTICO |
| Responsabilidades | 7 | 1 | 🔴 CRÍTICO |
| Imports | 12 | 10 | 🟠 ALTO |

### Métodos Atuais (32 total)

```typescript
// Webhook Processing
processWebhook(provider, instanceName, payload)

// Message Operations
markAsRead(companyId, conversationId)
sendMessage(companyId, instanceId, to, text, quotedMessageId?)
sendMediaMessage(companyId, instanceId, to, media, caption?, quotedMessageId?)
sendReaction(companyId, instanceId, to, messageId, emoji)

// Conversation Management
getConversations(companyId, filters)  // 12 filtros diferentes!
getMessages(companyId, conversationId, options)
togglePause(companyId, conversationId, paused)
archiveConversation(companyId, conversationId)
deleteConversation(companyId, conversationId)

// Instance Lifecycle
listInstances(companyId)
checkInstanceConnection(companyId, instanceId)
checkAllInstanceConnections(companyId)
createInstance(companyId, data)
updateInstanceDisplayName(companyId, instanceId, displayName)
connectInstance(companyId, instanceId)
deleteInstance(companyId, instanceId)

// Credential Management
getProviderCredentials(companyId)
createProviderCredential(companyId, data)
deleteProviderCredential(companyId, credentialId)
toggleProviderCredential(companyId, credentialId, enabled)

// Profile
getUserProfile(companyId, instanceId, phone)

// Private Helpers
resolveContactForMessage(...)  // ~100 linhas
buildRemoteJid(...)
getCredentials(...)
```

### Problemas Identificados

1. **7 responsabilidades distintas** em um único arquivo
2. **getConversations()** com 12 filtros inline (complexidade alta)
3. **resolveContactForMessage()** com 100+ linhas de lógica de resolução
4. **processWebhook()** mistura parsing + business logic + database
5. **Duplicação** de chamadas ao adapter em múltiplos métodos

---

## Estrutura Proposta

```
server/src/modules/whatsapp/
├── whatsapp.service.ts          # Facade/Coordinator (~150 linhas)
├── whatsapp.routes.ts           # Existente (manter)
├── whatsapp.types.ts            # Tipos compartilhados (~100 linhas)
├── services/
│   ├── index.ts                 # Re-exports
│   ├── webhook.service.ts       # Webhook processing (~400 linhas)
│   ├── message.service.ts       # Send/receive messages (~350 linhas)
│   ├── conversation.service.ts  # Conversation CRUD (~400 linhas)
│   ├── instance.service.ts      # Instance lifecycle (~350 linhas)
│   └── credential.service.ts    # Provider credentials (~200 linhas)
└── helpers/
    ├── contact-resolver.ts      # Resolução de contatos (~150 linhas)
    └── jid-builder.ts           # Construção de JIDs (~50 linhas)
```

---

## Mapeamento de Métodos

### webhook.service.ts
```typescript
export class WhatsAppWebhookService {
  // DE: whatsapp.service.ts
  processWebhook(provider, instanceName, payload)

  // NOVO: extraído de processWebhook
  private parseWebhookPayload(provider, payload)
  private handleMessageReceived(message)
  private handleStatusUpdate(status)
  private handleReactionReceived(reaction)
}
```

### message.service.ts
```typescript
export class WhatsAppMessageService {
  // DE: whatsapp.service.ts
  markAsRead(companyId, conversationId)
  sendMessage(companyId, instanceId, to, text, quotedMessageId?)
  sendMediaMessage(companyId, instanceId, to, media, caption?, quotedMessageId?)
  sendReaction(companyId, instanceId, to, messageId, emoji)

  // NOVO: extraído para helper
  private splitLongMessage(text, maxLength = 1000)
}
```

### conversation.service.ts
```typescript
export class WhatsAppConversationService {
  // DE: whatsapp.service.ts
  getConversations(companyId, filters)
  getMessages(companyId, conversationId, options)
  togglePause(companyId, conversationId, paused)
  archiveConversation(companyId, conversationId)
  deleteConversation(companyId, conversationId)

  // NOVO: extraído de getConversations
  private buildConversationFilters(filters)
  private buildConversationIncludes()
}
```

### instance.service.ts
```typescript
export class WhatsAppInstanceService {
  // DE: whatsapp.service.ts
  listInstances(companyId)
  checkInstanceConnection(companyId, instanceId)
  checkAllInstanceConnections(companyId)
  createInstance(companyId, data)
  updateInstanceDisplayName(companyId, instanceId, displayName)
  connectInstance(companyId, instanceId)
  deleteInstance(companyId, instanceId)
  getUserProfile(companyId, instanceId, phone)
}
```

### credential.service.ts
```typescript
export class WhatsAppCredentialService {
  // DE: whatsapp.service.ts
  getProviderCredentials(companyId)
  createProviderCredential(companyId, data)
  deleteProviderCredential(companyId, credentialId)
  toggleProviderCredential(companyId, credentialId, enabled)

  // DE: private method
  getCredentials(companyId, instanceId)
}
```

---

## Código: Before/After

### BEFORE: whatsapp.service.ts (atual)

```typescript
// server/src/modules/whatsapp/whatsapp.service.ts
import { prisma } from '../../config/database';
import { getAdapter } from '../../shared/adapters';
import { socketService } from '../../services/socket.service';
import { storageService } from '../../services/storage/storage.service';
// ... mais 8 imports

export class WhatsAppService {
  // 2.482 linhas de código misturado

  async processWebhook(...) { /* 200 linhas */ }
  async sendMessage(...) { /* 80 linhas */ }
  async getConversations(...) { /* 150 linhas com 12 filtros */ }
  // ... mais 29 métodos
}

export const whatsappService = new WhatsAppService();
```

### AFTER: Estrutura modular

**whatsapp.service.ts (Facade)**
```typescript
// server/src/modules/whatsapp/whatsapp.service.ts
import { WhatsAppWebhookService } from './services/webhook.service';
import { WhatsAppMessageService } from './services/message.service';
import { WhatsAppConversationService } from './services/conversation.service';
import { WhatsAppInstanceService } from './services/instance.service';
import { WhatsAppCredentialService } from './services/credential.service';

export class WhatsAppService {
  constructor(
    private webhook: WhatsAppWebhookService,
    private messages: WhatsAppMessageService,
    private conversations: WhatsAppConversationService,
    private instances: WhatsAppInstanceService,
    private credentials: WhatsAppCredentialService,
  ) {}

  // Delegate to specialized services
  processWebhook = this.webhook.processWebhook.bind(this.webhook);
  sendMessage = this.messages.sendMessage.bind(this.messages);
  getConversations = this.conversations.getConversations.bind(this.conversations);
  // ... delegates para outros métodos
}

// Singleton com DI
export const whatsappService = new WhatsAppService(
  new WhatsAppWebhookService(),
  new WhatsAppMessageService(),
  new WhatsAppConversationService(),
  new WhatsAppInstanceService(),
  new WhatsAppCredentialService(),
);
```

**services/message.service.ts**
```typescript
// server/src/modules/whatsapp/services/message.service.ts
import { prisma } from '../../../config/database';
import { getAdapter } from '../../../shared/adapters';
import { socketService } from '../../../services/socket.service';
import type { SendMessageParams, SendMediaParams } from '../whatsapp.types';

export class WhatsAppMessageService {
  async markAsRead(companyId: string, conversationId: string): Promise<void> {
    const conversation = await prisma.conversation.findFirst({
      where: { id: conversationId, companyId },
      include: { instance: true },
    });

    if (!conversation?.instance) return;

    const adapter = getAdapter(conversation.instance.provider);
    await adapter.markAsRead(/* ... */);
  }

  async sendMessage(params: SendMessageParams): Promise<Message> {
    const { companyId, instanceId, to, text, quotedMessageId } = params;

    // Validação
    const instance = await this.validateInstance(companyId, instanceId);

    // Envio
    const adapter = getAdapter(instance.provider);
    const result = await adapter.sendTextMessage(/* ... */);

    // Persistência
    return this.persistMessage(result);
  }

  async sendMediaMessage(params: SendMediaParams): Promise<Message> {
    // Similar ao sendMessage
  }

  async sendReaction(/* ... */): Promise<void> {
    // Lógica de reação
  }

  // Helpers privados
  private async validateInstance(companyId: string, instanceId: string) {
    const instance = await prisma.whatsAppInstance.findFirst({
      where: { id: instanceId, companyId },
    });
    if (!instance) throw new Error('Instance not found');
    return instance;
  }

  private splitLongMessage(text: string, maxLength = 1000): string[] {
    // Lógica de split
  }
}
```

**services/conversation.service.ts**
```typescript
// server/src/modules/whatsapp/services/conversation.service.ts
import { prisma } from '../../../config/database';
import type { ConversationFilters } from '../whatsapp.types';

export class WhatsAppConversationService {
  async getConversations(companyId: string, filters: ConversationFilters) {
    const where = this.buildFilters(companyId, filters);
    const include = this.buildIncludes();

    return prisma.conversation.findMany({
      where,
      include,
      orderBy: { lastMessageAt: 'desc' },
      take: filters.limit ?? 50,
      skip: filters.offset ?? 0,
    });
  }

  // Extraído: 12 filtros organizados
  private buildFilters(companyId: string, filters: ConversationFilters) {
    return {
      companyId,
      ...(filters.instanceId && { instanceId: filters.instanceId }),
      ...(filters.status && { status: filters.status }),
      ...(filters.assignedTo && { assignedToId: filters.assignedTo }),
      ...(filters.paused !== undefined && { paused: filters.paused }),
      ...(filters.archived !== undefined && { archived: filters.archived }),
      ...(filters.hasUnread && { unreadCount: { gt: 0 } }),
      ...(filters.search && {
        OR: [
          { contact: { name: { contains: filters.search, mode: 'insensitive' } } },
          { contact: { phone: { contains: filters.search } } },
        ],
      }),
      ...(filters.tags?.length && {
        contact: { tags: { some: { id: { in: filters.tags } } } },
      }),
      ...(filters.funnelId && {
        contact: { deals: { some: { funnelId: filters.funnelId } } },
      }),
      ...(filters.stageId && {
        contact: { deals: { some: { stageId: filters.stageId } } },
      }),
      ...(filters.sentiment && {
        contact: { sentiment: filters.sentiment },
      }),
    };
  }

  private buildIncludes() {
    return {
      contact: {
        include: {
          tags: true,
          deals: { include: { stage: true, funnel: true } },
        },
      },
      instance: true,
      messages: {
        take: 1,
        orderBy: { createdAt: 'desc' as const },
      },
    };
  }

  async togglePause(/* ... */) { /* ... */ }
  async archiveConversation(/* ... */) { /* ... */ }
  async deleteConversation(/* ... */) { /* ... */ }
}
```

**helpers/contact-resolver.ts**
```typescript
// server/src/modules/whatsapp/helpers/contact-resolver.ts
import { prisma } from '../../../config/database';

interface ResolveContactParams {
  companyId: string;
  phone: string;
  pushName?: string;
  lidPhone?: string;
}

export async function resolveContactForMessage(params: ResolveContactParams) {
  const { companyId, phone, pushName, lidPhone } = params;

  // 1. Buscar contato existente por phone ou LID
  let contact = await prisma.contact.findFirst({
    where: {
      companyId,
      OR: [
        { phone },
        { lidPhone },
        ...(lidPhone ? [{ phone: lidPhone }] : []),
      ],
    },
  });

  // 2. Se não existe, criar
  if (!contact) {
    contact = await prisma.contact.create({
      data: {
        companyId,
        phone,
        lidPhone,
        name: pushName || phone,
      },
    });
  }

  // 3. Atualizar LID se necessário
  if (lidPhone && !contact.lidPhone) {
    contact = await prisma.contact.update({
      where: { id: contact.id },
      data: { lidPhone },
    });
  }

  return contact;
}
```

---

## Dependências a Atualizar

Arquivos que importam `whatsappService`:

```typescript
// Buscar com: grep -r "whatsappService" server/src --include="*.ts"

// 1. Routes
server/src/modules/whatsapp/whatsapp.routes.ts  // Não precisa mudar (facade mantém API)

// 2. Workers
server/src/jobs/message.worker.ts               // Verificar imports específicos

// 3. Outros services
server/src/modules/flows/flow-engine.service.ts // Usa sendMessage
server/src/modules/ai/assistant.service.ts      // Usa sendMessage

// 4. Webhooks externos
server/src/app.ts                               // Webhook entry point
```

**Ação:** Como o `whatsappService` é um Facade que mantém a mesma API pública, a maioria dos imports **não precisa mudar**.

---

## Ordem de Extração

```
1. Criar whatsapp.types.ts (tipos compartilhados)
2. Criar helpers/contact-resolver.ts
3. Criar helpers/jid-builder.ts
4. Criar services/credential.service.ts (menos dependências)
5. Criar services/instance.service.ts
6. Criar services/message.service.ts
7. Criar services/conversation.service.ts
8. Criar services/webhook.service.ts (mais complexo, por último)
9. Refatorar whatsapp.service.ts para Facade
10. Atualizar imports se necessário
11. Rodar testes
```

---

## Checklist

### Preparação
- [ ] Criar branch `refactor/whatsapp-service`
- [ ] Backup do arquivo original
- [ ] Rodar testes existentes (baseline)

### Extração
- [ ] Criar `whatsapp.types.ts`
- [ ] Criar `helpers/contact-resolver.ts`
- [ ] Criar `helpers/jid-builder.ts`
- [ ] Criar `services/credential.service.ts`
- [ ] Criar `services/instance.service.ts`
- [ ] Criar `services/message.service.ts`
- [ ] Criar `services/conversation.service.ts`
- [ ] Criar `services/webhook.service.ts`
- [ ] Criar `services/index.ts` (re-exports)

### Refatoração
- [ ] Converter `whatsapp.service.ts` para Facade
- [ ] Atualizar imports em arquivos dependentes
- [ ] Remover código duplicado

### Validação
- [ ] Testes unitários passando
- [ ] Testes de integração passando
- [ ] Build sem erros
- [ ] Lint sem warnings
- [ ] PR review aprovado

### Documentação
- [ ] Atualizar este documento (status: ✅)
- [ ] Atualizar README da pasta

---

## Métricas Esperadas

| Métrica | Antes | Depois | Melhoria |
|---------|-------|--------|----------|
| Linhas (maior arquivo) | 2.482 | ~400 | 84%↓ |
| Métodos por arquivo | 32 | ~6 | 81%↓ |
| Responsabilidades | 7 | 1 | 86%↓ |
| Testabilidade | Baixa | Alta | ✅ |

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
