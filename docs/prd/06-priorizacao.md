# 6. Priorização de Features (MoSCoW)

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

[← Voltar para Índice PRD](README.md)

---

## Visão Geral

Este documento classifica todas as funcionalidades usando o framework **MoSCoW**:

- **M**ust Have - Obrigatório para MVP
- **S**hould Have - Importante, mas não bloqueante
- **C**ould Have - Desejável se houver tempo
- **W**on't Have - Fora do escopo atual

---

## 🔴 Must Have (MVP)

Funcionalidades **obrigatórias** para o lançamento. Sem elas, o produto não funciona.

| ID | Funcionalidade | Epic | Justificativa |
|----|----------------|------|---------------|
| US-001 | {{MUST_1}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-002 | {{MUST_2}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-003 | {{MUST_3}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-004 | {{MUST_4}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-005 | {{MUST_5}} | {{EPIC}} | {{JUSTIFICATIVA}} |

**Total Must Have:** {{TOTAL_MUST}}

---

## 🟡 Should Have

Funcionalidades **importantes** que agregam valor significativo, mas o sistema funciona sem elas.

| ID | Funcionalidade | Epic | Justificativa |
|----|----------------|------|---------------|
| US-006 | {{SHOULD_1}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-007 | {{SHOULD_2}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-008 | {{SHOULD_3}} | {{EPIC}} | {{JUSTIFICATIVA}} |

**Total Should Have:** {{TOTAL_SHOULD}}

---

## 🟢 Could Have

Funcionalidades **desejáveis** que melhoram a experiência, implementadas se sobrar tempo.

| ID | Funcionalidade | Epic | Justificativa |
|----|----------------|------|---------------|
| US-009 | {{COULD_1}} | {{EPIC}} | {{JUSTIFICATIVA}} |
| US-010 | {{COULD_2}} | {{EPIC}} | {{JUSTIFICATIVA}} |

**Total Could Have:** {{TOTAL_COULD}}

---

## ⚪ Won't Have (This Release)

Funcionalidades **fora do escopo** atual, mas podem ser consideradas no futuro.

| Funcionalidade | Motivo | Versão Futura |
|----------------|--------|---------------|
| {{WONT_1}} | {{MOTIVO_1}} | V1.1 |
| {{WONT_2}} | {{MOTIVO_2}} | V2.0 |
| {{WONT_3}} | {{MOTIVO_3}} | V2.0 |

**Total Won't Have:** {{TOTAL_WONT}}

---

## Resumo da Priorização

| Categoria | Quantidade | Percentual |
|-----------|------------|------------|
| 🔴 Must Have | {{TOTAL_MUST}} | {{PERCENT_MUST}}% |
| 🟡 Should Have | {{TOTAL_SHOULD}} | {{PERCENT_SHOULD}}% |
| 🟢 Could Have | {{TOTAL_COULD}} | {{PERCENT_COULD}}% |
| ⚪ Won't Have | {{TOTAL_WONT}} | - |
| **Total MVP** | **{{TOTAL_MVP}}** | **100%** |

---

## Critérios de Priorização

As funcionalidades foram priorizadas considerando:

1. **Valor para o Usuário** - Quanto resolve o problema da persona principal
2. **Viabilidade Técnica** - Complexidade de implementação
3. **Dependências** - Se bloqueia outras funcionalidades
4. **Risco** - Impacto se não for implementada
5. **Esforço** - Tempo e recursos necessários

### Matriz de Priorização

```
Alto Valor + Baixo Esforço  = Must Have (Fazer primeiro)
Alto Valor + Alto Esforço   = Should Have (Planejar bem)
Baixo Valor + Baixo Esforço = Could Have (Se der tempo)
Baixo Valor + Alto Esforço  = Won't Have (Não fazer agora)
```

---

## Sequência de Implementação

### Fase 1: Core (Semana 1-2)
1. US-001: {{DESCRICAO}}
2. US-002: {{DESCRICAO}}

### Fase 2: Essencial (Semana 3-4)
3. US-003: {{DESCRICAO}}
4. US-004: {{DESCRICAO}}

### Fase 3: Importante (Semana 5-6)
5. US-005: {{DESCRICAO}}
6. US-006: {{DESCRICAO}}

### Fase 4: Melhorias (Semana 7+)
7. US-007: {{DESCRICAO}}
8. US-008: {{DESCRICAO}}

---

[← Voltar para Índice PRD](README.md)
