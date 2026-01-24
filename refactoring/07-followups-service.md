# Refatoração: Followups Service

**Arquivo:** `server/src/modules/followups/followups.service.ts`
**Linhas:** 1.036
**Prioridade:** P1 - ALTO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 1.036 | 400 | 🔴 CRÍTICO |
| Responsabilidades | 4+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **CRUD de regras** - Criar, atualizar, deletar regras de followup
2. **Execução** - Processar e executar regras
3. **Agendamento** - Gerenciar delays e schedules
4. **Analytics** - Métricas e performance das regras

---

## Estrutura Proposta

```
server/src/modules/followups/
├── followups.service.ts          # Facade (~150 linhas)
├── followups.routes.ts           # Existente
├── followups.types.ts            # Tipos (~80 linhas)
├── services/
│   ├── index.ts
│   ├── followup-rules.service.ts # CRUD de regras (~250 linhas)
│   ├── followup-executor.service.ts # Execução (~300 linhas)
│   ├── followup-scheduler.service.ts # Agendamento (~200 linhas)
│   └── followup-analytics.service.ts # Métricas (~150 linhas)
└── helpers/
    ├── condition-evaluator.ts    # Avaliar condições (~100 linhas)
    └── action-executor.ts        # Executar ações (~100 linhas)
```

---

## Mapeamento de Métodos

### followup-rules.service.ts
```typescript
export class FollowupRulesService {
  // CRUD
  async list(companyId: string, filters?: RuleFilters)
  async getById(companyId: string, ruleId: string)
  async create(companyId: string, data: CreateRuleDTO)
  async update(companyId: string, ruleId: string, data: UpdateRuleDTO)
  async delete(companyId: string, ruleId: string)
  async duplicate(companyId: string, ruleId: string)

  // Status
  async toggleActive(companyId: string, ruleId: string, active: boolean)
  async pause(companyId: string, ruleId: string, until?: Date)
}
```

### followup-executor.service.ts
```typescript
export class FollowupExecutorService {
  // Execução principal
  async executeRule(ruleId: string, contactId: string, context: ExecutionContext)
  async executeAction(action: FollowupAction, contact: Contact)

  // Batch
  async executeBatch(ruleId: string, contactIds: string[])

  // Helpers
  private async evaluateConditions(conditions: Condition[], contact: Contact): Promise<boolean>
  private async logExecution(ruleId: string, contactId: string, result: ExecutionResult)
}
```

### followup-scheduler.service.ts
```typescript
export class FollowupSchedulerService {
  // Agendamento
  async scheduleExecution(ruleId: string, contactId: string, executeAt: Date)
  async cancelScheduled(scheduleId: string)
  async reschedule(scheduleId: string, newDate: Date)

  // Consultas
  async getPendingExecutions(companyId: string)
  async getScheduledForContact(contactId: string)

  // Processamento
  async processScheduledExecutions()
}
```

### followup-analytics.service.ts
```typescript
export class FollowupAnalyticsService {
  // Métricas por regra
  async getRulePerformance(ruleId: string, period: DateRange)
  async getRuleStats(ruleId: string)

  // Métricas gerais
  async getCompanyStats(companyId: string, period: DateRange)
  async getConversionRates(companyId: string)

  // Relatórios
  async generateReport(companyId: string, options: ReportOptions)
}
```

---

## Código: Before/After

### BEFORE
```typescript
// followups.service.ts - 1.036 linhas misturadas
export class FollowupsService {
  async createRule(...) { /* 80 linhas */ }
  async updateRule(...) { /* 60 linhas */ }
  async executeRule(...) { /* 150 linhas */ }
  async scheduleExecution(...) { /* 50 linhas */ }
  async getStats(...) { /* 100 linhas */ }
  // ... mais 15 métodos
}
```

### AFTER
```typescript
// followups.service.ts - Facade ~150 linhas
import { FollowupRulesService } from './services/followup-rules.service';
import { FollowupExecutorService } from './services/followup-executor.service';
import { FollowupSchedulerService } from './services/followup-scheduler.service';
import { FollowupAnalyticsService } from './services/followup-analytics.service';

export class FollowupsService {
  constructor(
    private rules: FollowupRulesService,
    private executor: FollowupExecutorService,
    private scheduler: FollowupSchedulerService,
    private analytics: FollowupAnalyticsService,
  ) {}

  // Delegates
  createRule = this.rules.create.bind(this.rules);
  updateRule = this.rules.update.bind(this.rules);
  executeRule = this.executor.executeRule.bind(this.executor);
  scheduleExecution = this.scheduler.scheduleExecution.bind(this.scheduler);
  getStats = this.analytics.getRuleStats.bind(this.analytics);
}
```

---

## Checklist

### Estrutura
- [ ] Criar `followups.types.ts`
- [ ] Criar pasta `services/`
- [ ] Criar pasta `helpers/`

### Services
- [ ] `followup-rules.service.ts`
- [ ] `followup-executor.service.ts`
- [ ] `followup-scheduler.service.ts`
- [ ] `followup-analytics.service.ts`

### Helpers
- [ ] `condition-evaluator.ts`
- [ ] `action-executor.ts`

### Finalização
- [x] Refatorar `followups.service.ts` para Facade
- [x] Atualizar imports
- [x] Build passou

---

**Status:** ✅ Completo
**Última Atualização:** 2026-01-23
