# Refatoração: Followup Rule Wizard

**Arquivo:** `src/components/followups/FollowupRuleWizard.tsx`
**Linhas:** 1.078
**Prioridade:** P2 - MÉDIO
**Estimativa:** 1 dia

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 1.078 | 300 | 🔴 CRÍTICO |
| Responsabilidades | 5+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **Step Navigation** - Controle dos passos do wizard
2. **Rule Configuration** - Nome, descrição, trigger
3. **Condition Builder** - Construtor de condições
4. **Action Builder** - Construtor de ações
5. **Preview & Submit** - Revisão e envio

---

## Estrutura Proposta

```
src/components/followups/FollowupWizard/
├── index.tsx                     # Re-export
├── FollowupWizard.tsx            # Container principal (~120 linhas)
├── WizardStepper.tsx             # Navegação dos passos (~80 linhas)
├── steps/
│   ├── BasicInfoStep.tsx         # Nome, descrição (~100 linhas)
│   ├── TriggerStep.tsx           # Configuração do trigger (~120 linhas)
│   ├── ConditionsStep.tsx        # Construtor de condições (~150 linhas)
│   ├── ActionsStep.tsx           # Construtor de ações (~150 linhas)
│   └── ReviewStep.tsx            # Revisão e confirmação (~100 linhas)
├── components/
│   ├── ConditionRow.tsx          # Linha de condição (~100 linhas)
│   ├── ActionRow.tsx             # Linha de ação (~100 linhas)
│   ├── TriggerSelector.tsx       # Seletor de trigger (~80 linhas)
│   └── ScheduleConfig.tsx        # Config de agendamento (~80 linhas)
└── hooks/
    ├── useWizard.ts              # Estado do wizard (~60 linhas)
    ├── useRuleBuilder.ts         # Construção da regra (~80 linhas)
    └── useRuleMutations.ts       # Mutations (~60 linhas)
```

---

## Código Refatorado

### Container Principal
```typescript
// src/components/followups/FollowupWizard/FollowupWizard.tsx

import { useState } from 'react';
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';
import { WizardStepper } from './WizardStepper';
import { BasicInfoStep } from './steps/BasicInfoStep';
import { TriggerStep } from './steps/TriggerStep';
import { ConditionsStep } from './steps/ConditionsStep';
import { ActionsStep } from './steps/ActionsStep';
import { ReviewStep } from './steps/ReviewStep';
import { useWizard } from './hooks/useWizard';
import { useRuleBuilder } from './hooks/useRuleBuilder';
import { useRuleMutations } from './hooks/useRuleMutations';

interface FollowupWizardProps {
  open: boolean;
  onClose: () => void;
  editRule?: FollowupRule;
}

const STEPS = ['Informações', 'Gatilho', 'Condições', 'Ações', 'Revisão'];

export function FollowupWizard({ open, onClose, editRule }: FollowupWizardProps) {
  const { currentStep, next, prev, goTo, canProceed } = useWizard(STEPS.length);
  const { rule, updateRule, resetRule } = useRuleBuilder(editRule);
  const { createRule, updateRule: saveRule } = useRuleMutations();

  const handleSubmit = async () => {
    if (editRule) {
      await saveRule.mutateAsync({ id: editRule.id, ...rule });
    } else {
      await createRule.mutateAsync(rule);
    }
    resetRule();
    onClose();
  };

  const handleClose = () => {
    resetRule();
    onClose();
  };

  const renderStep = () => {
    switch (currentStep) {
      case 0:
        return <BasicInfoStep rule={rule} onChange={updateRule} />;
      case 1:
        return <TriggerStep rule={rule} onChange={updateRule} />;
      case 2:
        return <ConditionsStep rule={rule} onChange={updateRule} />;
      case 3:
        return <ActionsStep rule={rule} onChange={updateRule} />;
      case 4:
        return <ReviewStep rule={rule} onSubmit={handleSubmit} />;
      default:
        return null;
    }
  };

  return (
    <Dialog open={open} onOpenChange={(open) => !open && handleClose()}>
      <DialogContent className="max-w-2xl">
        <DialogHeader>
          <DialogTitle>
            {editRule ? 'Editar Regra de Followup' : 'Nova Regra de Followup'}
          </DialogTitle>
        </DialogHeader>

        <WizardStepper
          steps={STEPS}
          currentStep={currentStep}
          onStepClick={goTo}
        />

        <div className="py-6">{renderStep()}</div>

        <WizardNavigation
          currentStep={currentStep}
          totalSteps={STEPS.length}
          onPrev={prev}
          onNext={next}
          canProceed={canProceed(rule)}
        />
      </DialogContent>
    </Dialog>
  );
}
```

### Hook useWizard
```typescript
// src/components/followups/FollowupWizard/hooks/useWizard.ts

import { useState, useCallback } from 'react';

export function useWizard(totalSteps: number) {
  const [currentStep, setCurrentStep] = useState(0);

  const next = useCallback(() => {
    setCurrentStep((prev) => Math.min(prev + 1, totalSteps - 1));
  }, [totalSteps]);

  const prev = useCallback(() => {
    setCurrentStep((prev) => Math.max(prev - 1, 0));
  }, []);

  const goTo = useCallback((step: number) => {
    if (step >= 0 && step < totalSteps) {
      setCurrentStep(step);
    }
  }, [totalSteps]);

  const canProceed = useCallback((rule: any) => {
    switch (currentStep) {
      case 0: // Basic Info
        return rule.name?.trim().length > 0;
      case 1: // Trigger
        return rule.trigger?.type != null;
      case 2: // Conditions
        return true; // Conditions are optional
      case 3: // Actions
        return rule.actions?.length > 0;
      case 4: // Review
        return true;
      default:
        return false;
    }
  }, [currentStep]);

  return {
    currentStep,
    next,
    prev,
    goTo,
    canProceed,
    isFirstStep: currentStep === 0,
    isLastStep: currentStep === totalSteps - 1,
  };
}
```

### Hook useRuleBuilder
```typescript
// src/components/followups/FollowupWizard/hooks/useRuleBuilder.ts

import { useState, useCallback } from 'react';

interface FollowupRule {
  name: string;
  description: string;
  trigger: {
    type: 'time_based' | 'event_based' | 'condition_based';
    config: any;
  } | null;
  conditions: Condition[];
  actions: Action[];
  isActive: boolean;
}

const DEFAULT_RULE: FollowupRule = {
  name: '',
  description: '',
  trigger: null,
  conditions: [],
  actions: [],
  isActive: true,
};

export function useRuleBuilder(editRule?: FollowupRule) {
  const [rule, setRule] = useState<FollowupRule>(editRule || DEFAULT_RULE);

  const updateRule = useCallback((updates: Partial<FollowupRule>) => {
    setRule((prev) => ({ ...prev, ...updates }));
  }, []);

  const addCondition = useCallback((condition: Condition) => {
    setRule((prev) => ({
      ...prev,
      conditions: [...prev.conditions, condition],
    }));
  }, []);

  const removeCondition = useCallback((index: number) => {
    setRule((prev) => ({
      ...prev,
      conditions: prev.conditions.filter((_, i) => i !== index),
    }));
  }, []);

  const addAction = useCallback((action: Action) => {
    setRule((prev) => ({
      ...prev,
      actions: [...prev.actions, action],
    }));
  }, []);

  const removeAction = useCallback((index: number) => {
    setRule((prev) => ({
      ...prev,
      actions: prev.actions.filter((_, i) => i !== index),
    }));
  }, []);

  const resetRule = useCallback(() => {
    setRule(DEFAULT_RULE);
  }, []);

  return {
    rule,
    updateRule,
    addCondition,
    removeCondition,
    addAction,
    removeAction,
    resetRule,
  };
}
```

### WizardStepper
```typescript
// src/components/followups/FollowupWizard/WizardStepper.tsx

import { cn } from '@/lib/utils';
import { Check } from 'lucide-react';

interface WizardStepperProps {
  steps: string[];
  currentStep: number;
  onStepClick: (step: number) => void;
}

export function WizardStepper({ steps, currentStep, onStepClick }: WizardStepperProps) {
  return (
    <div className="flex items-center justify-between">
      {steps.map((step, index) => (
        <div key={step} className="flex items-center">
          <button
            onClick={() => onStepClick(index)}
            disabled={index > currentStep}
            className={cn(
              'flex items-center gap-2 px-3 py-2 rounded-lg transition-colors',
              index === currentStep && 'bg-primary text-primary-foreground',
              index < currentStep && 'bg-green-100 text-green-800',
              index > currentStep && 'bg-muted text-muted-foreground'
            )}
          >
            {index < currentStep ? (
              <Check className="w-4 h-4" />
            ) : (
              <span className="w-6 h-6 rounded-full bg-current/10 flex items-center justify-center text-xs">
                {index + 1}
              </span>
            )}
            <span className="hidden sm:inline">{step}</span>
          </button>

          {index < steps.length - 1 && (
            <div className={cn(
              'w-8 h-0.5 mx-2',
              index < currentStep ? 'bg-green-500' : 'bg-muted'
            )} />
          )}
        </div>
      ))}
    </div>
  );
}
```

### BasicInfoStep
```typescript
// src/components/followups/FollowupWizard/steps/BasicInfoStep.tsx

import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Label } from '@/components/ui/label';
import { Switch } from '@/components/ui/switch';

interface BasicInfoStepProps {
  rule: any;
  onChange: (updates: any) => void;
}

export function BasicInfoStep({ rule, onChange }: BasicInfoStepProps) {
  return (
    <div className="space-y-4">
      <div>
        <Label htmlFor="name">Nome da Regra *</Label>
        <Input
          id="name"
          value={rule.name}
          onChange={(e) => onChange({ name: e.target.value })}
          placeholder="Ex: Followup de leads inativos"
        />
      </div>

      <div>
        <Label htmlFor="description">Descrição</Label>
        <Textarea
          id="description"
          value={rule.description}
          onChange={(e) => onChange({ description: e.target.value })}
          placeholder="Descreva o que esta regra faz..."
          rows={3}
        />
      </div>

      <div className="flex items-center justify-between">
        <div>
          <Label>Regra Ativa</Label>
          <p className="text-sm text-muted-foreground">
            A regra será executada automaticamente quando ativa
          </p>
        </div>
        <Switch
          checked={rule.isActive}
          onCheckedChange={(checked) => onChange({ isActive: checked })}
        />
      </div>
    </div>
  );
}
```

### ConditionsStep
```typescript
// src/components/followups/FollowupWizard/steps/ConditionsStep.tsx

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { ConditionRow } from '../components/ConditionRow';
import { Plus } from 'lucide-react';

interface ConditionsStepProps {
  rule: any;
  onChange: (updates: any) => void;
}

const CONDITION_TYPES = [
  { value: 'tag', label: 'Tag do contato' },
  { value: 'sentiment', label: 'Sentimento' },
  { value: 'last_message', label: 'Última mensagem' },
  { value: 'deal_stage', label: 'Estágio do deal' },
  { value: 'time_since_contact', label: 'Tempo desde contato' },
];

export function ConditionsStep({ rule, onChange }: ConditionsStepProps) {
  const addCondition = () => {
    const newCondition = {
      type: 'tag',
      operator: 'has',
      value: '',
    };
    onChange({ conditions: [...rule.conditions, newCondition] });
  };

  const updateCondition = (index: number, updates: any) => {
    const newConditions = [...rule.conditions];
    newConditions[index] = { ...newConditions[index], ...updates };
    onChange({ conditions: newConditions });
  };

  const removeCondition = (index: number) => {
    onChange({ conditions: rule.conditions.filter((_: any, i: number) => i !== index) });
  };

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div>
          <h3 className="font-medium">Condições</h3>
          <p className="text-sm text-muted-foreground">
            Define quando a regra será executada (opcional)
          </p>
        </div>
        <Button onClick={addCondition} variant="outline" size="sm">
          <Plus className="w-4 h-4 mr-2" />
          Adicionar
        </Button>
      </div>

      {rule.conditions.length === 0 ? (
        <div className="text-center py-8 text-muted-foreground">
          Nenhuma condição adicionada. A regra será executada para todos os contatos.
        </div>
      ) : (
        <div className="space-y-2">
          {rule.conditions.map((condition: any, index: number) => (
            <ConditionRow
              key={index}
              condition={condition}
              conditionTypes={CONDITION_TYPES}
              onChange={(updates) => updateCondition(index, updates)}
              onRemove={() => removeCondition(index)}
            />
          ))}
        </div>
      )}
    </div>
  );
}
```

### ActionsStep
```typescript
// src/components/followups/FollowupWizard/steps/ActionsStep.tsx

import { Button } from '@/components/ui/button';
import { ActionRow } from '../components/ActionRow';
import { Plus } from 'lucide-react';

interface ActionsStepProps {
  rule: any;
  onChange: (updates: any) => void;
}

const ACTION_TYPES = [
  { value: 'send_message', label: 'Enviar mensagem' },
  { value: 'assign_tag', label: 'Adicionar tag' },
  { value: 'remove_tag', label: 'Remover tag' },
  { value: 'move_deal_stage', label: 'Mover estágio do deal' },
  { value: 'send_notification', label: 'Enviar notificação' },
];

export function ActionsStep({ rule, onChange }: ActionsStepProps) {
  const addAction = () => {
    const newAction = {
      type: 'send_message',
      config: {},
    };
    onChange({ actions: [...rule.actions, newAction] });
  };

  const updateAction = (index: number, updates: any) => {
    const newActions = [...rule.actions];
    newActions[index] = { ...newActions[index], ...updates };
    onChange({ actions: newActions });
  };

  const removeAction = (index: number) => {
    onChange({ actions: rule.actions.filter((_: any, i: number) => i !== index) });
  };

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div>
          <h3 className="font-medium">Ações *</h3>
          <p className="text-sm text-muted-foreground">
            O que acontece quando a regra é executada
          </p>
        </div>
        <Button onClick={addAction} variant="outline" size="sm">
          <Plus className="w-4 h-4 mr-2" />
          Adicionar
        </Button>
      </div>

      {rule.actions.length === 0 ? (
        <div className="text-center py-8 text-muted-foreground border-2 border-dashed rounded-lg">
          Adicione pelo menos uma ação para a regra
        </div>
      ) : (
        <div className="space-y-2">
          {rule.actions.map((action: any, index: number) => (
            <ActionRow
              key={index}
              action={action}
              actionTypes={ACTION_TYPES}
              onChange={(updates) => updateAction(index, updates)}
              onRemove={() => removeAction(index)}
            />
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Checklist

### Estrutura
- [ ] Criar pasta `FollowupWizard/`
- [ ] Criar pasta `steps/`
- [ ] Criar pasta `components/`
- [ ] Criar pasta `hooks/`

### Hooks
- [ ] `useWizard.ts`
- [ ] `useRuleBuilder.ts`
- [ ] `useRuleMutations.ts`

### Steps
- [ ] `BasicInfoStep.tsx`
- [ ] `TriggerStep.tsx`
- [ ] `ConditionsStep.tsx`
- [ ] `ActionsStep.tsx`
- [ ] `ReviewStep.tsx`

### Componentes
- [ ] `FollowupWizard.tsx`
- [ ] `WizardStepper.tsx`
- [ ] `ConditionRow.tsx`
- [ ] `ActionRow.tsx`
- [ ] `TriggerSelector.tsx`
- [ ] `ScheduleConfig.tsx`

### Finalização
- [ ] Atualizar imports
- [ ] Remover arquivo antigo
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
