# Implementacoes Extras

Este documento registra o planejamento das melhorias combinadas para funis/deals e analytics.

## Escopo

1) Copiar negociacao para outro funil (sem perder historico no funil atual).
2) Corrigir taxa de conversao (ganhos / total de negociacoes criadas), respeitando filtros.
3) Renomear "Significado" para "Descricao" de etapa, tornar opcional e exibir via tooltip.
4) Definicoes no Analytics via tooltip (especialmente "Em Risco").
5) Motivo de perda (opcional) ao marcar como perdido + grafico no Analytics.
6) Ticket medio (total/ganhos/perdas), somente valores > 0.

## Relacionado

- Ver `docs/INCONSISTENCIAS-PLANO.md` para o plano de consistencia geral (isolamento, automacoes, analytics e IA builder).

## Decisoes ja alinhadas

- Acoes no card: criar copia do deal em outro funil (o original permanece).
- Ao copiar: usuario escolhe funil e etapa de destino.
- Copia permite deals fechados (WON/LOST), ex.: comercial -> onboarding.
- Motivo de perda: opcional.
- Ticket medio: considerar apenas valores > 0.
- Taxa de conversao respeita filtros ativos (status/tags/owner/periodo).
- Opcao A escolhida: copiar deal para novo funil mantendo historico do original (sem mover).

## UI/UX guardrails (para evitar erro de interpretacao)

- Acao no card: menu "Copiar para outro funil" com label claro (nao usar "Mover").
- Modal com resumo origem -> destino (funil/etapa) e confirmacao curta.
- Se etapa destino for won/lost, mostrar badge de alerta no select.
- Aviso explicito: "O deal original permanece no funil atual".
- Link bidirecional entre deals ("Copiado de" / "Copiado para") na ficha.
- Mostrar breve resumo no deal copiado: valor total e data de fechamento do deal original.
- Analytics: tooltips com definicao e contagem base para evitar leitura errada.
- No header das colunas, nao mostrar percentual sem legenda ou base (ex.: "25% (1 de 4)").
- Se percentual for conversao, rotular como "Conv. p/ proxima" e mover para analytics quando possivel.
- Se percentual for variacao temporal, rotular como "Delta vs periodo" com tooltip.
- Para "Sem valor", preferir contagem agregada no header ("Sem valor: 6").

## Plano de implementacao (alto nivel)

Fase 1 - Design e contratos
- Definir UX do menu do card e modal de copia (funil + etapa).
- Definir campos copiados e comportamento do status no deal novo.
- Confirmar se o contato permanece no funil original (padrao da opcao A).
- Ajustar contrato de API e payloads.

Fase 2 - Modelo de dados
- Adicionar campo opcional para motivo de perda (ex.: deals.loss_reason).
- (Opcional) Adicionar campo de rastreio (ex.: deals.cloned_from_deal_id).
- Ajustar migrations e schema Prisma.

Fase 3 - Backend
- Novo endpoint para duplicar deal (ex.: POST /api/deals/duplicate).
- Validar etapa do funil destino e criar deal novo.
- Registrar deal_activity de clonagem e status (ex.: created + cloned_from).
- Disparar flows para o novo deal (stage_changed) com contexto correto.

Fase 4 - Frontend
- Menu no card com acao "Copiar para outro funil".
- Modal de copia com selects de funil e etapa.
- Atualizacao de cache para kanban e analytics.
- Permitir copia de deals fechados.
- Exibir links "Copiado de" / "Copiado para" na ficha do deal.

Fase 5 - Analytics e UI de etapas
- Corrigir taxa de conversao para ganhos / criadas (com filtros).
- Adicionar ticket medio (total/ganhos/perdas, valores > 0).
- Tooltips de definicao (ex.: Em Risco = OPEN com expectedCloseDate vencida).
- Renomear "Significado" -> "Descricao" e tornar opcional.

Fase 6 - Testes e build
- Ajustar testes de deals/funnels e flows.
- Rodar build/test quando for implementar.

## Detalhes por melhoria

### 1) Copiar negociacao para outro funil

Objetivo
- Criar um novo deal no funil destino sem apagar o original.

UX
- Menu no card (tres pontos) com acao de copia.
- Modal com selecao de funil e etapa obrigatoria.
- Confirmacao antes de criar a copia.

Campos copiados (sugestao)
- title, contactId, value, currency, expectedCloseDate, priority, userId.
- companyName pode vir do deal ou do contato.

Status do novo deal
- Segue a etapa selecionada (se etapa for won/lost, status correspondente).

Atividades
- Registrar atividade no deal novo: "cloned_from" com dealId de origem.
- Registrar atividade no deal original: "cloned_to" com dealId criado.

Contato vinculado
- Opcao A: manter o contato no funil atual, nao mover automaticamente.

### 2) Taxa de conversao

Definicao
- Ganhos / total de negociacoes criadas (com filtros ativos).

Ajustes
- Garantir created_at disponivel no modelo de deals do frontend.
- Se filtro por periodo existir, usar created_at para o denominador.

### 3) Descricao da etapa

Mudancas
- "meaning" -> "description" (campo opcional).
- UI mostra icone de info na coluna e no mobile.

Compatibilidade
- Ajustar validacoes (zod) para nao obrigar descricao.
- Manter leitura de meaning existente em dados legados, se necessario.

### 4) Tooltips no Analytics

Definicoes sugeridas
- Em Risco: deals OPEN com expectedCloseDate < hoje.
- Taxa de Conversao: ganhos / criadas no periodo filtrado.
- Ticket Medio: media de valores > 0 (com contagem base).

### 5) Motivo de perda

Comportamento
- Input opcional no fluxo "Perder".
- Persistir no deal (loss_reason) e registrar no activity payload.

Analytics
- Grafico de distribuicao por motivo (somente motivos informados).

### 6) Ticket medio

Regras
- Somente deals com value > 0.
- Exibir: total, ganhos, perdas.

Formula
- ticket_medio = soma(value) / contagem(value > 0).

## Impacto tecnico (arquivos chave)

Frontend
- src/components/funnels/FunnelDealCard.tsx
- src/components/funnels/DealDetailsDrawer.tsx
- src/components/funnels/FunnelAnalyticsPanel.tsx
- src/components/funnels/FunnelStageColumn.tsx
- src/components/funnels/StageFormModal.tsx
- src/components/funnels/MobileFunnelView.tsx
- src/pages/Funnels.tsx
- src/hooks/useFunnelKanban.ts
- src/lib/api.ts

Backend
- server/src/modules/deals/deals.routes.ts
- server/src/modules/deals/deals.service.ts
- server/src/modules/funnels/funnels.service.ts
- server/src/modules/flows/flow-events.service.ts
- server/prisma/schema.prisma

## Pendencias para fechar antes de implementar

- Confirmar se a taxa de conversao deve considerar deals criadas no periodo via created_at.
- Confirmar formato do grafico de motivos de perda (barra/pizza/pareto).
