# Refatoração: API Client Frontend

**Arquivo:** `src/lib/api.ts`
**Linhas:** 2.037
**Prioridade:** P0 - CRÍTICO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 2.037 | 300 | 🔴 CRÍTICO |
| Exports | 74 | 15 | 🔴 CRÍTICO |
| Types/Interfaces | 57 | 20 | 🔴 CRÍTICO |

### Problema Principal

Arquivo monolítico com:
- 74 funções de API exportadas
- 57 interfaces/types
- Sem organização por domínio
- Difícil manutenção e tree-shaking

```typescript
// ATUAL: Tudo em um arquivo
export async function login(...) { }
export async function getContacts(...) { }
export async function createContact(...) { }
export async function getFunnels(...) { }
export async function getDeals(...) { }
export async function sendMessage(...) { }
// ... mais 68 funções

export interface Contact { }
export interface Deal { }
export interface Funnel { }
// ... mais 54 tipos
```

---

## Estrutura Proposta

```
src/lib/api/
├── index.ts              # Re-exports públicos (~50 linhas)
├── client.ts             # Axios instance + interceptors (~100 linhas)
├── types/
│   ├── index.ts          # Re-exports de tipos
│   ├── common.types.ts   # Tipos compartilhados (~100 linhas)
│   ├── auth.types.ts     # Tipos de auth (~50 linhas)
│   ├── contact.types.ts  # Tipos de contatos (~80 linhas)
│   ├── conversation.types.ts
│   ├── deal.types.ts
│   ├── funnel.types.ts
│   ├── flow.types.ts
│   ├── followup.types.ts
│   └── message.types.ts
├── auth.ts               # API de autenticação (~100 linhas)
├── contacts.ts           # API de contatos (~150 linhas)
├── conversations.ts      # API de conversas (~150 linhas)
├── deals.ts              # API de deals (~150 linhas)
├── funnels.ts            # API de funis (~100 linhas)
├── flows.ts              # API de flows (~100 linhas)
├── followups.ts          # API de followups (~100 linhas)
├── messages.ts           # API de mensagens (~100 linhas)
├── whatsapp.ts           # API de WhatsApp (~150 linhas)
├── dashboard.ts          # API de dashboard (~80 linhas)
└── admin.ts              # API administrativa (~100 linhas)
```

---

## Código: Client Base

```typescript
// src/lib/api/client.ts

import axios, { AxiosInstance, AxiosError, InternalAxiosRequestConfig } from 'axios';

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000';

// Criar instância
export const apiClient: AxiosInstance = axios.create({
  baseURL: API_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor - adicionar token
apiClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = localStorage.getItem('token');
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor - tratar erros
apiClient.interceptors.response.use(
  (response) => response,
  (error: AxiosError) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

// Helper para extrair data
export function extractData<T>(response: { data: T }): T {
  return response.data;
}

// Tipos de resposta padrão
export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export interface ApiError {
  success: false;
  error: string;
  details?: Array<{ field: string; message: string }>;
}
```

---

## Código: Tipos Compartilhados

```typescript
// src/lib/api/types/common.types.ts

export interface PaginationParams {
  page?: number;
  limit?: number;
  offset?: number;
}

export interface SortParams {
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}

export interface SearchParams {
  search?: string;
  q?: string;
}

export interface DateRangeParams {
  startDate?: string;
  endDate?: string;
}

export type BaseFilterParams = PaginationParams & SortParams & SearchParams & DateRangeParams;

// Entidades base
export interface BaseEntity {
  id: string;
  createdAt: string;
  updatedAt: string;
}

export interface CompanyEntity extends BaseEntity {
  companyId: string;
}
```

```typescript
// src/lib/api/types/contact.types.ts

import { BaseEntity, CompanyEntity } from './common.types';

export interface Tag extends BaseEntity {
  name: string;
  color: string;
}

export interface Contact extends CompanyEntity {
  name: string;
  phone: string;
  email?: string;
  notes?: string;
  sentiment?: 'positive' | 'neutral' | 'negative';
  tags: Tag[];
  customFields?: Record<string, any>;
}

export interface CreateContactParams {
  name: string;
  phone: string;
  email?: string;
  notes?: string;
}

export interface UpdateContactParams {
  name?: string;
  email?: string | null;
  notes?: string | null;
}

export interface ContactFilters {
  search?: string;
  tags?: string[];
  funnelId?: string;
  stageId?: string;
  sentiment?: 'positive' | 'neutral' | 'negative';
  hasDeals?: boolean;
}
```

---

## Código: APIs por Domínio

### Auth API

```typescript
// src/lib/api/auth.ts

import { apiClient, ApiResponse } from './client';

export interface User {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'user' | 'viewer';
  companyId: string;
}

export interface LoginParams {
  email: string;
  password: string;
}

export interface LoginResponse {
  token: string;
  user: User;
}

export interface RegisterParams {
  name: string;
  email: string;
  password: string;
  companyName?: string;
}

export const authApi = {
  async login(params: LoginParams): Promise<LoginResponse> {
    const { data } = await apiClient.post<ApiResponse<LoginResponse>>('/auth/login', params);
    return data.data;
  },

  async register(params: RegisterParams): Promise<LoginResponse> {
    const { data } = await apiClient.post<ApiResponse<LoginResponse>>('/auth/register', params);
    return data.data;
  },

  async logout(): Promise<void> {
    await apiClient.post('/auth/logout');
    localStorage.removeItem('token');
  },

  async me(): Promise<User> {
    const { data } = await apiClient.get<ApiResponse<User>>('/auth/me');
    return data.data;
  },

  async refreshToken(): Promise<{ token: string }> {
    const { data } = await apiClient.post<ApiResponse<{ token: string }>>('/auth/refresh');
    return data.data;
  },
};
```

### Contacts API

```typescript
// src/lib/api/contacts.ts

import { apiClient, ApiResponse, PaginatedResponse } from './client';
import type { Contact, CreateContactParams, UpdateContactParams, ContactFilters, Tag } from './types';

export const contactsApi = {
  // === CRUD ===
  async list(filters?: ContactFilters & { page?: number; limit?: number }): Promise<PaginatedResponse<Contact>> {
    const { data } = await apiClient.get<PaginatedResponse<Contact>>('/contacts', { params: filters });
    return data;
  },

  async getById(contactId: string): Promise<Contact> {
    const { data } = await apiClient.get<ApiResponse<Contact>>(`/contacts/${contactId}`);
    return data.data;
  },

  async create(params: CreateContactParams): Promise<Contact> {
    const { data } = await apiClient.post<ApiResponse<Contact>>('/contacts', params);
    return data.data;
  },

  async update(contactId: string, params: UpdateContactParams): Promise<Contact> {
    const { data } = await apiClient.patch<ApiResponse<Contact>>(`/contacts/${contactId}`, params);
    return data.data;
  },

  async delete(contactId: string): Promise<void> {
    await apiClient.delete(`/contacts/${contactId}`);
  },

  // === Tags ===
  async assignTags(contactId: string, tagIds: string[]): Promise<Contact> {
    const { data } = await apiClient.post<ApiResponse<Contact>>(`/contacts/${contactId}/tags`, { tagIds });
    return data.data;
  },

  async removeTags(contactId: string, tagIds: string[]): Promise<Contact> {
    const { data } = await apiClient.delete<ApiResponse<Contact>>(`/contacts/${contactId}/tags`, {
      data: { tagIds },
    });
    return data.data;
  },

  // === Notes ===
  async addNote(contactId: string, content: string): Promise<void> {
    await apiClient.post(`/contacts/${contactId}/notes`, { content });
  },

  // === Search ===
  async search(query: string, limit = 10): Promise<Contact[]> {
    const { data } = await apiClient.get<ApiResponse<Contact[]>>('/contacts/search', {
      params: { q: query, limit },
    });
    return data.data;
  },
};

// === Tags API ===
export const tagsApi = {
  async list(): Promise<Tag[]> {
    const { data } = await apiClient.get<ApiResponse<Tag[]>>('/tags');
    return data.data;
  },

  async create(name: string, color: string): Promise<Tag> {
    const { data } = await apiClient.post<ApiResponse<Tag>>('/tags', { name, color });
    return data.data;
  },

  async update(tagId: string, params: { name?: string; color?: string }): Promise<Tag> {
    const { data } = await apiClient.patch<ApiResponse<Tag>>(`/tags/${tagId}`, params);
    return data.data;
  },

  async delete(tagId: string): Promise<void> {
    await apiClient.delete(`/tags/${tagId}`);
  },
};
```

### Messages API

```typescript
// src/lib/api/messages.ts

import { apiClient, ApiResponse, PaginatedResponse } from './client';

export interface Message {
  id: string;
  conversationId: string;
  content: string;
  type: 'text' | 'image' | 'video' | 'audio' | 'document';
  fromMe: boolean;
  status: 'pending' | 'sent' | 'delivered' | 'read' | 'failed';
  mediaUrl?: string;
  quotedMessageId?: string;
  createdAt: string;
}

export interface SendMessageParams {
  to: string;
  text: string;
  quotedMessageId?: string;
}

export interface SendMediaParams {
  to: string;
  mediaUrl: string;
  mediaType: 'image' | 'video' | 'audio' | 'document';
  caption?: string;
  fileName?: string;
}

export const messagesApi = {
  async getByConversation(
    conversationId: string,
    params?: { page?: number; limit?: number }
  ): Promise<PaginatedResponse<Message>> {
    const { data } = await apiClient.get<PaginatedResponse<Message>>(
      `/conversations/${conversationId}/messages`,
      { params }
    );
    return data;
  },

  async send(instanceId: string, params: SendMessageParams): Promise<Message> {
    const { data } = await apiClient.post<ApiResponse<Message>>(
      `/whatsapp/instances/${instanceId}/messages`,
      params
    );
    return data.data;
  },

  async sendMedia(instanceId: string, params: SendMediaParams): Promise<Message> {
    const { data } = await apiClient.post<ApiResponse<Message>>(
      `/whatsapp/instances/${instanceId}/messages/media`,
      params
    );
    return data.data;
  },

  async sendReaction(
    instanceId: string,
    params: { to: string; messageId: string; emoji: string }
  ): Promise<void> {
    await apiClient.post(`/whatsapp/instances/${instanceId}/messages/reaction`, params);
  },

  async markAsRead(conversationId: string): Promise<void> {
    await apiClient.post(`/conversations/${conversationId}/read`);
  },
};
```

### Deals API

```typescript
// src/lib/api/deals.ts

import { apiClient, ApiResponse, PaginatedResponse } from './client';
import type { Contact } from './types';

export interface FunnelStage {
  id: string;
  name: string;
  color: string;
  order: number;
}

export interface Funnel {
  id: string;
  name: string;
  stages: FunnelStage[];
}

export interface Deal {
  id: string;
  title: string;
  value: number;
  status: 'open' | 'won' | 'lost';
  contactId: string;
  contact?: Contact;
  funnelId: string;
  funnel?: Funnel;
  stageId: string;
  stage?: FunnelStage;
  expectedCloseDate?: string;
  createdAt: string;
  updatedAt: string;
}

export interface DealFilters {
  funnelId?: string;
  stageId?: string;
  contactId?: string;
  status?: 'open' | 'won' | 'lost';
  minValue?: number;
  maxValue?: number;
}

export interface CreateDealParams {
  contactId: string;
  funnelId: string;
  stageId?: string;
  title?: string;
  value?: number;
}

export const dealsApi = {
  async list(filters?: DealFilters & { page?: number; limit?: number }): Promise<PaginatedResponse<Deal>> {
    const { data } = await apiClient.get<PaginatedResponse<Deal>>('/deals', { params: filters });
    return data;
  },

  async getById(dealId: string): Promise<Deal> {
    const { data } = await apiClient.get<ApiResponse<Deal>>(`/deals/${dealId}`);
    return data.data;
  },

  async create(params: CreateDealParams): Promise<Deal> {
    const { data } = await apiClient.post<ApiResponse<Deal>>('/deals', params);
    return data.data;
  },

  async update(dealId: string, params: Partial<CreateDealParams>): Promise<Deal> {
    const { data } = await apiClient.patch<ApiResponse<Deal>>(`/deals/${dealId}`, params);
    return data.data;
  },

  async moveToStage(dealId: string, stageId: string, reason?: string): Promise<Deal> {
    const { data } = await apiClient.post<ApiResponse<Deal>>(`/deals/${dealId}/move`, { stageId, reason });
    return data.data;
  },

  async delete(dealId: string): Promise<void> {
    await apiClient.delete(`/deals/${dealId}`);
  },

  async addNote(dealId: string, content: string, type = 'note'): Promise<void> {
    await apiClient.post(`/deals/${dealId}/notes`, { content, type });
  },

  async getNotes(dealId: string): Promise<any[]> {
    const { data } = await apiClient.get<ApiResponse<any[]>>(`/deals/${dealId}/notes`);
    return data.data;
  },
};

export const funnelsApi = {
  async list(): Promise<Funnel[]> {
    const { data } = await apiClient.get<ApiResponse<Funnel[]>>('/funnels');
    return data.data;
  },

  async getById(funnelId: string): Promise<Funnel> {
    const { data } = await apiClient.get<ApiResponse<Funnel>>(`/funnels/${funnelId}`);
    return data.data;
  },

  async create(name: string): Promise<Funnel> {
    const { data } = await apiClient.post<ApiResponse<Funnel>>('/funnels', { name });
    return data.data;
  },

  async addStage(funnelId: string, name: string, color: string): Promise<FunnelStage> {
    const { data } = await apiClient.post<ApiResponse<FunnelStage>>(`/funnels/${funnelId}/stages`, {
      name,
      color,
    });
    return data.data;
  },
};
```

---

## Index (Re-exports)

```typescript
// src/lib/api/index.ts

// Client
export { apiClient, type ApiResponse, type PaginatedResponse, type ApiError } from './client';

// APIs
export { authApi } from './auth';
export { contactsApi, tagsApi } from './contacts';
export { messagesApi } from './messages';
export { dealsApi, funnelsApi } from './deals';
export { conversationsApi } from './conversations';
export { flowsApi } from './flows';
export { followupsApi } from './followups';
export { whatsappApi } from './whatsapp';
export { dashboardApi } from './dashboard';
export { adminApi } from './admin';

// Types
export * from './types';

// Compatibilidade (deprecated - remover após migração)
// export { login, logout, getContacts, ... } from './legacy';
```

---

## Migração de Uso

### BEFORE

```typescript
// src/pages/Contacts.tsx - ANTES
import { getContacts, createContact, Contact } from '@/lib/api';

const contacts = await getContacts({ search: 'John' });
const newContact = await createContact({ name: 'John', phone: '+55...' });
```

### AFTER

```typescript
// src/pages/Contacts.tsx - DEPOIS
import { contactsApi, type Contact } from '@/lib/api';

const contacts = await contactsApi.list({ search: 'John' });
const newContact = await contactsApi.create({ name: 'John', phone: '+55...' });
```

---

## Checklist

### Estrutura
- [ ] Criar pasta `src/lib/api/`
- [ ] Criar `client.ts`
- [ ] Criar `types/common.types.ts`
- [ ] Criar `types/index.ts`

### APIs (um por vez)
- [ ] `auth.ts`
- [ ] `contacts.ts` + `types/contact.types.ts`
- [ ] `messages.ts` + `types/message.types.ts`
- [ ] `conversations.ts` + `types/conversation.types.ts`
- [ ] `deals.ts` + `types/deal.types.ts`
- [ ] `funnels.ts` + `types/funnel.types.ts`
- [ ] `flows.ts` + `types/flow.types.ts`
- [ ] `followups.ts` + `types/followup.types.ts`
- [ ] `whatsapp.ts` + `types/whatsapp.types.ts`
- [ ] `dashboard.ts`
- [ ] `admin.ts`

### Finalização
- [ ] Criar `index.ts` com re-exports
- [ ] Atualizar imports em componentes
- [ ] Remover arquivo `api.ts` antigo
- [ ] Testes

---

## Benefícios

| Aspecto | Antes | Depois |
|---------|-------|--------|
| Tree-shaking | Impossível | Funcional |
| Organização | Monolítico | Por domínio |
| Manutenção | Buscar em 2.000 linhas | Ir direto ao arquivo |
| Autocomplete | Lento | Rápido |
| Bundle size | 100% sempre | Apenas o usado |

---

**Status:** ✅ Completo
**Última Atualização:** 2026-01-23
