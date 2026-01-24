# Instruções do Projeto - Evo AI Connect

**Este arquivo é lido automaticamente pelo Claude Code em toda nova sessão**

---

## 🎯 FOCO ATUAL: REFATORAÇÃO

**Prioridade máxima: Refatoração do código antes de novas features.**

O usuário é **não-técnico** e usa **comandos simples**. Você deve:
1. Ler `doc/refactoring/README.md` SEMPRE ao iniciar
2. Identificar próxima refatoração por prioridade (P0 → P1 → P2)
3. Carregar documento específico e seguir checklist
4. Testar após cada mudança
5. Atualizar status no documento após completar
6. Usar linguagem simples ao reportar

---

## 📝 REGRA CRÍTICA: PERSISTÊNCIA DE PROGRESSO

**OBRIGATÓRIO:** Sempre que criar/modificar arquivos de código durante refatoração:

1. **ATUALIZAR `doc/refactoring/README.md`** com:
   - Status do item (⏳ → 🔄 X% → ✅)
   - Tabela de arquivos criados com linhas
   - Branch atual
   - Histórico de progresso

2. **Formato de atualização:**
```markdown
#### 📊 Detalhes: [XX]-[Nome] (Em Progresso)
**Branch:** `refactor/[nome]`
**Arquivos Criados:** X de Y

| Arquivo | Linhas | Status |
|---------|--------|--------|
| `arquivo.ts` | XXX | ✅/⏳ |
```

3. **Histórico de Progresso** (adicionar linha):
```markdown
| Data | Refatoração | Ação | Branch |
|------|-------------|------|--------|
| YYYY-MM-DD | XX-Nome | Descrição | `branch` |
```

**POR QUÊ:** Isso garante que o comando `status` mostre progresso correto mesmo após `clear` do contexto.

### Documentação de Refatoração
```
doc/refactoring/
├── README.md              ← ÍNDICE (ler primeiro!)
├── 01-06                  ← P0 Crítico (fazer primeiro)
├── 07-13                  ← P1 Alto
└── 14-18                  ← P2 Médio
```

---

## 🎯 COMANDOS DO USUÁRIO

Reconheça e responda a estes comandos:

### `status` (comando principal)
```
1. Read: doc/refactoring/README.md
2. Mostrar:
   - Progresso geral de refatoração (X/18 completos, Y em progresso)
   - Detalhes de refatoração em andamento (arquivos criados, % completo)
   - Branch atual se houver refatoração em progresso
   - Próxima refatoração pendente (por prioridade P0→P1→P2)
   - Arquivos críticos restantes
3. Se houver refatoração em progresso:
   - Mostrar tabela de arquivos criados vs pendentes
   - Sugerir "continue" para retomar
```

### `continue` ou `próximo`
```
1. Read: doc/refactoring/README.md (identificar próxima pendente)
2. Read: doc/refactoring/[XX]-[nome].md (carregar plano)
3. Executar refatoração seguindo checklist
4. Rodar testes após cada mudança
5. Atualizar status no documento (⏳ → ✅)
6. Reportar em linguagem simples
```

### `refatorar [número]` ou `refatorar [nome]`
```
Exemplos: "refatorar 01" ou "refatorar whatsapp"

1. Read: doc/refactoring/[número]-*.md
2. Mostrar plano resumido
3. Perguntar se pode iniciar
4. Executar checklist
5. Testar e atualizar status
```

### `teste` ou `validar`
```
1. Executar testes do projeto
2. Reportar resultados
3. Se falhar, tentar corrigir
```

### `o que falta?`
```
1. Read: doc/refactoring/README.md
2. Listar todas refatorações pendentes por prioridade
3. Mostrar estimativa total
```

### `explica [número]`
```
1. Explicar refatoração específica em linguagem SIMPLES
2. Mostrar benefício da mudança
3. Sem jargão técnico
```

---

### 📋 Referência Rápida de Refatorações

| # | Arquivo | Prioridade | Linhas |
|---|---------|------------|--------|
| 01 | whatsapp.service.ts | P0 | 2.482 |
| 02 | assistant.service.ts | P0 | 2.612 |
| 03 | validação Zod (routes) | P0 | 127 routes |
| 04 | api.ts (frontend) | P0 | 2.037 |
| 05 | Messages.tsx | P0 | 1.699 |
| 06 | use-chat.ts | P0 | 1.649 |
| 07-13 | Services diversos | P1 | 800-1.300 |
| 14-18 | Workers/Components | P2 | 600-1.100 |

---

## ⚡ PROTOCOLO OBRIGATÓRIO - Leia PRIMEIRO

### 1️⃣ Contextualização (SEMPRE ao iniciar sessão)

```
Read: doc/refactoring/README.md     (índice de refatorações)
```

**Token Budget:** ~3K tokens

### 2️⃣ Para Executar Refatoração

```
Read: doc/refactoring/[XX]-[nome].md   (plano específico ~400 linhas)
Seguir checklist do documento
Testar após cada mudança
Atualizar status: ⏳ → ✅
```

---

## 📚 Estrutura da Documentação

```
docs/
├── INDEX.md                    ← Navegação completa
├── STATUS.md                   ← Estado atual (ler sempre!)
├── prd/                        ← Requisitos de produto
│   ├── README.md              ← Índice do PRD
│   ├── 01-visao-objetivos.md
│   ├── 04-user-stories/       ← 9 Epics
│   │   ├── epic-01-autenticacao.md
│   │   ├── epic-02-whatsapp.md
│   │   ├── epic-03-contatos.md
│   │   ├── epic-04-funis.md
│   │   ├── epic-05-ia.md
│   │   ├── epic-06-deals.md
│   │   ├── epic-07-followups.md
│   │   ├── epic-08-dashboard.md
│   │   └── epic-09-flows.md
│   └── ...
└── spec/                       ← Especificações técnicas
    ├── README.md              ← Índice da SPEC
    ├── 03-modelo-dados.md
    ├── 04-contratos-api/      ← 8 Domínios
    │   ├── auth.md
    │   ├── whatsapp.md
    │   ├── contacts.md
    │   ├── funnels.md
    │   ├── deals.md
    │   ├── ai.md
    │   ├── followups.md
    │   └── flows.md
    └── ...

doc/
├── REFACTORING-PLAN.md         ← Plano master de refatoração
└── refactoring/                ← Documentação modular (18 docs)
    ├── README.md              ← Índice com progresso
    ├── 01-whatsapp-service.md ← P0 Backend
    ├── 02-assistant-service.md
    ├── 03-validation-schemas.md
    ├── 04-api-frontend.md     ← P0 Frontend
    ├── 05-messages-page.md
    ├── 06-use-chat-hook.md
    ├── 07-followups-service.md ← P1 Backend
    ├── ...                    ← (07-13 P1)
    ├── 14-message-worker.md   ← P2 Backend
    └── ...                    ← (14-18 P2)
```

---

## ❌ REGRAS CRÍTICAS

1. **NUNCA** ler múltiplos arquivos grandes simultaneamente
2. **SEMPRE** usar Grep/Glob antes de Read
3. **SEMPRE** consultar índices antes de ler conteúdo
4. **LIMITE:** <50K tokens por interação

---

## ✅ Estratégia de Leitura

**Para implementar feature:**
```
1. Read: docs/prd/04-user-stories/epic-XX.md  (requisitos)
2. Read: docs/spec/04-contratos-api/XX.md     (API)
3. Grep: "termo específico" em docs/spec/     (localizar)
4. Read: arquivo específico (offset + limit)   (detalhes)
```

**Para debugging:**
```
1. Identificar domínio
2. Read: docs/spec/04-contratos-api/[dominio].md
3. Read: docs/spec/06-maquina-estados.md (se precisar de estados)
```

---

## 📊 Orçamento de Tokens por Tarefa

| Tarefa | Tokens | Arquivos |
|--------|--------|----------|
| Visão geral | 15K | README + INDEX |
| User story | 8K | 1 epic |
| API endpoint | 10K | 1 domínio |
| Feature completa | 30K | US + API + Schema |

---

## 🔗 Links Rápidos

- **Início Rápido:** `AI-START.md` (~2K tokens)
- **Guia Completo:** `README.md` seção "🤖 Guia para Agentes de IA"
- **Protocolo Detalhado:** `.ai-instructions.md`
- **Manutenção de Docs:** `docs/MANUTENÇÃO.md`

---

## 🎯 Projeto: Evo AI Connect

**Descrição:** Plataforma de automação WhatsApp com IA, CRM integrado e gestão de funis de vendas

**Funcionalidades Principais:**
- CRM com contatos, deals e funis
- Integração WhatsApp via Evolution API/UAZAPI
- IA com GPT-4 para sugestões e análise

**Stack:**
- Backend: Fastify + TypeScript + Prisma + PostgreSQL
- Database: PostgreSQL 16 + pgvector + Redis
- Frontend: React + Vite + TailwindCSS + Shadcn/UI

**Status:** 📋 Documentação ✅ → 🚀 Implementação ⏳

---

## 🛠️ Skills Disponíveis

Skills especializadas em `.claude/skills/`:

| Skill | Quando Usar |
|-------|-------------|
| `backend.md` | APIs, módulos, serviços, workers |
| `frontend.md` | Páginas, componentes, UI |
| `ai-integration.md` | Features de AI, prompts, embeddings |
| `flow-engine.md` | Flows de automação, triggers |
| `whatsapp-integration.md` | Integração WhatsApp, webhooks |
| `database.md` | Schema Prisma, migrations, queries |
| `testing.md` | Testes Vitest, mocking |
| `devops.md` | Docker, deploy, infraestrutura |
| `refactoring.md` | Refatoração, qualidade, code smells |

**Ativação:** Automática por contexto ou mencione a skill explicitamente.

---

## 💬 LINGUAGEM PARA O USUÁRIO

**O usuário NÃO é técnico. Use linguagem SIMPLES:**

❌ Evite:
- "Implementei o endpoint REST com validação Zod"
- "Configurei o Prisma ORM com migrations"
- "Adicionei testes unitários com Jest e 95% de coverage"

✅ Use:
- "Criei a funcionalidade de criar [recurso]"
- "Configurei o banco de dados para guardar informações"
- "Adicionei verificações automáticas para garantir que tudo funciona"

**Formato de Resposta para Refatoração:**

```
✅ Refatoração Completa: [Nome do arquivo]

O que fiz:
- Dividi arquivo grande em [X] arquivos menores
- Cada arquivo agora tem responsabilidade única
- Código mais fácil de entender e manter

Resultado:
- Antes: [X] linhas em 1 arquivo
- Depois: [Y] linhas divididas em [Z] arquivos
- Testes: ✅ Todos passaram

Progresso: [X]/18 refatorações completas

Próxima: [Nome da próxima refatoração]
```

## 🔄 CICLO DE REFATORAÇÃO

Para comando `continue`:

```
1. READ doc/refactoring/README.md (índice)
2. Identificar próxima refatoração pendente (P0→P1→P2)
3. READ doc/refactoring/[XX]-[nome].md (plano)
4. Criar branch: git checkout -b refactor/[nome]
5. EXECUTAR checklist do documento
6. TESTAR após cada mudança
7. COMMIT atômico por extração
8. ATUALIZAR status no documento (⏳ → ✅)
9. REPORTAR em linguagem simples
10. Mostrar próxima refatoração
```

**Token Budget por Ciclo:** ~15-25K tokens

## 📋 Checklist de Refatoração

- [ ] Li doc/refactoring/README.md?
- [ ] Identifiquei a próxima refatoração P0?
- [ ] Carreguei o documento específico?
- [ ] Criei branch de refatoração?
- [ ] Vou testar após cada mudança?
- [ ] Vou atualizar status ao completar?

---

**Última Atualização:** 2026-01-22
**Versão:** 2.0.0 (Foco em Refatoração)
