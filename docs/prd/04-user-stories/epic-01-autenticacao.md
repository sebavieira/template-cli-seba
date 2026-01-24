# Epic 01: Autenticação e Gestão de Usuários

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Must Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que usuários se registrem, façam login e gerenciem suas equipes de forma segura, com suporte a múltiplas empresas e diferentes níveis de permissão.

---

## User Stories

### US-01.1: Registro de Usuário

**Como** um novo usuário,
**Quero** me registrar na plataforma com email e senha,
**Para que** eu possa começar a usar o sistema.

**Critérios de Aceitação:**
- [ ] Formulário aceita email válido e senha com mínimo 8 caracteres
- [ ] Validação de email único no sistema
- [ ] Senha é armazenada com hash bcrypt
- [ ] Após registro, usuário recebe sessão válida
- [ ] Criação automática de Company padrão para o usuário

**Prioridade:** Must Have

---

### US-01.2: Login com Email/Senha

**Como** um usuário registrado,
**Quero** fazer login com meu email e senha,
**Para que** eu possa acessar minha conta.

**Critérios de Aceitação:**
- [ ] Login com credenciais válidas retorna JWT token
- [ ] Token tem expiração configurável (padrão: 7 dias)
- [ ] Credenciais inválidas retornam erro 401
- [ ] Sessão é registrada com IP e timestamp
- [ ] Rate limiting de 5 tentativas por minuto por IP

**Prioridade:** Must Have

---

### US-01.3: Logout

**Como** um usuário logado,
**Quero** fazer logout da minha sessão,
**Para que** eu encerre meu acesso de forma segura.

**Critérios de Aceitação:**
- [ ] Token é invalidado no servidor
- [ ] Sessão é marcada como encerrada
- [ ] Redirecionamento para página de login

**Prioridade:** Must Have

---

### US-01.4: Refresh de Token

**Como** um usuário com sessão ativa,
**Quero** que meu token seja renovado automaticamente,
**Para que** eu não perca acesso durante uso contínuo.

**Critérios de Aceitação:**
- [ ] Refresh token com validade maior (30 dias)
- [ ] Novo access token emitido sem re-login
- [ ] Refresh token inválido força novo login

**Prioridade:** Must Have

---

### US-01.5: Perfil do Usuário

**Como** um usuário logado,
**Quero** visualizar e editar meu perfil,
**Para que** eu mantenha minhas informações atualizadas.

**Critérios de Aceitação:**
- [ ] Visualização de nome, email, timezone
- [ ] Edição de campos permitidos
- [ ] Upload de foto de perfil (opcional)
- [ ] Validação de campos obrigatórios

**Prioridade:** Should Have

---

### US-01.6: Gestão de Equipe

**Como** um admin da empresa,
**Quero** convidar usuários para minha equipe,
**Para que** outros vendedores possam usar o sistema.

**Critérios de Aceitação:**
- [ ] Convite por email com link de acesso
- [ ] Definição de role (Admin, Usuário)
- [ ] Lista de membros da equipe
- [ ] Remoção de membros
- [ ] Limite de usuários por plano

**Prioridade:** Should Have

---

### US-01.7: Controle de Acesso (RBAC)

**Como** super admin,
**Quero** definir permissões por role,
**Para que** usuários tenham acesso apenas ao necessário.

**Critérios de Aceitação:**
- [ ] Roles: SUPER_ADMIN, ADMIN, USER
- [ ] SUPER_ADMIN: acesso total a todas empresas
- [ ] ADMIN: gestão completa da própria empresa
- [ ] USER: acesso operacional sem configurações sensíveis

**Prioridade:** Must Have

---

## Modelo de Dados

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String
  publicId  String   @unique @default(cuid())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  sessions    Session[]
  profile     Profile?
  companies   UserCompany[]
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  ipAddress String?
  userAgent String?
  isValid   Boolean  @default(true)

  user User @relation(fields: [userId], references: [id])
}

model UserCompany {
  id        String   @id @default(cuid())
  userId    String
  companyId String
  role      AppRole  @default(USER)

  user    User    @relation(fields: [userId], references: [id])
  company Company @relation(fields: [companyId], references: [id])
}

enum AppRole {
  SUPER_ADMIN
  USER
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/auth/register` | Registro de usuário |
| POST | `/api/auth/login` | Login |
| POST | `/api/auth/logout` | Logout |
| POST | `/api/auth/refresh` | Refresh token |
| GET | `/api/auth/me` | Dados do usuário atual |
| GET | `/api/company/users` | Listar membros |
| POST | `/api/company/users/invite` | Convidar usuário |
| DELETE | `/api/company/users/:id` | Remover usuário |

---

[← Voltar para User Stories](README.md)
