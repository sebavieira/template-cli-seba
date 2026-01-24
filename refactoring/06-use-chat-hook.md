# Refatoração: Use Chat Hook

**Arquivo:** `src/components/editor/use-chat.ts`
**Linhas:** 1.649
**Prioridade:** P0 - CRÍTICO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 1.649 | 200 | 🔴 CRÍTICO |
| Responsabilidades | 5+ | 1 | 🔴 CRÍTICO |

### Responsabilidades Misturadas

1. **Chat básico** - Envio/recebimento de mensagens
2. **Streaming** - SSE/WebSocket para respostas AI
3. **Comentários** - Sistema de comentários inline
4. **Edição de tabelas** - Edição de dados tabulares
5. **Chunks de dados** - Processamento de dados em chunks
6. **Estado complexo** - Múltiplos estados interdependentes

---

## Estrutura Proposta

```
src/components/editor/
├── use-chat.ts                  # Hook principal (facade) (~150 linhas)
├── hooks/
│   ├── index.ts                 # Re-exports
│   ├── useChatCore.ts           # Chat básico (~200 linhas)
│   ├── useChatStreaming.ts      # Streaming de respostas (~250 linhas)
│   ├── useChatComments.ts       # Sistema de comentários (~200 linhas)
│   ├── useChatTableEdits.ts     # Edição de tabelas (~200 linhas)
│   └── useChatState.ts          # Estado centralizado (~150 linhas)
├── utils/
│   ├── chatChunks.ts            # Processamento de chunks (~100 linhas)
│   ├── chatFormatters.ts        # Formatação de mensagens (~80 linhas)
│   └── chatValidators.ts        # Validações (~50 linhas)
└── types/
    └── chat.types.ts            # Tipos do chat (~100 linhas)
```

---

## Tipos Base

```typescript
// src/components/editor/types/chat.types.ts

export interface ChatMessage {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: Date;
  status: 'pending' | 'streaming' | 'complete' | 'error';
  metadata?: {
    tokens?: number;
    model?: string;
    toolCalls?: ToolCall[];
  };
}

export interface ToolCall {
  id: string;
  name: string;
  arguments: Record<string, any>;
  result?: any;
  status: 'pending' | 'executing' | 'complete' | 'error';
}

export interface ChatComment {
  id: string;
  messageId: string;
  content: string;
  position: { start: number; end: number };
  author: string;
  createdAt: Date;
  resolved: boolean;
}

export interface TableEdit {
  id: string;
  messageId: string;
  tableId: string;
  changes: TableChange[];
  status: 'pending' | 'applied' | 'reverted';
}

export interface TableChange {
  row: number;
  column: string;
  oldValue: any;
  newValue: any;
}

export interface ChatState {
  messages: ChatMessage[];
  isStreaming: boolean;
  error: string | null;
  comments: ChatComment[];
  tableEdits: TableEdit[];
}

export interface UseChatOptions {
  conversationId?: string;
  systemPrompt?: string;
  model?: string;
  temperature?: number;
  onMessage?: (message: ChatMessage) => void;
  onError?: (error: Error) => void;
  onStreamStart?: () => void;
  onStreamEnd?: () => void;
}
```

---

## Hooks Especializados

### useChatState

```typescript
// src/components/editor/hooks/useChatState.ts

import { useReducer, useCallback } from 'react';
import type { ChatState, ChatMessage, ChatComment, TableEdit } from '../types/chat.types';

type ChatAction =
  | { type: 'ADD_MESSAGE'; payload: ChatMessage }
  | { type: 'UPDATE_MESSAGE'; payload: { id: string; updates: Partial<ChatMessage> } }
  | { type: 'SET_STREAMING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null }
  | { type: 'ADD_COMMENT'; payload: ChatComment }
  | { type: 'RESOLVE_COMMENT'; payload: string }
  | { type: 'ADD_TABLE_EDIT'; payload: TableEdit }
  | { type: 'REVERT_TABLE_EDIT'; payload: string }
  | { type: 'CLEAR_MESSAGES' };

const initialState: ChatState = {
  messages: [],
  isStreaming: false,
  error: null,
  comments: [],
  tableEdits: [],
};

function chatReducer(state: ChatState, action: ChatAction): ChatState {
  switch (action.type) {
    case 'ADD_MESSAGE':
      return { ...state, messages: [...state.messages, action.payload] };

    case 'UPDATE_MESSAGE':
      return {
        ...state,
        messages: state.messages.map((m) =>
          m.id === action.payload.id ? { ...m, ...action.payload.updates } : m
        ),
      };

    case 'SET_STREAMING':
      return { ...state, isStreaming: action.payload };

    case 'SET_ERROR':
      return { ...state, error: action.payload };

    case 'ADD_COMMENT':
      return { ...state, comments: [...state.comments, action.payload] };

    case 'RESOLVE_COMMENT':
      return {
        ...state,
        comments: state.comments.map((c) =>
          c.id === action.payload ? { ...c, resolved: true } : c
        ),
      };

    case 'ADD_TABLE_EDIT':
      return { ...state, tableEdits: [...state.tableEdits, action.payload] };

    case 'REVERT_TABLE_EDIT':
      return {
        ...state,
        tableEdits: state.tableEdits.map((e) =>
          e.id === action.payload ? { ...e, status: 'reverted' } : e
        ),
      };

    case 'CLEAR_MESSAGES':
      return { ...state, messages: [], comments: [], tableEdits: [] };

    default:
      return state;
  }
}

export function useChatState(initialMessages: ChatMessage[] = []) {
  const [state, dispatch] = useReducer(chatReducer, {
    ...initialState,
    messages: initialMessages,
  });

  const addMessage = useCallback((message: ChatMessage) => {
    dispatch({ type: 'ADD_MESSAGE', payload: message });
  }, []);

  const updateMessage = useCallback((id: string, updates: Partial<ChatMessage>) => {
    dispatch({ type: 'UPDATE_MESSAGE', payload: { id, updates } });
  }, []);

  const setStreaming = useCallback((isStreaming: boolean) => {
    dispatch({ type: 'SET_STREAMING', payload: isStreaming });
  }, []);

  const setError = useCallback((error: string | null) => {
    dispatch({ type: 'SET_ERROR', payload: error });
  }, []);

  const addComment = useCallback((comment: ChatComment) => {
    dispatch({ type: 'ADD_COMMENT', payload: comment });
  }, []);

  const resolveComment = useCallback((commentId: string) => {
    dispatch({ type: 'RESOLVE_COMMENT', payload: commentId });
  }, []);

  const addTableEdit = useCallback((edit: TableEdit) => {
    dispatch({ type: 'ADD_TABLE_EDIT', payload: edit });
  }, []);

  const revertTableEdit = useCallback((editId: string) => {
    dispatch({ type: 'REVERT_TABLE_EDIT', payload: editId });
  }, []);

  const clearMessages = useCallback(() => {
    dispatch({ type: 'CLEAR_MESSAGES' });
  }, []);

  return {
    state,
    addMessage,
    updateMessage,
    setStreaming,
    setError,
    addComment,
    resolveComment,
    addTableEdit,
    revertTableEdit,
    clearMessages,
  };
}
```

### useChatCore

```typescript
// src/components/editor/hooks/useChatCore.ts

import { useCallback } from 'react';
import { useMutation } from '@tanstack/react-query';
import { apiClient } from '@/lib/api';
import type { ChatMessage, UseChatOptions } from '../types/chat.types';
import { useChatState } from './useChatState';

interface SendMessageParams {
  content: string;
  attachments?: File[];
}

export function useChatCore(options: UseChatOptions = {}) {
  const { conversationId, systemPrompt, model = 'gpt-4', onMessage, onError } = options;

  const {
    state,
    addMessage,
    updateMessage,
    setError,
    clearMessages,
  } = useChatState();

  const sendMutation = useMutation({
    mutationFn: async (params: SendMessageParams) => {
      const userMessage: ChatMessage = {
        id: `user-${Date.now()}`,
        role: 'user',
        content: params.content,
        timestamp: new Date(),
        status: 'complete',
      };

      addMessage(userMessage);

      const response = await apiClient.post('/ai/chat', {
        conversationId,
        message: params.content,
        model,
        systemPrompt,
      });

      return response.data;
    },
    onSuccess: (data) => {
      const assistantMessage: ChatMessage = {
        id: data.id || `assistant-${Date.now()}`,
        role: 'assistant',
        content: data.content,
        timestamp: new Date(),
        status: 'complete',
        metadata: {
          tokens: data.usage?.total_tokens,
          model: data.model,
          toolCalls: data.tool_calls,
        },
      };

      addMessage(assistantMessage);
      onMessage?.(assistantMessage);
    },
    onError: (error: Error) => {
      setError(error.message);
      onError?.(error);
    },
  });

  const sendMessage = useCallback(
    (content: string, attachments?: File[]) => {
      sendMutation.mutate({ content, attachments });
    },
    [sendMutation]
  );

  const regenerateLastMessage = useCallback(() => {
    const lastUserMessage = [...state.messages]
      .reverse()
      .find((m) => m.role === 'user');

    if (lastUserMessage) {
      sendMessage(lastUserMessage.content);
    }
  }, [state.messages, sendMessage]);

  return {
    messages: state.messages,
    error: state.error,
    isLoading: sendMutation.isPending,
    sendMessage,
    regenerateLastMessage,
    clearMessages,
  };
}
```

### useChatStreaming

```typescript
// src/components/editor/hooks/useChatStreaming.ts

import { useCallback, useRef } from 'react';
import type { ChatMessage, UseChatOptions } from '../types/chat.types';
import { useChatState } from './useChatState';

export function useChatStreaming(options: UseChatOptions = {}) {
  const { onStreamStart, onStreamEnd, onError } = options;
  const abortControllerRef = useRef<AbortController | null>(null);

  const {
    state,
    addMessage,
    updateMessage,
    setStreaming,
    setError,
  } = useChatState();

  const sendStreamingMessage = useCallback(
    async (content: string) => {
      // Cancelar stream anterior se existir
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }

      abortControllerRef.current = new AbortController();

      // Adicionar mensagem do usuário
      const userMessage: ChatMessage = {
        id: `user-${Date.now()}`,
        role: 'user',
        content,
        timestamp: new Date(),
        status: 'complete',
      };
      addMessage(userMessage);

      // Criar placeholder para resposta
      const assistantId = `assistant-${Date.now()}`;
      const assistantMessage: ChatMessage = {
        id: assistantId,
        role: 'assistant',
        content: '',
        timestamp: new Date(),
        status: 'streaming',
      };
      addMessage(assistantMessage);

      setStreaming(true);
      onStreamStart?.();

      try {
        const response = await fetch('/api/ai/chat/stream', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message: content }),
          signal: abortControllerRef.current.signal,
        });

        if (!response.ok) {
          throw new Error('Stream failed');
        }

        const reader = response.body?.getReader();
        const decoder = new TextDecoder();
        let accumulatedContent = '';

        while (reader) {
          const { done, value } = await reader.read();
          if (done) break;

          const chunk = decoder.decode(value);
          const lines = chunk.split('\n').filter((line) => line.startsWith('data: '));

          for (const line of lines) {
            const data = line.slice(6); // Remove "data: "
            if (data === '[DONE]') continue;

            try {
              const parsed = JSON.parse(data);
              const content = parsed.choices?.[0]?.delta?.content || '';
              accumulatedContent += content;

              updateMessage(assistantId, { content: accumulatedContent });
            } catch {
              // Ignorar linhas inválidas
            }
          }
        }

        updateMessage(assistantId, { status: 'complete' });
      } catch (error) {
        if ((error as Error).name === 'AbortError') {
          updateMessage(assistantId, { status: 'complete', content: accumulatedContent + ' [cancelado]' });
        } else {
          setError((error as Error).message);
          updateMessage(assistantId, { status: 'error' });
          onError?.(error as Error);
        }
      } finally {
        setStreaming(false);
        onStreamEnd?.();
        abortControllerRef.current = null;
      }
    },
    [addMessage, updateMessage, setStreaming, setError, onStreamStart, onStreamEnd, onError]
  );

  const cancelStream = useCallback(() => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
  }, []);

  return {
    messages: state.messages,
    isStreaming: state.isStreaming,
    error: state.error,
    sendStreamingMessage,
    cancelStream,
  };
}
```

### useChatComments

```typescript
// src/components/editor/hooks/useChatComments.ts

import { useCallback } from 'react';
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { apiClient } from '@/lib/api';
import type { ChatComment } from '../types/chat.types';

interface UseChatCommentsOptions {
  conversationId: string;
}

export function useChatComments({ conversationId }: UseChatCommentsOptions) {
  const queryClient = useQueryClient();

  const { data: comments = [], isLoading } = useQuery({
    queryKey: ['chat-comments', conversationId],
    queryFn: async () => {
      const { data } = await apiClient.get(`/ai/conversations/${conversationId}/comments`);
      return data.data as ChatComment[];
    },
    enabled: !!conversationId,
  });

  const addCommentMutation = useMutation({
    mutationFn: async (comment: Omit<ChatComment, 'id' | 'createdAt' | 'resolved'>) => {
      const { data } = await apiClient.post(
        `/ai/conversations/${conversationId}/comments`,
        comment
      );
      return data.data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['chat-comments', conversationId] });
    },
  });

  const resolveCommentMutation = useMutation({
    mutationFn: async (commentId: string) => {
      await apiClient.patch(`/ai/comments/${commentId}/resolve`);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['chat-comments', conversationId] });
    },
  });

  const deleteCommentMutation = useMutation({
    mutationFn: async (commentId: string) => {
      await apiClient.delete(`/ai/comments/${commentId}`);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['chat-comments', conversationId] });
    },
  });

  const addComment = useCallback(
    (messageId: string, content: string, position: { start: number; end: number }) => {
      addCommentMutation.mutate({
        messageId,
        content,
        position,
        author: 'current-user', // TODO: pegar do contexto
      });
    },
    [addCommentMutation]
  );

  const resolveComment = useCallback(
    (commentId: string) => {
      resolveCommentMutation.mutate(commentId);
    },
    [resolveCommentMutation]
  );

  const deleteComment = useCallback(
    (commentId: string) => {
      deleteCommentMutation.mutate(commentId);
    },
    [deleteCommentMutation]
  );

  const getCommentsForMessage = useCallback(
    (messageId: string) => {
      return comments.filter((c) => c.messageId === messageId);
    },
    [comments]
  );

  return {
    comments,
    isLoading,
    addComment,
    resolveComment,
    deleteComment,
    getCommentsForMessage,
    unresolvedCount: comments.filter((c) => !c.resolved).length,
  };
}
```

---

## Hook Principal (Facade)

```typescript
// src/components/editor/use-chat.ts (REFATORADO)

import { useChatCore } from './hooks/useChatCore';
import { useChatStreaming } from './hooks/useChatStreaming';
import { useChatComments } from './hooks/useChatComments';
import { useChatTableEdits } from './hooks/useChatTableEdits';
import type { UseChatOptions } from './types/chat.types';

interface UseChatReturn {
  // Core
  messages: ChatMessage[];
  sendMessage: (content: string) => void;
  clearMessages: () => void;
  isLoading: boolean;
  error: string | null;

  // Streaming
  sendStreamingMessage: (content: string) => void;
  cancelStream: () => void;
  isStreaming: boolean;

  // Comments
  comments: ChatComment[];
  addComment: (messageId: string, content: string, position: { start: number; end: number }) => void;
  resolveComment: (commentId: string) => void;
  getCommentsForMessage: (messageId: string) => ChatComment[];

  // Table Edits
  tableEdits: TableEdit[];
  applyTableEdit: (messageId: string, tableId: string, changes: TableChange[]) => void;
  revertTableEdit: (editId: string) => void;
}

export function useChat(options: UseChatOptions = {}): UseChatReturn {
  const core = useChatCore(options);
  const streaming = useChatStreaming(options);
  const comments = useChatComments({
    conversationId: options.conversationId || '',
  });
  const tableEdits = useChatTableEdits({
    conversationId: options.conversationId || '',
  });

  // Combinar mensagens de core e streaming
  const messages = options.streaming
    ? streaming.messages
    : core.messages;

  return {
    // Core
    messages,
    sendMessage: options.streaming ? streaming.sendStreamingMessage : core.sendMessage,
    clearMessages: core.clearMessages,
    isLoading: core.isLoading,
    error: core.error || streaming.error,

    // Streaming
    sendStreamingMessage: streaming.sendStreamingMessage,
    cancelStream: streaming.cancelStream,
    isStreaming: streaming.isStreaming,

    // Comments
    comments: comments.comments,
    addComment: comments.addComment,
    resolveComment: comments.resolveComment,
    getCommentsForMessage: comments.getCommentsForMessage,

    // Table Edits
    tableEdits: tableEdits.edits,
    applyTableEdit: tableEdits.applyEdit,
    revertTableEdit: tableEdits.revertEdit,
  };
}

// Re-export para compatibilidade
export type { UseChatOptions, ChatMessage, ChatComment, TableEdit } from './types/chat.types';
```

---

## Checklist

### Estrutura
- [ ] Criar `types/chat.types.ts`
- [ ] Criar `hooks/` folder
- [ ] Criar `utils/` folder

### Hooks
- [ ] `useChatState.ts`
- [ ] `useChatCore.ts`
- [ ] `useChatStreaming.ts`
- [ ] `useChatComments.ts`
- [ ] `useChatTableEdits.ts`
- [ ] `hooks/index.ts`

### Utils
- [ ] `chatChunks.ts`
- [ ] `chatFormatters.ts`
- [ ] `chatValidators.ts`

### Integração
- [ ] Refatorar `use-chat.ts` para facade
- [ ] Atualizar imports nos componentes
- [ ] Testes unitários
- [ ] Testes de integração

---

## Métricas Esperadas

| Métrica | Antes | Depois | Melhoria |
|---------|-------|--------|----------|
| Linhas (maior arquivo) | 1.649 | ~250 | 85%↓ |
| Responsabilidades | 5+ | 1 por arquivo | 80%↓ |
| Testabilidade | Difícil | Fácil | ✅ |

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
