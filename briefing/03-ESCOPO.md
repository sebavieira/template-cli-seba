# Etapa 3: Escopo do Projeto

> Perguntas para definir o que o sistema FAZ e o que NÃO FAZ

**Tempo Estimado:** 5-10 minutos

---

## Introdução

```markdown
"Agora vamos definir os LIMITES do projeto.

Isso é muito importante para:
- Não criar expectativas erradas
- Focar no que é essencial primeiro
- Planejar o que vem depois

Vou perguntar o que ENTRA e o que NÃO ENTRA no sistema."
```

---

## Sequência de Perguntas

### 3.1 Funcionalidades Essenciais (Must Have)

```markdown
Pergunta:
"O que o sistema PRECISA fazer para ser útil?

Sem essas funcionalidades, não vale a pena.
Liste as funcionalidades essenciais."

Exemplos:
- "Precisa ter cadastro de clientes"
- "Precisa permitir agendamento"
- "Precisa mostrar relatório de vendas"
- "Precisa integrar com WhatsApp"

Captura: escopo.inclui (prioridade: must)
```

### 3.2 Funcionalidades Importantes (Should Have)

```markdown
Pergunta:
"O que seria BOM ter, mas o sistema ainda funciona sem isso?

São funcionalidades importantes, mas não bloqueiam o lançamento."

Exemplos:
- "Enviar lembretes automáticos"
- "Relatórios mais detalhados"
- "App para celular"
- "Múltiplos usuários"

Captura: escopo.inclui (prioridade: should)
```

### 3.3 Funcionalidades Desejáveis (Could Have)

```markdown
Pergunta:
"O que seria legal ter se der tempo?

Funcionalidades 'nice to have' - se sobrar tempo, implementamos."

Exemplos:
- "Personalização de cores"
- "Exportar para Excel"
- "Gráficos avançados"
- "Integração com Instagram"

Captura: escopo.inclui (prioridade: could)
```

### 3.4 O Que NÃO Faz

```markdown
Pergunta:
"O que o sistema NÃO vai fazer?

É importante definir limites claros.
O que você NÃO espera que o sistema faça?"

Exemplos:
- "Não vai processar pagamentos"
- "Não vai ter chat ao vivo"
- "Não vai funcionar offline"
- "Não vai ter app nativo (só web)"

Captura: escopo.nao_inclui
```

### 3.5 Futuro (Won't Have Now)

```markdown
Pergunta:
"O que você gostaria no FUTURO, mas não precisa agora?

Essas funcionalidades ficam para versões posteriores."

Exemplos:
- "Integração com sistema de contabilidade"
- "Múltiplas filiais"
- "Módulo de RH"
- "Versão para franquias"

Captura: escopo.futuro
```

### 3.6 Premissas

```markdown
Pergunta:
"Para o sistema funcionar, o que você ASSUME que já existe?

Por exemplo: internet, computador, conta em algum serviço..."

Exemplos:
- "Usuários têm smartphone com internet"
- "Empresa já tem email configurado"
- "Já existe conta no WhatsApp Business"
- "Já usa Google Agenda"

Captura: escopo.premissas
```

### 3.7 Restrições

```markdown
Pergunta:
"Tem alguma LIMITAÇÃO que eu preciso saber?

Pode ser orçamento, prazo, tecnologia obrigatória..."

Exemplos:
- "Orçamento máximo de R$ X"
- "Precisa funcionar no Windows 7"
- "Não pode usar serviço pago acima de R$ 100/mês"
- "Precisa estar pronto em 2 meses"

Captura: escopo.restricoes
```

---

## Resumo da Etapa

```markdown
"Excelente! Vamos recapitular o escopo:

**PRECISA TER (MVP):**
- [item 1]
- [item 2]
- [item 3]

**BOM TER (depois do MVP):**
- [item 1]
- [item 2]

**NÃO VAI TER:**
- [item 1]
- [item 2]

**FUTURO (V2+):**
- [item 1]
- [item 2]

**PREMISSAS:**
- [item 1]

**RESTRIÇÕES:**
- [item 1]

Está correto? Posso seguir para a próxima etapa (Funcionalidades Detalhadas)?"
```

---

## Dados Capturados

```yaml
escopo:
  inclui:
    must_have:
      - ""
    should_have:
      - ""
    could_have:
      - ""
  nao_inclui:
    - ""
  futuro:
    - ""
  premissas:
    - ""
  restricoes:
    - ""
```

---

## Dicas para a IA

### Se Usuário Quiser Tudo

```markdown
"Entendo que você quer bastante coisa!

Mas para lançar rápido, precisamos priorizar.
Se você pudesse escolher apenas 3 funcionalidades para ter PRIMEIRO,
quais seriam?"
```

### Se Usuário Não Souber o que Excluir

```markdown
"Deixa eu sugerir algumas coisas que geralmente ficam de fora no início:

- App nativo (usa web no celular)
- Múltiplos idiomas
- Integração com muitos sistemas
- Funcionalidades avançadas de relatório

Faz sentido deixar essas para depois?"
```

### Se Houver Contradição

```markdown
"Percebi que você quer [X], mas também disse que [Y].

Esses dois podem conflitar. Qual é mais importante?"
```

---

**Próxima Etapa:** `04-FUNCIONALIDADES.md`
