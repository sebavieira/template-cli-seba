# Epic 04: Funis de Vendas (Kanban)

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Must Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que vendedores visualizem e gerenciem seu pipeline de vendas em formato Kanban, com estágios customizáveis e movimentação drag-and-drop de contatos.

---

## User Stories

### US-04.1: Criação de Funil

**Como** um admin,
**Quero** criar funis de vendas,
**Para que** eu organize meus processos comerciais.

**Critérios de Aceitação:**
- [ ] Criação de funil com nome e descrição
- [ ] Múltiplos funis por empresa
- [ ] Estágios padrão criados automaticamente (Novo, Qualificação, Proposta, Negociação, Ganho, Perdido)
- [ ] Funil padrão definido para novos contatos

**Prioridade:** Must Have

---

### US-04.2: Gerenciamento de Estágios

**Como** um admin,
**Quero** customizar os estágios do meu funil,
**Para que** ele reflita meu processo de vendas.

**Critérios de Aceitação:**
- [ ] Criação de estágios com nome e cor
- [ ] Descrição opcional da etapa, exibida via ícone de informação
- [ ] Reordenação de estágios (drag-and-drop)
- [ ] Marcação de estágio como "Ganho" ou "Perdido"
- [ ] Exclusão de estágios (com migração de contatos)
- [ ] Mínimo de 2 estágios por funil

**Prioridade:** Must Have

---

### US-04.3: Movimentação de Contatos

**Como** um vendedor,
**Quero** mover contatos entre estágios,
**Para que** eu atualize o status das negociações.

**Critérios de Aceitação:**
- [ ] Drag-and-drop de cards entre colunas
- [ ] Movimentação via dropdown no card
- [ ] Registro de data de entrada no estágio
- [ ] Histórico de movimentações
- [ ] Atualização em tempo real para outros usuários

**Prioridade:** Must Have

---

### US-04.4: Visualização Kanban

**Como** um vendedor,
**Quero** ver meus contatos em formato Kanban,
**Para que** eu tenha visão geral do pipeline.

**Critérios de Aceitação:**
- [ ] Colunas representando estágios
- [ ] Cards com nome, telefone, última atividade
- [ ] Indicador de tags no card
- [ ] Indicador de mensagens não lidas
- [ ] Métricas no header da coluna com rótulo e base visível (ex.: "Conv. 25% (1 de 4)")
- [ ] Percentuais com tooltip explicando fórmula e período
- [ ] Scroll horizontal para muitos estágios
- [ ] Contagem de contatos por estágio

**Prioridade:** Must Have

---

### US-04.5: Limites de WIP

**Como** um admin,
**Quero** definir limites de WIP por estágio,
**Para que** minha equipe não sobrecarregue etapas.

**Critérios de Aceitação:**
- [ ] Configuração de limite por estágio
- [ ] Alerta visual quando limite atingido
- [ ] Bloqueio opcional de entrada quando excedido
- [ ] Relatório de estágios acima do limite

**Prioridade:** Could Have

---

### US-04.6: Filtros no Kanban

**Como** um vendedor,
**Quero** filtrar contatos no Kanban,
**Para que** eu veja apenas os relevantes.

**Critérios de Aceitação:**
- [ ] Filtro por tags
- [ ] Filtro por responsável (em equipes)
- [ ] Filtro por período de criação
- [ ] Busca por nome/telefone
- [ ] Combinação de filtros

**Prioridade:** Should Have

---

## Modelo de Dados

```prisma
model Funnel {
  id        String   @id @default(cuid())
  companyId String
  name      String
  isDefault Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  company Company        @relation(fields: [companyId], references: [id])
  stages  ContactStage[]
}

model ContactStage {
  id        String   @id @default(cuid())
  funnelId  String
  name      String
  description String?
  color     String   @default("#6366f1")
  order     Int
  isWon     Boolean  @default(false)
  isLost    Boolean  @default(false)
  wipLimit  Int?
  createdAt DateTime @default(now())

  funnel   Funnel    @relation(fields: [funnelId], references: [id])
  contacts Contact[]
  deals    Deal[]
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/funnels` | Listar funis |
| POST | `/api/funnels` | Criar funil |
| GET | `/api/funnels/:id` | Detalhes do funil com estágios |
| PUT | `/api/funnels/:id` | Atualizar funil |
| DELETE | `/api/funnels/:id` | Excluir funil |
| POST | `/api/funnels/stages` | Criar estágio |
| PUT | `/api/funnels/stages/:id` | Atualizar estágio |
| DELETE | `/api/funnels/stages/:id` | Excluir estágio |
| POST | `/api/funnels/stages/:id/bulk-move` | Mover contatos em lote |

---

## Interface do Kanban

```
┌────────────────────────────────────────────────────────────────┐
│  Funil: Vendas B2B                           [+ Novo Contato]  │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│ │  Novo   │ │ Qualif. │ │Proposta │ │ Negoc.  │ │  Ganho  │   │
│ │  (12)   │ │   (8)   │ │   (5)   │ │   (3)   │ │  (24)   │   │
│ ├─────────┤ ├─────────┤ ├─────────┤ ├─────────┤ ├─────────┤   │
│ │┌───────┐│ │┌───────┐│ │┌───────┐│ │┌───────┐│ │┌───────┐│   │
│ ││ João  ││ ││ Maria ││ ││Carlos ││ ││ Ana   ││ ││Pedro  ││   │
│ ││ @tag  ││ ││ @tag  ││ ││ @tag  ││ ││ @tag  ││ ││ @tag  ││   │
│ │└───────┘│ │└───────┘│ │└───────┘│ │└───────┘│ │└───────┘│   │
│ │┌───────┐│ │┌───────┐│ │         │ │         │ │┌───────┐│   │
│ ││ Lucas ││ ││ Paula ││ │         │ │         │ ││Roberta││   │
│ ││       ││ ││       ││ │         │ │         │ ││       ││   │
│ │└───────┘│ │└───────┘│ │         │ │         │ │└───────┘│   │
│ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘   │
└────────────────────────────────────────────────────────────────┘
```

---

[← Voltar para User Stories](README.md)
