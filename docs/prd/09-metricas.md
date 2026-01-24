# 9. Métricas de Sucesso e KPIs

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## 9.1 Métricas de Produto

### Aquisição

| Métrica | Definição | Meta MVP | Frequência |
|---------|-----------|----------|------------|
| Novos usuários | Cadastros por período | 50/mês | Semanal |
| Conversão cadastro | Visitantes → Cadastros | 15% | Semanal |
| Empresas ativas | Empresas com instância conectada | 30/mês | Semanal |
| Fonte de tráfego | Origem dos usuários | - | Mensal |

### Engajamento

| Métrica | Definição | Meta MVP | Frequência |
|---------|-----------|----------|------------|
| DAU | Usuários ativos diários | 100 | Diário |
| WAU | Usuários ativos semanais | 300 | Semanal |
| MAU | Usuários ativos mensais | 500 | Mensal |
| Sessão média | Tempo por sessão | 15 min | Semanal |
| Mensagens/dia | Mensagens processadas por dia | 5.000 | Diário |
| Respostas IA/dia | Respostas geradas pela IA | 1.000 | Diário |

### Retenção

| Métrica | Definição | Meta MVP | Frequência |
|---------|-----------|----------|------------|
| D1 | Retorno após 1 dia | 60% | Semanal |
| D7 | Retorno após 7 dias | 40% | Semanal |
| D30 | Retorno após 30 dias | 25% | Mensal |
| Churn | Taxa de abandono mensal | < 10% | Mensal |

---

## 9.2 Métricas de Negócio

### Financeiras

| Métrica | Definição | Meta 6 Meses | Meta 12 Meses |
|---------|-----------|--------------|---------------|
| MRR | Receita recorrente mensal | R$ 15.000 | R$ 50.000 |
| ARR | Receita recorrente anual | R$ 180.000 | R$ 600.000 |
| ARPU | Receita por usuário | R$ 99 | R$ 149 |
| LTV | Valor vitalício do cliente | R$ 1.188 | R$ 1.788 |
| CAC | Custo de aquisição | R$ 200 | R$ 300 |
| LTV/CAC | Proporção LTV para CAC | > 3x | > 5x |

### Crescimento

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Taxa de crescimento | MoM growth | 20% | Mensal |
| Net Promoter Score | Satisfação do cliente | > 40 | Trimestral |
| Taxa de conversão | Trial → Pago | 25% | Mensal |
| Upsell rate | Free → Premium | 15% | Mensal |

---

## 9.3 Métricas Técnicas

### Performance

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Tempo de resposta API | p95 latency | < 200ms | Contínuo |
| Tempo de resposta IA | Geração de resposta | < 10s | Contínuo |
| Uptime | Disponibilidade | > 99.5% | Mensal |
| Error rate | Taxa de erros 5xx | < 0.1% | Contínuo |
| TTFB | Time to first byte | < 100ms | Semanal |

### Qualidade

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Cobertura de testes | % de código testado | > 70% | Por deploy |
| Bugs em produção | Incidentes P1/P2 por semana | < 2 | Semanal |
| MTTR | Tempo médio de resolução | < 4h | Por incidente |
| Deploy frequency | Deploys por semana | 3+ | Semanal |
| Rollback rate | % de deploys revertidos | < 5% | Mensal |

### Infraestrutura

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| CPU Usage | Utilização média de CPU | < 60% | Contínuo |
| Memory Usage | Utilização de memória | < 70% | Contínuo |
| Disk I/O | Operações de disco | < 80% | Contínuo |
| Database connections | Conexões ativas | < 80 | Contínuo |
| Redis memory | Uso de memória Redis | < 500MB | Contínuo |

---

## 9.4 Métricas de IA

### Qualidade do Chatbot

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Taxa de automação | % de respostas geradas por IA | 60% | Semanal |
| Takeover rate | % de conversas assumidas por humano | < 30% | Semanal |
| Satisfação com IA | Feedback positivo | > 70% | Mensal |
| Precisão de sentimento | Acurácia da análise | > 85% | Mensal |

### Consumo de Tokens

| Métrica | Definição | Meta | Frequência |
|---------|-----------|------|------------|
| Tokens/mensagem | Média de tokens por resposta | < 500 | Semanal |
| Tokens/empresa/dia | Consumo diário por empresa | < 10.000 | Diário |
| Custo/mensagem | Custo médio de IA por resposta | < R$ 0.05 | Semanal |

---

## 9.5 Dashboard de KPIs

### Visão Executiva

```
┌─────────────────────────────────────────────────────────────────────┐
│                       DASHBOARD EVO AI CONNECT                       │
├─────────────────────────────────────────────────────────────────────┤
│  Usuários Ativos    │   Receita (MRR)     │   Uptime               │
│  [     500 MAU    ] │  [ R$ 15.000/mês  ] │  [     99.7%         ] │
├─────────────────────────────────────────────────────────────────────┤
│  Mensagens/Dia      │   Taxa Automação    │   NPS                  │
│  [     5.000      ] │  [     60%        ] │  [       42          ] │
├─────────────────────────────────────────────────────────────────────┤
│  Empresas Ativas    │   Churn Rate        │   LTV/CAC              │
│  [       30       ] │  [      8%        ] │  [      4.5x         ] │
└─────────────────────────────────────────────────────────────────────┘
```

### Metas por Período

| Período | Usuários MAU | MRR | Empresas | NPS |
|---------|--------------|-----|----------|-----|
| **Mês 1** | 100 | R$ 3.000 | 10 | 30 |
| **Mês 3** | 300 | R$ 10.000 | 25 | 35 |
| **Mês 6** | 500 | R$ 15.000 | 40 | 40 |
| **Ano 1** | 1.500 | R$ 50.000 | 100 | 50 |

---

## 9.6 Métricas de Funcionalidades

### WhatsApp

| Métrica | Definição | Meta |
|---------|-----------|------|
| Instâncias ativas | Conexões WhatsApp funcionando | > 95% |
| Tempo de conexão | QR Code → Conectado | < 30s |
| Mensagens entregues | Taxa de sucesso de envio | > 98% |
| Reconexão automática | Recuperação de desconexão | < 5min |

### CRM (Contatos e Funis)

| Métrica | Definição | Meta |
|---------|-----------|------|
| Contatos por empresa | Média de contatos | > 500 |
| Taxa de tagging | Contatos com tags | > 50% |
| Movimentação funil | Contatos movidos/semana | > 100 |
| Win rate | Deals ganhos vs total | > 25% |

### Deals

| Métrica | Definição | Meta |
|---------|-----------|------|
| Deals criados/mês | Oportunidades registradas | > 50 |
| Valor médio deal | Ticket médio | > R$ 1.000 |
| Tempo médio ciclo | Lead → Won | < 30 dias |
| Deals com notas | % com anotações | > 70% |

### Follow-ups

| Métrica | Definição | Meta |
|---------|-----------|------|
| Follow-ups enviados | Mensagens automáticas/mês | > 500 |
| Taxa de resposta | Respostas após follow-up | > 15% |
| Taxa de conversão | Follow-up → Deal | > 5% |

---

## 9.7 Alertas e Thresholds

### Alertas Críticos (Ação Imediata)

| Métrica | Threshold | Ação |
|---------|-----------|------|
| Uptime | < 99% | Escalar para engenharia |
| Error rate | > 5% | Investigar imediatamente |
| Latência p95 | > 500ms | Verificar performance |
| Conexões WhatsApp | < 90% online | Verificar Evolution API |
| OpenAI errors | > 10% | Verificar API key e limits |

### Alertas de Atenção (Monitorar)

| Métrica | Threshold | Ação |
|---------|-----------|------|
| Churn | > 15% | Analisar feedback e NPS |
| NPS | < 30 | Pesquisa com usuários |
| CAC/LTV | < 2x | Revisar estratégia de aquisição |
| Taxa automação | < 40% | Revisar prompts e configurações |
| Tokens/dia | > 50K | Otimizar uso de IA |

### Alertas de Crescimento

| Métrica | Threshold | Ação |
|---------|-----------|------|
| MoM Growth | < 10% | Intensificar marketing |
| Conversão trial | < 15% | Revisar onboarding |
| Retenção D30 | < 20% | Melhorar product-market fit |

---

## 9.8 Ferramentas de Monitoramento

### Recomendadas

| Ferramenta | Finalidade | Prioridade |
|------------|------------|------------|
| **Pino + ELK** | Logs estruturados | Must Have |
| **Grafana** | Dashboards de métricas | Should Have |
| **Prometheus** | Métricas de infraestrutura | Should Have |
| **Sentry** | Monitoramento de erros | Must Have |
| **Mixpanel/Amplitude** | Analytics de produto | Should Have |

### Implementação Atual

- ✅ Logs com Pino (JSON estruturado)
- ✅ Health checks em `/health`
- 📋 Dashboards internos (planejado)
- 📋 Alertas automáticos (planejado)

---

[← Voltar para Índice PRD](README.md)
