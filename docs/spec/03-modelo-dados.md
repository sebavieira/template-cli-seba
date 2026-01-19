# 3. Modelo de Dados

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

← [Voltar para SPEC](README.md)

---

## 3.1 Diagrama Entidade-Relacionamento

```mermaid
erDiagram
    USERS ||--o{ {{ENTIDADE_1}} : creates
    {{ENTIDADE_1}} ||--o{ {{ENTIDADE_2}} : contains
    {{ENTIDADE_2}} ||--o{ {{ENTIDADE_3}} : has

    USERS {
        uuid id PK
        string email UK
        string password_hash
        string name
        enum role
        timestamp created_at
        timestamp updated_at
    }

    {{ENTIDADE_1}} {
        uuid id PK
        uuid user_id FK
        string name
        jsonb config
        enum status
        timestamp created_at
        timestamp updated_at
    }

    {{ENTIDADE_2}} {
        uuid id PK
        uuid {{entidade_1}}_id FK
        string field1
        string field2
        enum status
        timestamp created_at
    }
```

---

## 3.2 Schemas das Tabelas

### Tabela: users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'user'
        CHECK (role IN ('admin', 'user', 'viewer')),
    email_verified_at TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

### Tabela: {{TABELA_1}}

```sql
CREATE TABLE {{tabela_1}} (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    config JSONB NOT NULL DEFAULT '{}',
    status VARCHAR(20) NOT NULL DEFAULT 'draft'
        CHECK (status IN ('draft', 'active', 'paused', 'completed')),
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_{{tabela_1}}_user_id ON {{tabela_1}}(user_id);
CREATE INDEX idx_{{tabela_1}}_status ON {{tabela_1}}(status);
CREATE INDEX idx_{{tabela_1}}_created_at ON {{tabela_1}}(created_at DESC);
```

### Tabela: {{TABELA_2}}

```sql
CREATE TABLE {{tabela_2}} (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    {{tabela_1}}_id UUID NOT NULL REFERENCES {{tabela_1}}(id) ON DELETE CASCADE,
    -- Adicione campos específicos
    field1 VARCHAR(255),
    field2 TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Índices
CREATE INDEX idx_{{tabela_2}}_{{tabela_1}}_id ON {{tabela_2}}({{tabela_1}}_id);
CREATE INDEX idx_{{tabela_2}}_status ON {{tabela_2}}(status);
```

---

## 3.3 Tipos Enumerados

```sql
-- Status genérico
CREATE TYPE status_enum AS ENUM (
    'draft',
    'active',
    'paused',
    'completed',
    'failed',
    'cancelled'
);

-- Roles de usuário
CREATE TYPE user_role AS ENUM (
    'admin',
    'user',
    'viewer'
);

-- Adicione outros enums conforme necessidade
```

---

## 3.4 Relacionamentos

| Tabela Origem | Tabela Destino | Tipo | Descrição |
|---------------|----------------|------|-----------|
| users | {{tabela_1}} | 1:N | Usuário cria múltiplos {{tabela_1}} |
| {{tabela_1}} | {{tabela_2}} | 1:N | {{tabela_1}} contém múltiplos {{tabela_2}} |

---

## 3.5 Constraints e Validações

### users
- `email`: Único, formato válido
- `password_hash`: Mínimo 60 caracteres (bcrypt)
- `role`: Valores permitidos: admin, user, viewer

### {{tabela_1}}
- `name`: 3-100 caracteres
- `status`: Valores permitidos conforme enum
- `user_id`: Deve existir em users

---

## 3.6 Índices para Performance

| Tabela | Índice | Tipo | Justificativa |
|--------|--------|------|---------------|
| users | idx_users_email | B-tree | Busca por login |
| {{tabela_1}} | idx_{{tabela_1}}_user_id | B-tree | Filtro por usuário |
| {{tabela_1}} | idx_{{tabela_1}}_status | B-tree | Filtro por status |

---

## 3.7 Campos JSONB

### Estrutura de config ({{tabela_1}})

```json
{
  "setting1": "value1",
  "setting2": {
    "nested": true
  },
  "options": ["opt1", "opt2"]
}
```

### Estrutura de metadata ({{tabela_2}})

```json
{
  "source": "string",
  "extra_data": {},
  "timestamps": {
    "processed_at": "ISO8601"
  }
}
```

---

## 3.8 Migrations

### Ordem de Execução

1. `001_create_users.sql`
2. `002_create_{{tabela_1}}.sql`
3. `003_create_{{tabela_2}}.sql`
4. `004_add_indexes.sql`

### Exemplo de Migration (Prisma)

```prisma
model User {
  id            String   @id @default(uuid())
  email         String   @unique
  passwordHash  String   @map("password_hash")
  name          String
  role          Role     @default(user)
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")

  // Relations
  {{tabela_1}}s {{Tabela_1}}[]

  @@map("users")
}

enum Role {
  admin
  user
  viewer
}
```

---

## 3.9 Soft Delete (Opcional)

Se usar soft delete, adicionar em cada tabela:

```sql
deleted_at TIMESTAMP DEFAULT NULL
```

E índice parcial:

```sql
CREATE INDEX idx_{{tabela}}_active ON {{tabela}}(id)
WHERE deleted_at IS NULL;
```

---

← [Voltar para SPEC](README.md) | [Próximo: Contratos de API →](04-contratos-api/README.md)
