# Etapa 4: Funcionalidades Detalhadas

> Perguntas para detalhar cada funcionalidade em User Stories

**Tempo Estimado:** 10-20 minutos

---

## Introdução

```markdown
"Agora vamos detalhar as funcionalidades!

Para cada funcionalidade que você listou, vou perguntar:
- QUEM vai usar
- O QUE vai fazer
- POR QUE é importante
- COMO saber se está funcionando

Isso ajuda a garantir que nada seja esquecido."
```

---

## Processo de Detalhamento

### Para Cada Funcionalidade, Criar Epic

```markdown
"Vamos começar com a funcionalidade: [NOME DA FUNCIONALIDADE]

Essa vai ser um 'Epic' (grupo de tarefas relacionadas)."
```

### Perguntas para o Epic

#### 4.1 Descrição do Epic

```markdown
Pergunta:
"Em poucas palavras, o que esse módulo/funcionalidade faz?"

Exemplo:
Epic: Agendamento
Descrição: "Permite que clientes agendem horários e barbeiros vejam a agenda"

Captura: epics[n].descricao
```

#### 4.2 User Stories do Epic

Para cada ação dentro do Epic:

```markdown
Pergunta:
"Quais AÇÕES os usuários fazem nessa funcionalidade?

Use o formato:
- Como [QUEM], quero [O QUE], para [POR QUE]"

Exemplo:
"Como CLIENTE, quero VER HORÁRIOS DISPONÍVEIS, para ESCOLHER o melhor para mim"
"Como BARBEIRO, quero BLOQUEAR HORÁRIO, para NÃO receber agendamentos"

Captura: epics[n].user_stories
```

#### 4.3 Critérios de Aceitação

```markdown
Pergunta:
"Para cada ação, como você sabe se está funcionando corretamente?

O que PRECISA acontecer para considerar pronto?"

Exemplo:
User Story: "Cliente ver horários disponíveis"
Critérios:
- Mostra apenas horários futuros
- Horários bloqueados não aparecem
- Atualiza em tempo real
- Funciona no celular

Captura: epics[n].user_stories[m].criterios_aceitacao
```

#### 4.4 Prioridade

```markdown
Pergunta:
"Qual a prioridade dessa funcionalidade?

- ESSENCIAL: Sem isso, sistema não funciona
- IMPORTANTE: Sistema funciona, mas fica incompleto
- DESEJÁVEL: Seria bom ter, mas pode esperar"

Captura: epics[n].user_stories[m].prioridade
```

---

## Template de Epic

Para cada Epic identificado, preencha:

```yaml
epic:
  nome: "Nome do Epic"
  descricao: "Descrição breve"
  user_stories:
    - id: "US-001"
      como: "tipo de usuário"
      quero: "ação que quer fazer"
      para_que: "benefício/motivo"
      prioridade: "must|should|could"
      criterios_aceitacao:
        - "critério 1"
        - "critério 2"
        - "critério 3"
      notas_tecnicas:
        - "nota 1"
      dependencias:
        - "US-XXX"
```

---

## Epics Comuns (Sugestões)

Se o usuário precisar de ajuda, sugira baseado no tipo de projeto:

### Para Sistema de Agendamento
1. Gestão de Agenda
2. Cadastro de Clientes
3. Notificações
4. Pagamentos
5. Relatórios

### Para E-commerce
1. Catálogo de Produtos
2. Carrinho de Compras
3. Checkout e Pagamento
4. Gestão de Pedidos
5. Conta do Cliente

### Para CRM
1. Gestão de Contatos
2. Pipeline de Vendas
3. Tarefas e Atividades
4. Relatórios
5. Integrações

### Para Sistema Interno
1. Autenticação
2. Gestão de Usuários
3. [Funcionalidade Principal]
4. Relatórios
5. Configurações

---

## Exemplo Completo de Epic

```yaml
epic:
  nome: "Gestão de Agenda"
  descricao: "Permite visualizar, criar e gerenciar agendamentos"
  user_stories:
    - id: "US-001"
      como: "cliente"
      quero: "ver horários disponíveis para agendamento"
      para_que: "eu possa escolher o melhor horário para mim"
      prioridade: "must"
      criterios_aceitacao:
        - "Sistema mostra apenas horários futuros"
        - "Horários já agendados não aparecem"
        - "Horários bloqueados não aparecem"
        - "Posso filtrar por profissional"
        - "Funciona no celular"
      notas_tecnicas:
        - "Cache de 5 minutos para disponibilidade"
      dependencias: []

    - id: "US-002"
      como: "cliente"
      quero: "agendar um horário"
      para_que: "garantir meu atendimento"
      prioridade: "must"
      criterios_aceitacao:
        - "Posso selecionar data, horário e serviço"
        - "Recebo confirmação na tela"
        - "Recebo confirmação por email/WhatsApp"
        - "Agendamento aparece para o profissional"
      notas_tecnicas:
        - "Implementar lock otimista para evitar double booking"
      dependencias:
        - "US-001"
```

---

## Resumo da Etapa

```markdown
"Ótimo! Documentei [N] Epics com [M] User Stories:

**Epic 1: [Nome]**
- US-001: [resumo]
- US-002: [resumo]

**Epic 2: [Nome]**
- US-003: [resumo]
- US-004: [resumo]

[continuar para outros epics]

Total: [X] funcionalidades essenciais, [Y] importantes, [Z] desejáveis

Está correto? Posso seguir para a última etapa (Técnico)?"
```

---

## Dicas para a IA

### Se Usuário For Muito Vago

```markdown
"Deixa eu ajudar! Para [funcionalidade], as ações comuns são:

1. [Ação 1] - por exemplo: criar novo registro
2. [Ação 2] - por exemplo: editar existente
3. [Ação 3] - por exemplo: visualizar lista
4. [Ação 4] - por exemplo: excluir

Quais dessas você precisa? Tem outras?"
```

### Se Tiver Muitas User Stories

```markdown
"Temos bastante coisa! Vamos focar nas essenciais primeiro.

Das [N] funcionalidades que listamos, quais são as TOP 5
que PRECISAM estar na primeira versão?"
```

### Quantidade Ideal

- MVP: 15-25 User Stories
- Cada Epic: 3-6 User Stories
- Total Epics: 4-8 para MVP

---

**Próxima Etapa:** `05-TECNICO.md`
