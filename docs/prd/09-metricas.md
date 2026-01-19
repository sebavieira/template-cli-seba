# 9. Métricas de Sucesso e KPIs

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

[← Voltar para Índice PRD](README.md)

---

## 9.1 Métricas de Produto

### Aquisição

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Novos usuários | Cadastros por período | {{META}} | Semanal |
| Conversão | Visitantes → Cadastros | {{META}}% | Semanal |
| Fonte de tráfego | Origem dos usuários | - | Mensal |

### Engajamento

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| DAU | Usuários ativos diários | {{META}} | Diário |
| WAU | Usuários ativos semanais | {{META}} | Semanal |
| MAU | Usuários ativos mensais | {{META}} | Mensal |
| Sessão média | Tempo por sessão | {{META}} min | Semanal |

### Retenção

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| D1 | Retorno após 1 dia | {{META}}% | Semanal |
| D7 | Retorno após 7 dias | {{META}}% | Semanal |
| D30 | Retorno após 30 dias | {{META}}% | Mensal |
| Churn | Taxa de abandono | < {{META}}% | Mensal |

---

## 9.2 Métricas de Negócio

### Financeiras

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| MRR | Receita recorrente mensal | R$ {{META}} | Mensal |
| ARR | Receita recorrente anual | R$ {{META}} | Trimestral |
| ARPU | Receita por usuário | R$ {{META}} | Mensal |
| LTV | Valor vitalício do cliente | R$ {{META}} | Trimestral |
| CAC | Custo de aquisição | R$ {{META}} | Mensal |
| LTV/CAC | Proporção LTV para CAC | > {{META}}x | Trimestral |

### Crescimento

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Taxa de crescimento | MoM growth | {{META}}% | Mensal |
| Net Promoter Score | Satisfação do cliente | > {{META}} | Trimestral |
| Taxa de conversão | Trial → Pago | {{META}}% | Mensal |

---

## 9.3 Métricas Técnicas

### Performance

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Tempo de resposta | API p95 latency | < {{META}}ms | Contínuo |
| Uptime | Disponibilidade | > {{META}}% | Mensal |
| Error rate | Taxa de erros | < {{META}}% | Contínuo |
| TTFB | Time to first byte | < {{META}}ms | Semanal |

### Qualidade

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Cobertura de testes | % de código testado | > {{META}}% | Por deploy |
| Bugs em produção | Incidentes por semana | < {{META}} | Semanal |
| Tempo de resolução | MTTR para bugs críticos | < {{META}}h | Por incidente |
| Deploy frequency | Deploys por semana | {{META}}+ | Semanal |

---

## 9.4 Dashboard de KPIs

### Visão Executiva

```
┌─────────────────────────────────────────────────────────┐
│                    DASHBOARD MVP                         │
├─────────────────────────────────────────────────────────┤
│  Usuários Ativos     │  Receita (MRR)    │  Uptime      │
│  [  {{VALOR}}  ]     │  [R$ {{VALOR}}]   │  [{{VALOR}}%]│
├─────────────────────────────────────────────────────────┤
│  NPS: {{VALOR}}      │  CAC: R$ {{VALOR}}│  Churn: {{}}%│
└─────────────────────────────────────────────────────────┘
```

### Metas por Período

| Período | Usuários | MRR | NPS |
|---------|----------|-----|-----|
| Mês 1 | {{META}} | R$ {{META}} | {{META}} |
| Mês 3 | {{META}} | R$ {{META}} | {{META}} |
| Mês 6 | {{META}} | R$ {{META}} | {{META}} |
| Ano 1 | {{META}} | R$ {{META}} | {{META}} |

---

## 9.5 Alertas e Thresholds

### Alertas Críticos (Ação Imediata)

| Métrica | Threshold | Ação |
|---------|-----------|------|
| Uptime | < 99% | Escalar para engenharia |
| Error rate | > 5% | Investigar imediatamente |
| Latência p95 | > 500ms | Verificar performance |

### Alertas de Atenção (Monitorar)

| Métrica | Threshold | Ação |
|---------|-----------|------|
| Churn | > 10% | Analisar feedback |
| NPS | < 30 | Pesquisa com usuários |
| CAC/LTV | < 3x | Revisar estratégia |

---

[← Voltar para Índice PRD](README.md)
