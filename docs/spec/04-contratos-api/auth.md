# 4.1 Autenticação

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

← [Voltar para Contratos de API](README.md)

---

## POST /api/auth/register

**Descrição:** Criar nova conta de usuário.

**Autenticação:** Não requerida

**Rate Limit:** 30 req/min

**Request Body:**

```json
{
  "email": "usuario@empresa.com",
  "password": "senhaSegura123",
  "fullName": "Nome Completo",
  "companyName": "Minha Empresa"
}
```

| Campo | Tipo | Obrigatório | Validação |
|-------|------|-------------|-----------|
| email | string | Sim | Email válido, único |
| password | string | Sim | Mínimo 6 caracteres |
| fullName | string | Não | 2-100 caracteres |
| companyName | string | Não | 2-100 caracteres |

**Response 201 Created:**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "usuario@empresa.com",
      "fullName": "Nome Completo"
    },
    "company": {
      "id": "uuid",
      "name": "Minha Empresa"
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Email já cadastrado"
}
```

**Regras de Negócio:**
- Email deve ser único no sistema
- Senha é hasheada com bcrypt (10 rounds)
- Empresa é criada automaticamente se `companyName` for fornecido
- Usuário recebe role `admin` na empresa criada
- Trial de 14 dias iniciado automaticamente

---

## POST /api/auth/login

**Descrição:** Autenticar usuário e obter tokens.

**Autenticação:** Não requerida

**Rate Limit:** 30 req/min

**Request Body:**

```json
{
  "email": "usuario@empresa.com",
  "password": "senhaSegura123"
}
```

| Campo | Tipo | Obrigatório |
|-------|------|-------------|
| email | string | Sim |
| password | string | Sim |

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "usuario@empresa.com",
      "fullName": "Nome Completo",
      "companyId": "uuid"
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 900
  }
}
```

**Response 401 Unauthorized:**

```json
{
  "success": false,
  "error": "Email ou senha incorretos"
}
```

**Tokens:**
- **Access Token:** Validade 15 minutos
- **Refresh Token:** Validade 7 dias
- **Algoritmo:** HS256

---

## POST /api/auth/logout

**Descrição:** Invalidar sessão atual.

**Autenticação:** Bearer token (opcional)

**Request Headers:**

```
Authorization: Bearer {access_token}
```

**Response 200 OK:**

```json
{
  "success": true,
  "message": "Logout realizado com sucesso"
}
```

**Comportamento:**
- Invalida o token na tabela `sessions`
- Retorna sucesso mesmo se token não existir/expirado
- Não requer autenticação obrigatória

---

## POST /api/auth/refresh

**Descrição:** Renovar access token usando refresh token.

**Autenticação:** Não requerida

**Rate Limit:** 30 req/min

**Request Body:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 900
  }
}
```

**Response 401 Unauthorized:**

```json
{
  "success": false,
  "error": "Token inválido ou expirado"
}
```

**Regras:**
- Refresh token é rotacionado a cada uso
- Token antigo é invalidado
- Sessão é atualizada no banco

---

## POST /api/auth/switch-company

**Descrição:** Trocar contexto para outra empresa.

**Autenticação:** Bearer token (JWT)

**Request Body:**

```json
{
  "companyId": "uuid-da-empresa"
}
```

**Response 200 OK:**

```json
{
  "success": true
}
```

**Response 400 Bad Request:**

```json
{
  "success": false,
  "error": "Você não pertence a esta empresa"
}
```

**Regras:**
- Usuário deve pertencer à empresa (tabela `user_companies`)
- Atualiza `companyId` no perfil do usuário
- Próximas requisições usarão o novo contexto

---

## GET /api/auth/me

**Descrição:** Obter perfil do usuário autenticado.

**Autenticação:** Bearer token (JWT)

**Request Headers:**

```
Authorization: Bearer {access_token}
```

**Response 200 OK:**

```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "usuario@empresa.com",
    "fullName": "Nome Completo",
    "phone": "+5511999999999",
    "timezone": "America/Sao_Paulo",
    "companyId": "uuid",
    "company": {
      "id": "uuid",
      "name": "Minha Empresa",
      "maxWhatsappInstances": 3,
      "planType": "pro"
    },
    "companyFeatures": {
      "maxContacts": 10000,
      "aiEnabled": true
    },
    "roles": ["USER"],
    "isActive": true,
    "companies": [
      {
        "id": "uuid-1",
        "name": "Empresa 1",
        "role": "admin"
      },
      {
        "id": "uuid-2",
        "name": "Empresa 2",
        "role": "member"
      }
    ],
    "activeCompanyId": "uuid-1"
  }
}
```

**Response 401 Unauthorized:**

```json
{
  "success": false,
  "error": "Não autenticado"
}
```

**Response 404 Not Found:**

```json
{
  "success": false,
  "error": "Usuário não encontrado"
}
```

**Campos Retornados:**
- `companies`: Lista de empresas que o usuário pertence
- `activeCompanyId`: Empresa atualmente selecionada
- `companyFeatures`: Features disponíveis no plano
- `roles`: Roles globais do sistema (SUPER_ADMIN, USER)

---

## Estrutura do JWT

### Payload do Access Token

```json
{
  "sub": "user-uuid",
  "email": "usuario@empresa.com",
  "companyId": "company-uuid",
  "iat": 1640000000,
  "exp": 1640000900
}
```

### Headers de Autenticação

Para rotas protegidas, incluir:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

## Códigos de Erro Comuns

| Código | Erro | Causa |
|--------|------|-------|
| 400 | Email já cadastrado | Tentativa de registro com email existente |
| 401 | Email ou senha incorretos | Credenciais inválidas no login |
| 401 | Token inválido ou expirado | Access/refresh token expirado |
| 401 | Não autenticado | Header Authorization ausente |
| 403 | Conta inativa | Usuário desativado |

---

**Anterior:** [← Índice API](README.md) | **Próximo:** [WhatsApp →](whatsapp.md)
