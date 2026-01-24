# Refatoração: Deals Service

**Arquivo:** `server/src/modules/deals/deals.service.ts`
**Linhas:** 982
**Prioridade:** P1 - ALTO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 982 | 400 | 🔴 CRÍTICO |
| Métodos | 53 | 10 | 🔴 CRÍTICO |
| Responsabilidades | 5+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **CRUD básico** - Criar, atualizar, deletar deals
2. **Gerenciamento de stages** - Mover entre estágios
3. **Notas e atividades** - Histórico do deal
4. **Anexos** - Upload e gerenciamento de arquivos
5. **Analytics** - Métricas e conversão

---

## Estrutura Proposta

```
server/src/modules/deals/
├── deals.service.ts              # Facade (~150 linhas)
├── deals.routes.ts               # Existente
├── deals.schemas.ts              # Validação Zod
├── deals.types.ts                # Tipos (~100 linhas)
├── services/
│   ├── index.ts
│   ├── deal-crud.service.ts      # CRUD básico (~200 linhas)
│   ├── deal-stage.service.ts     # Gerenciamento de stages (~200 linhas)
│   ├── deal-notes.service.ts     # Notas e atividades (~150 linhas)
│   ├── deal-attachments.service.ts # Anexos (~150 linhas)
│   └── deal-analytics.service.ts # Métricas (~150 linhas)
└── helpers/
    └── deal-value-calculator.ts  # Cálculos de valor (~50 linhas)
```

---

## Mapeamento de Métodos

### deal-crud.service.ts
```typescript
export class DealCrudService {
  async list(companyId: string, filters: DealFilters)
  async getById(companyId: string, dealId: string)
  async getByContact(companyId: string, contactId: string)
  async create(companyId: string, data: CreateDealDTO)
  async update(companyId: string, dealId: string, data: UpdateDealDTO)
  async delete(companyId: string, dealId: string)
  async bulkUpdate(companyId: string, dealIds: string[], data: BulkUpdateDTO)
}
```

### deal-stage.service.ts
```typescript
export class DealStageService {
  async moveToStage(companyId: string, dealId: string, stageId: string, reason?: string)
  async markAsWon(companyId: string, dealId: string, wonReason?: string)
  async markAsLost(companyId: string, dealId: string, lostReason: string)
  async reopen(companyId: string, dealId: string, stageId?: string)
  async getStageHistory(companyId: string, dealId: string)
  async getDealsInStage(companyId: string, stageId: string)
  async bulkMoveToStage(companyId: string, dealIds: string[], stageId: string)
}
```

### deal-notes.service.ts
```typescript
export class DealNotesService {
  async addNote(companyId: string, dealId: string, note: CreateNoteDTO)
  async updateNote(companyId: string, noteId: string, content: string)
  async deleteNote(companyId: string, noteId: string)
  async getNotes(companyId: string, dealId: string)
  async addActivity(companyId: string, dealId: string, activity: CreateActivityDTO)
  async getActivities(companyId: string, dealId: string)
  async getTimeline(companyId: string, dealId: string) // Notes + Activities combined
}
```

### deal-attachments.service.ts
```typescript
export class DealAttachmentsService {
  async addAttachment(companyId: string, dealId: string, attachment: AttachmentDTO)
  async removeAttachment(companyId: string, attachmentId: string)
  async getAttachments(companyId: string, dealId: string)
  async getAttachmentUrl(companyId: string, attachmentId: string)
}
```

### deal-analytics.service.ts
```typescript
export class DealAnalyticsService {
  async getFunnelStats(companyId: string, funnelId: string, period?: DateRange)
  async getConversionRate(companyId: string, funnelId: string)
  async getAverageTimeInStage(companyId: string, funnelId: string)
  async getTotalValue(companyId: string, filters?: DealFilters)
  async getWonDealsValue(companyId: string, period: DateRange)
  async getPipelineValue(companyId: string, funnelId: string)
  async getForecast(companyId: string, funnelId: string)
}
```

---

## Código: Stage Service

```typescript
// server/src/modules/deals/services/deal-stage.service.ts

import { prisma } from '../../../config/database';
import type { Deal, DealStageHistory } from '../deals.types';

export class DealStageService {
  async moveToStage(
    companyId: string,
    dealId: string,
    stageId: string,
    reason?: string
  ): Promise<Deal> {
    const deal = await prisma.deal.findFirst({
      where: { id: dealId, companyId },
      include: { stage: true },
    });

    if (!deal) {
      throw new Error('Deal not found');
    }

    // Validar que o stage pertence ao mesmo funil
    const stage = await prisma.funnelStage.findFirst({
      where: { id: stageId, funnelId: deal.funnelId },
    });

    if (!stage) {
      throw new Error('Stage not found in this funnel');
    }

    // Registrar histórico
    await prisma.dealStageHistory.create({
      data: {
        dealId,
        fromStageId: deal.stageId,
        toStageId: stageId,
        reason,
        movedAt: new Date(),
      },
    });

    // Atualizar deal
    const updated = await prisma.deal.update({
      where: { id: dealId },
      data: {
        stageId,
        stageChangedAt: new Date(),
      },
      include: {
        stage: true,
        funnel: true,
        contact: true,
      },
    });

    return updated;
  }

  async markAsWon(companyId: string, dealId: string, reason?: string): Promise<Deal> {
    const deal = await prisma.deal.findFirst({
      where: { id: dealId, companyId },
    });

    if (!deal) {
      throw new Error('Deal not found');
    }

    return prisma.deal.update({
      where: { id: dealId },
      data: {
        status: 'won',
        closedAt: new Date(),
        wonReason: reason,
      },
      include: { stage: true, funnel: true, contact: true },
    });
  }

  async markAsLost(companyId: string, dealId: string, reason: string): Promise<Deal> {
    const deal = await prisma.deal.findFirst({
      where: { id: dealId, companyId },
    });

    if (!deal) {
      throw new Error('Deal not found');
    }

    return prisma.deal.update({
      where: { id: dealId },
      data: {
        status: 'lost',
        closedAt: new Date(),
        lostReason: reason,
      },
      include: { stage: true, funnel: true, contact: true },
    });
  }

  async getStageHistory(companyId: string, dealId: string): Promise<DealStageHistory[]> {
    // Verificar acesso
    const deal = await prisma.deal.findFirst({
      where: { id: dealId, companyId },
    });

    if (!deal) {
      throw new Error('Deal not found');
    }

    return prisma.dealStageHistory.findMany({
      where: { dealId },
      include: {
        fromStage: true,
        toStage: true,
      },
      orderBy: { movedAt: 'desc' },
    });
  }
}
```

---

## Checklist

### Estrutura
- [ ] Criar `deals.types.ts`
- [ ] Criar `deals.schemas.ts`
- [ ] Criar pasta `services/`
- [ ] Criar pasta `helpers/`

### Services
- [ ] `deal-crud.service.ts`
- [ ] `deal-stage.service.ts`
- [ ] `deal-notes.service.ts`
- [ ] `deal-attachments.service.ts`
- [ ] `deal-analytics.service.ts`

### Helpers
- [ ] `deal-value-calculator.ts`

### Finalização
- [ ] Refatorar `deals.service.ts` para Facade
- [ ] Atualizar imports
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
