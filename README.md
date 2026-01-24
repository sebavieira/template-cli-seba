# Evo AI Connect - Documentação

> Plataforma de automação WhatsApp com IA, CRM integrado e gestão de funis de vendas

**Versão:** 1.0.0
**Última Atualização:** 2026-01-20

---

## O que é o Evo AI Connect?

O **Evo AI Connect** é uma plataforma completa para automação de vendas via WhatsApp, integrando:

1. **CRM Inteligente** - Gestão de contatos, deals e funis de vendas
2. **Integração WhatsApp** - Conexão com Evolution API/UAZAPI para envio/recebimento de mensagens
3. **Inteligência Artificial** - GPT-4 para sugestões de resposta e análise de conversas
4. **Automações** - Follow-ups automáticos e flows visuais
5. **Dashboard** - Métricas e relatórios em tempo real

---

## Stack Tecnológica

### Backend
- **Runtime:** Node.js 20.x
- **Framework:** Fastify 4.28.1
- **Linguagem:** TypeScript 5.6.3
- **ORM:** Prisma 5.22.0
- **Banco de Dados:** PostgreSQL 16 + pgvector
- **Queue:** Redis 7 + BullMQ 5.x

### Frontend
- **Framework:** React 18.3.1
- **Build Tool:** Vite 5.4.19
- **Estilização:** TailwindCSS + Shadcn/UI
- **Estado:** Zustand + TanStack Query
- **Realtime:** Socket.io Client

### Integrações
- **WhatsApp:** Evolution API / UAZAPI
- **IA:** OpenAI GPT-4
- **Embeddings:** pgvector para busca semântica

---

## Estrutura da Documentação

```
docs/
├── INDEX.md                    ← Navegação completa
├── STATUS.md                   ← Estado atual do projeto
├── MANUTENÇÃO.md               ← Guia de manutenção
│
├── prd/                        ← Requisitos de produto
│   ├── README.md               ← Índice do PRD
│   ├── 01-visao-objetivos.md
│   ├── 02-contexto-personas.md
│   ├── 03-escopo.md
│   ├── 04-user-stories/        ← 9 Epics
│   ├── 05-rnf.md
│   ├── 06-priorizacao.md
│   ├── 07-dependencias.md
│   ├── 08-compliance.md
│   ├── 09-metricas.md
│   ├── 10-riscos.md
│   └── 11-glossario.md
│
└── spec/                       ← Especificações técnicas
    ├── README.md               ← Índice da SPEC
    ├── 01-visao-geral.md
    ├── 02-arquitetura.md
    ├── 03-modelo-dados.md
    ├── 04-contratos-api/       ← 8 domínios
    ├── 05-diagramas-sequencia.md
    ├── 06-maquina-estados.md
    ├── 07-tratamento-erros.md
    ├── 08-estrategia-testes.md
    ├── 09-deployment.md
    ├── 10-observabilidade.md
    ├── 11-seguranca.md
    ├── 12-performance.md
    └── 13-rastreabilidade.md
```

---

## Como Usar Esta Documentação

### Para Desenvolvedores

1. **Início:** Leia `docs/STATUS.md` para ver o estado atual
2. **Requisitos:** Consulte `docs/prd/` para entender o que construir
3. **Implementação:** Use `docs/spec/` para detalhes técnicos

### Para Product Managers

1. **Visão Geral:** `docs/prd/01-visao-objetivos.md`
2. **Personas:** `docs/prd/02-contexto-personas.md`
3. **Roadmap:** `docs/prd/06-priorizacao.md`

### Para Agentes de IA

1. **Quick Start:** `AI-START.md` (~2K tokens)
2. **Protocolo:** `.ai-instructions.md`
3. **Config Claude:** `.claude/CLAUDE.md`

---

## Comandos do Sistema Autônomo

Ao usar com Claude Code ou outras IAs:

| Comando | Descrição |
|---------|-----------|
| `continue` | Continua implementando próxima tarefa |
| `status` | Mostra progresso atual |
| `teste` | Executa testes |
| `explica` | Explica última tarefa em linguagem simples |
| `pausa` | Salva estado para continuar depois |
| `o que falta?` | Lista próximas tarefas |

---

## Funcionalidades Principais

### Epic 01: Autenticação
- Registro e login de usuários
- JWT com refresh tokens
- RBAC (admin/user)

### Epic 02: WhatsApp
- Conexão de instâncias
- Envio/recebimento de mensagens
- Status de entrega

### Epic 03: Contatos
- CRUD de contatos
- Tags e segmentação
- Importação em massa

### Epic 04: Funis
- Criação de funis e etapas
- Configuração de cores e ordem

### Epic 05: IA
- Embeddings de conversas
- Sugestões de resposta
- Análise de sentimento

### Epic 06: Deals
- Gestão de negociações
- Notas com IA
- Movimentação entre etapas

### Epic 07: Follow-ups
- Agendamento automático
- Templates de mensagens
- Cancelamento inteligente

### Epic 08: Dashboard
- Métricas de funil
- KPIs de mensagens
- Relatórios exportáveis

### Epic 09: Flows
- Editor visual de automações
- Triggers e condições
- Execução automática

---

## Status do Projeto

| Fase | Status | Progresso |
|------|--------|-----------|
| Documentação | ✅ Completo | 100% |
| Setup/Infra | ⏳ Pendente | 0% |
| Autenticação | ⏳ Pendente | 0% |
| WhatsApp | ⏳ Pendente | 0% |
| CRM Core | ⏳ Pendente | 0% |
| IA & Automação | ⏳ Pendente | 0% |
| Dashboard | ⏳ Pendente | 0% |

**Progresso Total:** Fase 1 concluída (Documentação)

---

## Contribuindo

1. Leia `docs/STATUS.md` para identificar próxima tarefa
2. Consulte a documentação relevante (PRD + SPEC)
3. Implemente seguindo os padrões do projeto
4. Atualize `docs/STATUS.md` após completar

---

## Links Úteis

- [Status do Projeto](docs/STATUS.md)
- [Índice de Documentação](docs/INDEX.md)
- [PRD - Requisitos](docs/prd/README.md)
- [SPEC - Técnico](docs/spec/README.md)
- [Changelog](CHANGELOG.md)

---

<<<<<<< HEAD
**Versão:** 1.0.0
**Data:** 2026-01-19
=======
**Projeto:** Evo AI Connect
**Versão:** 0.2.0
**Data:** 2026-01-20
>>>>>>> a270160 (docs: update documentation)
