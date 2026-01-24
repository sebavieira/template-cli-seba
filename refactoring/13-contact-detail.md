# Refatoração: Contact Detail Page

**Arquivo:** `src/pages/ContactDetail.tsx`
**Linhas:** 843
**Prioridade:** P1 - ALTO
**Estimativa:** 1 dia

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 843 | 300 | 🔴 CRÍTICO |
| Responsabilidades | 4+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **Header** - Informações básicas do contato
2. **Tabs de dados** - Histórico, notas, deals, análise
3. **Ações** - Editar, tags, enviar mensagem
4. **Sidebar** - Informações resumidas

---

## Estrutura Proposta

```
src/pages/ContactDetail/
├── index.tsx                     # Container (~100 linhas)
├── ContactHeader.tsx             # Header com avatar e info (~150 linhas)
├── ContactTabs.tsx               # Gerenciador de tabs (~80 linhas)
├── tabs/
│   ├── HistoryTab.tsx            # Histórico de conversas (~150 linhas)
│   ├── NotesTab.tsx              # Notas do contato (~120 linhas)
│   ├── DealsTab.tsx              # Deals associados (~120 linhas)
│   └── AnalysisTab.tsx           # Análise/Sentiment (~100 linhas)
├── components/
│   ├── ContactActions.tsx        # Botões de ação (~80 linhas)
│   ├── TagsManager.tsx           # Gerenciador de tags (~100 linhas)
│   └── ContactSidebar.tsx        # Sidebar resumida (~120 linhas)
└── hooks/
    ├── useContact.ts             # Query do contato (~60 linhas)
    └── useContactMutations.ts    # Mutations (~80 linhas)
```

---

## Hooks

### useContact
```typescript
// src/pages/ContactDetail/hooks/useContact.ts

import { useQuery } from '@tanstack/react-query';
import { contactsApi } from '@/lib/api';

export function useContact(contactId: string) {
  return useQuery({
    queryKey: ['contacts', contactId],
    queryFn: () => contactsApi.getById(contactId),
    enabled: !!contactId,
  });
}

export function useContactHistory(contactId: string) {
  return useQuery({
    queryKey: ['contacts', contactId, 'history'],
    queryFn: () => contactsApi.getHistory(contactId),
    enabled: !!contactId,
  });
}

export function useContactDeals(contactId: string) {
  return useQuery({
    queryKey: ['contacts', contactId, 'deals'],
    queryFn: () => contactsApi.getDeals(contactId),
    enabled: !!contactId,
  });
}

export function useContactNotes(contactId: string) {
  return useQuery({
    queryKey: ['contacts', contactId, 'notes'],
    queryFn: () => contactsApi.getNotes(contactId),
    enabled: !!contactId,
  });
}
```

### useContactMutations
```typescript
// src/pages/ContactDetail/hooks/useContactMutations.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { contactsApi, tagsApi } from '@/lib/api';
import { useToast } from '@/hooks/use-toast';

export function useContactMutations(contactId: string) {
  const queryClient = useQueryClient();
  const { toast } = useToast();

  const updateContact = useMutation({
    mutationFn: (data: any) => contactsApi.update(contactId, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['contacts', contactId] });
      toast({ title: 'Contato atualizado' });
    },
  });

  const assignTags = useMutation({
    mutationFn: (tagIds: string[]) => contactsApi.assignTags(contactId, tagIds),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['contacts', contactId] });
      toast({ title: 'Tags atualizadas' });
    },
  });

  const addNote = useMutation({
    mutationFn: (content: string) => contactsApi.addNote(contactId, content),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['contacts', contactId, 'notes'] });
      toast({ title: 'Nota adicionada' });
    },
  });

  const deleteContact = useMutation({
    mutationFn: () => contactsApi.delete(contactId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['contacts'] });
      toast({ title: 'Contato removido' });
    },
  });

  return {
    updateContact,
    assignTags,
    addNote,
    deleteContact,
  };
}
```

---

## Componentes

### Container
```typescript
// src/pages/ContactDetail/index.tsx

import { useParams } from 'react-router-dom';
import { ContactHeader } from './ContactHeader';
import { ContactTabs } from './ContactTabs';
import { ContactSidebar } from './components/ContactSidebar';
import { useContact } from './hooks/useContact';
import { Skeleton } from '@/components/ui/skeleton';

export function ContactDetail() {
  const { contactId } = useParams<{ contactId: string }>();
  const { data: contact, isLoading, error } = useContact(contactId!);

  if (isLoading) {
    return <ContactDetailSkeleton />;
  }

  if (error || !contact) {
    return <ContactNotFound />;
  }

  return (
    <div className="flex h-full">
      <div className="flex-1 overflow-auto">
        <ContactHeader contact={contact} />
        <ContactTabs contactId={contact.id} />
      </div>
      <ContactSidebar contact={contact} />
    </div>
  );
}

function ContactDetailSkeleton() {
  return (
    <div className="p-6 space-y-4">
      <Skeleton className="h-32 w-full" />
      <Skeleton className="h-64 w-full" />
    </div>
  );
}

function ContactNotFound() {
  return (
    <div className="flex items-center justify-center h-full">
      <p className="text-muted-foreground">Contato não encontrado</p>
    </div>
  );
}
```

### ContactHeader
```typescript
// src/pages/ContactDetail/ContactHeader.tsx

import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { Badge } from '@/components/ui/badge';
import { ContactActions } from './components/ContactActions';
import { TagsManager } from './components/TagsManager';
import type { Contact } from '@/lib/api';

interface ContactHeaderProps {
  contact: Contact;
}

export function ContactHeader({ contact }: ContactHeaderProps) {
  return (
    <div className="p-6 border-b">
      <div className="flex items-start gap-4">
        <Avatar className="h-16 w-16">
          <AvatarImage src={contact.avatarUrl} />
          <AvatarFallback>{contact.name.slice(0, 2).toUpperCase()}</AvatarFallback>
        </Avatar>

        <div className="flex-1">
          <div className="flex items-center gap-2">
            <h1 className="text-2xl font-bold">{contact.name}</h1>
            {contact.sentiment && (
              <SentimentBadge sentiment={contact.sentiment} />
            )}
          </div>
          <p className="text-muted-foreground">{contact.phone}</p>
          {contact.email && <p className="text-muted-foreground">{contact.email}</p>}

          <TagsManager
            contactId={contact.id}
            currentTags={contact.tags}
            className="mt-2"
          />
        </div>

        <ContactActions contact={contact} />
      </div>
    </div>
  );
}

function SentimentBadge({ sentiment }: { sentiment: string }) {
  const colors: Record<string, string> = {
    positive: 'bg-green-100 text-green-800',
    neutral: 'bg-gray-100 text-gray-800',
    negative: 'bg-red-100 text-red-800',
  };

  return (
    <Badge className={colors[sentiment] || colors.neutral}>
      {sentiment}
    </Badge>
  );
}
```

### ContactTabs
```typescript
// src/pages/ContactDetail/ContactTabs.tsx

import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { HistoryTab } from './tabs/HistoryTab';
import { NotesTab } from './tabs/NotesTab';
import { DealsTab } from './tabs/DealsTab';
import { AnalysisTab } from './tabs/AnalysisTab';
import { MessageSquare, StickyNote, DollarSign, BarChart } from 'lucide-react';

interface ContactTabsProps {
  contactId: string;
}

export function ContactTabs({ contactId }: ContactTabsProps) {
  return (
    <Tabs defaultValue="history" className="p-6">
      <TabsList>
        <TabsTrigger value="history">
          <MessageSquare className="w-4 h-4 mr-2" />
          Histórico
        </TabsTrigger>
        <TabsTrigger value="notes">
          <StickyNote className="w-4 h-4 mr-2" />
          Notas
        </TabsTrigger>
        <TabsTrigger value="deals">
          <DollarSign className="w-4 h-4 mr-2" />
          Negócios
        </TabsTrigger>
        <TabsTrigger value="analysis">
          <BarChart className="w-4 h-4 mr-2" />
          Análise
        </TabsTrigger>
      </TabsList>

      <TabsContent value="history">
        <HistoryTab contactId={contactId} />
      </TabsContent>

      <TabsContent value="notes">
        <NotesTab contactId={contactId} />
      </TabsContent>

      <TabsContent value="deals">
        <DealsTab contactId={contactId} />
      </TabsContent>

      <TabsContent value="analysis">
        <AnalysisTab contactId={contactId} />
      </TabsContent>
    </Tabs>
  );
}
```

### NotesTab
```typescript
// src/pages/ContactDetail/tabs/NotesTab.tsx

import { useState } from 'react';
import { useContactNotes } from '../hooks/useContact';
import { useContactMutations } from '../hooks/useContactMutations';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { Card, CardContent } from '@/components/ui/card';
import { formatDistanceToNow } from 'date-fns';
import { ptBR } from 'date-fns/locale';

interface NotesTabProps {
  contactId: string;
}

export function NotesTab({ contactId }: NotesTabProps) {
  const [newNote, setNewNote] = useState('');
  const { data: notes = [], isLoading } = useContactNotes(contactId);
  const { addNote } = useContactMutations(contactId);

  const handleAddNote = () => {
    if (!newNote.trim()) return;
    addNote.mutate(newNote, {
      onSuccess: () => setNewNote(''),
    });
  };

  return (
    <div className="space-y-4">
      <div className="flex gap-2">
        <Textarea
          placeholder="Adicionar uma nota..."
          value={newNote}
          onChange={(e) => setNewNote(e.target.value)}
        />
        <Button onClick={handleAddNote} disabled={addNote.isPending}>
          Adicionar
        </Button>
      </div>

      {isLoading ? (
        <p>Carregando notas...</p>
      ) : (
        <div className="space-y-2">
          {notes.map((note) => (
            <Card key={note.id}>
              <CardContent className="p-4">
                <p className="whitespace-pre-wrap">{note.content}</p>
                <p className="text-sm text-muted-foreground mt-2">
                  {formatDistanceToNow(new Date(note.createdAt), {
                    addSuffix: true,
                    locale: ptBR,
                  })}
                </p>
              </CardContent>
            </Card>
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Checklist

### Estrutura
- [ ] Criar pasta `src/pages/ContactDetail/`
- [ ] Criar pasta `tabs/`
- [ ] Criar pasta `components/`
- [ ] Criar pasta `hooks/`

### Hooks
- [ ] `useContact.ts`
- [ ] `useContactMutations.ts`

### Componentes
- [ ] `index.tsx`
- [ ] `ContactHeader.tsx`
- [ ] `ContactTabs.tsx`
- [ ] `components/ContactActions.tsx`
- [ ] `components/TagsManager.tsx`
- [ ] `components/ContactSidebar.tsx`

### Tabs
- [ ] `tabs/HistoryTab.tsx`
- [ ] `tabs/NotesTab.tsx`
- [ ] `tabs/DealsTab.tsx`
- [ ] `tabs/AnalysisTab.tsx`

### Finalização
- [ ] Atualizar router
- [ ] Remover arquivo antigo
- [ ] Testes

---

**Status:** ✅ Completo
**Última Atualização:** 2026-01-23
