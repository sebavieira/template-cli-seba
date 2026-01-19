# 📊 Status do Projeto - {{NOME_PROJETO}}

**Última Atualização:** {{DATA_HORA}}
**Atualizado por:** {{AGENTE}}

---

## 📈 Progresso Geral

```
██████░░░░░░░░░░░░░░ 30%
```

| Métrica | Valor |
|---------|-------|
| **Progresso Total** | 30% |
| **Fase Atual** | Fase 2 - {{Nome da Fase}} |
| **Tarefas Completas** | 12/40 |
| **Última Tarefa** | FASE-2-TASK-003 |

---

## 🎯 Tarefa Atual

### FASE-2-TASK-004: {{Nome da Tarefa}}

**Epic:** [Epic 02](prd/04-user-stories/epic-02-{{nome}}.md)
**User Story:** US-007
**API:** [{{dominio}}](spec/04-contratos-api/{{dominio}}.md)

**Descrição:**
{{Descrição detalhada da tarefa}}

**Critérios de Conclusão:**
- [ ] Código implementado
- [ ] Testes unitários
- [ ] Testes de integração
- [ ] Documentação atualizada

---

## 📋 Fases do Projeto

### Fase 1: Fundação ✅
*Infraestrutura base e configuração*

| Task | Descrição | Status |
|------|-----------|--------|
| FASE-1-TASK-001 | Setup do projeto | ✅ Completo |
| FASE-1-TASK-002 | Configuração do banco | ✅ Completo |
| FASE-1-TASK-003 | Setup de testes | ✅ Completo |
| FASE-1-TASK-004 | CI/CD básico | ✅ Completo |

**Progresso da Fase:** 100% (4/4)

---

### Fase 2: {{Nome da Fase}} 🔄
*{{Descrição da fase}}*

| Task | Descrição | Status | Dependência |
|------|-----------|--------|-------------|
| FASE-2-TASK-001 | {{Tarefa 1}} | ✅ Completo | - |
| FASE-2-TASK-002 | {{Tarefa 2}} | ✅ Completo | TASK-001 |
| FASE-2-TASK-003 | {{Tarefa 3}} | ✅ Completo | TASK-002 |
| FASE-2-TASK-004 | {{Tarefa 4}} | 🔄 Em progresso | TASK-003 |
| FASE-2-TASK-005 | {{Tarefa 5}} | ⏳ Pendente | TASK-004 |
| FASE-2-TASK-006 | {{Tarefa 6}} | ⏳ Pendente | TASK-005 |

**Progresso da Fase:** 50% (3/6)

---

### Fase 3: {{Nome da Fase}} ⏳
*{{Descrição da fase}}*

| Task | Descrição | Status | Dependência |
|------|-----------|--------|-------------|
| FASE-3-TASK-001 | {{Tarefa 1}} | ⏳ Pendente | FASE-2 |
| FASE-3-TASK-002 | {{Tarefa 2}} | ⏳ Pendente | TASK-001 |
| ... | ... | ... | ... |

**Progresso da Fase:** 0% (0/X)

---

## 🚧 Bloqueadores

| ID | Descrição | Impacto | Status |
|----|-----------|---------|--------|
| BLOCK-001 | {{Descrição do bloqueador}} | Fase 3 | 🔴 Ativo |

**Ações:**
- {{Ação para resolver}}

---

## ✅ Tarefas Recentes (Últimas 5)

| Data | Task | Descrição |
|------|------|-----------|
| {{DATA}} | FASE-2-TASK-003 | {{Descrição}} |
| {{DATA}} | FASE-2-TASK-002 | {{Descrição}} |
| {{DATA}} | FASE-2-TASK-001 | {{Descrição}} |
| {{DATA}} | FASE-1-TASK-004 | {{Descrição}} |
| {{DATA}} | FASE-1-TASK-003 | {{Descrição}} |

---

## 📅 Próximas Tarefas

| Prioridade | Task | Descrição | Estimativa |
|------------|------|-----------|------------|
| 1 | FASE-2-TASK-004 | {{Descrição}} | {{Tempo}} |
| 2 | FASE-2-TASK-005 | {{Descrição}} | {{Tempo}} |
| 3 | FASE-2-TASK-006 | {{Descrição}} | {{Tempo}} |

---

## 📊 Métricas de Qualidade

| Métrica | Valor | Meta |
|---------|-------|------|
| Cobertura de Testes | {{X}}% | ≥80% |
| Testes Passando | {{Y}}/{{Z}} | 100% |
| Lint Errors | {{N}} | 0 |
| Build Status | ✅ | ✅ |

---

## 📝 Log de Atividades

### {{DATA}}

**Sessão:** {{ID_SESSAO}}

```
10:30 - Início da sessão
10:32 - Lido STATUS.md, identificada FASE-2-TASK-003
10:35 - Lido epic-02 e API de {{dominio}}
10:45 - Implementado {{funcionalidade}}
11:00 - Escritos 5 testes unitários
11:10 - Executados testes: 5/5 passed
11:15 - Atualizado STATUS.md
11:16 - Tarefa FASE-2-TASK-003 marcada como completa
```

---

## 🔗 Links Úteis

- [PRD Completo](prd/README.md)
- [SPEC Técnica](spec/README.md)
- [Epic Atual](prd/04-user-stories/epic-02-{{nome}}.md)
- [API Atual](spec/04-contratos-api/{{dominio}}.md)

---

## ⚡ Comandos Rápidos

```bash
# Rodar testes
npm test

# Build
npm run build

# Lint
npm run lint
```

---

**Legenda de Status:**
- ✅ Completo
- 🔄 Em progresso
- ⏳ Pendente
- 🚧 Bloqueado
- ❌ Cancelado
