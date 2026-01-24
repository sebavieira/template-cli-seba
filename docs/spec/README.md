# SPEC - Especificação Técnica

**Projeto:** Evo AI Connect
**Versão:** 1.0.0
**Status:** Ativo
**Última Atualização:** 2026-01-19
**Owner:** Tech Lead

---

## Índice da Especificação

### 1. [Visão Geral](01-visao-geral.md)
Objetivo do sistema, escopo do MVP e premissas técnicas.

### 2. [Arquitetura do Sistema](02-arquitetura.md)
Visão arquitetural, componentes e fluxo de dados.

### 3. [Modelo de Dados](03-modelo-dados.md)
Diagrama ER, schemas completos e relacionamentos.

### 4. [Contratos de API](04-contratos-api/)
Endpoints REST organizados por domínio:
- [Índice de Endpoints](04-contratos-api/README.md)
- [Autenticação](04-contratos-api/auth.md)
- [WhatsApp](04-contratos-api/whatsapp.md)
- [Contatos](04-contratos-api/contacts.md)
- [Funis](04-contratos-api/funnels.md)
- [Deals](04-contratos-api/deals.md)
- [IA/Prompts](04-contratos-api/ai.md)
- [Follow-ups](04-contratos-api/followups.md)
- [Flows](04-contratos-api/flows.md)

### 5. [Diagramas de Sequência](05-diagramas-sequencia.md)
Fluxos principais em Mermaid.

### 6. [Máquina de Estados](06-maquina-estados.md)
Estados, transições e regras.

### 7. [Tratamento de Erros](07-tratamento-erros.md)
Categorização, retry e circuit breaker.

### 8. [Estratégia de Testes](08-estrategia-testes.md)
Pirâmide de testes e exemplos.

### 9. [Deployment e Infraestrutura](09-deployment.md)
Arquitetura de deployment e variáveis de ambiente.

### 10. [Observabilidade](10-observabilidade.md)
Logs, métricas e alertas.

### 11. [Segurança](11-seguranca.md)
Autenticação, autorização e proteção de dados.

### 12. [Performance](12-performance.md)
Otimizações, caching e scaling.

### 13. [Rastreabilidade](13-rastreabilidade.md)
Matriz US → RF → Endpoint → Tests.

---

## Resumo Técnico

**Evo AI Connect** é uma plataforma de automação de WhatsApp com IA integrada.

### Stack Tecnológico

| Camada | Tecnologia | Versão |
|--------|------------|--------|
| **Backend** | Fastify + TypeScript | 4.28.1 / 5.6.3 |
| **Frontend** | React + Vite | 18.3.1 / 5.4.19 |
| **Database** | PostgreSQL + pgvector | 16.x / 0.5.x |
| **Cache/Queue** | Redis + BullMQ | 7.x / 5.x |
| **ORM** | Prisma | 5.22.0 |
| **UI** | Tailwind + Shadcn/UI | 3.4.17 |
| **Real-time** | Socket.io | 4.8.3 |

### Principais Domínios

1. **Auth** - Autenticação JWT com refresh tokens
2. **WhatsApp** - Integração Evolution/UAZAPI
3. **Contacts** - CRM com tags e notas
4. **Funnels** - Pipeline Kanban de vendas
5. **Deals** - Oportunidades com valores
6. **AI** - Chatbot OpenAI com prompts
7. **Follow-ups** - Automação de acompanhamento
8. **Flows** - Automação visual (em desenvolvimento)

---

## Como Usar Este Documento

**Para Desenvolvedores:**
- Comece com Visão Geral e Arquitetura
- Consulte Modelo de Dados para schemas
- Use Contratos de API como referência

**Para Arquitetos:**
- Arquitetura do Sistema - Visão macro
- Diagramas de Sequência - Fluxos
- Performance - Otimizações

**Para DevOps:**
- Deployment e Infraestrutura
- Observabilidade
- Segurança

**Para QA:**
- Estratégia de Testes
- Rastreabilidade
- Contratos de API

---

## Convenções

- ✅ Implementado
- 🚧 Em desenvolvimento
- 📋 Planejado
- ⚠️ Atenção
- 💡 Dica

---

**← [Voltar ao Projeto](../../README.md)**
