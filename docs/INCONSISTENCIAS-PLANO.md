# Inconsistencias e Plano de Refinos

Este documento consolida inconsistencias provaveis no sistema e um plano detalhado
para corrigir/refinar fluxos (WhatsApp, CRM, automacoes, analytics e IA).

## Objetivo

- Garantir isolamento por companyId e instanceId em toda a cadeia.
- Evitar mistura de historico, duplicidade de contatos e eventos repetidos.
- Tornar regras de funil e metricas consistentes e auditaveis.
- Habilitar IA para construir fluxos com seguranca e previsibilidade.

## Principios de consistencia

1) Fonte unica de verdade
- Identidade do contato e conversa derivam de chaves canonicas.
- Status do deal deriva da etapa final (e nao o contrario).

2) Idempotencia
- Cada mensagem e evento tem um identificador unico.
- Workers e triggers ignoram eventos repetidos.

3) Isolamento estrito
- Nenhuma consulta cruza companyId/instanceId.
- Conversas sao resolvidas por instanceId + contactNumber canonico.

4) Definicoes explicitas
- Metricas mostram base, denominador e periodo usado.
- IA sempre declara o que vai mudar antes de executar.

## Mapa de fluxos e funcoes (analise)

### Fluxo de mensagens (WhatsApp)
1) Webhook recebido -> normalizacao do numero
2) Resolver instancia (instanceId) e direcao (fromMe / self)
3) Resolver contato (companyId + phone canonico)
4) Resolver conversa (instanceId + contactNumber)
5) Persistir mensagem + disparar eventos (flows, IA, follow-ups)

Pontos criticos:
- Normalizacao inconsistente gera duplicados.
- Falta de instanceId na busca mistura historico.
- Self-messages podem ser gravadas como inbound.

Arquivos-chave:
- server/src/modules/whatsapp/whatsapp.service.ts
- server/src/modules/whatsapp/whatsapp.routes.ts
- server/src/jobs/message-buffer.worker.ts

### Fluxo de CRM (contatos, deals, funis)
1) Criar contato (companyId + phone unico)
2) Criar deal (funnelId + stageId coerentes)
3) Mover etapa (gera activity + evento de flow)
4) Fechar deal (etapa final -> status)

Pontos criticos:
- Status divergente de etapa final.
- Copia/movimento de deals sem regra clara de metrica.

Arquivos-chave:
- server/src/modules/deals/deals.service.ts
- server/src/modules/funnels/funnels.service.ts
- server/src/modules/flows/flow-events.service.ts

### Fluxo de automacoes
1) Evento entra (message_received, stage_changed, tag_added)
2) Flow engine decide caminho
3) Jobs (delay, wait_response, no_response)
4) Follow-ups e reentradas

Pontos criticos:
- Gatilhos duplicados (no_response + follow-up)
- Condicao de corrida entre timeout e resposta
- Buffer de mensagens duplicando execucao

Arquivos-chave:
- server/src/modules/flows/flow-engine.service.ts
- server/src/jobs/flow.worker.ts
- server/src/jobs/followup.worker.ts

### Fluxo de analytics
1) Coleta de dados (deals/messages)
2) Agregacao por filtro e periodo
3) Exibicao com base/denominador

Pontos criticos:
- Base incorreta (created_at vs stage_changed_at)
- Metricas sem legenda ou contagem base

Arquivos-chave:
- server/src/modules/funnels/funnels.service.ts
- src/components/funnels/FunnelAnalyticsPanel.tsx

### Fluxo de IA (acoes e builder)
1) Prompt determina acoes permitidas
2) IA sugere ou executa (modo hibrido)
3) Registro da acao e rollback

Pontos criticos:
- Acoes de alto risco sem confirmacao
- Falta de preview do fluxo antes de publicar

Arquivos-chave:
- server/src/services/chatbot-tool-executors.ts
- server/src/modules/ai/ai.service.ts

## Plano detalhado por fases

### Fase 0 - Diagnostico e baseline
Objetivo: medir inconsistencias e criar linha de base.

Tarefas:
- Criar queries de auditoria para duplicados (contacts/conversations/messages).
- Mapear taxas de erro por webhook e por instancia.
- Medir eventos duplicados (message_id repetido).
- Registrar origem do numero (raw, normalized, instanceId).

Como rodar (CONS-00):
- Script: `server/src/scripts/consistency-baseline.ts` (gera `.context/consistency/cons-00-report.json`)
- SQL: `.context/db/cons-00-baseline.sql`

Entregaveis:
- Relatorio de duplicidades e top 10 casos.
- Dashboard simples de consistencia (counts + trends).

Validacao:
- 0 duplicidades novas por 24h em ambiente controlado.

### Fase 1 - Identidade e isolamento
Objetivo: fixar chaves canonicas e isolamento por instancia.

Tarefas:
- Padronizar normalizacao E.164 (input, webhook, importacao).
- Criar campo canonico (normalizedPhone/normalizedNumber) se faltar.
- Garantir unique keys: contacts(companyId, phone) e conversations(instanceId, contactNumber).
- Ajustar resolucao de conversa para usar instanceId sempre.
- Definir tratamento de self-messages (fromMe + to = self).
- Criar rotina de merge com auditoria (move refs + log).

Entregaveis:
- Normalizador unico usado em todos os pontos.
- Script de merge idempotente com relatorio.

Validacao:
- Testes: nova empresa + mesmo numero nao traz historico.
- Teste: self-message nao vaza entre instancias.

### Fase 2 - Funis e deals
Objetivo: tornar status/etapa e copia consistentes.

Tarefas:
- Regra unica: etapa final define status (won/lost).
- Bloquear stageId de outro funil no backend.
- Explicitar regra de copia vs movimento (impacto em metricas).
- Exibir funil/etapa de origem no deal copiado.
- Criar fluxo claro para mudar funil no detalhe do deal.

Entregaveis:
- Invariantes documentados e testados.
- UI com texto explicito e tooltips.

Validacao:
- Movimentos geram activities e eventos 1x.
- Copia nao altera metrica do funil origem (regra definida).

### Fase 3 - Automacoes e filas
Objetivo: reduzir duplicidade e corrida entre jobs.

Tarefas:
- Idempotencia por message_id e flow_execution_id.
- Definir prioridade entre no_response e follow-up.
- Cancelamento de jobs quando resposta chega.
- Unificar regras de business_hours com timezone.
- Adicionar simulador dry-run para flows.

Entregaveis:
- Controle de dedupe em workers.
- Logs com correlationId por execucao.

Validacao:
- Rodar suite de testes do roteiro manual.
- Corrida timeout vs resposta sempre deterministica.

### Fase 4 - Analytics
Objetivo: tornar metricas compreensiveis e consistentes.

Tarefas:
- Definir base para cada metrica (created_at vs stage_changed_at).
- Exibir contagem base em todas as porcentagens.
- Padronizar filtros (status, owner, tags, periodo).
- Explicar formulas via tooltip contextual.

Entregaveis:
- Documento de definicoes de metricas.
- UI com tooltips e textos dinamicos por etapa.

Validacao:
- Revisao cruzada com dados reais (amostra).

### Fase 5 - IA builder ao vivo
Objetivo: IA construir fluxos com governanca.

Tarefas:
- Modo assistido: IA sugere, usuario confirma.
- Preview em tempo real (nodes, triggers, condicoes).
- Checagens de conflito (duplicidade de triggers, loops).
- Versionamento de fluxo (draft -> published).
- Registro de alteracoes e rollback rapido.

Entregaveis:
- Chat-guided builder com passos claros.
- Guardrails por nivel de autonomia.

Validacao:
- Criar fluxo simples via IA com 1 clique de confirmacao.
- Reverter fluxo e comparar diffs.

### Fase 6 - Observabilidade e QA
Objetivo: manter consistencia no tempo.

Tarefas:
- Checks noturnos de duplicidade.
- Alertas para crescimento anormal de conversas.
- Log de excecoes por instancia e webhook.
- Checklist de QA antes de deploy.

Entregaveis:
- Painel de consistencia e alertas.
- Procedimento de rollback e merge.

## Proximos passos sugeridos

1) Validar prioridades (qual fase atacar primeiro).
2) Aprovar regra de copia vs movimento para impactos de metrica.
3) Definir janela de tempo para dedupe + monitoramento.
