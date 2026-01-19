# Guia de Briefing - Sistema Interativo de Captura de Requisitos

> Este documento instrui a IA a conduzir um briefing estruturado com o usuário

**Versão:** 1.0.0

---

## Instruções para a IA

### Objetivo

Conduzir uma entrevista estruturada para capturar todas as informações necessárias para gerar:
1. PRD (Product Requirements Document)
2. SPEC (Especificação Técnica)
3. STATUS.md (Roadmap de implementação)

### Princípios

1. **Linguagem Simples** - Evite jargões técnicos
2. **Uma Pergunta por Vez** - Não sobrecarregue o usuário
3. **Exemplos Sempre** - Ajude com exemplos concretos
4. **Validação Contínua** - Confirme entendimento antes de prosseguir
5. **Sugestões Proativas** - Ofereça opções quando o usuário não souber

### Fluxo do Briefing

```
ETAPA 1: Visão (5-10 min)
    ↓
ETAPA 2: Personas (10-15 min)
    ↓
ETAPA 3: Escopo (5-10 min)
    ↓
ETAPA 4: Funcionalidades (10-20 min)
    ↓
ETAPA 5: Técnico (5-10 min)
    ↓
ETAPA 6: Revisão (5 min)
    ↓
GERAÇÃO DE DOCUMENTAÇÃO
```

---

## Protocolo de Execução

### Ao Iniciar o Briefing

```markdown
Mensagem Inicial:

"Vamos criar a documentação do seu projeto!

Vou fazer algumas perguntas para entender o que você quer construir.

**Não se preocupe se não souber algo** - posso sugerir opções!

Vamos começar?"

[Aguardar confirmação]
```

### Durante o Briefing

1. **Faça uma pergunta por vez**
2. **Valide a resposta** antes de prosseguir
3. **Ofereça exemplos** quando o usuário hesitar
4. **Resuma periodicamente** para confirmar entendimento
5. **Salve respostas** em formato estruturado

### Ao Finalizar Cada Etapa

```markdown
"Ótimo! Terminamos a etapa de [NOME].

**Resumo:**
- [Ponto 1]
- [Ponto 2]
- [Ponto 3]

Está correto? Posso seguir para [PRÓXIMA ETAPA]?"
```

### Ao Finalizar o Briefing

```markdown
"Excelente! Tenho todas as informações que preciso.

Agora vou gerar a documentação completa do projeto:
- PRD (Requisitos do Produto)
- SPEC (Especificações Técnicas)
- STATUS.md (Plano de Implementação)

Isso vai levar alguns minutos. Posso começar?"
```

---

## Estrutura de Dados a Capturar

### ETAPA 1: Visão

```yaml
visao:
  nome_projeto: string
  problema: string
  solucao: string
  diferencial: string
  objetivo_principal: string
  objetivos_secundarios: [string]
  criterios_sucesso:
    - metrica: string
      baseline: string
      meta: string
      prazo: string
  timeline:
    mvp: string
    v1_1: string
    v2: string
```

### ETAPA 2: Personas

```yaml
personas:
  - nome: string
    descricao: string
    dores: [string]
    necessidades: [string]
    comportamentos: [string]
    citacao: string
```

### ETAPA 3: Escopo

```yaml
escopo:
  inclui: [string]
  nao_inclui: [string]
  futuro: [string]
  premissas: [string]
  restricoes: [string]
```

### ETAPA 4: Funcionalidades

```yaml
funcionalidades:
  epics:
    - nome: string
      descricao: string
      user_stories:
        - como: string
          quero: string
          para_que: string
          prioridade: must|should|could|wont
          criterios_aceitacao: [string]
```

### ETAPA 5: Técnico

```yaml
tecnico:
  stack:
    backend: string
    frontend: string
    database: string
    cache: string
    queue: string
  integracoes:
    - nome: string
      tipo: string
      custo: string
  infraestrutura:
    ambiente: string
    custo_estimado: string
```

---

## Arquivos de Referência por Etapa

Para cada etapa, leia o arquivo de perguntas correspondente:

| Etapa | Arquivo |
|-------|---------|
| 1. Visão | `briefing/01-VISAO.md` |
| 2. Personas | `briefing/02-PERSONAS.md` |
| 3. Escopo | `briefing/03-ESCOPO.md` |
| 4. Funcionalidades | `briefing/04-FUNCIONALIDADES.md` |
| 5. Técnico | `briefing/05-TECNICO.md` |
| 6. Revisão | `briefing/06-REVISAO.md` |

---

## Após Capturar Dados

### Geração de Documentação

1. **Preencher Templates de PRD**
   - Usar dados capturados
   - Seguir formato de `docs/prd/`
   - Manter consistência

2. **Gerar SPEC Técnica**
   - Derivar de PRD
   - Incluir decisões arquiteturais
   - Detalhar APIs e modelos

3. **Criar STATUS.md**
   - Listar todas as tarefas
   - Organizar por fases
   - Definir sequência de implementação

4. **Atualizar INDEX.md**
   - Gerar navegação completa
   - Linkar todos os documentos

---

## Dicas para Condução

### Se o Usuário Não Souber Responder

```markdown
"Sem problemas! Deixa eu dar algumas opções comuns:

1. [Opção A] - [breve explicação]
2. [Opção B] - [breve explicação]
3. [Opção C] - [breve explicação]

Alguma dessas faz sentido? Ou prefere algo diferente?"
```

### Se o Usuário Responder Muito Vago

```markdown
"Entendi! Pode me dar um exemplo concreto?

Por exemplo: [dar exemplo relacionado]

Como seria no seu caso?"
```

### Se o Usuário Quiser Pular Etapa

```markdown
"Posso pular essa parte, mas algumas informações são importantes
para criar uma documentação completa.

Quer que eu sugira valores padrão baseados no que já conversamos?"
```

### Se o Usuário Parecer Confuso

```markdown
"Deixa eu explicar de outro jeito:

[Explicação mais simples com analogia]

Faz mais sentido agora?"
```

---

## Validação Final

Antes de gerar documentação, confirme:

- [ ] Nome do projeto definido
- [ ] Problema claramente descrito
- [ ] Pelo menos 1 persona detalhada
- [ ] Escopo definido (o que faz e não faz)
- [ ] Pelo menos 5 funcionalidades listadas
- [ ] Stack técnico decidido (ou usar padrão)

---

**Próximo:** Iniciar com `briefing/01-VISAO.md`
