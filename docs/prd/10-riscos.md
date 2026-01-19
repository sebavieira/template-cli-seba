# 10. Riscos e Mitigações

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

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

### RISCO-01: {{TITULO_RISCO}}

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Alta / Média / Baixa |
| **Impacto** | Alto / Médio / Baixo |
| **Classificação** | 🔴 🟠 🟡 🟢 |
| **Descrição** | {{DESCRICAO}} |
| **Indicadores** | {{SINAIS_DE_ALERTA}} |
| **Mitigação** | {{ESTRATEGIA}} |
| **Responsável** | {{RESPONSAVEL}} |

### RISCO-02: {{TITULO_RISCO}}

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | {{PROB}} |
| **Impacto** | {{IMPACTO}} |
| **Classificação** | {{CLASSIFICACAO}} |
| **Descrição** | {{DESCRICAO}} |
| **Mitigação** | {{ESTRATEGIA}} |

---

## 10.3 Riscos Técnicos

### RISCO-T01: Falha em Integração Externa

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Alto |
| **Classificação** | 🟠 Alto |
| **Descrição** | APIs de terceiros podem ficar indisponíveis ou mudar |
| **Mitigação** | Circuit breaker, fallbacks, monitoramento |

### RISCO-T02: Performance Inadequada

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Média |
| **Impacto** | Médio |
| **Classificação** | 🟡 Médio |
| **Descrição** | Sistema pode não escalar conforme esperado |
| **Mitigação** | Testes de carga, cache, otimização de queries |

### RISCO-T03: Vulnerabilidade de Segurança

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Baixa |
| **Impacto** | Alto |
| **Classificação** | 🟡 Médio |
| **Descrição** | Exposição de dados ou acesso não autorizado |
| **Mitigação** | Auditorias, pentests, boas práticas OWASP |

---

## 10.4 Riscos de Negócio

### RISCO-B01: Baixa Adoção

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | {{PROB}} |
| **Impacto** | Alto |
| **Classificação** | {{CLASS}} |
| **Descrição** | Usuários não adotarem o sistema |
| **Mitigação** | Validação com usuários, MVP focado, iteração rápida |

### RISCO-B02: Mudança Regulatória

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | Baixa |
| **Impacto** | Alto |
| **Classificação** | 🟡 Médio |
| **Descrição** | Mudanças em leis que afetam o produto |
| **Mitigação** | Monitoramento regulatório, arquitetura flexível |

---

## 10.5 Riscos Operacionais

### RISCO-O01: Indisponibilidade de Recursos

| Atributo | Valor |
|----------|-------|
| **Probabilidade** | {{PROB}} |
| **Impacto** | {{IMPACTO}} |
| **Descrição** | Falta de pessoas ou orçamento para desenvolver |
| **Mitigação** | Priorização rigorosa, escopo controlado |

---

## 10.6 Plano de Contingência

### Para Riscos Críticos

| Risco | Trigger | Ação de Contingência |
|-------|---------|---------------------|
| RISCO-T01 | API indisponível > 1h | Ativar modo offline/cache |
| RISCO-T03 | Breach detectado | Protocolo de incidente |
| RISCO-B01 | Adoção < 20% meta | Pivot no produto |

### Processo de Escalação

```
Nível 1: Time de Desenvolvimento
    ↓ (se não resolvido em 1h)
Nível 2: Tech Lead
    ↓ (se não resolvido em 4h)
Nível 3: Stakeholders
```

---

## 10.7 Monitoramento de Riscos

### Review Semanal

- [ ] Verificar status de cada risco
- [ ] Atualizar probabilidade/impacto se mudou
- [ ] Identificar novos riscos
- [ ] Validar efetividade das mitigações

### Indicadores de Alerta

| Risco | Indicador | Threshold |
|-------|-----------|-----------|
| T01 | Uptime de integrações | < 99% |
| T02 | Latência p95 | > 500ms |
| B01 | Conversão | < 10% |

---

[← Voltar para Índice PRD](README.md)
