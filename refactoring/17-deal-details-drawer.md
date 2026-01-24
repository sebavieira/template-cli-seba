# Refatoração: Deal Details Drawer

**Arquivo:** `src/components/funnels/DealDetailsDrawer.tsx`
**Linhas:** 1.147
**Prioridade:** P2 - MÉDIO
**Estimativa:** 1 dia

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 1.147 | 300 | 🔴 CRÍTICO |
| Responsabilidades | 5+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **Header** - Informações básicas do deal
2. **Tabs** - Timeline, notas, anexos, atividades
3. **Stage Management** - Mover entre estágios
4. **Notes** - Adicionar/editar notas
5. **Attachments** - Upload e listagem

---

## Estrutura Proposta

```
src/components/funnels/DealDetails/
├── index.tsx                     # Re-export
├── DealDetailsDrawer.tsx         # Container principal (~150 linhas)
├── DealHeader.tsx                # Header com info básica (~100 linhas)
├── DealTabs.tsx                  # Gerenciador de tabs (~80 linhas)
├── tabs/
│   ├── TimelineTab.tsx           # Timeline (~150 linhas)
│   ├── NotesTab.tsx              # Notas (~120 linhas)
│   ├── AttachmentsTab.tsx        # Anexos (~120 linhas)
│   └── ActivitiesTab.tsx         # Atividades (~100 linhas)
├── components/
│   ├── StageSelector.tsx         # Seletor de estágio (~100 linhas)
│   ├── DealValue.tsx             # Edição de valor (~80 linhas)
│   └── DealActions.tsx           # Ações (won/lost) (~80 linhas)
└── hooks/
    ├── useDeal.ts                # Query do deal (~60 linhas)
    └── useDealMutations.ts       # Mutations (~100 linhas)
```

---

## Código Refatorado

### Container Principal
```typescript
// src/components/funnels/DealDetails/DealDetailsDrawer.tsx

import {
  Sheet,
  SheetContent,
  SheetHeader,
  SheetTitle,
} from '@/components/ui/sheet';
import { DealHeader } from './DealHeader';
import { DealTabs } from './DealTabs';
import { useDeal } from './hooks/useDeal';
import { Skeleton } from '@/components/ui/skeleton';

interface DealDetailsDrawerProps {
  dealId: string | null;
  open: boolean;
  onClose: () => void;
}

export function DealDetailsDrawer({ dealId, open, onClose }: DealDetailsDrawerProps) {
  const { data: deal, isLoading } = useDeal(dealId!);

  return (
    <Sheet open={open} onOpenChange={(open) => !open && onClose()}>
      <SheetContent className="w-[600px] sm:max-w-[600px] p-0 overflow-hidden">
        {isLoading ? (
          <DealDetailsSkeleton />
        ) : deal ? (
          <div className="flex flex-col h-full">
            <SheetHeader className="p-6 border-b">
              <SheetTitle>{deal.title}</SheetTitle>
            </SheetHeader>

            <DealHeader deal={deal} />

            <div className="flex-1 overflow-auto">
              <DealTabs dealId={deal.id} />
            </div>
          </div>
        ) : (
          <div className="p-6">Deal não encontrado</div>
        )}
      </SheetContent>
    </Sheet>
  );
}

function DealDetailsSkeleton() {
  return (
    <div className="p-6 space-y-4">
      <Skeleton className="h-8 w-48" />
      <Skeleton className="h-24 w-full" />
      <Skeleton className="h-64 w-full" />
    </div>
  );
}
```

### DealHeader
```typescript
// src/components/funnels/DealDetails/DealHeader.tsx

import { Badge } from '@/components/ui/badge';
import { StageSelector } from './components/StageSelector';
import { DealValue } from './components/DealValue';
import { DealActions } from './components/DealActions';
import { formatCurrency } from '@/lib/utils';
import type { Deal } from '@/lib/api';

interface DealHeaderProps {
  deal: Deal;
}

export function DealHeader({ deal }: DealHeaderProps) {
  return (
    <div className="p-6 border-b space-y-4">
      {/* Contact Info */}
      <div className="flex items-center gap-3">
        <Avatar>
          <AvatarFallback>{deal.contact?.name.slice(0, 2)}</AvatarFallback>
        </Avatar>
        <div>
          <p className="font-medium">{deal.contact?.name}</p>
          <p className="text-sm text-muted-foreground">{deal.contact?.phone}</p>
        </div>
      </div>

      {/* Value and Stage */}
      <div className="flex items-center justify-between">
        <DealValue dealId={deal.id} value={deal.value} />
        <StageSelector
          dealId={deal.id}
          currentStageId={deal.stageId}
          funnelId={deal.funnelId}
        />
      </div>

      {/* Status Badge */}
      {deal.status !== 'open' && (
        <Badge variant={deal.status === 'won' ? 'default' : 'destructive'}>
          {deal.status === 'won' ? 'Ganho' : 'Perdido'}
        </Badge>
      )}

      {/* Actions */}
      {deal.status === 'open' && <DealActions dealId={deal.id} />}
    </div>
  );
}
```

### DealTabs
```typescript
// src/components/funnels/DealDetails/DealTabs.tsx

import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { TimelineTab } from './tabs/TimelineTab';
import { NotesTab } from './tabs/NotesTab';
import { AttachmentsTab } from './tabs/AttachmentsTab';
import { ActivitiesTab } from './tabs/ActivitiesTab';
import { Clock, StickyNote, Paperclip, Calendar } from 'lucide-react';

interface DealTabsProps {
  dealId: string;
}

export function DealTabs({ dealId }: DealTabsProps) {
  return (
    <Tabs defaultValue="timeline" className="p-6">
      <TabsList className="grid grid-cols-4">
        <TabsTrigger value="timeline">
          <Clock className="w-4 h-4 mr-2" />
          Timeline
        </TabsTrigger>
        <TabsTrigger value="notes">
          <StickyNote className="w-4 h-4 mr-2" />
          Notas
        </TabsTrigger>
        <TabsTrigger value="attachments">
          <Paperclip className="w-4 h-4 mr-2" />
          Anexos
        </TabsTrigger>
        <TabsTrigger value="activities">
          <Calendar className="w-4 h-4 mr-2" />
          Atividades
        </TabsTrigger>
      </TabsList>

      <TabsContent value="timeline">
        <TimelineTab dealId={dealId} />
      </TabsContent>

      <TabsContent value="notes">
        <NotesTab dealId={dealId} />
      </TabsContent>

      <TabsContent value="attachments">
        <AttachmentsTab dealId={dealId} />
      </TabsContent>

      <TabsContent value="activities">
        <ActivitiesTab dealId={dealId} />
      </TabsContent>
    </Tabs>
  );
}
```

### StageSelector
```typescript
// src/components/funnels/DealDetails/components/StageSelector.tsx

import { useQuery } from '@tanstack/react-query';
import { funnelsApi } from '@/lib/api';
import { useDealMutations } from '../hooks/useDealMutations';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';

interface StageSelectorProps {
  dealId: string;
  currentStageId: string;
  funnelId: string;
}

export function StageSelector({ dealId, currentStageId, funnelId }: StageSelectorProps) {
  const { data: funnel } = useQuery({
    queryKey: ['funnels', funnelId],
    queryFn: () => funnelsApi.getById(funnelId),
  });

  const { moveToStage } = useDealMutations(dealId);

  const handleStageChange = (stageId: string) => {
    if (stageId !== currentStageId) {
      moveToStage.mutate(stageId);
    }
  };

  return (
    <Select value={currentStageId} onValueChange={handleStageChange}>
      <SelectTrigger className="w-[180px]">
        <SelectValue placeholder="Selecione o estágio" />
      </SelectTrigger>
      <SelectContent>
        {funnel?.stages.map((stage) => (
          <SelectItem key={stage.id} value={stage.id}>
            <div className="flex items-center gap-2">
              <div
                className="w-2 h-2 rounded-full"
                style={{ backgroundColor: stage.color }}
              />
              {stage.name}
            </div>
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}
```

### Hooks
```typescript
// src/components/funnels/DealDetails/hooks/useDeal.ts

import { useQuery } from '@tanstack/react-query';
import { dealsApi } from '@/lib/api';

export function useDeal(dealId: string) {
  return useQuery({
    queryKey: ['deals', dealId],
    queryFn: () => dealsApi.getById(dealId),
    enabled: !!dealId,
  });
}

export function useDealNotes(dealId: string) {
  return useQuery({
    queryKey: ['deals', dealId, 'notes'],
    queryFn: () => dealsApi.getNotes(dealId),
    enabled: !!dealId,
  });
}

export function useDealTimeline(dealId: string) {
  return useQuery({
    queryKey: ['deals', dealId, 'timeline'],
    queryFn: () => dealsApi.getTimeline(dealId),
    enabled: !!dealId,
  });
}
```

```typescript
// src/components/funnels/DealDetails/hooks/useDealMutations.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { dealsApi } from '@/lib/api';
import { useToast } from '@/hooks/use-toast';

export function useDealMutations(dealId: string) {
  const queryClient = useQueryClient();
  const { toast } = useToast();

  const invalidateDeal = () => {
    queryClient.invalidateQueries({ queryKey: ['deals', dealId] });
    queryClient.invalidateQueries({ queryKey: ['deals'] });
  };

  const moveToStage = useMutation({
    mutationFn: (stageId: string) => dealsApi.moveToStage(dealId, stageId),
    onSuccess: () => {
      invalidateDeal();
      toast({ title: 'Estágio atualizado' });
    },
  });

  const updateValue = useMutation({
    mutationFn: (value: number) => dealsApi.update(dealId, { value }),
    onSuccess: () => {
      invalidateDeal();
      toast({ title: 'Valor atualizado' });
    },
  });

  const markAsWon = useMutation({
    mutationFn: () => dealsApi.update(dealId, { status: 'won' }),
    onSuccess: () => {
      invalidateDeal();
      toast({ title: 'Deal marcado como ganho!' });
    },
  });

  const markAsLost = useMutation({
    mutationFn: (reason: string) => dealsApi.update(dealId, { status: 'lost', lostReason: reason }),
    onSuccess: () => {
      invalidateDeal();
      toast({ title: 'Deal marcado como perdido' });
    },
  });

  const addNote = useMutation({
    mutationFn: (content: string) => dealsApi.addNote(dealId, content),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['deals', dealId, 'notes'] });
      toast({ title: 'Nota adicionada' });
    },
  });

  return {
    moveToStage,
    updateValue,
    markAsWon,
    markAsLost,
    addNote,
  };
}
```

---

## Checklist

- [ ] Criar pasta `DealDetails/`
- [ ] Criar `DealDetailsDrawer.tsx`
- [ ] Criar `DealHeader.tsx`
- [ ] Criar `DealTabs.tsx`
- [ ] Criar tabs (Timeline, Notes, Attachments, Activities)
- [ ] Criar components (StageSelector, DealValue, DealActions)
- [ ] Criar hooks
- [ ] Atualizar imports
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
