# Refatoração: Followup Worker

**Arquivo:** `server/src/jobs/followup.worker.ts`
**Linhas:** 613
**Prioridade:** P2 - MÉDIO
**Estimativa:** 1 dia

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 613 | 400 | 🟠 ALTO |
| Responsabilidades | 4+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **Rule Processing** - Processar regras de followup
2. **Condition Evaluation** - Avaliar condições
3. **Action Execution** - Executar ações (enviar mensagem, tag, etc)
4. **Scheduling** - Agendar próximas execuções

---

## Estrutura Proposta

```
server/src/jobs/
├── followup.worker.ts            # Worker principal (~80 linhas)
├── workers/
│   └── followup/
│       ├── index.ts
│       ├── followup-processor.ts # Coordenador (~150 linhas)
│       ├── condition-evaluator.ts # Condições (~150 linhas)
│       ├── action-executor.ts    # Ações (~200 linhas)
│       └── schedule-manager.ts   # Agendamento (~80 linhas)
```

---

## Código Refatorado

### Worker Principal
```typescript
// server/src/jobs/followup.worker.ts

import { Worker, Job } from 'bullmq';
import { connection } from './queues';
import { FollowupProcessor } from './workers/followup/followup-processor';

const processor = new FollowupProcessor();

const worker = new Worker(
  'followup',
  async (job: Job) => {
    return processor.process(job.data);
  },
  {
    connection,
    concurrency: 10,
  }
);

export { worker as followupWorker };
```

### Followup Processor
```typescript
// server/src/jobs/workers/followup/followup-processor.ts

import { prisma } from '../../../config/database';
import { ConditionEvaluator } from './condition-evaluator';
import { ActionExecutor } from './action-executor';
import { ScheduleManager } from './schedule-manager';

interface FollowupJobData {
  ruleId: string;
  contactId: string;
  conversationId?: string;
  companyId: string;
}

export class FollowupProcessor {
  private conditionEvaluator = new ConditionEvaluator();
  private actionExecutor = new ActionExecutor();
  private scheduleManager = new ScheduleManager();

  async process(data: FollowupJobData): Promise<void> {
    const { ruleId, contactId, companyId } = data;

    // 1. Buscar regra
    const rule = await prisma.followupRule.findUnique({
      where: { id: ruleId },
      include: { conditions: true, actions: true },
    });

    if (!rule || !rule.active) return;

    // 2. Buscar contato
    const contact = await prisma.contact.findUnique({
      where: { id: contactId },
      include: { tags: true, deals: true },
    });

    if (!contact) return;

    // 3. Avaliar condições
    const conditionsMet = await this.conditionEvaluator.evaluate(
      rule.conditions,
      contact,
      data.conversationId
    );

    if (!conditionsMet) {
      await this.logExecution(ruleId, contactId, 'skipped', 'Conditions not met');
      return;
    }

    // 4. Executar ações
    for (const action of rule.actions) {
      await this.actionExecutor.execute(action, contact, companyId);
    }

    // 5. Agendar próxima execução se recorrente
    if (rule.isRecurring && rule.intervalDays) {
      await this.scheduleManager.scheduleNext(ruleId, contactId, rule.intervalDays);
    }

    await this.logExecution(ruleId, contactId, 'completed');
  }

  private async logExecution(ruleId: string, contactId: string, status: string, reason?: string) {
    await prisma.followupExecution.create({
      data: { ruleId, contactId, status, reason, executedAt: new Date() },
    });
  }
}
```

### Condition Evaluator
```typescript
// server/src/jobs/workers/followup/condition-evaluator.ts

import { prisma } from '../../../config/database';

interface Condition {
  type: string;
  operator: string;
  value: any;
}

export class ConditionEvaluator {
  async evaluate(conditions: Condition[], contact: any, conversationId?: string): Promise<boolean> {
    for (const condition of conditions) {
      const result = await this.evaluateSingle(condition, contact, conversationId);
      if (!result) return false; // AND logic
    }
    return true;
  }

  private async evaluateSingle(condition: Condition, contact: any, conversationId?: string): Promise<boolean> {
    switch (condition.type) {
      case 'tag':
        return this.evaluateTag(condition, contact);

      case 'sentiment':
        return this.evaluateSentiment(condition, contact);

      case 'last_message':
        return this.evaluateLastMessage(condition, conversationId);

      case 'deal_stage':
        return this.evaluateDealStage(condition, contact);

      case 'time_since_contact':
        return this.evaluateTimeSinceContact(condition, contact);

      default:
        console.warn(`Unknown condition type: ${condition.type}`);
        return true;
    }
  }

  private evaluateTag(condition: Condition, contact: any): boolean {
    const contactTagIds = contact.tags.map((t: any) => t.id);

    switch (condition.operator) {
      case 'has':
        return contactTagIds.includes(condition.value);
      case 'not_has':
        return !contactTagIds.includes(condition.value);
      case 'has_any':
        return condition.value.some((id: string) => contactTagIds.includes(id));
      default:
        return false;
    }
  }

  private evaluateSentiment(condition: Condition, contact: any): boolean {
    switch (condition.operator) {
      case 'equals':
        return contact.sentiment === condition.value;
      case 'not_equals':
        return contact.sentiment !== condition.value;
      default:
        return false;
    }
  }

  private async evaluateLastMessage(condition: Condition, conversationId?: string): Promise<boolean> {
    if (!conversationId) return false;

    const lastMessage = await prisma.message.findFirst({
      where: { conversationId },
      orderBy: { createdAt: 'desc' },
    });

    if (!lastMessage) return false;

    const hoursSinceMessage = (Date.now() - lastMessage.createdAt.getTime()) / (1000 * 60 * 60);

    switch (condition.operator) {
      case 'older_than_hours':
        return hoursSinceMessage > condition.value;
      case 'newer_than_hours':
        return hoursSinceMessage < condition.value;
      default:
        return false;
    }
  }

  private evaluateDealStage(condition: Condition, contact: any): boolean {
    const dealStageIds = contact.deals.map((d: any) => d.stageId);

    switch (condition.operator) {
      case 'in_stage':
        return dealStageIds.includes(condition.value);
      case 'not_in_stage':
        return !dealStageIds.includes(condition.value);
      default:
        return false;
    }
  }

  private evaluateTimeSinceContact(condition: Condition, contact: any): boolean {
    const daysSinceContact = (Date.now() - contact.lastContactAt?.getTime()) / (1000 * 60 * 60 * 24);

    switch (condition.operator) {
      case 'more_than_days':
        return daysSinceContact > condition.value;
      case 'less_than_days':
        return daysSinceContact < condition.value;
      default:
        return false;
    }
  }
}
```

### Action Executor
```typescript
// server/src/jobs/workers/followup/action-executor.ts

import { whatsappService } from '../../../modules/whatsapp/whatsapp.service';
import { contactsService } from '../../../modules/contacts/contacts.service';
import { dealsService } from '../../../modules/deals/deals.service';

interface Action {
  type: string;
  config: any;
}

export class ActionExecutor {
  async execute(action: Action, contact: any, companyId: string): Promise<void> {
    switch (action.type) {
      case 'send_message':
        await this.sendMessage(action.config, contact, companyId);
        break;

      case 'assign_tag':
        await this.assignTag(action.config, contact, companyId);
        break;

      case 'remove_tag':
        await this.removeTag(action.config, contact, companyId);
        break;

      case 'move_deal_stage':
        await this.moveDealStage(action.config, contact, companyId);
        break;

      case 'send_notification':
        await this.sendNotification(action.config, contact, companyId);
        break;

      default:
        console.warn(`Unknown action type: ${action.type}`);
    }
  }

  private async sendMessage(config: any, contact: any, companyId: string) {
    const { template, instanceId } = config;
    const message = this.interpolateTemplate(template, contact);

    await whatsappService.sendMessage(companyId, instanceId, contact.phone, message);
  }

  private async assignTag(config: any, contact: any, companyId: string) {
    await contactsService.assignTags(companyId, contact.id, [config.tagId]);
  }

  private async removeTag(config: any, contact: any, companyId: string) {
    await contactsService.removeTags(companyId, contact.id, [config.tagId]);
  }

  private async moveDealStage(config: any, contact: any, companyId: string) {
    const deal = contact.deals.find((d: any) => d.funnelId === config.funnelId);
    if (deal) {
      await dealsService.moveToStage(companyId, deal.id, config.stageId);
    }
  }

  private async sendNotification(config: any, contact: any, companyId: string) {
    // TODO: Implement notification logic
  }

  private interpolateTemplate(template: string, contact: any): string {
    return template
      .replace(/\{\{name\}\}/g, contact.name)
      .replace(/\{\{phone\}\}/g, contact.phone)
      .replace(/\{\{email\}\}/g, contact.email || '');
  }
}
```

---

## Checklist

- [ ] Criar pasta `workers/followup/`
- [ ] Criar `followup-processor.ts`
- [ ] Criar `condition-evaluator.ts`
- [ ] Criar `action-executor.ts`
- [ ] Criar `schedule-manager.ts`
- [ ] Refatorar `followup.worker.ts`
- [ ] Testes

---

**Status:** ✅ Completo
**Última Atualização:** 2026-01-23
