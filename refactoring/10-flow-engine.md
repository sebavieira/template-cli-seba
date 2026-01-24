# Refatoração: Flow Engine Service

**Arquivo:** `server/src/modules/flows/flow-engine.service.ts`
**Linhas:** 674
**Prioridade:** P1 - ALTO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 674 | 400 | 🟠 ALTO |
| Maior método | 280 | 30 | 🔴 CRÍTICO |
| Responsabilidades | 4+ | 1 | 🟠 ALTO |

### Métodos Problemáticos

- **processNode()** - 280 linhas
- **moveNext()** - 200 linhas

### Responsabilidades Misturadas

1. **Trigger** - Detectar e disparar flows
2. **Execution State** - Gerenciar estado de execução
3. **Node Processing** - Processar cada tipo de nó
4. **Navigation** - Navegar entre nós

---

## Estrutura Proposta

```
server/src/modules/flows/
├── flow-engine.service.ts        # Coordinator (~150 linhas)
├── flows.service.ts              # CRUD existente
├── flows.routes.ts               # Existente
├── flows.types.ts                # Tipos (~100 linhas)
├── services/
│   ├── index.ts
│   ├── flow-trigger.service.ts   # Trigger de flows (~150 linhas)
│   ├── flow-state.service.ts     # Estado de execução (~200 linhas)
│   ├── flow-navigator.service.ts # Navegação (~150 linhas)
│   └── flow-metrics.service.ts   # Métricas (~100 linhas)
├── processors/
│   ├── index.ts                  # Registry de processors
│   ├── processor.interface.ts    # Interface base
│   ├── message.processor.ts      # Nó de mensagem (~80 linhas)
│   ├── condition.processor.ts    # Nó de condição (~100 linhas)
│   ├── delay.processor.ts        # Nó de delay (~60 linhas)
│   ├── action.processor.ts       # Nó de ação (~100 linhas)
│   ├── ai.processor.ts           # Nó de AI (~80 linhas)
│   └── webhook.processor.ts      # Nó de webhook (~80 linhas)
└── helpers/
    ├── condition-evaluator.ts    # Avaliar condições (~100 linhas)
    └── variable-resolver.ts      # Resolver variáveis (~80 linhas)
```

---

## Padrão: Strategy para Processors

### Interface Base

```typescript
// server/src/modules/flows/processors/processor.interface.ts

import type { FlowNode, FlowExecution, ProcessorResult } from '../flows.types';

export interface ProcessorContext {
  execution: FlowExecution;
  contact: Contact;
  conversation?: Conversation;
  variables: Record<string, any>;
}

export interface ProcessorResult {
  success: boolean;
  output?: any;
  nextNodeId?: string;
  error?: string;
  delay?: number; // ms to wait before next node
}

export interface INodeProcessor {
  type: string;
  process(node: FlowNode, context: ProcessorContext): Promise<ProcessorResult>;
}

export abstract class BaseProcessor implements INodeProcessor {
  abstract type: string;
  abstract process(node: FlowNode, context: ProcessorContext): Promise<ProcessorResult>;

  protected success(output?: any, nextNodeId?: string): ProcessorResult {
    return { success: true, output, nextNodeId };
  }

  protected error(message: string): ProcessorResult {
    return { success: false, error: message };
  }

  protected delay(ms: number, nextNodeId?: string): ProcessorResult {
    return { success: true, delay: ms, nextNodeId };
  }
}
```

### Message Processor

```typescript
// server/src/modules/flows/processors/message.processor.ts

import { BaseProcessor, ProcessorContext, ProcessorResult } from './processor.interface';
import { whatsappService } from '../../whatsapp/whatsapp.service';
import { resolveVariables } from '../helpers/variable-resolver';
import type { FlowNode } from '../flows.types';

export class MessageProcessor extends BaseProcessor {
  type = 'message';

  async process(node: FlowNode, context: ProcessorContext): Promise<ProcessorResult> {
    const { execution, contact, conversation } = context;
    const { template, mediaUrl, mediaType } = node.data;

    // Resolver variáveis no template
    const resolvedMessage = resolveVariables(template, {
      contact,
      conversation,
      variables: context.variables,
    });

    try {
      if (mediaUrl) {
        await whatsappService.sendMediaMessage(
          execution.companyId,
          conversation?.instanceId!,
          contact.phone,
          { url: mediaUrl, type: mediaType },
          resolvedMessage
        );
      } else {
        await whatsappService.sendMessage(
          execution.companyId,
          conversation?.instanceId!,
          contact.phone,
          resolvedMessage
        );
      }

      return this.success({ messageSent: true });
    } catch (error) {
      return this.error(`Failed to send message: ${error.message}`);
    }
  }
}
```

### Condition Processor

```typescript
// server/src/modules/flows/processors/condition.processor.ts

import { BaseProcessor, ProcessorContext, ProcessorResult } from './processor.interface';
import { evaluateCondition } from '../helpers/condition-evaluator';
import type { FlowNode } from '../flows.types';

export class ConditionProcessor extends BaseProcessor {
  type = 'condition';

  async process(node: FlowNode, context: ProcessorContext): Promise<ProcessorResult> {
    const { conditions, trueNodeId, falseNodeId } = node.data;

    const result = await evaluateCondition(conditions, context);

    return this.success(
      { conditionResult: result },
      result ? trueNodeId : falseNodeId
    );
  }
}
```

### Delay Processor

```typescript
// server/src/modules/flows/processors/delay.processor.ts

import { BaseProcessor, ProcessorContext, ProcessorResult } from './processor.interface';
import type { FlowNode } from '../flows.types';

export class DelayProcessor extends BaseProcessor {
  type = 'delay';

  async process(node: FlowNode, context: ProcessorContext): Promise<ProcessorResult> {
    const { duration, unit } = node.data;

    const msMultipliers: Record<string, number> = {
      seconds: 1000,
      minutes: 60 * 1000,
      hours: 60 * 60 * 1000,
      days: 24 * 60 * 60 * 1000,
    };

    const delayMs = duration * (msMultipliers[unit] || 1000);

    return this.delay(delayMs);
  }
}
```

### Registry

```typescript
// server/src/modules/flows/processors/index.ts

import { INodeProcessor } from './processor.interface';
import { MessageProcessor } from './message.processor';
import { ConditionProcessor } from './condition.processor';
import { DelayProcessor } from './delay.processor';
import { ActionProcessor } from './action.processor';
import { AIProcessor } from './ai.processor';
import { WebhookProcessor } from './webhook.processor';

const processors: INodeProcessor[] = [
  new MessageProcessor(),
  new ConditionProcessor(),
  new DelayProcessor(),
  new ActionProcessor(),
  new AIProcessor(),
  new WebhookProcessor(),
];

export const processorRegistry = new Map<string, INodeProcessor>(
  processors.map(p => [p.type, p])
);

export function getProcessor(type: string): INodeProcessor | undefined {
  return processorRegistry.get(type);
}
```

---

## Flow Engine Refatorado

```typescript
// server/src/modules/flows/flow-engine.service.ts

import { FlowTriggerService } from './services/flow-trigger.service';
import { FlowStateService } from './services/flow-state.service';
import { FlowNavigatorService } from './services/flow-navigator.service';
import { getProcessor } from './processors';
import type { FlowExecution, ProcessorContext } from './flows.types';

export class FlowEngineService {
  constructor(
    private trigger: FlowTriggerService,
    private state: FlowStateService,
    private navigator: FlowNavigatorService,
  ) {}

  async triggerFlow(flowId: string, contactId: string, conversationId?: string) {
    return this.trigger.trigger(flowId, contactId, conversationId);
  }

  async processNode(executionId: string): Promise<void> {
    const execution = await this.state.getExecution(executionId);
    if (!execution || execution.status !== 'running') return;

    const node = await this.navigator.getCurrentNode(execution);
    if (!node) {
      await this.state.completeExecution(executionId);
      return;
    }

    const processor = getProcessor(node.type);
    if (!processor) {
      await this.state.failExecution(executionId, `Unknown node type: ${node.type}`);
      return;
    }

    const context = await this.buildContext(execution);
    const result = await processor.process(node, context);

    if (!result.success) {
      await this.state.failExecution(executionId, result.error);
      return;
    }

    if (result.delay) {
      await this.state.scheduleResume(executionId, result.delay);
      return;
    }

    const nextNodeId = result.nextNodeId || await this.navigator.getNextNode(execution, node);

    if (nextNodeId) {
      await this.state.moveToNode(executionId, nextNodeId);
      await this.processNode(executionId); // Recursivo para próximo nó
    } else {
      await this.state.completeExecution(executionId);
    }
  }

  private async buildContext(execution: FlowExecution): Promise<ProcessorContext> {
    // Build context with contact, conversation, variables
  }
}
```

---

## Checklist

### Estrutura
- [ ] Criar `flows.types.ts` (expandir)
- [ ] Criar pasta `services/`
- [ ] Criar pasta `processors/`
- [ ] Criar pasta `helpers/`

### Processors
- [ ] `processor.interface.ts`
- [ ] `message.processor.ts`
- [ ] `condition.processor.ts`
- [ ] `delay.processor.ts`
- [ ] `action.processor.ts`
- [ ] `ai.processor.ts`
- [ ] `webhook.processor.ts`
- [ ] `processors/index.ts`

### Services
- [ ] `flow-trigger.service.ts`
- [ ] `flow-state.service.ts`
- [ ] `flow-navigator.service.ts`
- [ ] `flow-metrics.service.ts`

### Helpers
- [ ] `condition-evaluator.ts`
- [ ] `variable-resolver.ts`

### Finalização
- [ ] Refatorar `flow-engine.service.ts`
- [ ] Atualizar `node-executors.service.ts`
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
