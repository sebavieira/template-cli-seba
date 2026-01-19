# Etapa 5: Decisões Técnicas

> Perguntas sobre tecnologia, integrações e infraestrutura

**Tempo Estimado:** 5-10 minutos

---

## Introdução

```markdown
"Agora vamos falar da parte técnica.

NÃO SE PREOCUPE se você não entende de tecnologia!
Posso sugerir as melhores opções para o seu caso.

Vou perguntar algumas coisas e dar sugestões."
```

---

## Sequência de Perguntas

### 5.1 Preferência de Tecnologia

```markdown
Pergunta:
"Você tem preferência por alguma tecnologia específica?

Por exemplo:
- Linguagem de programação
- Framework
- Banco de dados

(Se não souber, eu sugiro baseado no projeto)"

Opções comuns:
- "Não tenho preferência, sugere o melhor"
- "Quero usar [tecnologia X] porque [motivo]"
- "Precisa ser compatível com [sistema existente]"

Captura: tecnico.preferencia
```

### 5.2 Stack Sugerido (Se Sem Preferência)

```markdown
Se usuário não tiver preferência:

"Baseado no seu projeto, sugiro:

**Backend (servidor):**
- Node.js + TypeScript (moderno, rápido, fácil de manter)

**Frontend (interface):**
- React + Next.js (moderno, boa experiência do usuário)

**Banco de Dados:**
- PostgreSQL (robusto, gratuito, escalável)

**Filas/Tarefas:**
- Redis + BullMQ (para processos em segundo plano)

Essa stack é moderna, bem documentada e tem muito suporte.

Concorda ou prefere algo diferente?"

Captura: tecnico.stack
```

### 5.3 Integrações Necessárias

```markdown
Pergunta:
"O sistema precisa se conectar com outros serviços?

Por exemplo:
- WhatsApp (enviar mensagens)
- Email (SendGrid, Gmail)
- Pagamentos (Stripe, PagSeguro)
- Calendário (Google Calendar)
- Outros sistemas que você já usa"

Captura: tecnico.integracoes
```

### Para Cada Integração:

```markdown
"Para a integração com [SERVIÇO]:

- Você já tem conta nesse serviço?
- Sabe o custo mensal aproximado?
- Já usa em outro sistema?

(Se não souber, pesquiso para você)"

Captura: tecnico.integracoes[n].detalhes
```

### 5.4 Requisitos de Infraestrutura

```markdown
Pergunta:
"Onde você quer hospedar o sistema?

Opções:
1. **Na nuvem (recomendado)** - mais fácil, paga pelo uso
2. **Servidor próprio** - você já tem infraestrutura
3. **Não sei** - posso sugerir

Se for na nuvem, tem preferência? (AWS, Google, Digital Ocean, Vercel...)"

Captura: tecnico.infraestrutura
```

### 5.5 Estimativa de Uso

```markdown
Pergunta:
"Para dimensionar o sistema, me conta:

- Quantos usuários vão acessar por dia?
- Quantas operações/transações por dia?
- Vai crescer muito rápido?"

Exemplos:
- "Uns 50 usuários, 100 agendamentos por dia"
- "Começamos pequeno, mas queremos escalar"
- "Máximo 10 usuários, uso interno"

Captura: tecnico.escala
```

### 5.6 Orçamento para Serviços

```markdown
Pergunta:
"Você tem um orçamento para serviços mensais?

Isso inclui:
- Hospedagem (servidor)
- Integrações (APIs pagas)
- Banco de dados
- Email/SMS

Faixas comuns:
- Mínimo: R$ 50-100/mês
- Padrão: R$ 100-300/mês
- Robusto: R$ 300-500/mês"

Captura: tecnico.orcamento
```

---

## Stacks Pré-Definidos

Se o usuário não tiver preferência, sugira baseado no tipo de projeto:

### Stack Simples (Projetos Pequenos)
```yaml
stack_simples:
  backend: "Node.js + Express + TypeScript"
  frontend: "React + Next.js"
  database: "PostgreSQL"
  hosting: "Vercel + Railway"
  custo: "R$ 50-100/mês"
```

### Stack Padrão (Maioria dos Projetos)
```yaml
stack_padrao:
  backend: "Node.js + Fastify + TypeScript"
  frontend: "React + Next.js + TailwindCSS"
  database: "PostgreSQL"
  cache: "Redis"
  queue: "BullMQ"
  hosting: "AWS/GCP/Digital Ocean"
  custo: "R$ 100-300/mês"
```

### Stack Robusto (Projetos Grandes)
```yaml
stack_robusto:
  backend: "Node.js + Fastify + TypeScript"
  frontend: "React + Next.js + TailwindCSS"
  database: "PostgreSQL + Read Replicas"
  cache: "Redis Cluster"
  queue: "BullMQ"
  search: "Elasticsearch"
  hosting: "Kubernetes (AWS EKS/GCP GKE)"
  custo: "R$ 500+/mês"
```

---

## Resumo da Etapa

```markdown
"Perfeito! Decisões técnicas definidas:

**Stack:**
- Backend: [tecnologia]
- Frontend: [tecnologia]
- Banco de Dados: [tecnologia]
- Outros: [lista]

**Integrações:**
- [Serviço 1]: [status/custo]
- [Serviço 2]: [status/custo]

**Infraestrutura:**
- Hospedagem: [onde]
- Escala esperada: [descrição]
- Custo estimado: R$ [valor]/mês

Está correto? Vamos para a revisão final?"
```

---

## Dados Capturados

```yaml
tecnico:
  stack:
    backend: ""
    frontend: ""
    database: ""
    cache: ""
    queue: ""
  integracoes:
    - nome: ""
      tipo: ""
      tem_conta: boolean
      custo_mensal: ""
  infraestrutura:
    hospedagem: ""
    ambiente: ""
    custo_estimado: ""
  escala:
    usuarios_dia: number
    operacoes_dia: number
    crescimento: ""
  orcamento_mensal: ""
```

---

## Dicas para a IA

### Se Usuário Não Entender Tecnologia

```markdown
"Não se preocupe com os nomes técnicos!

Vou escolher as melhores opções para você.
O importante é que vão funcionar bem e ter bom custo-benefício.

Posso seguir com as sugestões padrão?"
```

### Se Tiver Restrição de Orçamento

```markdown
"Entendi que o orçamento é limitado.

Vou priorizar opções gratuitas ou de baixo custo:
- Vercel (frontend grátis)
- Railway/Render (backend barato)
- PostgreSQL gratuito até certo limite

Conseguimos um sistema funcional por menos de R$ 50/mês!"
```

### Se Precisar de Integração Cara

```markdown
"Essa integração com [SERVIÇO] tem custo de [VALOR].

Alternativas mais baratas:
1. [Alternativa 1] - [custo]
2. [Alternativa 2] - [custo]
3. Implementar depois, quando tiver mais receita

O que prefere?"
```

---

**Próxima Etapa:** `06-REVISAO.md`
