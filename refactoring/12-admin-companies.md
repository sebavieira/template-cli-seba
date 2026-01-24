# Refatoração: Admin Companies Page

**Arquivo:** `src/pages/AdminCompanies.tsx`
**Linhas:** 1.318
**Prioridade:** P1 - ALTO
**Estimativa:** 1-2 dias

---

## Estado Atual

### Métricas
| Métrica | Valor | Limite | Status |
|---------|-------|--------|--------|
| Linhas | 1.318 | 300 | 🔴 CRÍTICO |
| Responsabilidades | 5+ | 1 | 🟠 ALTO |

### Responsabilidades Misturadas

1. **Lista de empresas** - Tabela com filtros e paginação
2. **CRUD de empresa** - Criar, editar, deletar
3. **Gerenciamento de usuários** - Usuários por empresa
4. **Configurações** - Settings da empresa
5. **Billing** - Informações de cobrança

---

## Estrutura Proposta

```
src/pages/Admin/
├── index.tsx                     # Re-export
├── AdminCompanies/
│   ├── index.tsx                 # Container principal (~100 linhas)
│   ├── CompanyList.tsx           # Lista/tabela (~200 linhas)
│   ├── CompanyFilters.tsx        # Filtros (~100 linhas)
│   ├── hooks/
│   │   ├── useCompanies.ts       # Query de empresas (~100 linhas)
│   │   ├── useCompanyMutations.ts # Mutations (~100 linhas)
│   │   └── useCompanyFilters.ts  # Filtros (~80 linhas)
│   └── modals/
│       ├── CreateCompanyModal.tsx  # Criar empresa (~150 linhas)
│       ├── EditCompanyModal.tsx    # Editar empresa (~150 linhas)
│       └── CompanyUsersModal.tsx   # Gerenciar usuários (~200 linhas)
└── components/
    ├── CompanyCard.tsx           # Card de empresa (~80 linhas)
    └── CompanyStats.tsx          # Estatísticas (~100 linhas)
```

---

## Hooks

### useCompanies
```typescript
// src/pages/Admin/AdminCompanies/hooks/useCompanies.ts

import { useQuery } from '@tanstack/react-query';
import { adminApi } from '@/lib/api';

interface UseCompaniesOptions {
  search?: string;
  status?: 'active' | 'inactive' | 'trial';
  page?: number;
  limit?: number;
}

export function useCompanies(options: UseCompaniesOptions = {}) {
  const { search, status, page = 1, limit = 20 } = options;

  return useQuery({
    queryKey: ['admin', 'companies', { search, status, page, limit }],
    queryFn: () => adminApi.listCompanies({ search, status, page, limit }),
  });
}

export function useCompany(companyId: string) {
  return useQuery({
    queryKey: ['admin', 'companies', companyId],
    queryFn: () => adminApi.getCompany(companyId),
    enabled: !!companyId,
  });
}

export function useCompanyUsers(companyId: string) {
  return useQuery({
    queryKey: ['admin', 'companies', companyId, 'users'],
    queryFn: () => adminApi.getCompanyUsers(companyId),
    enabled: !!companyId,
  });
}
```

### useCompanyMutations
```typescript
// src/pages/Admin/AdminCompanies/hooks/useCompanyMutations.ts

import { useMutation, useQueryClient } from '@tanstack/react-query';
import { adminApi } from '@/lib/api';
import { useToast } from '@/hooks/use-toast';

export function useCompanyMutations() {
  const queryClient = useQueryClient();
  const { toast } = useToast();

  const createCompany = useMutation({
    mutationFn: adminApi.createCompany,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'companies'] });
      toast({ title: 'Empresa criada com sucesso' });
    },
    onError: (error: Error) => {
      toast({ title: 'Erro ao criar empresa', description: error.message, variant: 'destructive' });
    },
  });

  const updateCompany = useMutation({
    mutationFn: ({ id, data }: { id: string; data: any }) => adminApi.updateCompany(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'companies'] });
      toast({ title: 'Empresa atualizada com sucesso' });
    },
    onError: (error: Error) => {
      toast({ title: 'Erro ao atualizar empresa', description: error.message, variant: 'destructive' });
    },
  });

  const deleteCompany = useMutation({
    mutationFn: adminApi.deleteCompany,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'companies'] });
      toast({ title: 'Empresa removida com sucesso' });
    },
    onError: (error: Error) => {
      toast({ title: 'Erro ao remover empresa', description: error.message, variant: 'destructive' });
    },
  });

  const toggleCompanyStatus = useMutation({
    mutationFn: ({ id, active }: { id: string; active: boolean }) =>
      adminApi.updateCompany(id, { active }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['admin', 'companies'] });
    },
  });

  return {
    createCompany,
    updateCompany,
    deleteCompany,
    toggleCompanyStatus,
  };
}
```

---

## Componentes

### Container Principal
```typescript
// src/pages/Admin/AdminCompanies/index.tsx

import { useState } from 'react';
import { CompanyList } from './CompanyList';
import { CompanyFilters } from './CompanyFilters';
import { CreateCompanyModal } from './modals/CreateCompanyModal';
import { EditCompanyModal } from './modals/EditCompanyModal';
import { CompanyUsersModal } from './modals/CompanyUsersModal';
import { useCompanies } from './hooks/useCompanies';
import { useCompanyFilters } from './hooks/useCompanyFilters';
import { Button } from '@/components/ui/button';
import { Plus } from 'lucide-react';

export function AdminCompanies() {
  const [createOpen, setCreateOpen] = useState(false);
  const [editCompanyId, setEditCompanyId] = useState<string | null>(null);
  const [usersCompanyId, setUsersCompanyId] = useState<string | null>(null);

  const { filters, setFilter, clearFilters } = useCompanyFilters();
  const { data, isLoading, error } = useCompanies(filters);

  return (
    <div className="container py-6">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-2xl font-bold">Empresas</h1>
        <Button onClick={() => setCreateOpen(true)}>
          <Plus className="w-4 h-4 mr-2" />
          Nova Empresa
        </Button>
      </div>

      <CompanyFilters
        filters={filters}
        onFilterChange={setFilter}
        onClear={clearFilters}
      />

      <CompanyList
        companies={data?.data ?? []}
        isLoading={isLoading}
        onEdit={(id) => setEditCompanyId(id)}
        onManageUsers={(id) => setUsersCompanyId(id)}
      />

      <CreateCompanyModal
        open={createOpen}
        onClose={() => setCreateOpen(false)}
      />

      <EditCompanyModal
        companyId={editCompanyId}
        open={!!editCompanyId}
        onClose={() => setEditCompanyId(null)}
      />

      <CompanyUsersModal
        companyId={usersCompanyId}
        open={!!usersCompanyId}
        onClose={() => setUsersCompanyId(null)}
      />
    </div>
  );
}
```

### CompanyList
```typescript
// src/pages/Admin/AdminCompanies/CompanyList.tsx

import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { MoreHorizontal, Edit, Users, Trash } from 'lucide-react';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Skeleton } from '@/components/ui/skeleton';

interface Company {
  id: string;
  name: string;
  plan: string;
  status: 'active' | 'inactive' | 'trial';
  usersCount: number;
  createdAt: string;
}

interface CompanyListProps {
  companies: Company[];
  isLoading: boolean;
  onEdit: (id: string) => void;
  onManageUsers: (id: string) => void;
}

export function CompanyList({ companies, isLoading, onEdit, onManageUsers }: CompanyListProps) {
  if (isLoading) {
    return <CompanyListSkeleton />;
  }

  return (
    <Table>
      <TableHeader>
        <TableRow>
          <TableHead>Nome</TableHead>
          <TableHead>Plano</TableHead>
          <TableHead>Status</TableHead>
          <TableHead>Usuários</TableHead>
          <TableHead>Criado em</TableHead>
          <TableHead className="w-[50px]" />
        </TableRow>
      </TableHeader>
      <TableBody>
        {companies.map((company) => (
          <TableRow key={company.id}>
            <TableCell className="font-medium">{company.name}</TableCell>
            <TableCell>{company.plan}</TableCell>
            <TableCell>
              <StatusBadge status={company.status} />
            </TableCell>
            <TableCell>{company.usersCount}</TableCell>
            <TableCell>{new Date(company.createdAt).toLocaleDateString()}</TableCell>
            <TableCell>
              <DropdownMenu>
                <DropdownMenuTrigger asChild>
                  <Button variant="ghost" size="icon">
                    <MoreHorizontal className="w-4 h-4" />
                  </Button>
                </DropdownMenuTrigger>
                <DropdownMenuContent align="end">
                  <DropdownMenuItem onClick={() => onEdit(company.id)}>
                    <Edit className="w-4 h-4 mr-2" />
                    Editar
                  </DropdownMenuItem>
                  <DropdownMenuItem onClick={() => onManageUsers(company.id)}>
                    <Users className="w-4 h-4 mr-2" />
                    Usuários
                  </DropdownMenuItem>
                </DropdownMenuContent>
              </DropdownMenu>
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  );
}

function StatusBadge({ status }: { status: string }) {
  const variants: Record<string, 'default' | 'secondary' | 'destructive'> = {
    active: 'default',
    trial: 'secondary',
    inactive: 'destructive',
  };

  return <Badge variant={variants[status] || 'default'}>{status}</Badge>;
}

function CompanyListSkeleton() {
  return (
    <div className="space-y-2">
      {[...Array(5)].map((_, i) => (
        <Skeleton key={i} className="h-16 w-full" />
      ))}
    </div>
  );
}
```

---

## Checklist

### Estrutura
- [ ] Criar pasta `src/pages/Admin/AdminCompanies/`
- [ ] Criar pasta `hooks/`
- [ ] Criar pasta `modals/`

### Hooks
- [ ] `useCompanies.ts`
- [ ] `useCompanyMutations.ts`
- [ ] `useCompanyFilters.ts`

### Componentes
- [ ] `index.tsx` (container)
- [ ] `CompanyList.tsx`
- [ ] `CompanyFilters.tsx`

### Modals
- [ ] `CreateCompanyModal.tsx`
- [ ] `EditCompanyModal.tsx`
- [ ] `CompanyUsersModal.tsx`

### Finalização
- [ ] Atualizar router
- [ ] Remover arquivo antigo
- [ ] Testes

---

**Status:** ✅ Completo
**Última Atualização:** 2026-01-23
