# Epic 03: CRM de Contatos

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Must Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que vendedores gerenciem seus contatos de forma organizada, com tags, notas, histórico de interações e funcionalidades de bloqueio.

---

## User Stories

### US-03.1: Listagem de Contatos

**Como** um vendedor,
**Quero** ver a lista de todos os meus contatos,
**Para que** eu possa encontrar e gerenciar clientes.

**Critérios de Aceitação:**
- [ ] Lista paginada (50 por página)
- [ ] Ordenação por nome, última atividade, data de criação
- [ ] Exibição de avatar, nome, telefone, tags
- [ ] Indicador de mensagens não lidas
- [ ] Performance: < 500ms para carregar

**Prioridade:** Must Have

---

### US-03.2: Criação de Contato

**Como** um vendedor,
**Quero** criar contatos manualmente,
**Para que** eu possa adicionar leads de outras fontes.

**Critérios de Aceitação:**
- [ ] Formulário com nome, telefone (obrigatório), email (opcional)
- [ ] Validação de formato de telefone brasileiro
- [ ] Verificação de duplicidade por telefone
- [ ] Associação opcional a estágio de funil

**Prioridade:** Must Have

---

### US-03.3: Edição de Contato

**Como** um vendedor,
**Quero** editar informações de um contato,
**Para que** eu mantenha dados atualizados.

**Critérios de Aceitação:**
- [ ] Edição de todos os campos do perfil
- [ ] Campos: nome, telefone, email, empresa, cargo, cidade
- [ ] Validação de campos
- [ ] Histórico de alterações (opcional)

**Prioridade:** Must Have

---

### US-03.4: Sistema de Tags

**Como** um vendedor,
**Quero** adicionar tags aos contatos,
**Para que** eu organize e filtre minha base.

**Critérios de Aceitação:**
- [ ] Criação de tags com nome e cor
- [ ] Atribuição de múltiplas tags por contato
- [ ] Remoção de tags
- [ ] Filtragem de contatos por tag
- [ ] Tags exclusivas por empresa

**Prioridade:** Must Have

---

### US-03.5: Notas em Contatos

**Como** um vendedor,
**Quero** adicionar notas em contatos,
**Para que** eu registre informações importantes.

**Critérios de Aceitação:**
- [ ] Criação de notas de texto livre
- [ ] Lista de notas ordenada por data
- [ ] Autor e timestamp em cada nota
- [ ] Edição e exclusão de notas próprias

**Prioridade:** Should Have

---

### US-03.6: Bloqueio de Contatos

**Como** um vendedor,
**Quero** bloquear contatos indesejados,
**Para que** eles não recebam respostas automáticas.

**Critérios de Aceitação:**
- [ ] Toggle de bloqueio no perfil do contato
- [ ] Contatos bloqueados não recebem respostas da IA
- [ ] Indicador visual de bloqueio na lista
- [ ] Filtro para ver apenas bloqueados

**Prioridade:** Should Have

---

### US-03.7: Busca e Filtros

**Como** um vendedor,
**Quero** buscar contatos por nome, telefone ou email,
**Para que** eu encontre rapidamente quem procuro.

**Critérios de Aceitação:**
- [ ] Busca em tempo real (debounced)
- [ ] Busca por nome, telefone, email
- [ ] Filtros combinados: tags, estágio, status
- [ ] Resultado instantâneo (< 200ms)

**Prioridade:** Must Have

---

## Modelo de Dados

```prisma
model Contact {
  id              String   @id @default(cuid())
  companyId       String
  phone           String
  name            String?
  email           String?
  avatarUrl       String?
  companyName     String?
  jobTitle        String?
  city            String?
  isBlocked       Boolean  @default(false)
  lastActivityAt  DateTime @default(now())
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  // Relacionamentos
  stageId         String?
  company         Company         @relation(fields: [companyId], references: [id])
  stage           ContactStage?   @relation(fields: [stageId], references: [id])
  notes           ContactNote[]
  tags            ContactTagAssignment[]
  conversations   WhatsAppConversation[]
  deals           Deal[]
  aiProfile       ContactAiProfile?

  @@unique([companyId, phone])
}

model ContactTag {
  id        String   @id @default(cuid())
  companyId String
  name      String
  color     String   @default("#6366f1")

  company     Company                @relation(fields: [companyId], references: [id])
  assignments ContactTagAssignment[]

  @@unique([companyId, name])
}

model ContactTagAssignment {
  id        String @id @default(cuid())
  contactId String
  tagId     String

  contact Contact    @relation(fields: [contactId], references: [id])
  tag     ContactTag @relation(fields: [tagId], references: [id])

  @@unique([contactId, tagId])
}

model ContactNote {
  id        String   @id @default(cuid())
  contactId String
  userId    String
  content   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  contact Contact @relation(fields: [contactId], references: [id])
  user    User    @relation(fields: [userId], references: [id])
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/contacts` | Listar contatos (com paginação e filtros) |
| POST | `/api/contacts` | Criar contato |
| GET | `/api/contacts/:id` | Detalhes do contato |
| PUT | `/api/contacts/:id` | Atualizar contato |
| DELETE | `/api/contacts/:id` | Excluir contato |
| POST | `/api/contacts/:id/toggle-block` | Bloquear/desbloquear |
| GET | `/api/contacts/:id/notes` | Listar notas |
| POST | `/api/contacts/:id/notes` | Criar nota |
| GET | `/api/contacts/tags` | Listar tags |
| POST | `/api/contacts/tags` | Criar tag |
| POST | `/api/contacts/tags/:id/assign` | Atribuir tag |

---

[← Voltar para User Stories](README.md)
