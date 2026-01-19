# SPEC - Especificação Técnica

**Versão:** 1.0.0
**Status:** Template
**Última Atualização:** {{DATA}}
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
- [{{DOMINIO_1}}](04-contratos-api/{{dominio_1}}.md)
- [{{DOMINIO_2}}](04-contratos-api/{{dominio_2}}.md)

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
