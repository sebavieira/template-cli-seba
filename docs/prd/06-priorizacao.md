# 6. Priorização de Features (MoSCoW)

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## Visão Geral

Este documento classifica todas as funcionalidades usando o framework **MoSCoW**:

- **M**ust Have - Obrigatório para MVP
- **S**hould Have - Importante, mas não bloqueante
- **C**ould Have - Desejável se houver tempo
- **W**on't Have - Fora do escopo atual

---

## 🔴 Must Have (MVP)

Funcionalidades **obrigatórias** para o lançamento. Sem elas, o produto não funciona.

| ID | Funcionalidade | Epic | Status |
|----|----------------|------|--------|
| US-01.1 | Registro de usuário | Autenticação | ✅ |
| US-01.2 | Login com email/senha | Autenticação | ✅ |
| US-01.3 | JWT com refresh token | Autenticação | ✅ |
| US-02.1 | Conexão de instância WhatsApp | WhatsApp | ✅ |
| US-02.3 | Recebimento de mensagens (webhook) | WhatsApp | ✅ |
| US-02.4 | Envio de mensagens | WhatsApp | ✅ |
| US-03.1 | Listagem de contatos | Contatos | ✅ |
| US-03.4 | Sistema de tags | Contatos | ✅ |
| US-04.1 | Criação de funil | Funis | ✅ |
| US-04.3 | Visualização Kanban | Funis | ✅ |
| US-05.1 | Configuração de prompt | IA | ✅ |
| US-05.4 | Respostas automáticas | IA | ✅ |
| US-08.1 | Dashboard principal | Dashboard | ✅ |

**Total Must Have:** 13 funcionalidades

---

## 🟡 Should Have

Funcionalidades **importantes** que agregam valor significativo, mas o sistema funciona sem elas.

| ID | Funcionalidade | Epic | Status |
|----|----------------|------|--------|
| US-01.6 | Gestão de equipe | Autenticação | ✅ |
| US-02.7 | Múltiplos provedores WhatsApp | WhatsApp | ✅ |
| US-03.5 | Notas em contatos | Contatos | ✅ |
| US-03.6 | Bloqueio de contatos | Contatos | ✅ |
| US-05.2 | Onboarding guiado | IA | ✅ |
| US-05.3 | Níveis de autonomia | IA | ✅ |
| US-05.6 | Análise de sentimento | IA | ✅ |
| US-06.1 | Criação de deals | Deals | ✅ |
| US-06.3 | Notas em deals | Deals | ✅ |
| US-07.1 | Regras de follow-up | Follow-ups | ✅ |
| US-08.4 | Notificações de equipe | Dashboard | ✅ |

**Total Should Have:** 11 funcionalidades

---

## 🟢 Could Have

Funcionalidades **desejáveis** que melhoram a experiência, implementadas se sobrar tempo.

| ID | Funcionalidade | Epic | Status |
|----|----------------|------|--------|
| US-04.5 | Limites de WIP | Funis | 📋 |
| US-06.5 | Anexos em deals | Deals | ✅ |
| US-06.6 | Métricas de deals | Deals | 📋 |
| US-07.5 | Post-actions em follow-ups | Follow-ups | ✅ |
| US-09.1 | Criação de flows | Flows | 🚧 |
| US-09.2 | Editor visual de flows | Flows | 🚧 |

**Total Could Have:** 6 funcionalidades

---

## ⚪ Won't Have (This Release)

Funcionalidades **fora do escopo** atual, mas podem ser consideradas no futuro.

| Funcionalidade | Motivo | Versão Futura |
|----------------|--------|---------------|
| Integração Instagram/Messenger | Foco em WhatsApp primeiro | V3.0 |
| App mobile nativo | Web responsivo atende | V2.5 |
| Relatórios PDF/Excel | Dashboard cobre necessidades | V2.0 |
| Integração ERPs | Complexidade alta | V3.0 |
| Transcrição de áudios | Custo adicional de API | V2.0 |
| Multi-idioma interface | Mercado BR prioritário | V2.5 |
| Integração calendário | Escopo diferente | V2.0 |
| Pagamentos no chat | E-commerce scope | V3.0 |
| White-label | Modelo de negócio diferente | V3.0 |

**Total Won't Have:** 9 funcionalidades

---

## Resumo da Priorização

| Categoria | Quantidade | Percentual |
|-----------|------------|------------|
| 🔴 Must Have | 13 | 43% |
| 🟡 Should Have | 11 | 37% |
| 🟢 Could Have | 6 | 20% |
| ⚪ Won't Have | 9 | - |
| **Total MVP** | **30** | **100%** |

---

## Critérios de Priorização

As funcionalidades foram priorizadas considerando:

1. **Valor para o Usuário** - Quanto resolve o problema da persona principal
2. **Viabilidade Técnica** - Complexidade de implementação
3. **Dependências** - Se bloqueia outras funcionalidades
4. **Risco** - Impacto se não for implementada
5. **Esforço** - Tempo e recursos necessários

### Matriz de Priorização

```
Alto Valor + Baixo Esforço  = Must Have (Fazer primeiro)
Alto Valor + Alto Esforço   = Should Have (Planejar bem)
Baixo Valor + Baixo Esforço = Could Have (Se der tempo)
Baixo Valor + Alto Esforço  = Won't Have (Não fazer agora)
```

---

## Sequência de Implementação

### Fase 1: Core (MVP)
1. Autenticação completa
2. Integração WhatsApp
3. CRM de Contatos básico
4. Funis Kanban

### Fase 2: IA (MVP)
5. Chatbot com OpenAI
6. Prompts customizáveis
7. Dashboard básico

### Fase 3: Melhorias (V1.1)
8. Deals e tracking
9. Follow-ups automáticos
10. Análise de sentimento
11. Notificações

### Fase 4: Avançado (V2.0)
12. Flow Builder
13. Relatórios avançados
14. API pública

---

[← Voltar para Índice PRD](README.md)
