# Epic 02: Integração WhatsApp

**Versão:** 1.0.0
**Status:** ✅ Concluído
**Prioridade:** Must Have

[← Voltar para User Stories](README.md)

---

## Objetivo do Epic

Permitir que empresas conectem seus números WhatsApp à plataforma, recebam e enviem mensagens em tempo real através de provedores como Evolution API e UAZAPI.

---

## User Stories

### US-02.1: Conexão de Instância WhatsApp

**Como** um admin,
**Quero** conectar meu número WhatsApp à plataforma,
**Para que** eu possa receber mensagens dos clientes.

**Critérios de Aceitação:**
- [ ] Criação de nova instância via Evolution API ou UAZAPI
- [ ] Geração de QR Code para autenticação
- [ ] Polling de status de conexão
- [ ] Indicação visual de status (conectado/desconectado)
- [ ] Armazenamento seguro de credenciais

**Prioridade:** Must Have

---

### US-02.2: Listagem de Instâncias

**Como** um admin,
**Quero** ver todas as instâncias WhatsApp conectadas,
**Para que** eu gerencie meus números ativos.

**Critérios de Aceitação:**
- [ ] Lista de todas instâncias da empresa
- [ ] Status de conexão em tempo real
- [ ] Nome do número/dispositivo
- [ ] Data de última atividade
- [ ] Ações: reconectar, desconectar, remover

**Prioridade:** Must Have

---

### US-02.3: Recebimento de Mensagens (Webhook)

**Como** sistema,
**Quero** receber mensagens via webhook,
**Para que** conversas sejam registradas em tempo real.

**Critérios de Aceitação:**
- [ ] Endpoint de webhook público e autenticado
- [ ] Processamento de mensagens de texto
- [ ] Processamento de mensagens de mídia (imagem, áudio, vídeo, documento)
- [ ] Criação/atualização automática de contato
- [ ] Criação automática de conversa
- [ ] Emissão de evento WebSocket para frontend

**Prioridade:** Must Have

---

### US-02.4: Envio de Mensagens

**Como** um vendedor,
**Quero** enviar mensagens para contatos,
**Para que** eu me comunique com meus clientes.

**Critérios de Aceitação:**
- [ ] Envio de texto simples
- [ ] Envio de mídia com caption
- [ ] Seleção de instância de envio
- [ ] Registro da mensagem como SENT
- [ ] Atualização de status de entrega (se disponível)

**Prioridade:** Must Have

---

### US-02.5: Visualização de Conversa

**Como** um vendedor,
**Quero** ver o histórico de mensagens com um contato,
**Para que** eu tenha contexto da conversa.

**Critérios de Aceitação:**
- [ ] Lista de mensagens ordenadas por data
- [ ] Diferenciação visual de enviadas/recebidas
- [ ] Exibição de mídias inline
- [ ] Scroll infinito ou paginação
- [ ] Atualização em tempo real via WebSocket

**Prioridade:** Must Have

---

### US-02.6: Indicador de Mensagem Não Lida

**Como** um vendedor,
**Quero** ver quais conversas têm mensagens não lidas,
**Para que** eu priorize meu atendimento.

**Critérios de Aceitação:**
- [ ] Badge com contagem de não lidas na lista de conversas
- [ ] Marcar como lida ao abrir conversa
- [ ] Ordenação por mensagens não lidas

**Prioridade:** Should Have

---

### US-02.7: Suporte a Múltiplos Provedores

**Como** um admin,
**Quero** escolher entre Evolution API e UAZAPI,
**Para que** eu use o provedor que melhor me atende.

**Critérios de Aceitação:**
- [ ] Configuração de credenciais por provedor
- [ ] Adapter pattern para abstração
- [ ] Fallback automático em caso de falha (opcional)
- [ ] Métricas por provedor

**Prioridade:** Should Have

---

### US-02.8: Armazenamento de Mídia

**Como** sistema,
**Quero** armazenar mídias recebidas de forma segura,
**Para que** usuários possam visualizá-las posteriormente.

**Critérios de Aceitação:**
- [ ] Download e armazenamento de imagens, áudios, vídeos, documentos
- [ ] Suporte a múltiplos provedores de storage (local, S3, R2, MinIO)
- [ ] URLs assinadas para acesso seguro
- [ ] Limite de tamanho configurável
- [ ] Limpeza de arquivos antigos (opcional)

**Prioridade:** Must Have

---

## Modelo de Dados

```prisma
model WhatsAppProviderCredential {
  id        String            @id @default(cuid())
  companyId String
  provider  WhatsAppProvider
  apiUrl    String
  apiKey    String            // Criptografado
  isActive  Boolean           @default(true)

  company   Company           @relation(fields: [companyId], references: [id])
  instances WhatsAppInstance[]
}

model WhatsAppInstance {
  id           String   @id @default(cuid())
  credentialId String
  instanceName String
  phoneNumber  String?
  status       String   @default("disconnected")
  qrCode       String?
  lastSeen     DateTime?

  credential    WhatsAppProviderCredential @relation(...)
  conversations WhatsAppConversation[]
}

model WhatsAppConversation {
  id          String   @id @default(cuid())
  instanceId  String
  contactId   String
  lastMessage DateTime @default(now())
  unreadCount Int      @default(0)

  instance WhatsAppInstance @relation(...)
  contact  Contact          @relation(...)
  messages WhatsAppMessage[]
}

model WhatsAppMessage {
  id             String      @id @default(cuid())
  conversationId String
  messageType    MessageType
  content        String?
  mediaUrl       String?
  mediaType      String?
  timestamp      DateTime    @default(now())
  isFromAi       Boolean     @default(false)

  conversation WhatsAppConversation @relation(...)
}

enum WhatsAppProvider {
  EVOLUTION
  UAZAPI
}

enum MessageType {
  RECEIVED
  SENT
}
```

---

## Endpoints de API

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/whatsapp/webhook` | Webhook para mensagens |
| POST | `/api/whatsapp/instances` | Criar instância |
| GET | `/api/whatsapp/instances` | Listar instâncias |
| GET | `/api/whatsapp/instances/:id` | Detalhes da instância |
| POST | `/api/whatsapp/instances/:id/qr` | Gerar QR Code |
| DELETE | `/api/whatsapp/instances/:id` | Remover instância |
| POST | `/api/whatsapp/send` | Enviar mensagem |
| GET | `/api/messages` | Listar mensagens de conversa |

---

## Fluxo de Processamento de Mensagem

```
WhatsApp (Cliente)
    ↓
Evolution API / UAZAPI
    ↓
Webhook POST /api/whatsapp/webhook
    ↓
Validação de assinatura/origem
    ↓
Extração de dados (remetente, conteúdo, mídia)
    ↓
Upsert de Contact
    ↓
Upsert de Conversation
    ↓
Criação de WhatsAppMessage
    ↓
Download de mídia (se aplicável)
    ↓
Enfileiramento para processamento IA
    ↓
Emissão de evento WebSocket
    ↓
Atualização do frontend em tempo real
```

---

[← Voltar para User Stories](README.md)
