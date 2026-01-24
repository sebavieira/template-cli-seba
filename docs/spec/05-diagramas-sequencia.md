# 5. Diagramas de Sequência

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 5.1 Fluxo de Autenticação

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant D as Database

    U->>F: Preenche login
    F->>A: POST /auth/login
    A->>D: Busca usuário por email
    D-->>A: Dados do usuário
    A->>A: Valida senha (bcrypt)
    A->>A: Gera JWT (access + refresh)
    A->>D: Cria sessão
    A-->>F: { accessToken, refreshToken, user }
    F->>F: Salva tokens (localStorage)
    F-->>U: Redireciona para dashboard
```

---

## 5.2 Fluxo de Mensagem WhatsApp (Recebida)

```mermaid
sequenceDiagram
    participant WA as WhatsApp
    participant E as Evolution API
    participant W as Webhook Handler
    participant Q as BullMQ
    participant WK as Message Worker
    participant D as Database
    participant AI as OpenAI
    participant S as Socket.io

    WA->>E: Usuário envia mensagem
    E->>W: POST /api/whatsapp/webhook
    W->>W: Valida instância
    W->>D: Busca/Cria conversa
    W->>D: Salva mensagem (RECEIVED)
    W->>Q: Enfileira job (message-worker)
    W-->>E: 200 OK (imediato)

    Q->>WK: Processa job
    WK->>D: Busca contexto (prompt, histórico)
    WK->>AI: Chat completion
    AI-->>WK: Resposta gerada
    WK->>E: Envia mensagem
    E->>WA: Entrega ao usuário
    WK->>D: Salva mensagem (SENT)
    WK->>S: Emite evento (new_message)
    S-->>F: Atualiza UI em tempo real
```

---

## 5.3 Fluxo de Envio de Mensagem (Manual)

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant E as Evolution API
    participant D as Database
    participant S as Socket.io

    U->>F: Digita mensagem
    F->>A: POST /api/whatsapp/send
    A->>A: Valida dados (Zod)
    A->>D: Busca instância e conversa
    A->>E: POST /message/sendText
    E-->>A: { key: { id: messageKeyId } }
    A->>D: Salva mensagem (SENT)
    A->>S: Emite evento (message_sent)
    A-->>F: 200 OK { messageId }
    F-->>U: Mensagem exibida
```

---

## 5.4 Fluxo de Criação de Contato

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant D as Database

    U->>F: Preenche formulário
    F->>A: POST /api/contacts
    A->>A: Valida dados (Zod)
    A->>D: Verifica duplicidade (phoneNumber)

    alt Contato existe
        D-->>A: Contato existente
        A-->>F: 400 { error: "Número já cadastrado" }
    else Contato novo
        A->>D: INSERT contact
        D-->>A: Contato criado
        A-->>F: 201 Created { contact }
    end

    F-->>U: Feedback visual
```

---

## 5.5 Fluxo de Movimentação no Funil

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant D as Database
    participant Q as BullMQ
    participant FL as Flow Worker

    U->>F: Arrasta deal para novo estágio
    F->>A: POST /api/deals/move
    A->>D: Atualiza deal.stageId
    A->>D: Registra atividade (STAGE_CHANGED)

    opt Flow ativo para STAGE_CHANGED
        A->>Q: Enfileira trigger de flow
        Q->>FL: Processa trigger
        FL->>FL: Executa ações do flow
    end

    A-->>F: 200 OK { deal }
    F-->>U: Kanban atualizado
```

---

## 5.6 Fluxo de Follow-up Automático

```mermaid
sequenceDiagram
    participant CR as Cron Job
    participant Q as BullMQ
    participant FW as Followup Worker
    participant D as Database
    participant E as Evolution API
    participant WA as WhatsApp

    CR->>Q: Verifica follow-ups pendentes
    Q->>FW: Processa job
    FW->>D: Busca follow-ups (scheduledFor <= now)

    loop Para cada follow-up
        FW->>D: Verifica se conversa teve resposta
        alt Sem resposta recente
            FW->>D: Busca mensagem da regra
            FW->>E: POST /message/sendText
            E->>WA: Envia mensagem
            FW->>D: Atualiza status (SENT)
            FW->>D: Agenda próxima tentativa (se < maxAttempts)
        else Com resposta
            FW->>D: Cancela follow-up (RESPONDED)
        end
    end
```

---

## 5.7 Fluxo de Análise de IA

```mermaid
sequenceDiagram
    participant A as API
    participant Q as BullMQ
    participant AW as AI Worker
    participant D as Database
    participant AI as OpenAI
    participant V as pgvector

    A->>Q: Enfileira análise (após N mensagens)
    Q->>AW: Processa job
    AW->>D: Busca histórico de mensagens
    AW->>AI: Gera embedding do texto
    AI-->>AW: Vector [1536 dims]
    AW->>V: Salva embedding
    AW->>AI: Analisa sentimento/urgência/intenção
    AI-->>AW: { sentiment, urgency, intent, topics }
    AW->>D: Atualiza ConversationAiState
    AW->>D: Atualiza ContactAiProfile
```

---

## 5.8 Fluxo de Refresh Token

```mermaid
sequenceDiagram
    participant F as Frontend
    participant A as API
    participant D as Database

    F->>A: Request com access token expirado
    A-->>F: 401 Unauthorized

    F->>A: POST /auth/refresh { token: refreshToken }
    A->>D: Valida refresh token

    alt Token válido
        A->>A: Gera novo access token
        A->>A: Gera novo refresh token
        A->>D: Atualiza sessão
        A-->>F: { accessToken, refreshToken }
        F->>F: Salva novos tokens
        F->>A: Retry request original
    else Token inválido
        A-->>F: 401 { error: "Token expirado" }
        F->>F: Redireciona para login
    end
```

---

## 5.9 Fluxo de Conexão WhatsApp (QR Code)

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant E as Evolution API
    participant WH as Webhook
    participant WA as WhatsApp

    U->>F: Clica "Conectar"
    F->>A: POST /api/whatsapp/instances/:id/connect
    A->>E: POST /instance/connect
    E-->>A: { qrcode: "data:image/png..." }
    A-->>F: { qrcode, status: "qrcode" }
    F-->>U: Exibe QR Code

    U->>WA: Escaneia QR Code
    WA->>E: Confirmação de conexão
    E->>WH: POST /webhook (connection.update)
    WH->>A: Atualiza instância (isConnected: true)
    A->>F: Socket.io (instance_connected)
    F-->>U: Status: Conectado
```

---

## 5.10 Fluxo de Webhook WhatsApp

```mermaid
sequenceDiagram
    participant E as Evolution API
    participant W as Webhook Handler
    participant A as API
    participant D as Database
    participant Q as Queue

    E->>W: POST /api/whatsapp/webhook/:instanceName
    W->>W: Parse event type

    alt messages.upsert
        W->>D: Busca instância por nome
        W->>D: Busca/Cria conversa
        W->>D: Salva mensagem
        W->>Q: Enfileira AI processing
    else connection.update
        W->>D: Atualiza status de conexão
    else qrcode.updated
        W->>D: Armazena QR Code
    else messages.update
        W->>D: Atualiza status de entrega
    end

    W-->>E: 200 OK (sempre, evita retries)
```

---

## Como Ler os Diagramas

### Participantes

- **Usuário (U)**: Pessoa usando o sistema
- **Frontend (F)**: React SPA
- **API (A)**: Fastify Backend
- **Database (D)**: PostgreSQL
- **Queue (Q)**: BullMQ/Redis
- **Worker (WK/FW/AW)**: Processadores de jobs
- **Evolution API (E)**: Provedor WhatsApp
- **OpenAI (AI)**: Serviço de IA
- **Socket.io (S)**: WebSocket server
- **WhatsApp (WA)**: Aplicativo do usuário final

### Notações

- `->`: Chamada síncrona
- `-->`: Resposta
- `opt`: Bloco opcional
- `alt/else`: Condicionais
- `loop`: Repetição

---

← [Voltar para SPEC](README.md) | [Próximo: Máquina de Estados →](06-maquina-estados.md)
