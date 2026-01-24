# Evolution API v2 - Estrutura de Payloads de Mídia

Documentação de referência para payloads de webhook da Evolution API v2, incluindo imagens, vídeos, documentos, áudios e mensagens com resposta/comentário.

---

## 1. Tipos de Mídia Suportados

| Tipo | Campo no Payload | Campos Principais |
|------|------------------|-------------------|
| **Imagem** | `imageMessage` | `mimetype`, `caption`, `url`, `base64` |
| **Vídeo** | `videoMessage` | `mimetype`, `caption`, `url`, `base64` |
| **Documento** | `documentMessage` | `mimetype`, `caption`, `url`, `base64`, `fileName`, `title` |
| **Áudio** | `audioMessage` | `mimetype`, `url`, `base64` |
| **Áudio Gravado (PTT)** | `pttMessage` | `mimetype`, `url`, `base64` |

---

## 2. Estrutura de Payload por Tipo de Mídia

### 2.1 Imagem (imageMessage)

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5XXXXXXXXXX"
    },
    "pushName": "Nome do Contato",
    "message": {
      "imageMessage": {
        "mimetype": "image/jpeg",
        "caption": "Legenda opcional da imagem",
        "url": "https://mmg.whatsapp.net/...",
        "base64": "data:image/jpeg;base64,/9j/4AAQ..."
      }
    },
    "messageTimestamp": 1706123456
  }
}
```

**Campos importantes:**
- `mimetype`: Tipo do arquivo (image/jpeg, image/png, image/webp, etc.)
- `caption`: Texto/legenda enviada junto com a imagem (opcional)
- `url`: URL temporária do WhatsApp para download
- `base64`: Alternativa ao URL, dados em base64

---

### 2.2 Vídeo (videoMessage)

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5XXXXXXXXXX"
    },
    "pushName": "Nome do Contato",
    "message": {
      "videoMessage": {
        "mimetype": "video/mp4",
        "caption": "Legenda do vídeo",
        "url": "https://mmg.whatsapp.net/...",
        "base64": "data:video/mp4;base64,AAAAIG...",
        "seconds": 15,
        "gifPlayback": false
      }
    },
    "messageTimestamp": 1706123456
  }
}
```

**Campos importantes:**
- `mimetype`: Tipo do arquivo (video/mp4, video/3gpp, etc.)
- `caption`: Texto/legenda do vídeo (opcional)
- `seconds`: Duração do vídeo em segundos
- `gifPlayback`: Se true, é um GIF animado

---

### 2.3 Documento (documentMessage)

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5XXXXXXXXXX"
    },
    "pushName": "Nome do Contato",
    "message": {
      "documentMessage": {
        "mimetype": "application/pdf",
        "caption": "Segue o contrato para análise",
        "fileName": "contrato-2024.pdf",
        "title": "Contrato de Prestação de Serviços",
        "url": "https://mmg.whatsapp.net/...",
        "base64": "data:application/pdf;base64,JVBERi0..."
      }
    },
    "messageTimestamp": 1706123456
  }
}
```

**Campos importantes:**
- `mimetype`: Tipo do arquivo (application/pdf, application/vnd.ms-excel, etc.)
- `caption`: Descrição do documento (opcional)
- `fileName`: Nome original do arquivo
- `title`: Título do documento (opcional)

---

### 2.4 Áudio (audioMessage)

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5XXXXXXXXXX"
    },
    "pushName": "Nome do Contato",
    "message": {
      "audioMessage": {
        "mimetype": "audio/ogg; codecs=opus",
        "url": "https://mmg.whatsapp.net/...",
        "base64": "data:audio/ogg;base64,T2dnUw...",
        "seconds": 8,
        "ptt": true
      }
    },
    "messageTimestamp": 1706123456
  }
}
```

**Campos importantes:**
- `mimetype`: Normalmente "audio/ogg; codecs=opus" para áudios do WhatsApp
- `seconds`: Duração do áudio em segundos
- `ptt`: Se true, é uma mensagem de voz (Push-To-Talk), se false é arquivo de áudio

---

## 3. Mensagens com Resposta/Comentário (contextInfo)

Quando alguém responde ou comenta uma mensagem, o payload inclui um objeto `contextInfo` que contém informações sobre a mensagem original.

### 3.1 Resposta de Texto a uma Imagem

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5YYYYYYYYYY"
    },
    "pushName": "Nome do Contato",
    "message": {
      "extendedTextMessage": {
        "text": "Que foto bonita! Onde foi tirada?",
        "contextInfo": {
          "stanzaId": "BAE5XXXXXXXXXX",
          "participant": "5511999999999@s.whatsapp.net",
          "quotedMessage": {
            "imageMessage": {
              "mimetype": "image/jpeg",
              "caption": "Foto da viagem"
            }
          }
        }
      }
    },
    "messageTimestamp": 1706123789
  }
}
```

### 3.2 Resposta de Imagem a uma Mensagem de Texto

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5ZZZZZZZZZZ"
    },
    "pushName": "Nome do Contato",
    "message": {
      "imageMessage": {
        "mimetype": "image/jpeg",
        "caption": "Olha essa foto também!",
        "url": "https://mmg.whatsapp.net/...",
        "contextInfo": {
          "stanzaId": "BAE5XXXXXXXXXX",
          "participant": "5511888888888@s.whatsapp.net",
          "quotedMessage": {
            "conversation": "Me manda mais fotos da viagem"
          }
        }
      }
    },
    "messageTimestamp": 1706124000
  }
}
```

### 3.3 Resposta de Vídeo a um Documento

```json
{
  "event": "messages.upsert",
  "instance": "minha-instancia",
  "data": {
    "key": {
      "remoteJid": "5511999999999@s.whatsapp.net",
      "fromMe": false,
      "id": "BAE5WWWWWWWWWW"
    },
    "pushName": "Nome do Contato",
    "message": {
      "videoMessage": {
        "mimetype": "video/mp4",
        "caption": "Vídeo explicando o documento",
        "url": "https://mmg.whatsapp.net/...",
        "contextInfo": {
          "stanzaId": "BAE5XXXXXXXXXX",
          "quotedMessage": {
            "documentMessage": {
              "mimetype": "application/pdf",
              "fileName": "proposta.pdf",
              "caption": "Proposta comercial"
            }
          }
        }
      }
    },
    "messageTimestamp": 1706124500
  }
}
```

---

## 4. Campos do contextInfo

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `stanzaId` | string | ID único da mensagem original sendo respondida |
| `participant` | string | JID do autor da mensagem original |
| `quotedMessage` | object | Cópia da mensagem original |

### 4.1 Tipos de quotedMessage

| Tipo Original | Campo em quotedMessage |
|---------------|------------------------|
| Texto simples | `conversation` |
| Texto formatado | `extendedTextMessage.text` |
| Imagem | `imageMessage.caption` |
| Vídeo | `videoMessage.caption` |
| Documento | `documentMessage.caption` + `documentMessage.fileName` |
| Áudio | `audioMessage` (sem texto) |

---

## 5. Identificadores de Tipo de Mensagem (messageType)

O campo `messageType` pode ter os seguintes valores:

### Imagens
- `imageMessage`
- `image`
- `photo`
- `picture`

### Vídeos
- `videoMessage`
- `video`
- `gif`
- `gifMessage`

### Documentos
- `documentMessage`
- `document`
- `file`

### Áudios
- `audioMessage`
- `pttMessage`
- `audio`
- `ptt`
- `voice`
- `voice_message`

---

## 6. Detecção de Tipo por MIME

| Prefixo MIME | Tipo Detectado |
|--------------|----------------|
| `image/*` | `image` |
| `video/*` | `video` |
| `audio/*` | `audio` |
| Outros | `document` |

### Exemplos de MIME Types Comuns

| Extensão | MIME Type |
|----------|-----------|
| .jpg, .jpeg | `image/jpeg` |
| .png | `image/png` |
| .webp | `image/webp` |
| .gif | `image/gif` |
| .mp4 | `video/mp4` |
| .3gp | `video/3gpp` |
| .pdf | `application/pdf` |
| .doc | `application/msword` |
| .docx | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` |
| .xls | `application/vnd.ms-excel` |
| .xlsx | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` |
| .ogg | `audio/ogg; codecs=opus` |
| .mp3 | `audio/mpeg` |

---

## 7. Locais de Extração de Mídia (Prioridade)

O sistema busca URL/base64 da mídia nestes locais, em ordem:

1. `message.[tipo]Message.url`
2. `message.[tipo]Message.base64`
3. `message.[tipo]Message.mediaUrl`
4. `data.mediaUrl`
5. `data.base64`
6. `base64` (raiz do payload)

---

## 8. Estrutura Normalizada do Projeto

O parser do projeto (`evolution.parser.ts`) normaliza todos os payloads para esta estrutura:

```typescript
interface NormalizedWebhookEvent {
  // Identificação
  eventType: 'message.received' | 'message.sent' | 'message.updated' | 'message.reaction';
  instanceName: string;
  fromMe: boolean;

  // Contato
  contactNumber?: string;
  contactPhone?: string;
  contactLid?: string;
  contactJid?: string;
  contactName?: string;

  // Mensagem
  messageText?: string;        // Texto ou legenda da mídia
  messageKeyId?: string;       // ID único da mensagem
  messageStatus?: 'RECEIVED' | 'SENT' | 'DELIVERED' | 'READ' | 'FAILED' | 'PENDING';

  // Resposta/Comentário
  replyToMessageId?: string;   // stanzaId da mensagem original
  replyToMessageText?: string; // Texto da mensagem original

  // Mídia Genérica
  mediaType?: 'image' | 'video' | 'audio' | 'document' | 'ptt';
  mediaMimeType?: string;
  mediaUrl?: string;
  mediaFileName?: string;      // Apenas para documentos

  // Áudio Específico
  hasAudio?: boolean;
  audioMimeType?: string;
  audioUrl?: string;

  // Reação
  reactionMessageId?: string;
  reactionEmoji?: string;
  reactionTimestamp?: number;

  // Metadados
  timestamp: number;
  raw: unknown;                // Payload original completo
}
```

---

## 9. Arquivos de Referência no Projeto

| Arquivo | Descrição |
|---------|-----------|
| `server/src/shared/adapters/evolution/evolution.parser.ts` | Parser principal de webhooks |
| `server/src/shared/adapters/helpers/message-extractor.ts` | Utilitários de extração de mídia |
| `server/src/shared/adapters/types.ts` | Tipos TypeScript |

---

## 10. Notas Importantes

1. **URLs Temporárias**: As URLs do WhatsApp (`mmg.whatsapp.net`) são temporárias e expiram após algumas horas.

2. **Base64 vs URL**: Alguns provedores enviam base64 diretamente, outros apenas URL. O sistema deve suportar ambos.

3. **contextInfo em qualquer mídia**: O `contextInfo` pode aparecer em qualquer tipo de mensagem (texto, imagem, vídeo, documento, áudio) quando é uma resposta.

4. **stanzaId para vincular respostas**: Use o `stanzaId` para identificar qual mensagem está sendo respondida.

5. **Caption opcional**: Nem toda mídia tem legenda. Imagens, vídeos e documentos podem ou não ter `caption`.

6. **PTT vs Audio**: `ptt: true` indica mensagem de voz gravada, `ptt: false` indica arquivo de áudio enviado.

---

*Última atualização: Janeiro 2026*
*Baseado na Evolution API v2 e código do projeto*
