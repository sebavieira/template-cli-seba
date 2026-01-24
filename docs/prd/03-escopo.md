# 3. Escopo do Produto

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md) | [Anterior: Contexto](02-contexto-personas.md) | [Próximo: User Stories →](04-user-stories/README.md)

---

## 3.1 No Escopo (MVP)

Funcionalidades que **SERÃO** implementadas na primeira versão:

### Funcionalidades Essenciais (Must Have)

1. **Integração WhatsApp via Evolution API**
   - Conexão de múltiplas instâncias WhatsApp por empresa
   - Envio e recebimento de mensagens em tempo real
   - Suporte a texto, imagem, áudio, documento e vídeo
   - QR Code para conexão de novos números

2. **CRM de Contatos**
   - Cadastro automático de contatos a partir de conversas
   - Perfil completo com nome, telefone, email, empresa
   - Sistema de tags para organização
   - Notas e histórico de interações
   - Bloqueio de contatos indesejados

3. **Funis de Vendas (Kanban)**
   - Criação de múltiplos funis por empresa
   - Estágios customizáveis com cores
   - Drag-and-drop de contatos entre estágios
   - Marcação de estágios como "Ganho" ou "Perdido"
   - Limite de WIP (Work in Progress) por estágio

4. **Chatbot com Inteligência Artificial**
   - Respostas automáticas usando OpenAI GPT-4
   - Prompt de sistema personalizável por empresa
   - Contexto de conversa para respostas coerentes
   - Níveis de autonomia configuráveis (básico, híbrido, completo)
   - Ações autônomas: adicionar tags, mover estágios

5. **Dashboard de Métricas**
   - Contagem de mensagens enviadas/recebidas
   - Total de contatos e conversas ativas
   - Atividades recentes
   - Conversas aguardando resposta

6. **Autenticação e Multi-tenancy**
   - Login com email/senha
   - Múltiplas empresas isoladas
   - Roles: Super Admin, Admin, Usuário
   - Convite de usuários para equipe

### Funcionalidades Importantes (Should Have)

7. **Respostas Rápidas (Quick Replies)**
   - Templates de resposta com atalhos
   - Biblioteca de respostas por empresa/usuário
   - Inserção rápida durante conversa

8. **Sistema de Prompts com Onboarding**
   - Wizard guiado para configurar IA
   - Perguntas sobre tom, personalidade, restrições
   - Geração automática de prompt otimizado

---

## 3.2 Fora do Escopo (MVP)

Funcionalidades que **NÃO SERÃO** implementadas no MVP:

- ❌ Integração com Instagram, Facebook Messenger, Telegram
- ❌ Aplicativo mobile nativo (iOS/Android)
- ❌ Relatórios exportáveis em PDF/Excel
- ❌ Integração com ERPs (SAP, TOTVS, etc.)
- ❌ Transcrição automática de áudios
- ❌ Suporte a múltiplos idiomas na interface
- ❌ Integração com calendário (Google Calendar, Outlook)
- ❌ Pagamentos integrados (links de pagamento no chat)
- ❌ Marketplace de templates de prompts
- ❌ White-label para revenda

---

## 3.3 Roadmap Futuro

### V1.1 (Após MVP)
- 📋 **Deals (Negócios)**: Gerenciamento de oportunidades de venda com valor, probabilidade e tracking
- 📋 **Follow-ups Automáticos**: Regras condicionais para envio de mensagens de acompanhamento
- 📋 **Análise de Sentimento**: Detecção de sentimento da conversa em tempo real
- 📋 **Notificações de Equipe**: Alertas para eventos importantes (lead quente, sentiment negativo)
- 📋 **Editor Rico para Notas**: Markdown, formatação, anexos em notas de deals

### V2.0 (Longo Prazo)
- 📋 **Flow Builder Visual**: Editor drag-and-drop para automações complexas
- 📋 **API Pública**: REST API para integrações com sistemas externos
- 📋 **Webhooks Customizáveis**: Eventos para integração com outras ferramentas
- 📋 **Relatórios Avançados**: Analytics com gráficos, exportação, agendamento
- 📋 **App Mobile**: Aplicativo para iOS e Android para gestores
- 📋 **Integração UAZAPI**: Suporte a segundo provedor de WhatsApp

---

## 3.4 Premissas

Condições que assumimos como verdadeiras:

1. **Usuários têm conta WhatsApp Business**
   - Impacto se falsa: Precisaremos de guia de migração de conta pessoal

2. **Empresas usam WhatsApp como canal principal de vendas**
   - Impacto se falsa: Funcionalidades multicanal teriam maior prioridade

3. **Usuários aceitam IA respondendo em seu nome**
   - Impacto se falsa: Modo de sugestão seria mais utilizado que automação

4. **Conexão de internet estável nos dispositivos**
   - Impacto se falsa: Necessidade de modo offline/sincronização

---

## 3.5 Restrições

Limitações conhecidas do projeto:

### Restrições Técnicas
- WhatsApp não oferece API oficial gratuita - dependemos de provedores terceiros (Evolution API, UAZAPI)
- Limite de tokens da OpenAI por requisição (contexto máximo de ~128K tokens)
- WebSocket requer conexão ativa para atualizações em tempo real

### Restrições de Negócio
- Orçamento inicial limitado para infraestrutura cloud
- Equipe de desenvolvimento enxuta (foco em features core)
- Modelo de precificação baseado em volume de mensagens/tokens

### Restrições Legais/Regulatórias
- Conformidade com LGPD para dados de contatos
- Termos de uso do WhatsApp (proibição de spam, mensagens em massa não solicitadas)
- Políticas de uso da OpenAI (conteúdo gerado)

---

## 3.6 Dependências Externas

| Dependência | Tipo | Status | Responsável |
|-------------|------|--------|-------------|
| Evolution API | Integração WhatsApp | ✅ Integrado | Equipe Evolution |
| UAZAPI | Integração WhatsApp (backup) | ✅ Integrado | Equipe UAZAPI |
| OpenAI API | IA Conversacional | ✅ Integrado | OpenAI |
| PostgreSQL | Banco de Dados | ✅ Configurado | Infraestrutura |
| Redis | Cache e Filas | ✅ Configurado | Infraestrutura |
| AWS S3 / Cloudflare R2 | Armazenamento de Mídia | ✅ Configurado | Infraestrutura |

---

[← Voltar para Índice PRD](README.md) | [Próximo: User Stories →](04-user-stories/README.md)
