# Refatoração: Message Worker

**Arquivo:** `server/src/jobs/message.worker.ts`
**Linhas:** 617
**Prioridade:** P2 - MÉDIO
**Estimativa:** 1 dia

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 617 | 400 | 🟠 ALTO |
| Maior função | 300+ | 30 | 🔴 CRÍTICO |
| Responsabilidades | 5+ | 1 | 🟠 ALTO |

### Método Problemático: processMessage()

```typescript
// ATUAL: 300+ linhas em uma função
async function processMessage(job: Job) {
  // 1. Buscar conversation/prompt/context (60 linhas)
  // 2. Audio transcription (20 linhas)
  // 3. AI completion + tool calling (100 linhas)
  // 4. Message sending (80 linhas)
  // 5. Metrics logging (40 linhas)
}
```

### Responsabilidades Misturadas

1. **Context Building** - Buscar conversa, prompt, memória
2. **Audio Transcription** - Transcrever áudio via Deepgram
3. **AI Completion** - Chamar OpenAI, processar tool calls
4. **Message Sending** - Dividir e enviar mensagens
5. **Metrics** - Registrar uso de tokens

---

## Estrutura Proposta

```
server/src/jobs/
├── message.worker.ts             # Worker principal (~100 linhas)
├── workers/
│   └── message/
│       ├── index.ts              # Exports
│       ├── message-processor.ts  # Coordenador (~150 linhas)
│       ├── ai-completion.service.ts # AI (~150 linhas)
│       ├── message-sender.service.ts # Envio (~100 linhas)
│       ├── audio-transcriber.service.ts # Áudio (~80 linhas)
│       └── metrics-recorder.service.ts # Métricas (~50 linhas)
└── helpers/
    └── text-splitter.ts          # Split de mensagens (~50 linhas)
```

---

## Código Refatorado

### Worker Principal
```typescript
// server/src/jobs/message.worker.ts

import { Worker, Job } from 'bullmq';
import { connection } from './queues';
import { MessageProcessor } from './workers/message/message-processor';

const processor = new MessageProcessor();

const worker = new Worker(
  'message',
  async (job: Job) => {
    return processor.process(job.data);
  },
  {
    connection,
    concurrency: 5,
  }
);

worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err.message);
});

export { worker as messageWorker };
```

### Message Processor
```typescript
// server/src/jobs/workers/message/message-processor.ts

import { prisma } from '../../../config/database';
import { AICompletionService } from './ai-completion.service';
import { MessageSenderService } from './message-sender.service';
import { AudioTranscriberService } from './audio-transcriber.service';
import { MetricsRecorderService } from './metrics-recorder.service';

interface MessageJobData {
  conversationId: string;
  messageId: string;
  instanceId: string;
  companyId: string;
}

export class MessageProcessor {
  private ai = new AICompletionService();
  private sender = new MessageSenderService();
  private transcriber = new AudioTranscriberService();
  private metrics = new MetricsRecorderService();

  async process(data: MessageJobData): Promise<void> {
    const { conversationId, messageId, companyId } = data;

    // 1. Buscar contexto
    const context = await this.buildContext(conversationId, messageId);
    if (!context) return;

    // 2. Transcrever áudio se necessário
    let messageContent = context.message.content;
    if (context.message.type === 'audio') {
      messageContent = await this.transcriber.transcribe(context.message.mediaUrl);
    }

    // 3. Gerar resposta AI
    const aiResponse = await this.ai.complete({
      companyId,
      conversationId,
      message: messageContent,
      prompt: context.prompt,
      memory: context.memory,
      contact: context.contact,
    });

    // 4. Enviar resposta
    await this.sender.send({
      instanceId: context.instanceId,
      to: context.contact.phone,
      text: aiResponse.content,
      companyId,
    });

    // 5. Registrar métricas
    await this.metrics.record({
      companyId,
      conversationId,
      tokensUsed: aiResponse.tokensUsed,
      model: aiResponse.model,
    });
  }

  private async buildContext(conversationId: string, messageId: string) {
    const conversation = await prisma.conversation.findUnique({
      where: { id: conversationId },
      include: {
        contact: true,
        instance: true,
        messages: {
          where: { id: messageId },
          take: 1,
        },
      },
    });

    if (!conversation) return null;

    const prompt = await prisma.prompt.findFirst({
      where: { companyId: conversation.companyId, isDefault: true },
    });

    const memory = await prisma.conversationMemory.findUnique({
      where: { conversationId },
    });

    return {
      conversation,
      message: conversation.messages[0],
      contact: conversation.contact,
      instanceId: conversation.instanceId,
      prompt: prompt?.content || '',
      memory: memory?.summary || '',
    };
  }
}
```

### AI Completion Service
```typescript
// server/src/jobs/workers/message/ai-completion.service.ts

import { OpenAI } from 'openai';
import { toolExecutor } from '../../../modules/ai/services/tool-executor.service';

interface CompletionParams {
  companyId: string;
  conversationId: string;
  message: string;
  prompt: string;
  memory: string;
  contact: { name: string; phone: string };
}

interface CompletionResult {
  content: string;
  tokensUsed: number;
  model: string;
  toolCalls?: any[];
}

export class AICompletionService {
  private openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

  async complete(params: CompletionParams): Promise<CompletionResult> {
    const { companyId, conversationId, message, prompt, memory, contact } = params;

    const messages = this.buildMessages(prompt, memory, contact, message);

    const response = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages,
      tools: this.getTools(),
      tool_choice: 'auto',
      max_tokens: 1000,
    });

    const assistantMessage = response.choices[0].message;

    // Handle tool calls
    if (assistantMessage.tool_calls?.length) {
      return this.handleToolCalls(assistantMessage.tool_calls, companyId, conversationId, messages);
    }

    return {
      content: assistantMessage.content || '',
      tokensUsed: response.usage?.total_tokens || 0,
      model: response.model,
    };
  }

  private buildMessages(prompt: string, memory: string, contact: any, userMessage: string) {
    const systemMessage = `${prompt}\n\nContexto do contato:\nNome: ${contact.name}\nTelefone: ${contact.phone}\n\nMemória da conversa:\n${memory}`;

    return [
      { role: 'system' as const, content: systemMessage },
      { role: 'user' as const, content: userMessage },
    ];
  }

  private async handleToolCalls(toolCalls: any[], companyId: string, conversationId: string, messages: any[]) {
    // Execute tools and get results
    const results = [];
    for (const call of toolCalls) {
      const result = await toolExecutor.execute(
        call.function.name,
        JSON.parse(call.function.arguments),
        { companyId, conversationId }
      );
      results.push({ tool_call_id: call.id, role: 'tool', content: JSON.stringify(result) });
    }

    // Continue with tool results
    const followUp = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [...messages, { role: 'assistant', tool_calls: toolCalls }, ...results],
    });

    return {
      content: followUp.choices[0].message.content || '',
      tokensUsed: followUp.usage?.total_tokens || 0,
      model: followUp.model,
      toolCalls,
    };
  }

  private getTools() {
    // Return OpenAI tools definition
    return [];
  }
}
```

### Message Sender Service
```typescript
// server/src/jobs/workers/message/message-sender.service.ts

import { whatsappService } from '../../../modules/whatsapp/whatsapp.service';
import { splitText } from '../../helpers/text-splitter';

interface SendParams {
  instanceId: string;
  to: string;
  text: string;
  companyId: string;
}

export class MessageSenderService {
  private readonly MAX_MESSAGE_LENGTH = 1000;

  async send(params: SendParams): Promise<void> {
    const { instanceId, to, text, companyId } = params;

    // Split long messages
    const parts = splitText(text, this.MAX_MESSAGE_LENGTH);

    for (const part of parts) {
      await whatsappService.sendMessage(companyId, instanceId, to, part);

      // Small delay between messages to maintain order
      if (parts.length > 1) {
        await this.delay(500);
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### Text Splitter Helper
```typescript
// server/src/jobs/helpers/text-splitter.ts

export function splitText(text: string, maxLength: number): string[] {
  if (text.length <= maxLength) {
    return [text];
  }

  const parts: string[] = [];
  let remaining = text;

  while (remaining.length > 0) {
    if (remaining.length <= maxLength) {
      parts.push(remaining);
      break;
    }

    // Find a good break point
    let breakPoint = maxLength;

    // Try to break at paragraph
    const paragraphBreak = remaining.lastIndexOf('\n\n', maxLength);
    if (paragraphBreak > maxLength * 0.5) {
      breakPoint = paragraphBreak;
    } else {
      // Try to break at sentence
      const sentenceBreak = Math.max(
        remaining.lastIndexOf('. ', maxLength),
        remaining.lastIndexOf('! ', maxLength),
        remaining.lastIndexOf('? ', maxLength)
      );
      if (sentenceBreak > maxLength * 0.5) {
        breakPoint = sentenceBreak + 1;
      } else {
        // Break at word
        const wordBreak = remaining.lastIndexOf(' ', maxLength);
        if (wordBreak > maxLength * 0.5) {
          breakPoint = wordBreak;
        }
      }
    }

    parts.push(remaining.slice(0, breakPoint).trim());
    remaining = remaining.slice(breakPoint).trim();
  }

  return parts;
}
```

---

## Checklist

- [ ] Criar pasta `workers/message/`
- [ ] Criar `message-processor.ts`
- [ ] Criar `ai-completion.service.ts`
- [ ] Criar `message-sender.service.ts`
- [ ] Criar `audio-transcriber.service.ts`
- [ ] Criar `metrics-recorder.service.ts`
- [ ] Criar `helpers/text-splitter.ts`
- [ ] Refatorar `message.worker.ts`
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
