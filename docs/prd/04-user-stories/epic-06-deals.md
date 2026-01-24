# Epic 06: Deals e Negócios

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Should Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que vendedores gerenciem oportunidades de venda (deals) com valores monetários, probabilidades e tracking completo de atividades e notas.

---

## User Stories

### US-06.1: Criação de Deal

**Como** um vendedor,
**Quero** criar um deal associado a um contato,
**Para que** eu acompanhe oportunidades de venda.

**Critérios de Aceitação:**
- [ ] Criação com título, valor, contato associado
- [ ] Associação automática ao estágio do contato
- [ ] Probabilidade de fechamento (0-100%)
- [ ] Data de previsão de fechamento (opcional)
- [ ] Múltiplos deals por contato

**Prioridade:** Should Have

---

### US-06.2: Movimentação de Deals

**Como** um vendedor,
**Quero** mover deals entre estágios,
**Para que** eu atualize o progresso da negociação.

**Critérios de Aceitação:**
- [ ] Drag-and-drop no Kanban
- [ ] Atualização de status: OPEN, WON, LOST
- [ ] Registro de data de entrada no estágio
- [ ] Motivo de perda opcional quando marcado como LOST
- [ ] Histórico de movimentações

**Prioridade:** Should Have

---

### US-06.3: Notas em Deals

**Como** um vendedor,
**Quero** adicionar notas ricas em deals,
**Para que** eu registre detalhes da negociação.

**Critérios de Aceitação:**
- [ ] Editor de texto rico (Plate.js)
- [ ] Suporte a formatação (bold, italic, listas)
- [ ] Suporte a links e menções
- [ ] Múltiplas notas por deal
- [ ] Autor e timestamp em cada nota

**Prioridade:** Should Have

---

### US-06.4: Atividades de Deal

**Como** um vendedor,
**Quero** ver histórico de atividades do deal,
**Para que** eu acompanhe tudo que aconteceu.

**Critérios de Aceitação:**
- [ ] Timeline de atividades
- [ ] Tipos: criação, movimentação, nota adicionada, status alterado
- [ ] Filtro por tipo de atividade
- [ ] Autor de cada atividade
- [ ] Ordenação cronológica

**Prioridade:** Should Have

---

### US-06.5: Anexos em Deals

**Como** um vendedor,
**Quero** anexar arquivos aos deals,
**Para que** eu centralize documentos da negociação.

**Critérios de Aceitação:**
- [ ] Upload de arquivos (PDF, imagens, docs)
- [ ] Limite de tamanho (10MB por arquivo)
- [ ] Lista de anexos com preview
- [ ] Download e exclusão
- [ ] Integração com storage (S3/R2)

**Prioridade:** Could Have

---

### US-06.6: Copiar Deal para Outro Funil

**Como** um vendedor,
**Quero** copiar um deal para outro funil,
**Para que** eu continue o fluxo em outra etapa sem perder o histórico original.

**Critérios de Aceitação:**
- [ ] Ação "Copiar para outro funil" disponível no card
- [ ] Seleção obrigatória de funil e etapa de destino
- [ ] Deal original permanece inalterado
- [ ] Deal novo herda os dados principais (titulo, valor, contato, responsável, datas)
- [ ] Registro de atividade de clonagem (origem e destino)

**Prioridade:** Should Have

---

### US-06.7: Métricas de Deals

**Como** um gestor,
**Quero** ver métricas dos deals,
**Para que** eu acompanhe a performance comercial.

**Critérios de Aceitação:**
- [ ] Valor total em pipeline
- [ ] Valor ganho vs perdido
- [ ] Ticket médio (total/ganhos/perdas) apenas com valores > 0
- [ ] Taxa de conversão por estágio
- [ ] Tempo médio de ciclo de vendas
- [ ] Deals por vendedor
- [ ] Motivos de perda mais frequentes (quando informado)

**Prioridade:** Could Have

---

## Modelo de Dados

```prisma
model Deal {
  id            String     @id @default(cuid())
  companyId     String
  contactId     String
  stageId       String
  clonedFromDealId String?
  title         String
  value         Decimal?   @db.Decimal(15, 2)
  probability   Int?       // 0-100
  status        DealStatus @default(OPEN)
  expectedClose DateTime?
  lostReason    String?
  stageEnteredAt DateTime  @default(now())
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt

  company     Company         @relation(fields: [companyId], references: [id])
  contact     Contact         @relation(fields: [contactId], references: [id])
  stage       ContactStage    @relation(fields: [stageId], references: [id])
  notes       DealNote[]
  activities  DealActivity[]
  attachments DealAttachment[]
}

model DealNote {
  id        String   @id @default(cuid())
  dealId    String
  userId    String
  content   String   @db.Text // HTML do editor rico
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  deal Deal @relation(fields: [dealId], references: [id])
  user User @relation(fields: [userId], references: [id])
}

model DealActivity {
  id        String   @id @default(cuid())
  dealId    String
  userId    String?
  type      String   // created, moved, note_added, status_changed, cloned_from, cloned_to
  metadata  Json?    // Dados adicionais da atividade
  createdAt DateTime @default(now())

  deal Deal  @relation(fields: [dealId], references: [id])
  user User? @relation(fields: [userId], references: [id])
}

model DealAttachment {
  id        String   @id @default(cuid())
  dealId    String
  userId    String
  filename  String
  fileUrl   String
  fileSize  Int
  mimeType  String
  createdAt DateTime @default(now())

  deal Deal @relation(fields: [dealId], references: [id])
  user User @relation(fields: [userId], references: [id])
}

enum DealStatus {
  OPEN
  WON
  LOST
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/deals` | Listar deals |
| POST | `/api/deals` | Criar deal |
| GET | `/api/deals/:id` | Detalhes do deal |
| PUT | `/api/deals/:id` | Atualizar deal |
| DELETE | `/api/deals/:id` | Excluir deal |
| POST | `/api/deals/:id/move` | Mover para outro estágio |
| GET | `/api/deals/:id/notes` | Listar notas |
| POST | `/api/deals/:id/notes` | Criar nota |
| PUT | `/api/deals/:id/notes/:noteId` | Editar nota |
| GET | `/api/deals/:id/activities` | Listar atividades |
| POST | `/api/deals/:id/attachments` | Upload de anexo |
| DELETE | `/api/deals/:id/attachments/:attachmentId` | Remover anexo |

---

## Interface de Deal

```
┌─────────────────────────────────────────────────────────────┐
│  Deal: Proposta Software CRM                    [Editar] [X]│
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Contato: João Silva          Valor: R$ 25.000,00          │
│  Estágio: Negociação          Probabilidade: 70%           │
│  Previsão: 15/02/2026         Status: Em Aberto            │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  [Notas] [Atividades] [Anexos]                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📝 Notas                                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Reunião realizada em 10/01. Cliente pediu desconto  │   │
│  │ de 10%. Aguardando aprovação do gerente.            │   │
│  │                          - Ana, 10/01/2026 14:30    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  [+ Adicionar Nota]                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

[← Voltar para User Stories](README.md)
