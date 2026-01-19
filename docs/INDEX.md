# 📚 Índice da Documentação

**Navegação completa para toda a documentação do projeto**

---

## 🚀 Quick Links

| Documento | Descrição | Quando Usar |
|-----------|-----------|-------------|
| [STATUS.md](STATUS.md) | Estado atual do projeto | **SEMPRE ao iniciar** |
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
| [Epic 01](prd/04-user-stories/epic-01-{{nome}}.md) | {{Descrição}} | US-001 a US-00X |
| [Epic 02](prd/04-user-stories/epic-02-{{nome}}.md) | {{Descrição}} | US-00X a US-00Y |
| ... | ... | ... |

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
| 10 | [Observabilidade](spec/10-observabilidade.md) | Logs, métricas | ~4K |
| 11 | [Segurança](spec/11-seguranca.md) | Auth, OWASP | ~5K |
| 12 | [Performance](spec/12-performance.md) | Otimizações | ~4K |
| 13 | [Rastreabilidade](spec/13-rastreabilidade.md) | Matriz US→Testes | ~3K |

### APIs por Domínio

| Domínio | Endpoints | Prioridade |
|---------|-----------|------------|
| [{{Domínio 1}}](spec/04-contratos-api/{{dominio1}}.md) | CRUD + ações | Must |
| [{{Domínio 2}}](spec/04-contratos-api/{{dominio2}}.md) | CRUD + ações | Must |
| ... | ... | ... |

---

## 📊 Documentos Operacionais

| Documento | Descrição |
|-----------|-----------|
| [STATUS.md](STATUS.md) | Estado atual e progresso |
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

**Última Atualização:** {{DATA}}
