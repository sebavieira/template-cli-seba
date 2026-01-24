# Refatoração: Messages Page

**Arquivo:** `src/pages/Messages.tsx`
**Linhas:** 1.699
**Prioridade:** P0 - CRÍTICO
**Estimativa:** 2-3 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 1.699 | 300 | 🔴 CRÍTICO |
| Hooks | 81 | 10 | 🔴 CRÍTICO |
| Responsabilidades | 8+ | 1 | 🔴 CRÍTICO |

### O Problema: 81 Hooks em Um Componente

```typescript
// ATUAL: Hooks espalhados
function MessagesPage() {
  // State hooks (20+)
  const [conversations, setConversations] = useState([]);
  const [selectedConversation, setSelectedConversation] = useState(null);
  const [messages, setMessages] = useState([]);
  const [search, setSearch] = useState('');
  const [filters, setFilters] = useState({});
  const [isLoading, setIsLoading] = useState(false);
  // ... mais 14 useState

  // Query hooks (15+)
  const { data: conversationsData } = useQuery(...);
  const { data: messagesData } = useQuery(...);
  const { data: contactData } = useQuery(...);
  // ... mais 12 useQuery

  // Mutation hooks (10+)
  const sendMessageMutation = useMutation(...);
  const markAsReadMutation = useMutation(...);
  // ... mais 8 useMutation

  // Effect hooks (15+)
  useEffect(() => { /* fetch conversations */ }, [...]);
  useEffect(() => { /* fetch messages */ }, [...]);
  useEffect(() => { /* websocket */ }, [...]);
  // ... mais 12 useEffect

  // Callback hooks (20+)
  const handleSendMessage = useCallback(...);
  const handleSelectConversation = useCallback(...);
  // ... mais 18 useCallback

  // Memo hooks (5+)
  const filteredConversations = useMemo(...);
  // ...

  // 1.000+ linhas de JSX
  return (
    <div>...</div>
  );
}
```

### Responsabilidades Misturadas

1. **Lista de conversas** - Fetch, filtros, busca, paginação
2. **Thread de mensagens** - Fetch, scroll, virtualização
3. **Envio de mensagens** - Texto, mídia, reações
4. **WebSocket** - Conexão, eventos, reconexão
5. **Estado UI** - Modais, drawers, loading
6. **Filtros** - Tags, status, atribuição
7. **Ações em lote** - Arquivar, pausar, deletar
8. **Integração CRM** - Sidebar de contato

---

## Estrutura Proposta

```
src/pages/Messages/
├── index.tsx                    # Container principal (~100 linhas)
├── MessagesPage.tsx             # Layout e composição (~150 linhas)
├── hooks/
│   ├── index.ts                 # Re-exports
│   ├── useConversations.ts      # Lista de conversas (~150 linhas)
│   ├── useMessages.ts           # Thread de mensagens (~120 linhas)
│   ├── useMessageActions.ts     # Envio/reações (~100 linhas)
│   ├── useConversationFilters.ts # Filtros (~80 linhas)
│   ├── useWebSocketMessages.ts  # WebSocket (~100 linhas)
│   └── useConversationActions.ts # Ações em lote (~80 linhas)
├── components/
│   ├── ConversationList/
│   │   ├── index.tsx            # Lista (~150 linhas)
│   │   ├── ConversationItem.tsx # Item individual (~80 linhas)
│   │   └── ConversationFilters.tsx # Filtros UI (~100 linhas)
│   ├── MessageThread/
│   │   ├── index.tsx            # Thread (~150 linhas)
│   │   ├── MessageBubble.tsx    # Bolha (~100 linhas)
│   │   ├── MessageInput.tsx     # Input (~120 linhas)
│   │   └── MessageMedia.tsx     # Mídia (~80 linhas)
│   └── ContactSidebar/
│       └── index.tsx            # Sidebar CRM (~150 linhas)
└── types.ts                     # Tipos locais (~50 linhas)
```

---

## Hooks Customizados

### useConversations

```typescript
// src/pages/Messages/hooks/useConversations.ts

import { useQuery, useQueryClient } from '@tanstack/react-query';
import { conversationsApi } from '@/lib/api';
import type { Conversation, ConversationFilters } from '../types';

interface UseConversationsOptions {
  filters?: ConversationFilters;
  enabled?: boolean;
}

interface UseConversationsReturn {
  conversations: Conversation[];
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
  hasMore: boolean;
  loadMore: () => void;
  total: number;
}

export function useConversations(options: UseConversationsOptions = {}): UseConversationsReturn {
  const { filters = {}, enabled = true } = options;
  const queryClient = useQueryClient();

  const {
    data,
    isLoading,
    error,
    refetch,
    fetchNextPage,
    hasNextPage,
  } = useInfiniteQuery({
    queryKey: ['conversations', filters],
    queryFn: ({ pageParam = 1 }) =>
      conversationsApi.list({ ...filters, page: pageParam, limit: 20 }),
    getNextPageParam: (lastPage) =>
      lastPage.page < lastPage.totalPages ? lastPage.page + 1 : undefined,
    enabled,
    staleTime: 30000, // 30 segundos
  });

  const conversations = data?.pages.flatMap((page) => page.data) ?? [];
  const total = data?.pages[0]?.total ?? 0;

  const loadMore = () => {
    if (hasNextPage) {
      fetchNextPage();
    }
  };

  // Invalidar ao receber nova mensagem (chamado pelo WebSocket hook)
  const invalidate = () => {
    queryClient.invalidateQueries({ queryKey: ['conversations'] });
  };

  return {
    conversations,
    isLoading,
    error: error as Error | null,
    refetch,
    hasMore: !!hasNextPage,
    loadMore,
    total,
  };
}
```

### useMessages

```typescript
// src/pages/Messages/hooks/useMessages.ts

import { useQuery, useQueryClient } from '@tanstack/react-query';
import { messagesApi } from '@/lib/api';
import type { Message } from '../types';

interface UseMessagesOptions {
  conversationId: string | null;
  enabled?: boolean;
}

interface UseMessagesReturn {
  messages: Message[];
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
  hasMore: boolean;
  loadMore: () => void;
  addOptimisticMessage: (message: Partial<Message>) => void;
}

export function useMessages({ conversationId, enabled = true }: UseMessagesOptions): UseMessagesReturn {
  const queryClient = useQueryClient();

  const {
    data,
    isLoading,
    error,
    refetch,
    fetchNextPage,
    hasNextPage,
  } = useInfiniteQuery({
    queryKey: ['messages', conversationId],
    queryFn: ({ pageParam = 1 }) =>
      messagesApi.getByConversation(conversationId!, { page: pageParam, limit: 50 }),
    getNextPageParam: (lastPage) =>
      lastPage.page < lastPage.totalPages ? lastPage.page + 1 : undefined,
    enabled: enabled && !!conversationId,
    staleTime: 10000, // 10 segundos
  });

  const messages = data?.pages.flatMap((page) => page.data).reverse() ?? [];

  const loadMore = () => {
    if (hasNextPage) {
      fetchNextPage();
    }
  };

  // Adicionar mensagem otimista (antes da confirmação do servidor)
  const addOptimisticMessage = (message: Partial<Message>) => {
    queryClient.setQueryData(['messages', conversationId], (old: any) => {
      if (!old) return old;
      const optimistic = {
        id: `temp-${Date.now()}`,
        status: 'pending',
        createdAt: new Date().toISOString(),
        ...message,
      };
      return {
        ...old,
        pages: old.pages.map((page: any, index: number) =>
          index === 0 ? { ...page, data: [optimistic, ...page.data] } : page
        ),
      };
    });
  };

  return {
    messages,
    isLoading,
    error: error as Error | null,
    refetch,
    hasMore: !!hasNextPage,
    loadMore,
    addOptimisticMessage,
  };
}
```

### useMessageActions

```typescript
// src/pages/Messages/hooks/useMessageActions.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { messagesApi } from '@/lib/api';
import { useToast } from '@/hooks/use-toast';

interface UseMessageActionsOptions {
  instanceId: string | null;
  conversationId: string | null;
  onMessageSent?: () => void;
}

export function useMessageActions({
  instanceId,
  conversationId,
  onMessageSent,
}: UseMessageActionsOptions) {
  const queryClient = useQueryClient();
  const { toast } = useToast();

  const sendTextMutation = useMutation({
    mutationFn: (params: { to: string; text: string; quotedMessageId?: string }) =>
      messagesApi.send(instanceId!, params),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', conversationId] });
      queryClient.invalidateQueries({ queryKey: ['conversations'] });
      onMessageSent?.();
    },
    onError: (error) => {
      toast({
        title: 'Erro ao enviar mensagem',
        description: error.message,
        variant: 'destructive',
      });
    },
  });

  const sendMediaMutation = useMutation({
    mutationFn: (params: {
      to: string;
      mediaUrl: string;
      mediaType: 'image' | 'video' | 'audio' | 'document';
      caption?: string;
    }) => messagesApi.sendMedia(instanceId!, params),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', conversationId] });
      onMessageSent?.();
    },
    onError: (error) => {
      toast({
        title: 'Erro ao enviar mídia',
        description: error.message,
        variant: 'destructive',
      });
    },
  });

  const sendReactionMutation = useMutation({
    mutationFn: (params: { to: string; messageId: string; emoji: string }) =>
      messagesApi.sendReaction(instanceId!, params),
  });

  const markAsReadMutation = useMutation({
    mutationFn: () => messagesApi.markAsRead(conversationId!),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['conversations'] });
    },
  });

  return {
    sendText: sendTextMutation.mutate,
    sendMedia: sendMediaMutation.mutate,
    sendReaction: sendReactionMutation.mutate,
    markAsRead: markAsReadMutation.mutate,
    isSending: sendTextMutation.isPending || sendMediaMutation.isPending,
  };
}
```

### useWebSocketMessages

```typescript
// src/pages/Messages/hooks/useWebSocketMessages.ts

import { useEffect, useCallback } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { useWebSocket } from '@/contexts/WebSocketContext';
import type { Message, Conversation } from '../types';

interface UseWebSocketMessagesOptions {
  conversationId: string | null;
  onNewMessage?: (message: Message) => void;
  onStatusUpdate?: (messageId: string, status: string) => void;
}

export function useWebSocketMessages({
  conversationId,
  onNewMessage,
  onStatusUpdate,
}: UseWebSocketMessagesOptions) {
  const { socket, isConnected } = useWebSocket();
  const queryClient = useQueryClient();

  const handleNewMessage = useCallback(
    (data: { message: Message; conversationId: string }) => {
      // Atualizar cache de mensagens
      if (data.conversationId === conversationId) {
        queryClient.setQueryData(['messages', conversationId], (old: any) => {
          if (!old) return old;
          return {
            ...old,
            pages: old.pages.map((page: any, index: number) =>
              index === 0
                ? { ...page, data: [data.message, ...page.data.filter((m: Message) => m.id !== data.message.id)] }
                : page
            ),
          };
        });
        onNewMessage?.(data.message);
      }

      // Atualizar lista de conversas
      queryClient.invalidateQueries({ queryKey: ['conversations'] });
    },
    [conversationId, queryClient, onNewMessage]
  );

  const handleStatusUpdate = useCallback(
    (data: { messageId: string; status: string }) => {
      queryClient.setQueryData(['messages', conversationId], (old: any) => {
        if (!old) return old;
        return {
          ...old,
          pages: old.pages.map((page: any) => ({
            ...page,
            data: page.data.map((m: Message) =>
              m.id === data.messageId ? { ...m, status: data.status } : m
            ),
          })),
        };
      });
      onStatusUpdate?.(data.messageId, data.status);
    },
    [conversationId, queryClient, onStatusUpdate]
  );

  useEffect(() => {
    if (!socket || !isConnected) return;

    socket.on('message:new', handleNewMessage);
    socket.on('message:status', handleStatusUpdate);

    return () => {
      socket.off('message:new', handleNewMessage);
      socket.off('message:status', handleStatusUpdate);
    };
  }, [socket, isConnected, handleNewMessage, handleStatusUpdate]);

  return { isConnected };
}
```

### useConversationFilters

```typescript
// src/pages/Messages/hooks/useConversationFilters.ts

import { useState, useCallback, useMemo } from 'react';
import { useSearchParams } from 'react-router-dom';

export interface ConversationFilters {
  search?: string;
  status?: 'open' | 'closed' | 'pending';
  assignedTo?: string;
  paused?: boolean;
  archived?: boolean;
  tags?: string[];
  funnelId?: string;
  stageId?: string;
}

export function useConversationFilters() {
  const [searchParams, setSearchParams] = useSearchParams();

  // Parse filters from URL
  const filters: ConversationFilters = useMemo(() => ({
    search: searchParams.get('search') || undefined,
    status: searchParams.get('status') as any || undefined,
    assignedTo: searchParams.get('assignedTo') || undefined,
    paused: searchParams.get('paused') === 'true' ? true : undefined,
    archived: searchParams.get('archived') === 'true' ? true : undefined,
    tags: searchParams.get('tags')?.split(',').filter(Boolean) || undefined,
    funnelId: searchParams.get('funnelId') || undefined,
    stageId: searchParams.get('stageId') || undefined,
  }), [searchParams]);

  const setFilter = useCallback((key: keyof ConversationFilters, value: any) => {
    setSearchParams((prev) => {
      const next = new URLSearchParams(prev);
      if (value === undefined || value === null || value === '' || (Array.isArray(value) && value.length === 0)) {
        next.delete(key);
      } else if (Array.isArray(value)) {
        next.set(key, value.join(','));
      } else {
        next.set(key, String(value));
      }
      return next;
    });
  }, [setSearchParams]);

  const clearFilters = useCallback(() => {
    setSearchParams({});
  }, [setSearchParams]);

  const hasActiveFilters = useMemo(() => {
    return Object.values(filters).some((v) => v !== undefined);
  }, [filters]);

  return {
    filters,
    setFilter,
    clearFilters,
    hasActiveFilters,
  };
}
```

---

## Componentes

### MessagesPage (Container)

```typescript
// src/pages/Messages/MessagesPage.tsx

import { useState } from 'react';
import { ConversationList } from './components/ConversationList';
import { MessageThread } from './components/MessageThread';
import { ContactSidebar } from './components/ContactSidebar';
import {
  useConversations,
  useMessages,
  useMessageActions,
  useConversationFilters,
  useWebSocketMessages,
} from './hooks';

export function MessagesPage() {
  const [selectedId, setSelectedId] = useState<string | null>(null);
  const [showSidebar, setShowSidebar] = useState(true);

  // Hooks compostos
  const { filters, setFilter, clearFilters } = useConversationFilters();
  const { conversations, isLoading: loadingConversations, loadMore } = useConversations({ filters });
  const { messages, isLoading: loadingMessages, addOptimisticMessage } = useMessages({
    conversationId: selectedId,
  });

  const selectedConversation = conversations.find((c) => c.id === selectedId);

  const { sendText, sendMedia, markAsRead, isSending } = useMessageActions({
    instanceId: selectedConversation?.instanceId ?? null,
    conversationId: selectedId,
    onMessageSent: () => {
      // Scroll to bottom handled by MessageThread
    },
  });

  // WebSocket
  useWebSocketMessages({
    conversationId: selectedId,
    onNewMessage: (msg) => {
      if (!msg.fromMe) {
        // Play notification sound
      }
    },
  });

  const handleSelectConversation = (id: string) => {
    setSelectedId(id);
    markAsRead();
  };

  const handleSendMessage = (text: string, quotedMessageId?: string) => {
    if (!selectedConversation) return;

    // Optimistic update
    addOptimisticMessage({
      content: text,
      fromMe: true,
      type: 'text',
    });

    sendText({
      to: selectedConversation.contact.phone,
      text,
      quotedMessageId,
    });
  };

  return (
    <div className="flex h-screen">
      {/* Lista de conversas */}
      <ConversationList
        conversations={conversations}
        selectedId={selectedId}
        onSelect={handleSelectConversation}
        isLoading={loadingConversations}
        onLoadMore={loadMore}
        filters={filters}
        onFilterChange={setFilter}
      />

      {/* Thread de mensagens */}
      <MessageThread
        conversation={selectedConversation}
        messages={messages}
        isLoading={loadingMessages}
        isSending={isSending}
        onSendMessage={handleSendMessage}
        onSendMedia={sendMedia}
      />

      {/* Sidebar do contato */}
      {showSidebar && selectedConversation && (
        <ContactSidebar
          contact={selectedConversation.contact}
          onClose={() => setShowSidebar(false)}
        />
      )}
    </div>
  );
}
```

---

## Migração Gradual

### Fase 1: Criar hooks (não quebra nada)
```
1. Criar pasta src/pages/Messages/hooks/
2. Criar cada hook individualmente
3. Testar hooks isoladamente
```

### Fase 2: Criar componentes (não quebra nada)
```
1. Criar pasta src/pages/Messages/components/
2. Criar componentes menores
3. Testar componentes isoladamente
```

### Fase 3: Integrar (substituição)
```
1. Criar MessagesPage.tsx novo
2. Importar hooks e componentes
3. Substituir src/pages/Messages.tsx por import do novo
4. Testar integração completa
```

---

## Checklist

### Hooks
- [ ] `useConversations.ts`
- [ ] `useMessages.ts`
- [ ] `useMessageActions.ts`
- [ ] `useConversationFilters.ts`
- [ ] `useWebSocketMessages.ts`
- [ ] `useConversationActions.ts`
- [ ] `hooks/index.ts`

### Componentes
- [ ] `ConversationList/index.tsx`
- [ ] `ConversationList/ConversationItem.tsx`
- [ ] `ConversationList/ConversationFilters.tsx`
- [ ] `MessageThread/index.tsx`
- [ ] `MessageThread/MessageBubble.tsx`
- [ ] `MessageThread/MessageInput.tsx`
- [ ] `ContactSidebar/index.tsx`

### Integração
- [ ] `MessagesPage.tsx`
- [ ] `index.tsx` (re-export)
- [ ] Atualizar router
- [ ] Remover `Messages.tsx` antigo

### Testes
- [ ] Testes de hooks
- [ ] Testes de componentes
- [ ] Teste E2E da página

---

## Métricas Esperadas

| Métrica | Antes | Depois | Melhoria |
|---------|-------|--------|----------|
| Linhas (maior arquivo) | 1.699 | ~150 | 91%↓ |
| Hooks no componente | 81 | ~6 | 93%↓ |
| Testabilidade | Impossível | Fácil | ✅ |
| Reusabilidade | Nenhuma | Alta | ✅ |

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
