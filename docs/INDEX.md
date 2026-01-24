# 📚 Índice da Documentação - Evo AI Connect

**Navegação completa para toda a documentação do projeto**

---

## 🚀 Quick Links

| Documento | Descrição | Quando Usar |
|-----------|-----------|-------------|
| [STATUS.md](STATUS.md) | Estado atual do projeto | **SEMPRE ao iniciar** |
| [../IMPLEMENTACOES-EXTRAS.md](../IMPLEMENTACOES-EXTRAS.md) | Planejamento de melhorias extras | Quando houver novas frentes |
| [INCONSISTENCIAS-PLANO.md](INCONSISTENCIAS-PLANO.md) | Plano de inconsistencias e refinamentos | Quando houver problemas de consistencia |
| [../README.md](../README.md) | Visão geral do projeto | Contextualização |
| [prd/README.md](prd/README.md) | Índice do PRD | Requisitos |
| [spec/README.md](spec/README.md) | Índice da SPEC | Implementação |

---

## 📋 PRD - Product Requirements Document

### Documentos Principais

| # | Documento | Conteúdo | Tokens |
|---|-----------|----------|--------|
| 01 | [Visão e Objetivos](prd/01-visao-objetivos.md) | Problema, solução, métricas | ~3K |
| 02 | [Contexto e Personas](prd/02-contexto-personas.md) | Usuários, jornadas | ~4K |
| 03 | [Escopo](prd/03-escopo.md) | In/out, fases, MoSCoW | ~3K |
| 04 | [User Stories](prd/04-user-stories/) | Epics e histórias | ~8K/epic |
| 05 | [RNFs](prd/05-rnf.md) | Requisitos não funcionais | ~3K |
| 06 | [Priorização](prd/06-priorizacao.md) | MoSCoW, roadmap | ~2K |
| 07 | [Dependências](prd/07-dependencias.md) | Integrações externas | ~3K |
| 08 | [Compliance](prd/08-compliance.md) | LGPD, políticas | ~3K |
| 09 | [Métricas](prd/09-metricas.md) | KPIs, sucesso | ~2K |
| 10 | [Riscos](prd/10-riscos.md) | Riscos, mitigações | ~2K |
| 11 | [Glossário](prd/11-glossario.md) | Termos técnicos | ~2K |

### User Stories por Epic

| Epic | Descrição | User Stories |
|------|-----------|--------------|
| [Epic 01](prd/04-user-stories/epic-01-autenticacao.md) | Autenticação e Controle de Acesso | US-001 a US-005 |
| [Epic 02](prd/04-user-stories/epic-02-whatsapp.md) | Integração WhatsApp | US-006 a US-010 |
| [Epic 03](prd/04-user-stories/epic-03-contatos.md) | Gestão de Contatos | US-011 a US-015 |
| [Epic 04](prd/04-user-stories/epic-04-funis.md) | Funis e Etapas | US-016 a US-020 |
| [Epic 05](prd/04-user-stories/epic-05-ia.md) | Inteligência Artificial | US-021 a US-025 |
| [Epic 06](prd/04-user-stories/epic-06-deals.md) | Gestão de Deals | US-026 a US-030 |
| [Epic 07](prd/04-user-stories/epic-07-followups.md) | Follow-ups Automáticos | US-031 a US-035 |
| [Epic 08](prd/04-user-stories/epic-08-dashboard.md) | Dashboard e Métricas | US-036 a US-040 |
| [Epic 09](prd/04-user-stories/epic-09-flows.md) | Flows de Automação | US-041 a US-045 |

---

## 🔧 SPEC - Technical Specification

### Documentos Principais

| # | Documento | Conteúdo | Tokens |
|---|-----------|----------|--------|
| 01 | [Visão Geral](spec/01-visao-geral.md) | Overview técnico | ~3K |
| 02 | [Arquitetura](spec/02-arquitetura.md) | C4, componentes | ~5K |
| 03 | [Modelo de Dados](spec/03-modelo-dados.md) | ERD, schemas | ~6K |
| 04 | [Contratos API](spec/04-contratos-api/) | Endpoints | ~5K/domínio |
| 05 | [Diagramas Sequência](spec/05-diagramas-sequencia.md) | Fluxos | ~4K |
| 06 | [Máquina Estados](spec/06-maquina-estados.md) | Transições | ~3K |
| 07 | [Tratamento Erros](spec/07-tratamento-erros.md) | Códigos, retry | ~4K |
| 08 | [Estratégia Testes](spec/08-estrategia-testes.md) | Pirâmide, exemplos | ~5K |
| 09 | [Deployment](spec/09-deployment.md) | Docker, CI/CD | ~4K |
| 10 | [Observabilidade](spec/10-observabilidade.md) | Logs, métricas, alertas | ~4K |
| 11 | [Segurança](spec/11-seguranca.md) | Auth, OWASP, encryption | ~5K |
| 12 | [Performance](spec/12-performance.md) | Otimizações, caching | ~4K |
| 13 | [Rastreabilidade](spec/13-rastreabilidade.md) | Matriz US→Testes | ~3K |

### APIs por Domínio

| Domínio | Endpoints | Prioridade |
|---------|-----------|------------|
| [Auth](spec/04-contratos-api/auth.md) | Login, Registro, Refresh, Logout | Must |
| [WhatsApp](spec/04-contratos-api/whatsapp.md) | Instâncias, Mensagens, Webhooks | Must |
| [Contacts](spec/04-contratos-api/contacts.md) | CRUD Contatos, Tags, Importação | Must |
| [Funnels](spec/04-contratos-api/funnels.md) | CRUD Funis, Etapas | Must |
| [Deals](spec/04-contratos-api/deals.md) | CRUD Deals, Notas, Movimentação | Must |
| [AI](spec/04-contratos-api/ai.md) | Embeddings, Sugestões, Análise | Should |
| [Follow-ups](spec/04-contratos-api/followups.md) | Agendamento, Execução | Should |
| [Flows](spec/04-contratos-api/flows.md) | Editor Visual, Execução | Could |

---

## 📊 Documentos Operacionais

| Documento | Descrição |
|-----------|-----------|
| [STATUS.md](STATUS.md) | Estado atual e progresso |
| [../IMPLEMENTACOES-EXTRAS.md](../IMPLEMENTACOES-EXTRAS.md) | Planejamento adicional de melhorias |
| [INCONSISTENCIAS-PLANO.md](INCONSISTENCIAS-PLANO.md) | Plano de inconsistencias e refinamentos |
| [MANUTENÇÃO.md](MANUTENÇÃO.md) | Guia de manutenção dos docs |
| [../CHANGELOG.md](../CHANGELOG.md) | Histórico de mudanças |

---

## 🤖 Documentos para IA

| Documento | Descrição |
|-----------|-----------|
| [../AI-START.md](../AI-START.md) | Quick start para IA (~2K tokens) |
| [../.ai-instructions.md](../.ai-instructions.md) | Protocolo completo |
| [../.claude/CLAUDE.md](../.claude/CLAUDE.md) | Config Claude Code |

---

## 📖 Estratégia de Leitura

### Para Implementar Feature

```
1. docs/STATUS.md           → Identificar tarefa
2. docs/prd/04-user-stories/epic-XX.md → Requisitos
3. docs/spec/04-contratos-api/XX.md    → API
4. Implementar
```

### Para Debugging

```
1. docs/spec/04-contratos-api/[domínio].md → Contrato
2. docs/spec/07-tratamento-erros.md         → Códigos erro
3. docs/spec/06-maquina-estados.md          → Estados válidos
```

### Para Entender Contexto

```
1. docs/prd/01-visao-objetivos.md → Problema/Solução
2. docs/prd/02-contexto-personas.md → Usuários
3. docs/prd/03-escopo.md → Limites
```

---

## 🔢 Orçamento de Tokens

| Tipo de Leitura | Tokens | Arquivos |
|-----------------|--------|----------|
| Quick Start | 2K | AI-START.md |
| Contextualização | 8K | STATUS + README + INDEX |
| Feature simples | 15K | 1 epic + 1 API |
| Feature complexa | 30K | Múltiplos |
| **Limite** | **50K** | Por interação |

---

**Última Atualização:** 2026-01-21
