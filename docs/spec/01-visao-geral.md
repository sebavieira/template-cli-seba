# 1. Visão Geral Técnica

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para SPEC](README.md)

---

## 1.1 Objetivo do Sistema

O **Evo AI Connect** é uma plataforma SaaS de automação de WhatsApp com Inteligência Artificial integrada, projetada para equipes de vendas de pequenas e médias empresas.

**Objetivo Técnico:** Fornecer uma infraestrutura escalável e multi-tenant que permita:
- Integração com WhatsApp via APIs de terceiros (Evolution API / UAZAPI)
- Processamento de mensagens em tempo real com respostas automáticas via OpenAI
- Gestão de contatos e pipeline de vendas (CRM)
- Automação de follow-ups e fluxos de trabalho

---

## 1.2 Escopo do MVP

### Incluído no MVP

| Funcionalidade | Descrição | Complexidade | Status |
|----------------|-----------|--------------|--------|
| **Autenticação** | JWT com refresh tokens, RBAC multi-tenant | Média | ✅ |
| **WhatsApp** | Conexão, envio/recebimento de mensagens | Alta | ✅ |
| **CRM Contatos** | Tags, notas, busca, filtros | Média | ✅ |
| **Funis Kanban** | Estágios, drag-and-drop, WIP limits | Média | ✅ |
| **Chatbot IA** | Prompts customizáveis, níveis de autonomia | Alta | ✅ |
| **Deals** | Oportunidades com valor, pipeline tracking | Média | ✅ |
| **Follow-ups** | Regras de disparo, fila de processamento | Alta | ✅ |
| **Dashboard** | Métricas, notificações de equipe | Média | ✅ |

### Excluído do MVP

- ❌ Integração Instagram/Messenger
- ❌ Aplicativo mobile nativo
- ❌ Relatórios PDF/Excel
- ❌ Integração com ERPs
- ❌ Transcrição de áudios
- ❌ Multi-idioma na interface
- ❌ Flow Builder visual completo (em desenvolvimento)

---

## 1.3 Personas Técnicas

### Desenvolvedor Backend

**Responsabilidades:**
- Implementar APIs REST com Fastify
- Gerenciar Prisma ORM e migrations
- Configurar integrações (Evolution, OpenAI)
- Implementar workers BullMQ

**Ferramentas:**
- IDE: VS Code com extensões TypeScript
- Terminal: Node.js 20.x, pnpm
- Database: PostgreSQL 16, Redis 7
- Git: GitHub

### Desenvolvedor Frontend

**Responsabilidades:**
- Implementar interfaces React com TypeScript
- Consumir APIs via TanStack Query
- Garantir UX/acessibilidade com Shadcn/UI
- Gerenciar estado com Zustand

**Ferramentas:**
- IDE: VS Code com extensões React
- Build: Vite 5.x
- Estilo: Tailwind CSS

### DevOps/SRE

**Responsabilidades:**
- CI/CD pipeline com GitHub Actions
- Monitoramento com Pino logs
- Infraestrutura Docker/Docker Compose
- Backup e disaster recovery

---

## 1.4 Premissas Técnicas

| Premissa | Impacto se Falsa |
|----------|------------------|
| Evolution API disponível 99% do tempo | Fallback para UAZAPI necessário |
| OpenAI API com latência < 5s | UX degradada, timeouts |
| PostgreSQL suporta pgvector | Alternativa para embeddings |
| Usuários têm conexão estável | Otimizar para offline-first |
| Equipe familiar com TypeScript | Curva de aprendizado maior |

---

## 1.5 Restrições Técnicas

| Restrição | Motivo | Alternativa |
|-----------|--------|-------------|
| Somente WhatsApp inicialmente | Foco no mercado principal | Instagram/Telegram no futuro |
| Evolution API não oficial | Custo-benefício | WhatsApp Business API oficial |
| OpenAI como único provider | Qualidade das respostas | Modelos alternativos (Anthropic) |
| Self-hosted obrigatório | Controle de dados LGPD | SaaS gerenciado futuro |

---

## 1.6 Decisões Arquiteturais (ADRs)

### ADR-001: Fastify como Framework Web

- **Status:** Aceito
- **Contexto:** Necessidade de framework Node.js performático com suporte a TypeScript
- **Decisão:** Fastify em vez de Express por melhor performance e schema validation nativo
- **Consequências:** Melhor throughput, menos middleware necessário, curva de aprendizado ligeiramente maior

### ADR-002: Prisma como ORM

- **Status:** Aceito
- **Contexto:** Necessidade de ORM com TypeScript type-safety e migrations
- **Decisão:** Prisma em vez de TypeORM por melhor DX e schema-first approach
- **Consequências:** Types gerados automaticamente, migrations declarativas, learning curve para queries complexas

### ADR-003: Multi-tenant via Company ID

- **Status:** Aceito
- **Contexto:** Múltiplas empresas usando a mesma infraestrutura
- **Decisão:** Isolamento via `companyId` em todas as tabelas principais
- **Consequências:** Queries sempre filtradas por empresa, índices necessários, sem sharding físico

### ADR-004: BullMQ para Processamento Assíncrono

- **Status:** Aceito
- **Contexto:** Processamento de mensagens, follow-ups e IA não podem bloquear requests
- **Decisão:** BullMQ com Redis para filas de jobs
- **Consequências:** Workers separados, retry automático, monitoramento via dashboard BullMQ

### ADR-005: Socket.io para Real-time

- **Status:** Aceito
- **Contexto:** Mensagens e notificações devem aparecer em tempo real
- **Decisão:** Socket.io em vez de WebSockets puros por melhor DX e fallbacks
- **Consequências:** Namespace por empresa, rooms por conversa, overhead de abstração

### ADR-006: pgvector para Embeddings

- **Status:** Aceito
- **Contexto:** Busca semântica em conversas para memória da IA
- **Decisão:** Extensão pgvector no PostgreSQL para armazenar embeddings OpenAI
- **Consequências:** Sem banco vetorial separado, queries híbridas SQL + vetor

---

## 1.7 Stack Tecnológico Resumido

| Camada | Tecnologia | Versão |
|--------|------------|--------|
| **Runtime** | Node.js | 20.x LTS |
| **Backend** | Fastify + TypeScript | 4.28.1 / 5.6.3 |
| **Frontend** | React + Vite | 18.3.1 / 5.4.19 |
| **Banco de Dados** | PostgreSQL + pgvector | 16.x / 0.5.x |
| **Cache/Queue** | Redis + BullMQ | 7.x / 5.x |
| **ORM** | Prisma | 5.22.0 |
| **UI Library** | Shadcn/UI + Tailwind | Latest / 3.4.17 |
| **State** | TanStack Query + Zustand | 5.x / 4.x |
| **Real-time** | Socket.io | 4.8.3 |
| **Validation** | Zod | 3.x |
| **Logs** | Pino | 8.x |

---

← [Voltar para SPEC](README.md) | [Próximo: Arquitetura →](02-arquitetura.md)
