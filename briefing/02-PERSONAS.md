# Etapa 2: Personas e Usuários

> Perguntas para identificar quem vai usar o sistema

**Tempo Estimado:** 10-15 minutos

---

## Introdução

```markdown
"Agora vamos falar sobre as PESSOAS que vão usar o sistema.

Pense em todos os tipos de usuários diferentes:
- Quem vai usar no dia a dia?
- Quem vai administrar?
- Quem são os clientes (se houver)?

Vou perguntar sobre cada tipo de usuário."
```

---

## Sequência de Perguntas

### 2.1 Identificar Tipos de Usuário

```markdown
Pergunta:
"Quem são os TIPOS de pessoas que vão usar o sistema?

Não precisa nome de pessoas reais - são perfis/papéis."

Exemplos:
- Dono da loja
- Funcionário
- Cliente final
- Gerente
- Administrador

Captura: lista de tipos de persona
```

### Para CADA Tipo de Usuário, Pergunte:

### 2.2 Descrição do Usuário

```markdown
Pergunta:
"Me conta mais sobre [TIPO DE USUÁRIO]:
- Quem é essa pessoa?
- Qual o trabalho/papel dela?
- Como é o dia a dia?"

Exemplo para "Dono de Barbearia":
"É o proprietário, trabalha das 9h às 20h, atende clientes
e também cuida da administração. Usa muito o WhatsApp."

Captura: personas[n].descricao
```

### 2.3 Dores e Problemas

```markdown
Pergunta:
"Quais são as principais DORES de [TIPO DE USUÁRIO]?

O que mais incomoda? O que toma tempo? O que é difícil?"

Exemplos:
- "Perde muito tempo respondendo WhatsApp"
- "Não consegue ver agenda facilmente"
- "Esquece de confirmar agendamentos"
- "Não sabe quanto faturou no mês"

Captura: personas[n].dores
```

### 2.4 Necessidades

```markdown
Pergunta:
"O que [TIPO DE USUÁRIO] PRECISA fazer no sistema?

Liste as ações principais."

Exemplos:
- Ver agenda do dia
- Cadastrar novo cliente
- Registrar pagamento
- Enviar mensagem para cliente

Captura: personas[n].necessidades
```

### 2.5 Comportamentos e Preferências

```markdown
Pergunta:
"Como [TIPO DE USUÁRIO] costuma trabalhar?

- Usa mais computador ou celular?
- Prefere algo simples ou completo?
- Tem familiaridade com tecnologia?"

Exemplos:
- "Usa mais o celular"
- "Precisa ser muito simples, não tem paciência"
- "Não entende muito de tecnologia"

Captura: personas[n].comportamentos
```

### 2.6 Citação Representativa

```markdown
Pergunta:
"Se [TIPO DE USUÁRIO] pudesse resumir o que precisa em uma frase,
o que diria?

Complete: 'Eu quero...'"

Exemplos:
- "Eu quero parar de perder tempo no telefone"
- "Eu quero ver minha agenda de forma fácil"
- "Eu quero poder agendar sem precisar ligar"

Captura: personas[n].citacao
```

---

## Repetir para Outras Personas

```markdown
"Ótimo! Documentei [PERSONA 1].

Você mencionou também [PERSONA 2].
Vamos falar sobre ela agora?"

[Repetir perguntas 2.2 a 2.6]
```

---

## Priorização de Personas

```markdown
Pergunta:
"De todos os usuários que conversamos, quem é o PRINCIPAL?

Ou seja, para quem o sistema é mais importante?"

Captura: persona_principal
```

---

## Resumo da Etapa

```markdown
"Perfeito! Identificamos [N] tipos de usuários:

**1. [Nome Persona 1]** (Principal)
- Descrição: [resumo]
- Principal dor: [dor mais importante]
- Principal necessidade: [necessidade mais importante]

**2. [Nome Persona 2]**
- Descrição: [resumo]
- Principal dor: [dor mais importante]
- Principal necessidade: [necessidade mais importante]

[Repetir para outras personas]

Está correto? Posso seguir para a próxima etapa (Escopo)?"
```

---

## Dados Capturados

```yaml
personas:
  - nome: ""
    tipo: "principal|secundario"
    descricao: ""
    dores:
      - ""
    necessidades:
      - ""
    comportamentos:
      - ""
    citacao: ""
```

---

## Dicas para a IA

### Se Usuário Listar Apenas 1 Tipo

```markdown
"Só [esse tipo] vai usar o sistema?

Às vezes esquecemos de alguns usuários. Por exemplo:
- Quem vai administrar?
- Tem clientes que vão acessar?
- Outros funcionários?"
```

### Se Usuário For Vago

```markdown
"Deixa eu dar um exemplo:

Para uma barbearia, os usuários seriam:
1. O Barbeiro (agenda atendimentos)
2. O Cliente (agenda pelo app)
3. O Dono (vê relatórios)

No seu caso, como seria?"
```

### Persona Mínima Viável

Se usuário tiver dificuldade, capture pelo menos:
- Quem é
- Principal dor
- Principal necessidade

---

**Próxima Etapa:** `03-ESCOPO.md`
