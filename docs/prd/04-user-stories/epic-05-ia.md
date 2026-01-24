# Epic 05: Chatbot com IA

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Must Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que empresas configurem chatbots inteligentes que respondem automaticamente usando OpenAI, com personalidade customizável e diferentes níveis de autonomia para ações.

---

## User Stories

### US-05.1: Configuração de Prompt

**Como** um admin,
**Quero** configurar o prompt de sistema do chatbot,
**Para que** ele responda conforme minha marca.

**Critérios de Aceitação:**
- [ ] Editor de texto para prompt de sistema
- [ ] Variáveis de contexto disponíveis (nome do contato, empresa, etc.)
- [ ] Preview de resposta antes de salvar
- [ ] Limite de caracteres com contador
- [ ] Histórico de versões do prompt

**Prioridade:** Must Have

---

### US-05.2: Onboarding Guiado

**Como** um novo usuário,
**Quero** configurar meu chatbot através de perguntas guiadas,
**Para que** eu não precise escrever prompts técnicos.

**Critérios de Aceitação:**
- [ ] Wizard de 5-7 etapas
- [ ] Perguntas sobre: nome do negócio, tom de voz, serviços, restrições
- [ ] Geração automática de prompt otimizado
- [ ] Opção de editar prompt gerado
- [ ] Possibilidade de refazer onboarding

**Prioridade:** Should Have

**Perguntas do Onboarding:**
1. Qual o nome do seu negócio?
2. O que você vende? (produto/serviço)
3. Quais os principais produtos/serviços?
4. Qual tom de voz? (formal, casual, amigável)
5. Quais informações não devem ser compartilhadas?
6. Qual o horário de funcionamento?
7. Há alguma promoção ativa?

---

### US-05.3: Níveis de Autonomia

**Como** um admin,
**Quero** definir o nível de autonomia do chatbot,
**Para que** ele aja conforme minha preferência.

**Critérios de Aceitação:**
- [ ] Nível 0: Apenas responder mensagens
- [ ] Nível 1: Responder + adicionar tags automaticamente
- [ ] Nível 2: Responder + tags + mover estágios automaticamente
- [ ] Configuração por empresa
- [ ] Log de ações autônomas realizadas

**Prioridade:** Must Have

**Níveis Detalhados:**

| Nível | Nome | Ações Permitidas |
|-------|------|------------------|
| 0 | Básico | Responder mensagens |
| 1 | Híbrido | Responder + Adicionar tags + Pausar bot |
| 2 | Completo | Todas as ações incluindo mover estágios |

---

### US-05.4: Respostas Automáticas

**Como** sistema,
**Quero** gerar respostas automáticas via OpenAI,
**Para que** contatos recebam atendimento instantâneo.

**Critérios de Aceitação:**
- [ ] Integração com OpenAI API (GPT-4)
- [ ] Contexto das últimas N mensagens (configurável)
- [ ] Timeout de resposta (máximo 30s)
- [ ] Fallback para mensagem padrão em caso de erro
- [ ] Tracking de tokens consumidos

**Prioridade:** Must Have

---

### US-05.5: Ações Autônomas

**Como** chatbot,
**Quero** executar ações baseadas na conversa,
**Para que** o CRM seja atualizado automaticamente.

**Critérios de Aceitação:**
- [ ] Adicionar tags baseado em detecção de intenção
- [ ] Mover contato de estágio baseado em qualificação
- [ ] Pausar bot quando humano deve assumir
- [ ] Notificar equipe sobre leads quentes
- [ ] Log completo de ações com justificativa

**Prioridade:** Should Have

**Ações Disponíveis:**

| Ação | Trigger | Resultado |
|------|---------|-----------|
| `add_tag` | IA detecta interesse em produto | Tag adicionada ao contato |
| `move_stage` | IA detecta intenção de compra | Contato movido para "Negociação" |
| `pause_bot` | IA detecta necessidade humana | Bot pausa, notifica equipe |
| `notify_team` | Lead quente detectado | Notificação enviada |

---

### US-05.6: Análise de Sentimento

**Como** um vendedor,
**Quero** ver o sentimento da conversa,
**Para que** eu priorize conversas negativas.

**Critérios de Aceitação:**
- [ ] Análise de sentimento: positivo, neutro, negativo
- [ ] Score de engajamento (0-100)
- [ ] Detecção de urgência
- [ ] Indicador visual na lista de conversas
- [ ] Cache de análise (atualiza a cada 10 min)

**Prioridade:** Should Have

---

### US-05.7: Respostas Rápidas (Quick Replies)

**Como** um vendedor,
**Quero** usar templates de resposta,
**Para que** eu responda mais rápido.

**Critérios de Aceitação:**
- [ ] Biblioteca de respostas por empresa
- [ ] Atalhos de teclado (ex: /preco, /endereco)
- [ ] Variáveis dinâmicas (nome do cliente, etc.)
- [ ] Criação e edição de templates
- [ ] Busca por texto ou atalho

**Prioridade:** Should Have

---

## Modelo de Dados

```prisma
model UserPrompt {
  id        String   @id @default(cuid())
  companyId String   @unique
  content   String   @db.Text
  version   Int      @default(1)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  company   Company           @relation(fields: [companyId], references: [id])
  onboarding PromptOnboarding?
}

model PromptOnboarding {
  id              String  @id @default(cuid())
  promptId        String  @unique
  businessName    String?
  businessType    BusinessType?
  products        String?
  tone            String?
  restrictions    String?
  businessHours   String?
  promotions      String?
  isCompleted     Boolean @default(false)

  prompt UserPrompt @relation(fields: [promptId], references: [id])
}

model ConversationAiState {
  id             String   @id @default(cuid())
  conversationId String   @unique
  sentiment      String?  // positive, neutral, negative
  engagementScore Int?    // 0-100
  urgency        String?  // low, medium, high
  intent         String?  // purchase, support, inquiry
  lastAnalyzedAt DateTime @default(now())

  conversation WhatsAppConversation @relation(...)
}

model QuickReply {
  id        String  @id @default(cuid())
  companyId String
  userId    String?
  shortcut  String
  content   String
  isGlobal  Boolean @default(false)

  company Company @relation(fields: [companyId], references: [id])
  user    User?   @relation(fields: [userId], references: [id])

  @@unique([companyId, shortcut])
}

enum BusinessType {
  PRODUCT
  SERVICE
  BOTH
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/user-prompts` | Obter prompt atual |
| POST | `/api/user-prompts` | Criar/atualizar prompt |
| GET | `/api/user-prompts/onboarding` | Status do onboarding |
| POST | `/api/user-prompts/onboarding` | Salvar etapa do onboarding |
| POST | `/api/ai/prompt` | Testar prompt com input |
| GET | `/api/ai/sentiment` | Análise de sentimento |
| POST | `/api/ai/suggestions` | Obter sugestões de resposta |
| GET | `/api/quick-replies` | Listar respostas rápidas |
| POST | `/api/quick-replies` | Criar resposta rápida |

---

## Fluxo de Processamento IA

```
Mensagem Recebida
    ↓
Buffer de Mensagens (agrupa se múltiplas em sequência)
    ↓
Recupera Contexto (últimas N mensagens)
    ↓
Monta Prompt: Sistema + Contexto + Mensagem
    ↓
Chama OpenAI API
    ↓
Analisa Resposta para Ações Autônomas
    ↓
Executa Ações (se autonomyLevel permite)
    ↓
Envia Resposta ao Contato
    ↓
Registra Tokens Consumidos
    ↓
Atualiza Análise de Sentimento (assíncrono)
```

---

[← Voltar para User Stories](README.md)
