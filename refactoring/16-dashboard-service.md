# Refatoração: Dashboard Service

**Arquivo:** `server/src/modules/dashboard/dashboard.service.ts`
**Linhas:** 624
**Prioridade:** P2 - MÉDIO
**Estimativa:** 1 dia

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 624 | 400 | 🟠 ALTO |
| Responsabilidades | 4+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **Conversation Stats** - Métricas de conversas
2. **Message Stats** - Métricas de mensagens
3. **Funnel Stats** - Métricas de funis
4. **AI Stats** - Uso de AI/tokens

---

## Estrutura Proposta

```
server/src/modules/dashboard/
├── dashboard.service.ts          # Facade (~100 linhas)
├── dashboard.routes.ts           # Existente
├── services/
│   ├── index.ts
│   ├── conversation-stats.service.ts (~150 linhas)
│   ├── message-stats.service.ts  (~120 linhas)
│   ├── funnel-stats.service.ts   (~150 linhas)
│   └── ai-stats.service.ts       (~100 linhas)
└── helpers/
    └── date-range.helper.ts      # Helpers de data (~50 linhas)
```

---

## Código Refatorado

### Facade
```typescript
// server/src/modules/dashboard/dashboard.service.ts

import { ConversationStatsService } from './services/conversation-stats.service';
import { MessageStatsService } from './services/message-stats.service';
import { FunnelStatsService } from './services/funnel-stats.service';
import { AIStatsService } from './services/ai-stats.service';

interface DashboardFilters {
  startDate?: Date;
  endDate?: Date;
  instanceId?: string;
}

export class DashboardService {
  constructor(
    private conversations = new ConversationStatsService(),
    private messages = new MessageStatsService(),
    private funnels = new FunnelStatsService(),
    private ai = new AIStatsService(),
  ) {}

  async getOverview(companyId: string, filters?: DashboardFilters) {
    const [conversations, messages, funnels, aiUsage] = await Promise.all([
      this.conversations.getSummary(companyId, filters),
      this.messages.getSummary(companyId, filters),
      this.funnels.getSummary(companyId, filters),
      this.ai.getSummary(companyId, filters),
    ]);

    return { conversations, messages, funnels, aiUsage };
  }

  // Delegates para rotas específicas
  getConversationStats = this.conversations.getDetailed.bind(this.conversations);
  getMessageStats = this.messages.getDetailed.bind(this.messages);
  getFunnelStats = this.funnels.getDetailed.bind(this.funnels);
  getAIStats = this.ai.getDetailed.bind(this.ai);
}

export const dashboardService = new DashboardService();
```

### Conversation Stats
```typescript
// server/src/modules/dashboard/services/conversation-stats.service.ts

import { prisma } from '../../../config/database';
import { getDateRange, DateRangeType } from '../helpers/date-range.helper';

interface ConversationSummary {
  total: number;
  active: number;
  newToday: number;
  avgResponseTime: number;
  byStatus: Record<string, number>;
}

export class ConversationStatsService {
  async getSummary(companyId: string, filters?: any): Promise<ConversationSummary> {
    const { startDate, endDate } = getDateRange(filters);

    const [total, active, newToday, byStatus] = await Promise.all([
      this.getTotalConversations(companyId, startDate, endDate),
      this.getActiveConversations(companyId),
      this.getNewConversationsToday(companyId),
      this.getConversationsByStatus(companyId, startDate, endDate),
    ]);

    const avgResponseTime = await this.getAverageResponseTime(companyId, startDate, endDate);

    return { total, active, newToday, avgResponseTime, byStatus };
  }

  async getDetailed(companyId: string, filters?: any) {
    const { startDate, endDate } = getDateRange(filters);

    return {
      summary: await this.getSummary(companyId, filters),
      timeline: await this.getTimeline(companyId, startDate, endDate),
      byInstance: await this.getByInstance(companyId, startDate, endDate),
      topContacts: await this.getTopContacts(companyId, startDate, endDate),
    };
  }

  private async getTotalConversations(companyId: string, start: Date, end: Date) {
    return prisma.conversation.count({
      where: {
        companyId,
        createdAt: { gte: start, lte: end },
      },
    });
  }

  private async getActiveConversations(companyId: string) {
    return prisma.conversation.count({
      where: {
        companyId,
        status: 'open',
        archived: false,
      },
    });
  }

  private async getNewConversationsToday(companyId: string) {
    const today = new Date();
    today.setHours(0, 0, 0, 0);

    return prisma.conversation.count({
      where: {
        companyId,
        createdAt: { gte: today },
      },
    });
  }

  private async getConversationsByStatus(companyId: string, start: Date, end: Date) {
    const results = await prisma.conversation.groupBy({
      by: ['status'],
      where: {
        companyId,
        createdAt: { gte: start, lte: end },
      },
      _count: true,
    });

    return results.reduce((acc, r) => {
      acc[r.status] = r._count;
      return acc;
    }, {} as Record<string, number>);
  }

  private async getAverageResponseTime(companyId: string, start: Date, end: Date) {
    // Calculate average time between customer message and response
    // This is a simplified version
    return 0; // TODO: Implement
  }

  private async getTimeline(companyId: string, start: Date, end: Date) {
    // Get conversations by day
    return []; // TODO: Implement
  }

  private async getByInstance(companyId: string, start: Date, end: Date) {
    return prisma.conversation.groupBy({
      by: ['instanceId'],
      where: {
        companyId,
        createdAt: { gte: start, lte: end },
      },
      _count: true,
    });
  }

  private async getTopContacts(companyId: string, start: Date, end: Date, limit = 10) {
    return prisma.conversation.findMany({
      where: {
        companyId,
        createdAt: { gte: start, lte: end },
      },
      include: { contact: true },
      orderBy: { messageCount: 'desc' },
      take: limit,
    });
  }
}
```

### Date Range Helper
```typescript
// server/src/modules/dashboard/helpers/date-range.helper.ts

export type DateRangeType = 'today' | 'yesterday' | 'last7days' | 'last30days' | 'thisMonth' | 'lastMonth' | 'custom';

interface DateRange {
  startDate: Date;
  endDate: Date;
}

export function getDateRange(filters?: { startDate?: Date; endDate?: Date; range?: DateRangeType }): DateRange {
  if (filters?.startDate && filters?.endDate) {
    return { startDate: filters.startDate, endDate: filters.endDate };
  }

  const now = new Date();
  const range = filters?.range || 'last30days';

  switch (range) {
    case 'today':
      const today = new Date(now);
      today.setHours(0, 0, 0, 0);
      return { startDate: today, endDate: now };

    case 'yesterday':
      const yesterday = new Date(now);
      yesterday.setDate(yesterday.getDate() - 1);
      yesterday.setHours(0, 0, 0, 0);
      const yesterdayEnd = new Date(yesterday);
      yesterdayEnd.setHours(23, 59, 59, 999);
      return { startDate: yesterday, endDate: yesterdayEnd };

    case 'last7days':
      const last7 = new Date(now);
      last7.setDate(last7.getDate() - 7);
      return { startDate: last7, endDate: now };

    case 'last30days':
    default:
      const last30 = new Date(now);
      last30.setDate(last30.getDate() - 30);
      return { startDate: last30, endDate: now };

    case 'thisMonth':
      const thisMonthStart = new Date(now.getFullYear(), now.getMonth(), 1);
      return { startDate: thisMonthStart, endDate: now };

    case 'lastMonth':
      const lastMonthStart = new Date(now.getFullYear(), now.getMonth() - 1, 1);
      const lastMonthEnd = new Date(now.getFullYear(), now.getMonth(), 0);
      return { startDate: lastMonthStart, endDate: lastMonthEnd };
  }
}

export function formatDateForQuery(date: Date): string {
  return date.toISOString().split('T')[0];
}
```

---

## Checklist

- [ ] Criar pasta `services/`
- [ ] Criar `conversation-stats.service.ts`
- [ ] Criar `message-stats.service.ts`
- [ ] Criar `funnel-stats.service.ts`
- [ ] Criar `ai-stats.service.ts`
- [ ] Criar `helpers/date-range.helper.ts`
- [ ] Refatorar `dashboard.service.ts`
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
