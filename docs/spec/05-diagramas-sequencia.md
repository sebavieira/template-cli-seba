# 5. Diagramas de Sequência

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

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
    A->>A: Gera JWT
    A-->>F: { token, user }
    F->>F: Salva token
    F-->>U: Redireciona para dashboard
```

---

## 5.2 Fluxo de Criação de {{Entidade}}

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant D as Database
    participant Q as Queue

    U->>F: Preenche formulário
    F->>A: POST /{{entidade}}
    A->>A: Valida dados (Zod)
    A->>D: INSERT {{entidade}}
    D-->>A: {{entidade}} criado

    opt Processamento Assíncrono
        A->>Q: Enfileira job
        Q-->>A: Job ID
    end

    A-->>F: 201 Created
    F-->>U: Sucesso
```

---

## 5.3 Fluxo de Listagem com Paginação

```mermaid
sequenceDiagram
    participant U as Usuário
    participant F as Frontend
    participant A as API
    participant C as Cache
    participant D as Database

    U->>F: Acessa lista
    F->>A: GET /{{entidade}}?page=1

    A->>C: Verifica cache
    alt Cache hit
        C-->>A: Dados em cache
    else Cache miss
        A->>D: SELECT com paginação
        D-->>A: Resultados
        A->>C: Salva em cache (TTL: 5min)
    end

    A-->>F: { data, pagination }
    F-->>U: Renderiza lista
```

---

## 5.4 Fluxo de Processamento Assíncrono

```mermaid
sequenceDiagram
    participant A as API
    participant Q as Queue (BullMQ)
    participant W as Worker
    participant D as Database
    participant E as External API

    A->>Q: Adiciona job
    Q-->>A: Job ID

    loop Worker polling
        Q->>W: Próximo job
        W->>D: Atualiza status: processing
        W->>E: Chamada externa

        alt Sucesso
            E-->>W: 200 OK
            W->>D: Atualiza status: completed
        else Falha
            E-->>W: Error
            W->>Q: Retry (exponential backoff)
        end
    end
```

---

## 5.5 Fluxo de Tratamento de Erros

```mermaid
sequenceDiagram
    participant F as Frontend
    participant A as API
    participant D as Database
    participant L as Logger

    F->>A: Request inválido
    A->>A: Validação falha
    A->>L: Log de erro
    A-->>F: 400 Bad Request

    F->>A: Request válido
    A->>D: Query
    D-->>A: Database error
    A->>L: Log de erro crítico
    A->>A: Circuit Breaker check
    A-->>F: 500 Internal Error
```

---

## 5.6 Fluxo de Webhook/Callback

```mermaid
sequenceDiagram
    participant E as External Service
    participant W as Webhook Handler
    participant A as API
    participant D as Database
    participant Q as Queue

    E->>W: POST /webhooks/{{provider}}
    W->>W: Valida signature
    W->>W: Parse payload
    W->>Q: Enfileira processamento
    W-->>E: 200 OK (imediato)

    Q->>A: Processa evento
    A->>D: Atualiza dados
    A->>A: Dispara ações
```

---

## Como Ler os Diagramas

### Participantes

- **Usuário (U)**: Pessoa usando o sistema
- **Frontend (F)**: Interface web/mobile
- **API (A)**: Backend server
- **Database (D)**: PostgreSQL
- **Queue (Q)**: BullMQ/Redis
- **Worker (W)**: Processador de jobs
- **Cache (C)**: Redis cache
- **External (E)**: APIs de terceiros

### Notações

- `->`: Chamada síncrona
- `-->`: Resposta
- `opt`: Bloco opcional
- `alt/else`: Condicionais
- `loop`: Repetição

---

← [Voltar para SPEC](README.md) | [Próximo: Máquina de Estados →](06-maquina-estados.md)
