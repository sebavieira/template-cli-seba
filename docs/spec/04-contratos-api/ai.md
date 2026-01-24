# 4.6 IA / Prompts

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para Contratos de API](README.md)

---

## Análise de IA

### POST /api/ai/analyze

**Descrição:** Disparar análise de IA em uma conversa.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "conversationId": "uuid",
  "force": false
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| conversationId | string | Sim | UUID da conversa |
| force | boolean | Não | Forçar re-análise (default: false) |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "analysis": {
      "sentiment": "positive",
      "urgency": "medium",
      "intent": "purchase_inquiry",
      "topics": ["preço", "prazo", "garantia"],
      "suggestedActions": [
        {
          "type": "SEND_PROPOSAL",
          "description": "Cliente demonstrou interesse em compra"
        }
      ],
      "summary": "Cliente interessado em produto X, solicitou informações sobre preço e prazo de entrega.",
      "analyzedAt": "2026-01-19T10:00:00Z"
    }
  },
  "analysis": {
    "sentiment": "positive",
    "urgency": "medium"
  }
}
```

**Sentimentos Possíveis:**

| Valor | Descrição |
|-------|-----------|
| positive | Conversa positiva/satisfeita |
| neutral | Conversa neutra |
| negative | Conversa negativa/insatisfeita |
| mixed | Sentimentos mistos |

**Níveis de Urgência:**

| Valor | Descrição |
|-------|-----------|
| low | Baixa urgência |
| medium | Urgência moderada |
| high | Alta urgência |
| critical | Urgência crítica |

---

### GET /api/ai/conversations/:id/state

**Descrição:** Obter estado atual da IA para uma conversa.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `id`: UUID da conversa

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "conversationId": "uuid",
    "sentiment": "positive",
    "urgency": "low",
    "intent": "support_request",
    "topics": ["dúvida", "funcionalidade"],
    "lastAnalyzedAt": "2026-01-19T10:00:00Z",
    "messageCount": 15,
    "aiEnabled": true,
    "isPaused": false,
    "summary": "Cliente com dúvidas sobre funcionalidades do produto.",
    "suggestedResponse": "Olá! Posso ajudar com suas dúvidas sobre as funcionalidades.",
    "actionsPerformed": [
      {
        "action": "TAG_ADDED",
        "tag": "Dúvida",
        "timestamp": "2026-01-19T09:00:00Z"
      }
    ]
  }
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Estado não encontrado"
}
```

---

### POST /api/ai/conversations/states

**Descrição:** Obter estados de múltiplas conversas (batch).

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "conversationIds": ["uuid-1", "uuid-2", "uuid-3"]
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "uuid-1": {
      "sentiment": "positive",
      "urgency": "low",
      "lastAnalyzedAt": "2026-01-19T10:00:00Z"
    },
    "uuid-2": {
      "sentiment": "negative",
      "urgency": "high",
      "lastAnalyzedAt": "2026-01-19T09:30:00Z"
    },
    "uuid-3": null
  }
}
```

---

## Prompts de Ação

### GET /api/ai/prompts

**Descrição:** Listar prompts de ação configurados.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Resposta de Boas-Vindas",
      "actionType": "WELCOME",
      "prompt": "Você é um assistente virtual amigável...",
      "isActive": true,
      "priority": 1,
      "conditions": {
        "triggerOnFirstMessage": true
      },
      "createdAt": "2026-01-01T00:00:00Z"
    },
    {
      "id": "uuid",
      "name": "Tratamento de Reclamação",
      "actionType": "COMPLAINT_HANDLING",
      "prompt": "Quando o cliente demonstrar insatisfação...",
      "isActive": true,
      "priority": 2,
      "conditions": {
        "sentimentTrigger": "negative"
      },
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ]
}
```

---

### POST /api/ai/prompts

**Descrição:** Criar novo prompt de ação.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Qualificação de Lead",
  "actionType": "LEAD_QUALIFICATION",
  "prompt": "Identifique o interesse do cliente e faça perguntas para qualificar...",
  "isActive": true,
  "priority": 3,
  "conditions": {
    "funnelStage": "Novo",
    "triggerOnKeywords": ["interessado", "orçamento", "preço"]
  }
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| name | string | Sim | Nome do prompt |
| actionType | string | Sim | Tipo de ação |
| prompt | string | Sim | Texto do prompt |
| isActive | boolean | Não | Ativo (default: true) |
| priority | number | Não | Prioridade de execução |
| conditions | object | Não | Condições de disparo |

**Tipos de Ação:**

| Tipo | Descrição |
|------|-----------|
| WELCOME | Mensagem de boas-vindas |
| LEAD_QUALIFICATION | Qualificação de leads |
| COMPLAINT_HANDLING | Tratamento de reclamações |
| FAQ_RESPONSE | Resposta a perguntas frequentes |
| FOLLOW_UP | Follow-up automático |
| CUSTOM | Ação personalizada |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Qualificação de Lead",
    "actionType": "LEAD_QUALIFICATION",
    "isActive": true,
    "createdAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### PUT /api/ai/prompts/:id

**Descrição:** Atualizar prompt de ação.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "name": "Qualificação de Lead (Atualizado)",
  "prompt": "Novo texto do prompt...",
  "isActive": false
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Qualificação de Lead (Atualizado)",
    "isActive": false,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### DELETE /api/ai/prompts/:id

**Descrição:** Excluir prompt de ação.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Prompt removido"
}
```

---

### POST /api/ai/prompts/:id/toggle

**Descrição:** Ativar ou desativar prompt.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Qualificação de Lead",
    "isActive": false,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

## Onboarding de IA

### GET /api/ai/onboarding/:instanceId

**Descrição:** Obter dados de onboarding da instância.

**Autenticação:** Bearer token (JWT)

**Path Parameters:**
- `instanceId`: UUID da instância WhatsApp

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "instanceId": "uuid",
    "businessName": "Minha Empresa",
    "businessType": "ECOMMERCE",
    "businessDescription": "Loja de produtos eletrônicos",
    "targetAudience": "Consumidores finais, 25-45 anos",
    "tone": "professional_friendly",
    "mainProducts": ["Smartphones", "Notebooks", "Acessórios"],
    "faq": [
      {
        "question": "Qual o prazo de entrega?",
        "answer": "Entregamos em até 5 dias úteis"
      }
    ],
    "workingHours": {
      "start": "09:00",
      "end": "18:00",
      "timezone": "America/Sao_Paulo",
      "workDays": [1, 2, 3, 4, 5]
    },
    "generatedPrompt": "Você é um assistente virtual da Minha Empresa...",
    "isComplete": true,
    "completedAt": "2026-01-15T10:00:00Z"
  }
}
```

---

### POST /api/ai/onboarding/:instanceId

**Descrição:** Salvar progresso do onboarding.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "businessName": "Minha Empresa",
  "businessType": "ECOMMERCE",
  "businessDescription": "Loja de produtos eletrônicos online",
  "targetAudience": "Consumidores finais interessados em tecnologia",
  "tone": "professional_friendly",
  "mainProducts": ["Smartphones", "Notebooks"],
  "currentStep": 3
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| businessName | string | Não | Nome do negócio |
| businessType | string | Não | Tipo de negócio |
| businessDescription | string | Não | Descrição do negócio |
| targetAudience | string | Não | Público-alvo |
| tone | string | Não | Tom de voz da IA |
| mainProducts | string[] | Não | Produtos/serviços principais |
| faq | object[] | Não | Perguntas frequentes |
| workingHours | object | Não | Horário de funcionamento |
| currentStep | number | Não | Etapa atual do onboarding |

**Tons de Voz Disponíveis:**

| Valor | Descrição |
|-------|-----------|
| formal | Formal e corporativo |
| professional_friendly | Profissional mas amigável |
| casual | Casual e descontraído |
| enthusiastic | Entusiasmado e empolgado |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "instanceId": "uuid",
    "currentStep": 3,
    "updatedAt": "2026-01-19T10:00:00Z"
  }
}
```

---

### POST /api/ai/onboarding/generate

**Descrição:** Gerar prompt automaticamente com base nos dados de onboarding.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "onboardingData": {
    "businessName": "Minha Empresa",
    "businessType": "ECOMMERCE",
    "businessDescription": "Loja de produtos eletrônicos",
    "targetAudience": "Consumidores finais",
    "tone": "professional_friendly",
    "mainProducts": ["Smartphones", "Notebooks"],
    "faq": [
      {
        "question": "Qual o prazo de entrega?",
        "answer": "Até 5 dias úteis"
      }
    ]
  }
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "generatedPrompt": "Você é um assistente virtual da Minha Empresa, uma loja de produtos eletrônicos especializada em Smartphones e Notebooks.\n\nSeu tom deve ser profissional mas amigável. Você atende consumidores finais interessados em tecnologia.\n\nPerguntas Frequentes:\n- Prazo de entrega: Até 5 dias úteis\n\nSempre seja educado, objetivo e ajude o cliente a encontrar o produto ideal.",
    "promptLength": 420,
    "estimatedTokens": 85
  }
}
```

---

## Assistente Interno

### POST /api/ai/assistant/chat

**Descrição:** Enviar mensagem para o assistente interno da plataforma.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "message": "Como configuro um follow-up automático?",
  "conversationId": "uuid (opcional)",
  "history": [
    {
      "role": "user",
      "content": "Olá"
    },
    {
      "role": "assistant",
      "content": "Olá! Como posso ajudar?"
    }
  ]
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| message | string | Sim | Mensagem do usuário |
| conversationId | string | Não | ID da conversa existente |
| history | array | Não | Histórico de mensagens |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "response": "Para configurar um follow-up automático, siga estes passos:\n\n1. Acesse o menu 'Automações'\n2. Clique em 'Novo Follow-up'\n3. Defina o gatilho (ex: após 24h sem resposta)\n4. Configure a mensagem a ser enviada\n5. Salve a regra\n\nPosso ajudar com mais alguma coisa?",
    "conversationId": "uuid",
    "messageId": "uuid"
  }
}
```

---

### GET /api/ai/assistant/conversations

**Descrição:** Listar conversas com o assistente interno.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Configuração de Follow-ups",
      "lastMessageAt": "2026-01-19T10:00:00Z",
      "messageCount": 5
    }
  ]
}
```

---

### GET /api/ai/assistant/conversations/:id/messages

**Descrição:** Listar mensagens de uma conversa com o assistente.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "role": "user",
      "content": "Como configuro um follow-up automático?",
      "createdAt": "2026-01-19T10:00:00Z"
    },
    {
      "id": "uuid",
      "role": "assistant",
      "content": "Para configurar um follow-up automático...",
      "createdAt": "2026-01-19T10:00:05Z"
    }
  ]
}
```

---

### DELETE /api/ai/assistant/conversations/:id

**Descrição:** Excluir conversa com o assistente.

**Autenticação:** Bearer token (JWT)

**Response 200 OK:**

```json
{
  "success": true
}
```

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Erro na análise | Falha ao processar análise de IA |
| 400 | Dados de onboarding obrigatórios | Campo obrigatório ausente |
| 401 | Não autenticado | Token inválido ou ausente |
| 404 | Estado não encontrado | Conversa sem análise de IA |
| 500 | Erro no chat do assistente | Falha na API do OpenAI |

---

**Anterior:** [← Deals](deals.md) | **Próximo:** [Follow-ups →](followups.md)
