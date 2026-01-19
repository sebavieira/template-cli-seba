# Guia de Início Rápido

> Para usuários não-técnicos que querem criar um novo projeto

---

## Você NÃO precisa saber programação!

Este template foi criado para que **qualquer pessoa** possa iniciar um projeto de software profissional.

A IA (Claude) vai:
1. Fazer perguntas sobre seu projeto
2. Criar toda a documentação
3. Implementar o código
4. Testar tudo automaticamente

**Você só precisa responder perguntas!**

---

## Como Começar (5 minutos)

### Passo 1: Copie a Pasta

Copie esta pasta `template/` para onde você quer criar seu projeto.

Renomeie para o nome do seu projeto (ex: `meu-app/`)

### Passo 2: Abra o Claude Code

Abra o terminal na pasta do seu projeto e digite:

```
claude
```

### Passo 3: Inicie o Briefing

Digite:

```
briefing
```

### Passo 4: Responda as Perguntas

A IA vai perguntar sobre:

1. **O que é seu projeto?**
   - Qual problema resolve?
   - Para quem é?

2. **Quem vai usar?**
   - Tipos de usuários
   - O que eles precisam?

3. **O que o sistema faz?**
   - Funcionalidades principais
   - O que NÃO faz (importante!)

4. **Detalhes técnicos** (a IA sugere se você não souber)
   - Tecnologias
   - Integrações

### Passo 5: A IA Cria Tudo

Após responder, a IA vai:
- Criar documentação completa
- Organizar em pastas
- Preparar para implementação

### Passo 6: Comece a Construir

Digite apenas:

```
continue
```

A IA implementa funcionalidade por funcionalidade!

---

## Comandos que Você Vai Usar

| Você digita | O que acontece |
|-------------|----------------|
| `continue` | IA implementa próxima funcionalidade |
| `status` | Mostra progresso (quantos % pronto) |
| `teste` | Verifica se tudo está funcionando |
| `explica` | IA explica o que fez em linguagem simples |
| `pausa` | Salva progresso para continuar depois |
| `o que falta?` | Lista o que ainda precisa fazer |

---

## Dicas Importantes

### Seja Específico nas Respostas

**Ruim:** "Quero um app de vendas"

**Bom:** "Quero um sistema para minha loja de roupas onde clientes podem ver produtos, adicionar no carrinho e comprar com cartão ou pix"

### Não Tenha Medo de Errar

- Você pode voltar e corrigir
- A IA pergunta se não entender
- O briefing pode ser refeito

### Peça Exemplos

Se não entender uma pergunta, diga:
- "Me dá um exemplo"
- "Não entendi, pode explicar?"
- "O que você sugere?"

---

## Exemplo de Sessão

```
Você: briefing

IA: Vamos começar! Qual é o nome do seu projeto?

Você: Sistema de Agendamento para Barbearia

IA: Ótimo! Agora me conta: qual problema esse sistema resolve?

Você: Hoje meus clientes ligam para agendar e eu perco muito tempo no telefone.
      Quero que eles agendam sozinhos pelo celular.

IA: Entendi! Quem são os usuários do sistema?

Você: Os clientes que querem agendar e eu (dono) para ver a agenda

IA: Perfeito! O que os clientes precisam fazer no sistema?
...
```

---

## Depois do Briefing

A pasta do seu projeto vai ficar assim:

```
meu-projeto/
├── docs/           ← Documentação (criada pela IA)
│   ├── prd/       ← Requisitos do produto
│   └── spec/      ← Especificações técnicas
├── backend/        ← Código do servidor (criado depois)
├── frontend/       ← Interface visual (criada depois)
└── STATUS.md       ← Progresso do projeto
```

**Você não precisa entender esses arquivos!**

A IA usa eles para saber o que fazer.

---

## Precisa de Ajuda?

Digite para a IA:
- "não entendi"
- "me explica de novo"
- "o que significa [termo]?"

A IA está aqui para ajudar!

---

**Boa sorte com seu projeto!**
