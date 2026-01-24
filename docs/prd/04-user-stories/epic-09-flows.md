# Epic 09: Fluxos de Automação

**Versão:** 1.0.0
**Status:** 🚧 Em Desenvolvimento
**Prioridade:** Could Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que usuários criem fluxos de automação visual com editor drag-and-drop, usando nodes condicionais, ações e integrações para automatizar processos complexos.

---

## User Stories

### US-09.1: Criação de Flow

**Como** um admin,
**Quero** criar fluxos de automação,
**Para que** eu automatize processos repetitivos.

**Critérios de Aceitação:**
- [ ] Nome e descrição do flow
- [ ] Seleção de trigger inicial
- [ ] Canvas vazio para começar
- [ ] Salvar como rascunho ou publicar
- [ ] Duplicar flows existentes

**Prioridade:** Could Have

---

### US-09.2: Editor Visual

**Como** um admin,
**Quero** um editor visual drag-and-drop,
**Para que** eu monte fluxos sem programar.

**Critérios de Aceitação:**
- [ ] Canvas infinito com zoom/pan
- [ ] Biblioteca de nodes (sidebar)
- [ ] Drag-and-drop de nodes
- [ ] Conexão entre nodes (draw lines)
- [ ] Configuração de node (panel lateral)
- [ ] Undo/redo
- [ ] Auto-save

**Prioridade:** Could Have

---

### US-09.3: Tipos de Nodes

**Como** um admin,
**Quero** diferentes tipos de nodes,
**Para que** eu construa lógicas complexas.

**Critérios de Aceitação:**
- [ ] **Trigger**: Evento que inicia o flow
- [ ] **Condition**: Lógica if/else
- [ ] **Action**: Executar ação
- [ ] **Delay**: Aguardar tempo
- [ ] **Webhook**: Chamar API externa
- [ ] **AI**: Gerar conteúdo ou decisão

**Prioridade:** Could Have

**Catálogo de Nodes:**

| Categoria | Node | Descrição |
|-----------|------|-----------|
| Trigger | message_received | Mensagem recebida |
| Trigger | tag_added | Tag adicionada |
| Trigger | stage_changed | Estágio alterado |
| Trigger | scheduled | Agendamento (cron) |
| Trigger | webhook_received | Webhook externo |
| Condition | if_contains | Texto contém palavra |
| Condition | if_tag | Contato tem tag |
| Condition | if_stage | Contato em estágio |
| Condition | if_sentiment | Sentimento detectado |
| Action | send_message | Enviar mensagem |
| Action | add_tag | Adicionar tag |
| Action | remove_tag | Remover tag |
| Action | move_stage | Mover estágio |
| Action | create_deal | Criar deal |
| Action | notify_team | Notificar equipe |
| Utility | delay | Aguardar tempo |
| Utility | webhook | Chamar API |
| AI | ai_generate | Gerar texto com IA |
| AI | ai_classify | Classificar input |

---

### US-09.4: Execução de Flows

**Como** sistema,
**Quero** executar flows quando triggers ocorrem,
**Para que** automações funcionem em tempo real.

**Critérios de Aceitação:**
- [ ] Detecção de triggers em eventos
- [ ] Execução de nodes em sequência
- [ ] Avaliação de condições
- [ ] Execução de ações
- [ ] Tratamento de erros
- [ ] Retry em caso de falha
- [ ] Timeout por execução

**Prioridade:** Could Have

---

### US-09.5: Histórico de Execuções

**Como** um admin,
**Quero** ver histórico de execuções de flows,
**Para que** eu monitore e debug automações.

**Critérios de Aceitação:**
- [ ] Lista de execuções por flow
- [ ] Status: running, completed, failed
- [ ] Tempo de execução
- [ ] Logs de cada node executado
- [ ] Dados de input/output
- [ ] Filtros por período e status

**Prioridade:** Could Have

---

### US-09.6: Ativação/Desativação

**Como** um admin,
**Quero** ativar e desativar flows,
**Para que** eu controle quais automações estão ativas.

**Critérios de Aceitação:**
- [ ] Toggle de ativação
- [ ] Flows inativos não executam
- [ ] Indicador visual de status
- [ ] Log de ativação/desativação

**Prioridade:** Could Have

---

## Modelo de Dados

```prisma
model AutomationFlow {
  id          String   @id @default(cuid())
  companyId   String
  name        String
  description String?
  isActive    Boolean  @default(false)
  isDraft     Boolean  @default(true)
  triggerType String
  triggerConfig Json?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  company     Company          @relation(fields: [companyId], references: [id])
  nodes       FlowNode[]
  connections FlowConnection[]
  executions  FlowExecution[]
}

model FlowNode {
  id        String @id @default(cuid())
  flowId    String
  type      String // trigger, condition, action, delay, webhook, ai
  name      String
  config    Json   // Configuração específica do node
  positionX Int
  positionY Int

  flow AutomationFlow @relation(fields: [flowId], references: [id])
  connectionsFrom FlowConnection[] @relation("FromNode")
  connectionsTo   FlowConnection[] @relation("ToNode")
}

model FlowConnection {
  id         String  @id @default(cuid())
  flowId     String
  fromNodeId String
  toNodeId   String
  label      String? // Para branches de condition (true/false)

  flow     AutomationFlow @relation(fields: [flowId], references: [id])
  fromNode FlowNode       @relation("FromNode", fields: [fromNodeId], references: [id])
  toNode   FlowNode       @relation("ToNode", fields: [toNodeId], references: [id])
}

model FlowExecution {
  id        String   @id @default(cuid())
  flowId    String
  contactId String?
  status    String   @default("running") // running, completed, failed
  startedAt DateTime @default(now())
  completedAt DateTime?
  errorMessage String?
  logs      Json?    // Array de logs de cada node

  flow    AutomationFlow @relation(fields: [flowId], references: [id])
  contact Contact?       @relation(fields: [contactId], references: [id])
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/flows` | Listar flows |
| POST | `/api/flows` | Criar flow |
| GET | `/api/flows/:id` | Detalhes do flow |
| PUT | `/api/flows/:id` | Atualizar flow |
| DELETE | `/api/flows/:id` | Excluir flow |
| PATCH | `/api/flows/:id/toggle` | Ativar/desativar |
| POST | `/api/flows/:id/execute` | Executar manualmente |
| GET | `/api/flows/:id/executions` | Histórico de execuções |
| GET | `/api/flows/executions/:execId` | Detalhes da execução |

---

## Interface do Editor

```
┌────────────────────────────────────────────────────────────────┐
│  Flow: Qualificação Automática                    [💾] [▶️ ]  │
├────────────┬───────────────────────────────────────────────────┤
│            │                                                   │
│  Triggers  │     ┌─────────┐                                  │
│  ─────────│     │ Trigger │                                  │
│  📩 Message│     │ Message │──────┐                           │
│  🏷️ Tag    │     └─────────┘      │                           │
│  📊 Stage  │                       ▼                           │
│            │               ┌─────────────┐                    │
│  Conditions│               │  Condition  │                    │
│  ─────────│               │ Contains    │                    │
│  ❓ If/Else│               │  "preço"    │                    │
│  🏷️ Has Tag│               └──────┬──────┘                    │
│            │                ╱           ╲                      │
│  Actions   │              Yes           No                     │
│  ─────────│              ▼             ▼                      │
│  💬 Message│       ┌─────────┐   ┌─────────┐                  │
│  🏷️ Add Tag│       │  Send   │   │ Add Tag │                  │
│  📊 Move   │       │ Message │   │"curioso"│                  │
│            │       └─────────┘   └─────────┘                  │
│  Utility   │                                                   │
│  ─────────│                                                   │
│  ⏱️ Delay  │                                                   │
│  🌐 Webhook│                                                   │
│            │                                                   │
└────────────┴───────────────────────────────────────────────────┘
```

---

[← Voltar para User Stories](README.md)
