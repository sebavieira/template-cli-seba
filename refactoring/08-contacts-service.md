# Refatoração: Contacts Service

**Arquivo:** `server/src/modules/contacts/contacts.service.ts`
**Linhas:** 995
**Prioridade:** P1 - ALTO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 995 | 400 | 🔴 CRÍTICO |
| Responsabilidades | 4+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **CRUD básico** - Criar, atualizar, deletar contatos
2. **Tags** - Gerenciamento de tags
3. **Enriquecimento** - Dados adicionais, merge
4. **Segmentação** - Filtros avançados, busca

---

## Estrutura Proposta

```
server/src/modules/contacts/
├── contacts.service.ts           # Facade (~150 linhas)
├── contacts.routes.ts            # Existente
├── contacts.schemas.ts           # Validação Zod
├── contacts.types.ts             # Tipos (~80 linhas)
├── services/
│   ├── index.ts
│   ├── contact-crud.service.ts   # CRUD básico (~200 linhas)
│   ├── contact-tags.service.ts   # Gerenciamento de tags (~150 linhas)
│   ├── contact-enrichment.service.ts # Enriquecimento (~150 linhas)
│   └── contact-search.service.ts # Busca e filtros (~200 linhas)
└── helpers/
    ├── phone-normalizer.ts       # Normalização de telefone (~50 linhas)
    └── contact-merger.ts         # Merge de contatos (~100 linhas)
```

---

## Mapeamento de Métodos

### contact-crud.service.ts
```typescript
export class ContactCrudService {
  async list(companyId: string, options: ListOptions)
  async getById(companyId: string, contactId: string)
  async getByPhone(companyId: string, phone: string)
  async create(companyId: string, data: CreateContactDTO)
  async update(companyId: string, contactId: string, data: UpdateContactDTO)
  async delete(companyId: string, contactId: string)
  async bulkCreate(companyId: string, contacts: CreateContactDTO[])
  async bulkUpdate(companyId: string, updates: BulkUpdateDTO[])
}
```

### contact-tags.service.ts
```typescript
export class ContactTagsService {
  async assignTags(companyId: string, contactId: string, tagIds: string[])
  async removeTags(companyId: string, contactId: string, tagIds: string[])
  async replaceTags(companyId: string, contactId: string, tagIds: string[])
  async getContactsByTag(companyId: string, tagId: string)
  async bulkAssignTags(companyId: string, contactIds: string[], tagIds: string[])
}
```

### contact-enrichment.service.ts
```typescript
export class ContactEnrichmentService {
  async enrichContact(companyId: string, contactId: string)
  async updateSentiment(companyId: string, contactId: string, sentiment: Sentiment)
  async addNote(companyId: string, contactId: string, note: string)
  async getNotes(companyId: string, contactId: string)
  async updateCustomFields(companyId: string, contactId: string, fields: Record<string, any>)
}
```

### contact-search.service.ts
```typescript
export class ContactSearchService {
  async search(companyId: string, query: string, options?: SearchOptions)
  async advancedSearch(companyId: string, filters: AdvancedFilters)
  async getBySegment(companyId: string, segment: SegmentDefinition)
  async exportContacts(companyId: string, filters: ExportFilters)
}
```

---

## Código: Helpers

### phone-normalizer.ts
```typescript
// server/src/modules/contacts/helpers/phone-normalizer.ts

export function normalizePhone(phone: string): string {
  // Remover caracteres não numéricos
  let normalized = phone.replace(/\D/g, '');

  // Adicionar código do país se não existir
  if (normalized.length === 11 && normalized.startsWith('9')) {
    normalized = '55' + normalized;
  } else if (normalized.length === 10) {
    normalized = '55' + normalized;
  }

  return normalized;
}

export function formatPhoneForDisplay(phone: string): string {
  const normalized = normalizePhone(phone);

  if (normalized.length === 13) {
    // +55 11 99999-9999
    return `+${normalized.slice(0, 2)} ${normalized.slice(2, 4)} ${normalized.slice(4, 9)}-${normalized.slice(9)}`;
  }

  return phone;
}

export function isValidBrazilianPhone(phone: string): boolean {
  const normalized = normalizePhone(phone);
  return /^55\d{10,11}$/.test(normalized);
}
```

### contact-merger.ts
```typescript
// server/src/modules/contacts/helpers/contact-merger.ts

import { Contact } from '../contacts.types';

export interface MergeResult {
  merged: Contact;
  conflicts: MergeConflict[];
}

export interface MergeConflict {
  field: string;
  sourceValue: any;
  targetValue: any;
  resolution: 'source' | 'target' | 'manual';
}

export function mergeContacts(
  source: Contact,
  target: Contact,
  strategy: 'prefer_source' | 'prefer_target' | 'prefer_newest' = 'prefer_newest'
): MergeResult {
  const conflicts: MergeConflict[] = [];
  const merged = { ...target };

  const fieldsToMerge = ['name', 'email', 'notes', 'customFields'] as const;

  for (const field of fieldsToMerge) {
    if (source[field] && target[field] && source[field] !== target[field]) {
      conflicts.push({
        field,
        sourceValue: source[field],
        targetValue: target[field],
        resolution: strategy === 'prefer_source' ? 'source' : 'target',
      });

      if (strategy === 'prefer_source') {
        merged[field] = source[field];
      } else if (strategy === 'prefer_newest') {
        merged[field] = new Date(source.updatedAt) > new Date(target.updatedAt)
          ? source[field]
          : target[field];
      }
    } else if (source[field] && !target[field]) {
      merged[field] = source[field];
    }
  }

  // Merge tags (união)
  const allTagIds = new Set([
    ...source.tags.map(t => t.id),
    ...target.tags.map(t => t.id),
  ]);
  merged.tagIds = Array.from(allTagIds);

  return { merged, conflicts };
}
```

---

## Checklist

### Estrutura
- [ ] Criar `contacts.types.ts`
- [ ] Criar `contacts.schemas.ts`
- [ ] Criar pasta `services/`
- [ ] Criar pasta `helpers/`

### Services
- [ ] `contact-crud.service.ts`
- [ ] `contact-tags.service.ts`
- [ ] `contact-enrichment.service.ts`
- [ ] `contact-search.service.ts`

### Helpers
- [ ] `phone-normalizer.ts`
- [ ] `contact-merger.ts`

### Finalização
- [ ] Refatorar `contacts.service.ts` para Facade
- [ ] Atualizar imports
- [ ] Testes

---

**Status:** ⏳ Pendente
**Última Atualização:** 2026-01-22
