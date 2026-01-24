# PRD - Product Requirements Document

**Projeto:** Evo AI Connect
**Versão:** 1.0.0
**Status:** Ativo
**Última Atualização:** 2026-01-19
**Owner:** Product Team

---

## Índice de Navegação

### 1. [Sumário Executivo](01-visao-objetivos.md)
Visão do produto, objetivos de negócio, critérios de sucesso e roadmap.

**Conteúdo:**
- Declaração de Visão
- Objetivos de Negócio
- Critérios de Sucesso
- Timeline e Roadmap

---

### 2. [Contexto do Produto](02-contexto-personas.md)
Problema sendo resolvido, personas detalhadas e necessidades dos usuários.

**Conteúdo:**
- Declaração do Problema
- Personas Detalhadas (Carlos, Ana, Roberto)
- Necessidades e Dores
- Análise Competitiva

---

### 3. [Escopo do Produto](03-escopo.md)
Definição do que está incluído no MVP e fora do escopo.

**Conteúdo:**
- No Escopo (MVP)
- Fora do Escopo
- Roadmap Futuro
- Premissas e Restrições

---

### 4. [User Stories e Requisitos](04-user-stories/README.md)
User stories detalhadas organizadas em 9 epics.

**Epics:**
- [Epic 01: Autenticação](04-user-stories/epic-01-autenticacao.md)
- [Epic 02: WhatsApp](04-user-stories/epic-02-whatsapp.md)
- [Epic 03: Contatos](04-user-stories/epic-03-contatos.md)
- [Epic 04: Funis](04-user-stories/epic-04-funis.md)
- [Epic 05: IA](04-user-stories/epic-05-ia.md)
- [Epic 06: Deals](04-user-stories/epic-06-deals.md)
- [Epic 07: Follow-ups](04-user-stories/epic-07-followups.md)
- [Epic 08: Dashboard](04-user-stories/epic-08-dashboard.md)
- [Epic 09: Flows](04-user-stories/epic-09-flows.md)

---

### 5. [Requisitos Não Funcionais](05-rnf.md)
Performance, escalabilidade, segurança e compliance.

**Conteúdo:**
- Performance e Latência (< 200ms p95)
- Escalabilidade (500 usuários simultâneos)
- Disponibilidade (99.5% uptime)
- Segurança (JWT, HTTPS, rate limiting)
- Compliance (LGPD)
- Observabilidade (logs, métricas)

---

### 6. [Priorização de Features (MoSCoW)](06-priorizacao.md)
Classificação das funcionalidades.

**Conteúdo:**
- Must Have (obrigatório para MVP)
- Should Have (importante)
- Could Have (desejável)
- Won't Have (fora do escopo)

---

### 7. [Dependências e Integrações](07-dependencias.md)
APIs externas e serviços de terceiros.

**Integrações:**
- Evolution API / UAZAPI (WhatsApp)
- OpenAI API (IA)
- PostgreSQL + Redis
- AWS S3 / Cloudflare R2

---

### 8. [Compliance e Políticas](08-compliance.md)
Requisitos legais e regulatórios.

---

### 9. [Métricas de Sucesso](09-metricas.md)
KPIs do produto e negócio.

---

### 10. [Riscos e Mitigações](10-riscos.md)
Riscos identificados e estratégias.

---

### 11. [Glossário](11-glossario.md)
Termos e definições.

---

## Resumo do Produto

**Evo AI Connect** é uma plataforma de automação de WhatsApp com IA integrada para equipes de vendas. Combina:

- **CRM Visual**: Gerenciamento de contatos com tags e notas
- **Funis Kanban**: Pipeline de vendas visual
- **Chatbot IA**: Respostas automáticas com OpenAI
- **Deals**: Tracking de oportunidades de venda
- **Follow-ups**: Automação de acompanhamento
- **Flows**: Automação visual (em desenvolvimento)

## Como Usar Este PRD

1. **Leitura Sequencial:** Comece pelo Sumário Executivo
2. **Consulta Específica:** Use o índice acima
3. **Desenvolvimento:** Foque em User Stories e RNFs
4. **Stakeholders:** Leia Contexto, Escopo e Métricas

## Versionamento

| Versão | Data | Descrição |
|--------|------|-----------|
| 1.0.0 | 2026-01-19 | Documentação inicial completa |

---

**← [Voltar ao Projeto](../../README.md)**
