# Etapa 6: Revisão Final

> Validação e confirmação antes de gerar documentação

**Tempo Estimado:** 5 minutos

---

## Introdução

```markdown
"Chegamos na última etapa!

Vou apresentar um RESUMO de tudo que conversamos.
Leia com calma e me diz se precisa corrigir alguma coisa.

Depois disso, vou gerar toda a documentação do projeto."
```

---

## Checklist de Validação

### Verificar Dados Obrigatórios

```markdown
Antes de apresentar resumo, verifique:

□ Nome do projeto definido
□ Problema claramente descrito
□ Pelo menos 1 persona com dores e necessidades
□ Escopo definido (inclui e não inclui)
□ Pelo menos 1 Epic com User Stories
□ Stack técnico decidido
```

### Se Faltar Informação

```markdown
"Percebi que falta definir [ITEM].

Posso:
1. Perguntar agora rapidamente
2. Usar um valor padrão
3. Deixar para definir depois

O que prefere?"
```

---

## Apresentação do Resumo

```markdown
"Aqui está o resumo completo do seu projeto:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 🎯 VISÃO DO PROJETO

**Nome:** [nome_projeto]

**Problema:**
[problema descrito]

**Solução:**
[solução descrita]

**Objetivo Principal:**
[objetivo_principal]

**Metas de Sucesso:**
| Métrica | Hoje | Meta | Prazo |
|---------|------|------|-------|
| [métrica 1] | [baseline] | [meta] | [prazo] |
| [métrica 2] | [baseline] | [meta] | [prazo] |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 👥 USUÁRIOS

**Persona Principal:** [nome]
- [descrição breve]
- Dor principal: [dor]
- Necessidade principal: [necessidade]

**Outras Personas:**
- [Nome 2]: [descrição breve]
- [Nome 3]: [descrição breve]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## ✅ ESCOPO

**O Sistema FAZ:**
- [item 1]
- [item 2]
- [item 3]

**O Sistema NÃO FAZ:**
- [item 1]
- [item 2]

**Futuro (V2+):**
- [item 1]
- [item 2]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 📋 FUNCIONALIDADES

**Epic 1: [Nome]**
- US-001: [descrição]
- US-002: [descrição]

**Epic 2: [Nome]**
- US-003: [descrição]
- US-004: [descrição]

[... outros epics ...]

**Total:** [X] User Stories ([Y] essenciais, [Z] importantes)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## 🛠️ TECNOLOGIA

**Stack:**
- Backend: [backend]
- Frontend: [frontend]
- Banco de Dados: [database]

**Integrações:**
- [serviço 1]
- [serviço 2]

**Custo Estimado:** R$ [valor]/mês

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Está tudo correto?**

1. ✅ Sim, pode gerar a documentação
2. ✏️ Preciso corrigir algo
3. ➕ Quero adicionar mais informações
"
```

---

## Tratando Correções

### Se Precisar Corrigir

```markdown
"Sem problemas! O que precisa corrigir?

Pode me dizer:
- A seção (Visão, Personas, Escopo, Funcionalidades, Técnico)
- O que está errado
- Como deveria ser"
```

### Se Quiser Adicionar

```markdown
"Claro! O que quer adicionar?

Lembre que sempre podemos atualizar depois também."
```

---

## Confirmação Final

```markdown
"Perfeito! Vou gerar a documentação completa:

📁 **Arquivos que serão criados:**

docs/
├── INDEX.md
├── STATUS.md
├── prd/
│   ├── README.md
│   ├── 01-visao-objetivos.md
│   ├── 02-contexto-personas.md
│   ├── 03-escopo.md
│   ├── 04-user-stories/
│   │   ├── README.md
│   │   └── [epics...]
│   ├── 05-rnf.md
│   └── [outros...]
└── spec/
    ├── README.md
    ├── 01-visao-geral.md
    ├── 02-arquitetura.md
    └── [outros...]

Isso vai levar alguns minutos.

**Posso começar?**"
```

---

## Após Confirmação

### Processo de Geração

```markdown
"Gerando documentação...

✅ INDEX.md criado
✅ STATUS.md criado
✅ PRD: Visão e Objetivos
✅ PRD: Contexto e Personas
✅ PRD: Escopo
✅ PRD: User Stories (X epics)
✅ PRD: Requisitos Não Funcionais
✅ SPEC: Visão Geral
✅ SPEC: Arquitetura
✅ SPEC: Modelo de Dados
✅ SPEC: Contratos de API
[... continuar listando ...]

**Documentação gerada com sucesso!**"
```

### Próximos Passos

```markdown
"Pronto! Seu projeto está documentado.

**Próximos passos:**

1. **Revise a documentação** em `docs/`
   - Veja se faz sentido
   - Corrija detalhes se precisar

2. **Comece a implementar**
   - Digite: `continue`
   - Vou seguir o STATUS.md automaticamente

3. **Acompanhe o progresso**
   - Digite: `status`
   - Vejo o que foi feito e o que falta

**Comandos úteis:**
- `continue` → Implementa próxima tarefa
- `status` → Mostra progresso
- `teste` → Executa testes
- `explica` → Explica última tarefa

Quer começar a implementar agora?"
```

---

## Dados Finais Consolidados

```yaml
projeto:
  nome: ""
  versao: "1.0.0"
  data_criacao: ""

visao:
  problema: ""
  solucao: ""
  diferencial: ""
  objetivo_principal: ""
  objetivos_secundarios: []
  criterios_sucesso: []
  timeline:
    mvp: ""
    v1_1: ""
    v2: ""

personas:
  - nome: ""
    tipo: "principal"
    descricao: ""
    dores: []
    necessidades: []

escopo:
  inclui:
    must_have: []
    should_have: []
    could_have: []
  nao_inclui: []
  futuro: []
  premissas: []
  restricoes: []

funcionalidades:
  epics:
    - nome: ""
      user_stories: []

tecnico:
  stack:
    backend: ""
    frontend: ""
    database: ""
  integracoes: []
  infraestrutura: ""
  custo_estimado: ""
```

---

## Após Gerar Documentação

A IA deve:

1. **Criar todos os arquivos de PRD** usando dados capturados
2. **Criar todos os arquivos de SPEC** derivados do PRD
3. **Criar STATUS.md** com roadmap de implementação
4. **Atualizar INDEX.md** com navegação completa
5. **Confirmar sucesso** ao usuário

---

**Fim do Briefing - Início da Implementação**
