# Refatoração: WhatsApp Adapters

**Arquivos:**
- `server/src/shared/adapters/evolution.adapter.ts` (901 linhas)
- `server/src/shared/adapters/uazapi.adapter.ts` (899 linhas)

**Linhas Total:** ~1.800
**Prioridade:** P1 - ALTO
**Estimativa:** 2 dias

---

## Estado Atual

### Métricas
| Arquivo | Linhas | Limite | Status |
|---------|--------|--------|--------|
| `evolution.adapter.ts` | 901 | 400 | 🔴 CRÍTICO |
| `uazapi.adapter.ts` | 899 | 400 | 🔴 CRÍTICO |

### Problema Principal

**70% de código duplicado** entre os dois adapters:
- Mesma interface
- Mesma estrutura de métodos
- Diferença apenas nos detalhes de API

### Métodos Comuns (duplicados)

```typescript
// Ambos adapters têm:
createInstance()
connectInstance()
checkStatus()
deleteInstance()
configureWebhook()
sendTextMessage()
sendMediaMessage()
sendReactionMessage()
fetchAudioBase64()
fetchMediaBase64()
fetchProfilePicture()
parseWebhookPayload()  // 200+ linhas cada!
```

---

## Estrutura Proposta

```
server/src/shared/adapters/
├── index.ts                      # Factory e exports (~50 linhas)
├── adapter.interface.ts          # Interface comum (~80 linhas)
├── base.adapter.ts               # Implementação base (~300 linhas)
├── evolution/
│   ├── evolution.adapter.ts      # Específico Evolution (~250 linhas)
│   ├── evolution.parser.ts       # Parser de webhook (~150 linhas)
│   └── evolution.types.ts        # Tipos (~50 linhas)
├── uazapi/
│   ├── uazapi.adapter.ts         # Específico UAZAPI (~250 linhas)
│   ├── uazapi.parser.ts          # Parser de webhook (~150 linhas)
│   └── uazapi.types.ts           # Tipos (~50 linhas)
└── helpers/
    ├── media-fetcher.ts          # Fetch de mídia comum (~100 linhas)
    ├── message-normalizer.ts     # Normalização de mensagens (~80 linhas)
    └── status-mapper.ts          # Mapeamento de status (~50 linhas)
```

---

## Interface Comum

```typescript
// server/src/shared/adapters/adapter.interface.ts

export interface WhatsAppCredentials {
  apiUrl: string;
  apiKey: string;
  instanceToken?: string;
}

export interface InstanceConfig {
  name: string;
  webhookUrl: string;
  webhookEvents?: string[];
}

export interface SendMessageOptions {
  to: string;
  text: string;
  quotedMessageId?: string;
}

export interface SendMediaOptions {
  to: string;
  mediaUrl: string;
  mediaType: 'image' | 'video' | 'audio' | 'document';
  caption?: string;
  fileName?: string;
}

export interface WebhookPayload {
  event: string;
  instanceName: string;
  data: {
    messageId?: string;
    remoteJid?: string;
    fromMe?: boolean;
    pushName?: string;
    message?: {
      type: 'text' | 'image' | 'video' | 'audio' | 'document' | 'reaction';
      content?: string;
      mediaUrl?: string;
      caption?: string;
      quotedMessageId?: string;
      reaction?: { emoji: string; messageId: string };
    };
    status?: 'pending' | 'sent' | 'delivered' | 'read' | 'failed';
    timestamp?: number;
  };
}

export interface IWhatsAppAdapter {
  provider: 'evolution' | 'uazapi';

  // Instance Management
  createInstance(credentials: WhatsAppCredentials, config: InstanceConfig): Promise<{ instanceId: string }>;
  connectInstance(credentials: WhatsAppCredentials, instanceName: string): Promise<{ qrCode?: string; status: string }>;
  checkStatus(credentials: WhatsAppCredentials, instanceName: string): Promise<{ connected: boolean; status: string }>;
  deleteInstance(credentials: WhatsAppCredentials, instanceName: string): Promise<void>;

  // Messaging
  sendTextMessage(credentials: WhatsAppCredentials, instanceName: string, options: SendMessageOptions): Promise<{ messageId: string }>;
  sendMediaMessage(credentials: WhatsAppCredentials, instanceName: string, options: SendMediaOptions): Promise<{ messageId: string }>;
  sendReactionMessage(credentials: WhatsAppCredentials, instanceName: string, to: string, messageId: string, emoji: string): Promise<void>;

  // Media
  fetchMediaBase64(credentials: WhatsAppCredentials, mediaUrl: string): Promise<string>;
  fetchAudioBase64(credentials: WhatsAppCredentials, audioUrl: string): Promise<string>;
  fetchProfilePicture(credentials: WhatsAppCredentials, instanceName: string, phone: string): Promise<string | null>;

  // Webhook
  parseWebhookPayload(rawPayload: any): WebhookPayload | null;
}
```

---

## Base Adapter

```typescript
// server/src/shared/adapters/base.adapter.ts

import axios, { AxiosInstance } from 'axios';
import { IWhatsAppAdapter, WhatsAppCredentials, WebhookPayload } from './adapter.interface';

export abstract class BaseWhatsAppAdapter implements IWhatsAppAdapter {
  abstract provider: 'evolution' | 'uazapi';

  protected createClient(credentials: WhatsAppCredentials): AxiosInstance {
    return axios.create({
      baseURL: credentials.apiUrl,
      timeout: 30000,
      headers: this.getHeaders(credentials),
    });
  }

  protected abstract getHeaders(credentials: WhatsAppCredentials): Record<string, string>;

  // === Métodos comuns implementados ===

  async fetchMediaBase64(credentials: WhatsAppCredentials, mediaUrl: string): Promise<string> {
    const client = this.createClient(credentials);
    const response = await client.get(mediaUrl, { responseType: 'arraybuffer' });
    return Buffer.from(response.data).toString('base64');
  }

  async fetchAudioBase64(credentials: WhatsAppCredentials, audioUrl: string): Promise<string> {
    return this.fetchMediaBase64(credentials, audioUrl);
  }

  // === Métodos abstratos (específicos por provider) ===

  abstract createInstance(credentials: WhatsAppCredentials, config: InstanceConfig): Promise<{ instanceId: string }>;
  abstract connectInstance(credentials: WhatsAppCredentials, instanceName: string): Promise<{ qrCode?: string; status: string }>;
  abstract checkStatus(credentials: WhatsAppCredentials, instanceName: string): Promise<{ connected: boolean; status: string }>;
  abstract deleteInstance(credentials: WhatsAppCredentials, instanceName: string): Promise<void>;
  abstract sendTextMessage(credentials: WhatsAppCredentials, instanceName: string, options: SendMessageOptions): Promise<{ messageId: string }>;
  abstract sendMediaMessage(credentials: WhatsAppCredentials, instanceName: string, options: SendMediaOptions): Promise<{ messageId: string }>;
  abstract sendReactionMessage(credentials: WhatsAppCredentials, instanceName: string, to: string, messageId: string, emoji: string): Promise<void>;
  abstract fetchProfilePicture(credentials: WhatsAppCredentials, instanceName: string, phone: string): Promise<string | null>;
  abstract parseWebhookPayload(rawPayload: any): WebhookPayload | null;
}
```

---

## Evolution Adapter

```typescript
// server/src/shared/adapters/evolution/evolution.adapter.ts

import { BaseWhatsAppAdapter } from '../base.adapter';
import { WhatsAppCredentials, InstanceConfig, SendMessageOptions, SendMediaOptions, WebhookPayload } from '../adapter.interface';
import { parseEvolutionWebhook } from './evolution.parser';

export class EvolutionAdapter extends BaseWhatsAppAdapter {
  provider = 'evolution' as const;

  protected getHeaders(credentials: WhatsAppCredentials): Record<string, string> {
    return {
      'Content-Type': 'application/json',
      'apikey': credentials.apiKey,
    };
  }

  async createInstance(credentials: WhatsAppCredentials, config: InstanceConfig) {
    const client = this.createClient(credentials);

    const response = await client.post('/instance/create', {
      instanceName: config.name,
      integration: 'WHATSAPP-BAILEYS',
      qrcode: true,
    });

    // Configurar webhook
    await client.post(`/webhook/set/${config.name}`, {
      enabled: true,
      url: config.webhookUrl,
      webhookByEvents: true,
      events: config.webhookEvents || [
        'MESSAGES_UPSERT',
        'MESSAGES_UPDATE',
        'CONNECTION_UPDATE',
      ],
    });

    return { instanceId: response.data.instance?.instanceId || config.name };
  }

  async connectInstance(credentials: WhatsAppCredentials, instanceName: string) {
    const client = this.createClient(credentials);

    const response = await client.get(`/instance/connect/${instanceName}`);

    return {
      qrCode: response.data.base64,
      status: response.data.state || 'connecting',
    };
  }

  async checkStatus(credentials: WhatsAppCredentials, instanceName: string) {
    const client = this.createClient(credentials);

    const response = await client.get(`/instance/connectionState/${instanceName}`);
    const state = response.data.state || response.data.instance?.state;

    return {
      connected: state === 'open',
      status: state,
    };
  }

  async deleteInstance(credentials: WhatsAppCredentials, instanceName: string) {
    const client = this.createClient(credentials);
    await client.delete(`/instance/delete/${instanceName}`);
  }

  async sendTextMessage(credentials: WhatsAppCredentials, instanceName: string, options: SendMessageOptions) {
    const client = this.createClient(credentials);

    const response = await client.post(`/message/sendText/${instanceName}`, {
      number: options.to,
      text: options.text,
      ...(options.quotedMessageId && {
        quoted: { key: { id: options.quotedMessageId } },
      }),
    });

    return { messageId: response.data.key?.id };
  }

  async sendMediaMessage(credentials: WhatsAppCredentials, instanceName: string, options: SendMediaOptions) {
    const client = this.createClient(credentials);

    const endpoint = options.mediaType === 'audio'
      ? `/message/sendWhatsAppAudio/${instanceName}`
      : `/message/sendMedia/${instanceName}`;

    const response = await client.post(endpoint, {
      number: options.to,
      mediatype: options.mediaType,
      media: options.mediaUrl,
      caption: options.caption,
      fileName: options.fileName,
    });

    return { messageId: response.data.key?.id };
  }

  async sendReactionMessage(
    credentials: WhatsAppCredentials,
    instanceName: string,
    to: string,
    messageId: string,
    emoji: string
  ) {
    const client = this.createClient(credentials);

    await client.post(`/message/sendReaction/${instanceName}`, {
      key: { remoteJid: to, id: messageId },
      reaction: emoji,
    });
  }

  async fetchProfilePicture(credentials: WhatsAppCredentials, instanceName: string, phone: string) {
    const client = this.createClient(credentials);

    try {
      const response = await client.post(`/chat/fetchProfilePictureUrl/${instanceName}`, {
        number: phone,
      });
      return response.data.profilePictureUrl || null;
    } catch {
      return null;
    }
  }

  parseWebhookPayload(rawPayload: any): WebhookPayload | null {
    return parseEvolutionWebhook(rawPayload);
  }
}
```

---

## Evolution Parser

```typescript
// server/src/shared/adapters/evolution/evolution.parser.ts

import { WebhookPayload } from '../adapter.interface';
import { normalizeStatus } from '../helpers/status-mapper';

export function parseEvolutionWebhook(raw: any): WebhookPayload | null {
  if (!raw || !raw.event) return null;

  const event = raw.event;
  const instanceName = raw.instance || raw.instanceName;
  const data = raw.data || {};

  // Message received
  if (event === 'messages.upsert' || event === 'MESSAGES_UPSERT') {
    const message = data.message || data;
    const key = message.key || {};

    return {
      event: 'message',
      instanceName,
      data: {
        messageId: key.id,
        remoteJid: key.remoteJid,
        fromMe: key.fromMe || false,
        pushName: message.pushName || data.pushName,
        message: parseMessageContent(message),
        timestamp: message.messageTimestamp || Date.now(),
      },
    };
  }

  // Status update
  if (event === 'messages.update' || event === 'MESSAGES_UPDATE') {
    const update = Array.isArray(data) ? data[0] : data;
    const key = update.key || {};

    return {
      event: 'status',
      instanceName,
      data: {
        messageId: key.id,
        remoteJid: key.remoteJid,
        status: normalizeStatus(update.status || update.update?.status),
      },
    };
  }

  // Connection update
  if (event === 'connection.update' || event === 'CONNECTION_UPDATE') {
    return {
      event: 'connection',
      instanceName,
      data: {
        status: data.state || data.connection,
      },
    };
  }

  return null;
}

function parseMessageContent(message: any) {
  const msg = message.message || message;

  if (msg.conversation || msg.extendedTextMessage) {
    return {
      type: 'text' as const,
      content: msg.conversation || msg.extendedTextMessage?.text,
    };
  }

  if (msg.imageMessage) {
    return {
      type: 'image' as const,
      mediaUrl: msg.imageMessage.url,
      caption: msg.imageMessage.caption,
    };
  }

  if (msg.videoMessage) {
    return {
      type: 'video' as const,
      mediaUrl: msg.videoMessage.url,
      caption: msg.videoMessage.caption,
    };
  }

  if (msg.audioMessage) {
    return {
      type: 'audio' as const,
      mediaUrl: msg.audioMessage.url,
    };
  }

  if (msg.documentMessage) {
    return {
      type: 'document' as const,
      mediaUrl: msg.documentMessage.url,
      caption: msg.documentMessage.fileName,
    };
  }

  if (msg.reactionMessage) {
    return {
      type: 'reaction' as const,
      reaction: {
        emoji: msg.reactionMessage.text,
        messageId: msg.reactionMessage.key?.id,
      },
    };
  }

  return { type: 'text' as const, content: '' };
}
```

---

## Factory

```typescript
// server/src/shared/adapters/index.ts

import { IWhatsAppAdapter } from './adapter.interface';
import { EvolutionAdapter } from './evolution/evolution.adapter';
import { UazapiAdapter } from './uazapi/uazapi.adapter';

const adapters: Record<string, IWhatsAppAdapter> = {
  evolution: new EvolutionAdapter(),
  uazapi: new UazapiAdapter(),
};

export function getAdapter(provider: 'evolution' | 'uazapi'): IWhatsAppAdapter {
  const adapter = adapters[provider];
  if (!adapter) {
    throw new Error(`Unknown provider: ${provider}`);
  }
  return adapter;
}

export * from './adapter.interface';
export { EvolutionAdapter } from './evolution/evolution.adapter';
export { UazapiAdapter } from './uazapi/uazapi.adapter';
```

---

## Checklist

### Estrutura
- [ ] Criar `adapter.interface.ts`
- [ ] Criar `base.adapter.ts`
- [ ] Criar pasta `evolution/`
- [ ] Criar pasta `uazapi/`
- [ ] Criar pasta `helpers/`

### Evolution
- [ ] `evolution/evolution.adapter.ts`
- [ ] `evolution/evolution.parser.ts`
- [ ] `evolution/evolution.types.ts`

### UAZAPI
- [ ] `uazapi/uazapi.adapter.ts`
- [ ] `uazapi/uazapi.parser.ts`
- [ ] `uazapi/uazapi.types.ts`

### Helpers
- [ ] `helpers/media-fetcher.ts`
- [ ] `helpers/message-normalizer.ts`
- [ ] `helpers/status-mapper.ts`

### Finalização
- [ ] Atualizar `index.ts`
- [ ] Remover arquivos antigos
- [ ] Testes

---

## Métricas Esperadas

| Métrica | Antes | Depois | Melhoria |
|---------|-------|--------|----------|
| Total linhas | 1.800 | ~1.200 | 33%↓ |
| Duplicação | 70% | 0% | 100%↓ |
| Maior arquivo | 901 | ~300 | 67%↓ |

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
