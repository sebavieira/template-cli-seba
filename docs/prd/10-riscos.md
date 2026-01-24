# 10. Riscos e Mitigações

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## 10.1 Matriz de Riscos

### Classificação

| Probabilidade / Impacto | Baixo | Médio | Alto |
|-------------------------|-------|-------|------|
| **Alta** | 🟡 Médio | 🟠 Alto | 🔴 Crítico |
| **Média** | 🟢 Baixo | 🟡 Médio | 🟠 Alto |
| **Baixa** | 🟢 Baixo | 🟢 Baixo | 🟡 Médio |

---

## 10.2 Riscos de Produto

### RISCO-P01: Baixa Adoção do Produto

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Alto |
| **Classificação** | 🟠 Alto |
| **Descrição** | Usuários podem não adotar o sistema ou abandonar após trial |
| **Indicadores** | Conversão < 15%, Churn > 15%, D7 < 30% |
| **Mitigação** | Onboarding guiado, feedback contínuo, iteração rápida no produto |
| **Responsável** | Product Team |

### RISCO-P02: Chatbot com Respostas Inadequadas

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Alto |
| **Classificação** | 🟠 Alto |
| **Descrição** | IA pode gerar respostas incorretas, inapropriadas ou fora de contexto |
| **Indicadores** | Takeover rate > 40%, Feedback negativo > 30%, Reclamações |
| **Mitigação** | Prompts bem estruturados, níveis de autonomia, supervisão humana obrigatória |
| **Responsável** | AI Team |

### RISCO-P03: Complexidade de Uso

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Médio |
| **Classificação** | 🟡 Médio |
| **Descrição** | Sistema pode ser difícil de usar para usuários não técnicos |
| **Indicadores** | Tempo de onboarding > 1h, Tickets de suporte alto, NPS < 30 |
| **Mitigação** | Interface intuitiva, onboarding passo-a-passo, tutoriais em vídeo |
| **Responsável** | UX Team |

---

## 10.3 Riscos Técnicos

### RISCO-T01: Indisponibilidade da Evolution API

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Alto |
| **Classificação** | 🟠 Alto |
| **Descrição** | API de WhatsApp pode ficar indisponível ou instável |
| **Indicadores** | Uptime < 99%, Erros de conexão > 5%, Desconexões frequentes |
| **Mitigação** | UAZAPI como backup, reconexão automática, cache de mensagens |
| **Responsável** | Backend Team |

### RISCO-T02: Custos Elevados de OpenAI

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Alta |
| **Impacto** | Médio |
| **Classificação** | 🟠 Alto |
| **Descrição** | Consumo de tokens pode exceder orçamento ou margem de lucro |
| **Indicadores** | Custo/mensagem > R$ 0.10, Tokens/empresa > 50K/dia |
| **Mitigação** | Limites por empresa, caching de respostas, modelos mais baratos, prompts otimizados |
| **Responsável** | Backend Team |

### RISCO-T03: Performance Inadequada

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Médio |
| **Classificação** | 🟡 Médio |
| **Descrição** | Sistema pode não escalar conforme esperado |
| **Indicadores** | Latência p95 > 500ms, CPU > 80%, Memory > 90% |
| **Mitigação** | Testes de carga, cache agressivo, otimização de queries, horizontal scaling |
| **Responsável** | DevOps Team |

### RISCO-T04: Vulnerabilidade de Segurança

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Baixa |
| **Impacto** | Alto |
| **Classificação** | 🟡 Médio |
| **Descrição** | Exposição de dados sensíveis ou acesso não autorizado |
| **Indicadores** | Alertas de segurança, Tentativas de invasão, Vazamento de dados |
| **Mitigação** | Auditorias regulares, pentests, OWASP Top 10, rate limiting |
| **Responsável** | Security Team |

### RISCO-T05: Perda de Dados

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Baixa |
| **Impacto** | Alto |
| **Classificação** | 🟡 Médio |
| **Descrição** | Falha no banco de dados ou storage com perda de informações |
| **Indicadores** | Erros de banco, Corrupção de dados, Backup falhando |
| **Mitigação** | Backups diários, replicação, testes de restore, point-in-time recovery |
| **Responsável** | DevOps Team |

---

## 10.4 Riscos de Negócio

### RISCO-B01: Mudanças nas Políticas do WhatsApp/Meta

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Alto |
| **Classificação** | 🟠 Alto |
| **Descrição** | Meta pode restringir APIs não oficiais ou mudar políticas de uso |
| **Indicadores** | Comunicados oficiais, Bloqueios de conta, Mudanças em termos |
| **Mitigação** | Monitorar políticas, preparar migração para Business API oficial, diversificar canais |
| **Responsável** | Product Team |

### RISCO-B02: Concorrência Intensa

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Alta |
| **Impacto** | Médio |
| **Classificação** | 🟠 Alto |
| **Descrição** | Competidores com mais recursos ou features similares |
| **Indicadores** | Perda de market share, Comparações desfavoráveis, Churn para concorrentes |
| **Mitigação** | Foco em diferenciação (IA + simplicidade), nicho específico, preço competitivo |
| **Responsável** | Product + Marketing |

### RISCO-B03: Mudança Regulatória (LGPD/ANPD)

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Baixa |
| **Impacto** | Alto |
| **Classificação** | 🟡 Médio |
| **Descrição** | Novas regulamentações que afetem tratamento de dados ou automação |
| **Indicadores** | Novas leis/regulamentos, Fiscalização ANPD, Multas no setor |
| **Mitigação** | Compliance por design, consultoria jurídica, arquitetura flexível |
| **Responsável** | Legal + Product |

### RISCO-B04: Dependência de Fornecedores

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Médio |
| **Classificação** | 🟡 Médio |
| **Descrição** | Dependência crítica de OpenAI, Evolution API, ou infraestrutura |
| **Indicadores** | Aumento de preços, Mudanças em termos, Descontinuação de serviço |
| **Mitigação** | Múltiplos provedores, abstrações de integração, modelos alternativos |
| **Responsável** | Tech Lead |

---

## 10.5 Riscos Operacionais

### RISCO-O01: Sobrecarga da Equipe

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Médio |
| **Classificação** | 🟡 Médio |
| **Descrição** | Equipe pequena com muitas demandas simultâneas |
| **Indicadores** | Burnout, Atrasos frequentes, Queda de qualidade |
| **Mitigação** | Priorização rigorosa (MoSCoW), automação, escopo controlado |
| **Responsável** | Tech Lead |

### RISCO-O02: Suporte Insuficiente

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Médio |
| **Classificação** | 🟡 Médio |
| **Descrição** | Volume de tickets de suporte excede capacidade de atendimento |
| **Indicadores** | Tempo de resposta > 24h, Tickets acumulados, NPS baixo |
| **Mitigação** | FAQ/docs, chatbot de suporte, automação de tickets comuns |
| **Responsável** | Support Team |

---

## 10.6 Plano de Contingência

### Para Riscos Críticos

| Risco | Trigger | Ação de Contingência |
|-------|---------|---------------------|
| **T01** - Evolution API | Indisponível > 1h | Ativar UAZAPI como fallback |
| **T02** - Custos OpenAI | Custo > 150% orçamento | Reduzir limites, usar GPT-3.5 |
| **T04** - Breach | Acesso não autorizado detectado | Protocolo de incidente (8.6) |
| **T05** - Perda dados | Corrupção detectada | Restore de backup, notificar usuários |
| **B01** - Políticas Meta | Bloqueio de contas | Migrar para Business API |
| **P01** - Baixa adoção | Conversão < 10% | Pivot no produto/nicho |

### Processo de Escalação

```
Nível 1: Desenvolvedor de Plantão
    ↓ (se não resolvido em 1h para P1, 4h para P2)
Nível 2: Tech Lead
    ↓ (se não resolvido em 4h para P1, 24h para P2)
Nível 3: Stakeholders / CEO
```

### Classificação de Incidentes

| Prioridade | Descrição | SLA Resposta | SLA Resolução |
|------------|-----------|--------------|---------------|
| **P1 - Crítico** | Sistema indisponível para todos | 15 min | 4h |
| **P2 - Alto** | Funcionalidade crítica afetada | 1h | 24h |
| **P3 - Médio** | Funcionalidade não crítica afetada | 4h | 72h |
| **P4 - Baixo** | Melhoria ou bug menor | 24h | 1 semana |

---

## 10.7 Monitoramento de Riscos

### Review Semanal

- [ ] Verificar status de cada risco ativo
- [ ] Atualizar probabilidade/impacto se mudou
- [ ] Identificar novos riscos
- [ ] Validar efetividade das mitigações
- [ ] Atualizar planos de contingência

### Review Mensal

- [ ] Análise de tendências
- [ ] Revisão de riscos de negócio
- [ ] Atualização da matriz de riscos
- [ ] Comunicação para stakeholders

### Indicadores de Alerta

| Risco | Indicador | Threshold | Ação |
|-------|-----------|-----------|------|
| T01 | Uptime Evolution API | < 99% | Preparar failover |
| T02 | Custo tokens/mês | > R$ 2.000 | Otimizar prompts |
| T03 | Latência p95 | > 300ms | Investigar performance |
| P01 | Churn mensal | > 12% | Pesquisa com churned users |
| P02 | Takeover rate | > 35% | Revisar prompts e autonomia |
| B02 | NPS | < 25 | Pesquisa qualitativa |

---

## 10.8 Histórico de Riscos

| Data | Risco | Evento | Ação Tomada | Resultado |
|------|-------|--------|-------------|-----------|
| - | - | - | - | - |

*Tabela atualizada conforme riscos se materializarem*

---

[← Voltar para Índice PRD](README.md)
