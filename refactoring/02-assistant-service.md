# Refatoração: Assistant Service

**Arquivo:** `server/src/modules/ai/assistant.service.ts`
**Linhas:** 2.612
**Prioridade:** P0 - CRÍTICO
**Estimativa:** 3-4 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 2.612 | 400 | 🔴 CRÍTICO |
| Maior método | 2.100+ | 30 | 🔴 CRÍTICO |
| Imports | 16 | 10 | 🔴 CRÍTICO |
| Cases no switch | 40+ | 5 | 🔴 CRÍTICO |

### O Problema Principal: `executeTool()`

```typescript
// ATUAL: Um método gigante com 40+ cases
async executeTool(toolName: string, args: any, context: ToolContext) {
  switch (toolName) {
    case 'list_contacts': /* 50 linhas */ break;
    case 'create_contact': /* 60 linhas */ break;
    case 'update_contact': /* 40 linhas */ break;
    case 'list_tags': /* 30 linhas */ break;
    case 'create_tag': /* 40 linhas */ break;
    case 'assign_tags': /* 50 linhas */ break;
    case 'list_funnels': /* 30 linhas */ break;
    case 'list_stages': /* 30 linhas */ break;
    case 'create_deal': /* 80 linhas */ break;
    case 'update_deal_stage': /* 60 linhas */ break;
    case 'list_flows': /* 30 linhas */ break;
    case 'trigger_flow': /* 50 linhas */ break;
    case 'create_followup_rule': /* 100 linhas */ break;
    // ... mais 27 cases
    default: throw new Error(`Unknown tool: ${toolName}`);
  }
}
```

### Responsabilidades Misturadas

1. **Chat/Conversation** - Gerenciar conversas com AI
2. **Tool Definitions** - Definir schemas das ferramentas
3. **Tool Execution** - Executar 40+ ferramentas diferentes
4. **Memory** - Gerenciar memória de conversa
5. **Integrations** - Contatos, Funnels, Deals, Flows, Followups

---

## Estrutura Proposta

```
server/src/modules/ai/
├── assistant.service.ts         # Core chat logic (~400 linhas)
├── assistant.types.ts           # Tipos compartilhados (~150 linhas)
├── services/
│   ├── index.ts                 # Re-exports
│   └── tool-executor.service.ts # Coordinator (~200 linhas)
├── tools/
│   ├── index.ts                 # Registry de tools
│   ├── tool.interface.ts        # Interface base
│   ├── contact.tools.ts         # Ferramentas de contatos (~200 linhas)
│   ├── tag.tools.ts             # Ferramentas de tags (~150 linhas)
│   ├── funnel.tools.ts          # Ferramentas de funis (~200 linhas)
│   ├── deal.tools.ts            # Ferramentas de deals (~250 linhas)
│   ├── flow.tools.ts            # Ferramentas de flows (~200 linhas)
│   ├── followup.tools.ts        # Ferramentas de followups (~200 linhas)
│   └── message.tools.ts         # Ferramentas de mensagens (~150 linhas)
└── definitions/
    └── openai-tools.ts          # Schemas OpenAI (~400 linhas)
```

---

## Padrão: Strategy Pattern para Tools

### Interface Base

```typescript
// server/src/modules/ai/tools/tool.interface.ts

export interface ToolContext {
  companyId: string;
  userId: string;
  conversationId?: string;
  contactId?: string;
}

export interface ToolResult {
  success: boolean;
  data?: any;
  error?: string;
  message?: string;
}

export interface ITool {
  name: string;
  execute(args: Record<string, any>, context: ToolContext): Promise<ToolResult>;
}

export abstract class BaseTool implements ITool {
  abstract name: string;
  abstract execute(args: Record<string, any>, context: ToolContext): Promise<ToolResult>;

  protected success(data: any, message?: string): ToolResult {
    return { success: true, data, message };
  }

  protected error(message: string): ToolResult {
    return { success: false, error: message };
  }
}
```

### Registry de Tools

```typescript
// server/src/modules/ai/tools/index.ts

import { ITool } from './tool.interface';
import { ListContactsTool, CreateContactTool, UpdateContactTool } from './contact.tools';
import { ListTagsTool, CreateTagTool, AssignTagsTool } from './tag.tools';
import { ListFunnelsTool, ListStagesTool } from './funnel.tools';
import { CreateDealTool, UpdateDealStageTool } from './deal.tools';
import { ListFlowsTool, TriggerFlowTool } from './flow.tools';
import { CreateFollowupRuleTool } from './followup.tools';
import { SendMessageTool, ScheduleMessageTool } from './message.tools';

// Registry - facilita adicionar novas tools
const tools: ITool[] = [
  // Contacts
  new ListContactsTool(),
  new CreateContactTool(),
  new UpdateContactTool(),

  // Tags
  new ListTagsTool(),
  new CreateTagTool(),
  new AssignTagsTool(),

  // Funnels
  new ListFunnelsTool(),
  new ListStagesTool(),

  // Deals
  new CreateDealTool(),
  new UpdateDealStageTool(),

  // Flows
  new ListFlowsTool(),
  new TriggerFlowTool(),

  // Followups
  new CreateFollowupRuleTool(),

  // Messages
  new SendMessageTool(),
  new ScheduleMessageTool(),
];

// Map para lookup O(1)
export const toolRegistry = new Map<string, ITool>(
  tools.map(tool => [tool.name, tool])
);

export function getTool(name: string): ITool | undefined {
  return toolRegistry.get(name);
}

export function getAllTools(): ITool[] {
  return tools;
}
```

### Exemplo: Contact Tools

```typescript
// server/src/modules/ai/tools/contact.tools.ts

import { BaseTool, ToolContext, ToolResult } from './tool.interface';
import { prisma } from '../../../config/database';

export class ListContactsTool extends BaseTool {
  name = 'list_contacts';

  async execute(args: { search?: string; limit?: number }, context: ToolContext): Promise<ToolResult> {
    const { companyId } = context;
    const { search, limit = 10 } = args;

    const contacts = await prisma.contact.findMany({
      where: {
        companyId,
        ...(search && {
          OR: [
            { name: { contains: search, mode: 'insensitive' } },
            { phone: { contains: search } },
            { email: { contains: search, mode: 'insensitive' } },
          ],
        }),
      },
      take: limit,
      include: { tags: true },
    });

    return this.success(contacts, `Found ${contacts.length} contacts`);
  }
}

export class CreateContactTool extends BaseTool {
  name = 'create_contact';

  async execute(
    args: { name: string; phone: string; email?: string },
    context: ToolContext
  ): Promise<ToolResult> {
    const { companyId } = context;
    const { name, phone, email } = args;

    // Validação
    if (!name || !phone) {
      return this.error('Name and phone are required');
    }

    // Verificar duplicata
    const existing = await prisma.contact.findFirst({
      where: { companyId, phone },
    });

    if (existing) {
      return this.error(`Contact with phone ${phone} already exists`);
    }

    // Criar
    const contact = await prisma.contact.create({
      data: { companyId, name, phone, email },
    });

    return this.success(contact, `Contact "${name}" created successfully`);
  }
}

export class UpdateContactTool extends BaseTool {
  name = 'update_contact';

  async execute(
    args: { contactId: string; name?: string; email?: string },
    context: ToolContext
  ): Promise<ToolResult> {
    const { companyId } = context;
    const { contactId, ...updateData } = args;

    // Verificar existência
    const contact = await prisma.contact.findFirst({
      where: { id: contactId, companyId },
    });

    if (!contact) {
      return this.error('Contact not found');
    }

    // Atualizar
    const updated = await prisma.contact.update({
      where: { id: contactId },
      data: updateData,
    });

    return this.success(updated, `Contact updated successfully`);
  }
}
```

### Exemplo: Deal Tools

```typescript
// server/src/modules/ai/tools/deal.tools.ts

import { BaseTool, ToolContext, ToolResult } from './tool.interface';
import { prisma } from '../../../config/database';

export class CreateDealTool extends BaseTool {
  name = 'create_deal';

  async execute(
    args: {
      contactId: string;
      funnelId: string;
      stageId?: string;
      title?: string;
      value?: number;
    },
    context: ToolContext
  ): Promise<ToolResult> {
    const { companyId } = context;
    const { contactId, funnelId, stageId, title, value } = args;

    // Validar contato
    const contact = await prisma.contact.findFirst({
      where: { id: contactId, companyId },
    });
    if (!contact) {
      return this.error('Contact not found');
    }

    // Validar funil
    const funnel = await prisma.funnel.findFirst({
      where: { id: funnelId, companyId },
      include: { stages: { orderBy: { order: 'asc' } } },
    });
    if (!funnel) {
      return this.error('Funnel not found');
    }

    // Determinar stage (primeiro se não especificado)
    const targetStageId = stageId || funnel.stages[0]?.id;
    if (!targetStageId) {
      return this.error('Funnel has no stages');
    }

    // Verificar deal existente
    const existingDeal = await prisma.deal.findFirst({
      where: { contactId, funnelId },
    });
    if (existingDeal) {
      return this.error('Contact already has a deal in this funnel');
    }

    // Criar deal
    const deal = await prisma.deal.create({
      data: {
        companyId,
        contactId,
        funnelId,
        stageId: targetStageId,
        title: title || `Deal - ${contact.name}`,
        value: value || 0,
      },
      include: { stage: true, funnel: true },
    });

    return this.success(deal, `Deal created in stage "${deal.stage.name}"`);
  }
}

export class UpdateDealStageTool extends BaseTool {
  name = 'update_deal_stage';

  async execute(
    args: { dealId: string; stageId: string },
    context: ToolContext
  ): Promise<ToolResult> {
    const { companyId } = context;
    const { dealId, stageId } = args;

    // Validar deal
    const deal = await prisma.deal.findFirst({
      where: { id: dealId, companyId },
    });
    if (!deal) {
      return this.error('Deal not found');
    }

    // Validar stage (deve ser do mesmo funil)
    const stage = await prisma.funnelStage.findFirst({
      where: { id: stageId, funnelId: deal.funnelId },
    });
    if (!stage) {
      return this.error('Stage not found in this funnel');
    }

    // Atualizar
    const updated = await prisma.deal.update({
      where: { id: dealId },
      data: { stageId },
      include: { stage: true },
    });

    return this.success(updated, `Deal moved to stage "${stage.name}"`);
  }
}
```

---

## Tool Executor Service

```typescript
// server/src/modules/ai/services/tool-executor.service.ts

import { getTool, getAllTools } from '../tools';
import type { ToolContext, ToolResult } from '../tools/tool.interface';

export class ToolExecutorService {
  async execute(
    toolName: string,
    args: Record<string, any>,
    context: ToolContext
  ): Promise<ToolResult> {
    const tool = getTool(toolName);

    if (!tool) {
      return {
        success: false,
        error: `Unknown tool: ${toolName}`,
      };
    }

    try {
      return await tool.execute(args, context);
    } catch (error) {
      console.error(`Tool execution error [${toolName}]:`, error);
      return {
        success: false,
        error: error instanceof Error ? error.message : 'Tool execution failed',
      };
    }
  }

  getAvailableTools(): string[] {
    return getAllTools().map(t => t.name);
  }
}

export const toolExecutor = new ToolExecutorService();
```

---

## Assistant Service Refatorado

```typescript
// server/src/modules/ai/assistant.service.ts (AFTER)

import { OpenAI } from 'openai';
import { prisma } from '../../config/database';
import { toolExecutor } from './services/tool-executor.service';
import { OPENAI_TOOLS } from './definitions/openai-tools';
import type { ToolContext } from './tools/tool.interface';

export class AssistantService {
  private openai: OpenAI;

  constructor() {
    this.openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
  }

  async chat(
    companyId: string,
    userId: string,
    conversationId: string,
    message: string
  ) {
    const context: ToolContext = { companyId, userId, conversationId };

    // Build messages with history
    const messages = await this.buildMessages(conversationId, message);

    // Call OpenAI with tools
    const response = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages,
      tools: OPENAI_TOOLS,
      tool_choice: 'auto',
    });

    const assistantMessage = response.choices[0].message;

    // Handle tool calls
    if (assistantMessage.tool_calls?.length) {
      return this.handleToolCalls(assistantMessage.tool_calls, context, messages);
    }

    // Save and return response
    await this.saveMessage(conversationId, 'assistant', assistantMessage.content);
    return { content: assistantMessage.content };
  }

  private async handleToolCalls(
    toolCalls: OpenAI.ChatCompletionMessageToolCall[],
    context: ToolContext,
    messages: OpenAI.ChatCompletionMessageParam[]
  ) {
    const toolResults = [];

    for (const toolCall of toolCalls) {
      const { name, arguments: argsString } = toolCall.function;
      const args = JSON.parse(argsString);

      // Execute via Strategy Pattern
      const result = await toolExecutor.execute(name, args, context);

      toolResults.push({
        tool_call_id: toolCall.id,
        role: 'tool' as const,
        content: JSON.stringify(result),
      });
    }

    // Continue conversation with tool results
    messages.push({ role: 'assistant', tool_calls: toolCalls });
    messages.push(...toolResults);

    const followUp = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages,
    });

    return { content: followUp.choices[0].message.content };
  }

  private async buildMessages(conversationId: string, newMessage: string) {
    // Buscar histórico
    const history = await prisma.assistantMessage.findMany({
      where: { conversationId },
      orderBy: { createdAt: 'asc' },
      take: 20,
    });

    const messages: OpenAI.ChatCompletionMessageParam[] = [
      { role: 'system', content: this.getSystemPrompt() },
      ...history.map(m => ({ role: m.role as 'user' | 'assistant', content: m.content })),
      { role: 'user', content: newMessage },
    ];

    return messages;
  }

  private getSystemPrompt(): string {
    return `You are an AI assistant for a CRM system...`;
  }

  private async saveMessage(conversationId: string, role: string, content: string) {
    // Persistir mensagem
  }
}

export const assistantService = new AssistantService();
```

---

## Mapeamento de Tools por Arquivo

| Arquivo | Tools | Linhas Est. |
|---------|-------|-------------|
| `contact.tools.ts` | list_contacts, create_contact, update_contact, search_contacts | ~200 |
| `tag.tools.ts` | list_tags, create_tag, assign_tags, remove_tags | ~150 |
| `funnel.tools.ts` | list_funnels, list_stages, get_funnel_stats | ~200 |
| `deal.tools.ts` | create_deal, update_deal_stage, update_deal_value, add_deal_note | ~250 |
| `flow.tools.ts` | list_flows, trigger_flow, validate_flow | ~200 |
| `followup.tools.ts` | create_followup_rule, list_followup_rules, pause_followup | ~200 |
| `message.tools.ts` | send_message, schedule_message, send_template | ~150 |

**Total:** ~1.350 linhas distribuídas vs 2.100+ linhas em um método

---

## Checklist

### Preparação
- [ ] Criar branch `refactor/assistant-service`
- [ ] Backup do arquivo original
- [ ] Rodar testes existentes

### Estrutura
- [ ] Criar `assistant.types.ts`
- [ ] Criar `tools/tool.interface.ts`
- [ ] Criar `tools/index.ts` (registry)

### Tools (um por vez)
- [ ] `tools/contact.tools.ts`
- [ ] `tools/tag.tools.ts`
- [ ] `tools/funnel.tools.ts`
- [ ] `tools/deal.tools.ts`
- [ ] `tools/flow.tools.ts`
- [ ] `tools/followup.tools.ts`
- [ ] `tools/message.tools.ts`

### Finalização
- [ ] Criar `services/tool-executor.service.ts`
- [ ] Criar `definitions/openai-tools.ts`
- [ ] Refatorar `assistant.service.ts`
- [ ] Testar cada tool individualmente
- [ ] Testes de integração

### Validação
- [ ] Build sem erros
- [ ] Lint sem warnings
- [ ] PR review

---

## Benefícios do Strategy Pattern

| Aspecto | Antes | Depois |
|---------|-------|--------|
| Adicionar tool | Editar 2.600 linhas | Criar arquivo ~50 linhas |
| Testar tool | Mock de tudo | Teste isolado |
| Encontrar bug | Buscar em 2.600 linhas | Ir direto ao arquivo |
| Code review | Revisar tudo | Revisar apenas a tool |

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
