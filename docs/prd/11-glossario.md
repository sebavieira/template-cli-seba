# 11. Glossário

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## Termos de Negócio

| Termo | Definição |
|-------|-----------|
| **Lead** | Contato potencialmente interessado em comprar |
| **Prospect** | Lead que demonstrou interesse ativo |
| **Conversão** | Ação de transformar lead em cliente |
| **Pipeline** | Funil de vendas com estágios definidos |
| **Win Rate** | Taxa de negócios ganhos vs total |
| **Churn** | Taxa de cancelamento/perda de clientes |
| **Follow-up** | Acompanhamento de contato para manter engajamento |
| **SDR** | Sales Development Representative - vendedor de prospecção |
| **Closer** | Vendedor responsável por fechamento |

---

## Termos do Produto

| Termo | Definição |
|-------|-----------|
| **Instância** | Conexão ativa de um número WhatsApp |
| **Conversa** | Histórico de mensagens com um contato |
| **Tag** | Etiqueta para organizar contatos |
| **Estágio** | Fase do funil em que o contato se encontra |
| **Deal** | Oportunidade de venda com valor monetário |
| **Flow** | Automação visual com nodes e conexões |
| **Prompt** | Instrução de sistema para a IA |
| **Quick Reply** | Resposta pré-definida para uso rápido |
| **Autonomia** | Nível de ações que a IA pode executar sozinha |
| **Sentimento** | Análise emocional da conversa (positivo/neutro/negativo) |

---

## Termos Técnicos

| Termo | Definição |
|-------|-----------|
| **API** | Application Programming Interface - interface de comunicação entre sistemas |
| **Endpoint** | URL específica que recebe requisições da API |
| **JWT** | JSON Web Token - formato de token para autenticação |
| **CRUD** | Create, Read, Update, Delete - operações básicas de dados |
| **MVP** | Minimum Viable Product - versão mínima funcional |
| **ORM** | Object-Relational Mapping - mapeamento de objetos para banco |
| **REST** | Representational State Transfer - arquitetura de APIs |
| **Webhook** | Callback HTTP que notifica eventos em tempo real |
| **WebSocket** | Protocolo para comunicação bidirecional em tempo real |
| **Multi-tenant** | Arquitetura onde múltiplas empresas usam o mesmo sistema isoladamente |
| **Worker** | Processo background que executa tarefas assíncronas |
| **Queue** | Fila de tarefas para processamento ordenado |
| **Token** | Unidade de texto processada por modelos de IA |
| **Embedding** | Representação vetorial de texto para busca semântica |

---

## Siglas e Acrônimos

| Sigla | Significado |
|-------|-------------|
| **PRD** | Product Requirements Document |
| **SPEC** | Especificação Técnica |
| **US** | User Story |
| **RNF** | Requisito Não Funcional |
| **MoSCoW** | Must/Should/Could/Won't - framework de priorização |
| **LGPD** | Lei Geral de Proteção de Dados |
| **SLA** | Service Level Agreement |
| **KPI** | Key Performance Indicator |
| **NPS** | Net Promoter Score |
| **RBAC** | Role-Based Access Control |
| **CRM** | Customer Relationship Management |
| **WIP** | Work In Progress |
| **IA** | Inteligência Artificial |
| **GPT** | Generative Pre-trained Transformer |
| **LLM** | Large Language Model |

---

## Status e Estados

### Contatos

| Estado | Significado |
|--------|-------------|
| **Ativo** | Contato com atividade recente |
| **Inativo** | Sem atividade há mais de 30 dias |
| **Bloqueado** | Contato impedido de receber mensagens automáticas |

### Deals

| Estado | Significado | Cor |
|--------|-------------|-----|
| **Open** | Deal em andamento | 🔵 |
| **Won** | Deal ganho/fechado | 🟢 |
| **Lost** | Deal perdido | 🔴 |

### Follow-ups

| Estado | Significado |
|--------|-------------|
| **Pending** | Aguardando envio |
| **Sent** | Enviado com sucesso |
| **Responded** | Contato respondeu |
| **Failed** | Falha no envio |
| **Cancelled** | Cancelado (contato respondeu antes) |

### Flows

| Estado | Significado |
|--------|-------------|
| **Draft** | Rascunho, não executável |
| **Active** | Publicado e executando |
| **Paused** | Pausado temporariamente |

---

## Prioridades

| Prioridade | Significado |
|------------|-------------|
| **Must Have** | Obrigatório - sistema não funciona sem |
| **Should Have** | Importante - agrega valor significativo |
| **Could Have** | Desejável - melhoria se houver tempo |
| **Won't Have** | Fora do escopo atual |

---

## Métricas do Produto

| Métrica | Definição | Como Calcular |
|---------|-----------|---------------|
| **Taxa de Resposta** | % de mensagens respondidas | (Respondidas / Recebidas) × 100 |
| **Tempo de Resposta** | Tempo médio até primeira resposta | Média(timestamp_resposta - timestamp_mensagem) |
| **Taxa de Automação** | % de respostas geradas por IA | (Respostas IA / Total Respostas) × 100 |
| **Win Rate** | % de deals ganhos | (Deals Won / Total Deals) × 100 |
| **Conversão de Estágio** | % de contatos que avançam | (Avançaram / Total no Estágio) × 100 |

---

## Referências

- [Documentação interna do projeto](../INDEX.md)
- [PRD Completo](README.md)
- [SPEC Técnica](../spec/README.md)

---

[← Voltar para Índice PRD](README.md)
