# 8. Compliance e Políticas

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## 8.1 LGPD (Lei Geral de Proteção de Dados)

### Princípios Aplicáveis

| Princípio | Implementação |
|-----------|---------------|
| **Finalidade** | Dados coletados apenas para fins específicos e declarados |
| **Adequação** | Tratamento compatível com finalidades informadas |
| **Necessidade** | Coleta limitada ao mínimo necessário |
| **Transparência** | Informações claras sobre tratamento de dados |
| **Segurança** | Medidas técnicas para proteger dados |

### Requisitos de Implementação

1. **Consentimento**
   - Opt-in explícito para coleta de dados
   - Granularidade por tipo de dado/finalidade
   - Registro de consentimento com timestamp

2. **Direito ao Acesso**
   - Endpoint para exportar dados do usuário
   - Formato legível (JSON ou CSV)
   - Prazo: 15 dias úteis

3. **Direito à Exclusão**
   - Mecanismo de solicitação de exclusão
   - Exclusão em cascata de dados relacionados
   - Prazo: 15 dias úteis

4. **Portabilidade**
   - Exportação em formato interoperável
   - Estrutura documentada

### Dados Coletados

| Dado | Finalidade | Base Legal | Retenção |
|------|------------|------------|----------|
| **Nome e Email** | Identificação e comunicação | Execução de contrato | Duração da conta |
| **Telefone (WhatsApp)** | Conexão de instância | Consentimento | Duração da instância |
| **Conversas WhatsApp** | CRM e histórico | Execução de contrato | Configurável (30-365 dias) |
| **Dados de Contatos** | Gestão de relacionamento | Legítimo interesse | Duração da conta |
| **Logs de Acesso** | Segurança e auditoria | Obrigação legal | 6 meses |
| **Métricas de Uso** | Melhoria do produto | Legítimo interesse | Anonimizado após 12 meses |

---

## 8.2 Políticas por Canal

### WhatsApp (Canal Principal)

**Compliance com Meta Business Policies:**

| Requisito | Implementação | Status |
|-----------|---------------|--------|
| Opt-in explícito | Contato deve iniciar conversa ou consentir | ✅ |
| Templates aprovados | Suporte a templates HSM | 📋 Planejado |
| Janela de 24h | Respostas apenas dentro da janela | ✅ |
| Bloqueio de spam | Rate limiting e monitoramento | ✅ |
| Identificação clara | Nome da empresa visível | ✅ |

**Limitações Técnicas:**

- Máximo de 1.000 mensagens/dia por instância (Evolution API)
- Templates obrigatórios para iniciar conversas (Business API)
- Janela de 24h para respostas após última mensagem do contato
- Proibido envio em massa sem consentimento

### Email (Notificações do Sistema)

- Compliance com CAN-SPAM / LGPD
- Opt-out em todas as mensagens
- Identificação clara do remetente
- Assunto não enganoso
- Usado apenas para notificações do sistema (não marketing)

---

## 8.3 Segurança de Dados

### Classificação de Dados

| Dado | Classificação | Proteção |
|------|---------------|----------|
| **Senhas** | Crítico | Hash bcrypt (10 rounds), nunca armazenado em texto |
| **JWT Tokens** | Crítico | Expiração curta (15min access, 7d refresh) |
| **API Keys WhatsApp** | Crítico | Criptografado em repouso, acesso restrito |
| **Conversas** | Alto | Isolamento multi-tenant, sem acesso entre empresas |
| **Dados de Contato** | Alto | Isolamento por empresa, backup criptografado |
| **Prompts IA** | Médio | Armazenado por empresa, sem compartilhamento |
| **Logs** | Médio | Retenção 6 meses, sem PII em logs públicos |

### Controles de Acesso

| Controle | Implementação |
|----------|---------------|
| Autenticação | JWT obrigatório para todas as rotas protegidas |
| Autorização | RBAC (owner, admin, member) por recurso |
| Isolamento | Multi-tenant com companyId em todas as queries |
| Audit Trail | Log de ações sensíveis (login, exclusão, alteração) |
| Sessões | Timeout configurável, invalidação remota |
| Rate Limiting | 100 requests/min por IP |

### Criptografia

| Camada | Tecnologia |
|--------|------------|
| **Em trânsito** | TLS 1.3 (HTTPS obrigatório) |
| **Em repouso** | PostgreSQL com encryption at rest |
| **Senhas** | bcrypt com salt único |
| **Tokens** | JWT com assinatura HS256/RS256 |

---

## 8.4 Termos e Políticas

### Documentos Necessários

- [x] Termos de Uso
- [x] Política de Privacidade
- [ ] Política de Cookies (se aplicável)
- [ ] SLA para planos pagos

### Conteúdo da Política de Privacidade

1. **Identificação do Controlador**
   - Nome da empresa
   - CNPJ
   - Endereço
   - Contato do DPO

2. **Dados Coletados**
   - Dados de cadastro
   - Dados de uso
   - Dados de terceiros (contatos WhatsApp)

3. **Finalidades do Tratamento**
   - Prestação do serviço
   - Comunicação
   - Melhoria do produto
   - Segurança

4. **Compartilhamento**
   - Provedores de infraestrutura
   - APIs de terceiros (OpenAI, Evolution)
   - Não venda de dados

5. **Direitos do Titular**
   - Acesso
   - Correção
   - Exclusão
   - Portabilidade
   - Revogação de consentimento

---

## 8.5 Checklist de Compliance

### LGPD

- [x] Política de privacidade redigida
- [x] Consentimento implementado no cadastro
- [x] Mecanismo de opt-out funcional
- [ ] Exportação de dados disponível (endpoint planejado)
- [x] Exclusão de dados implementada (soft delete + hard delete)
- [x] Logs de auditoria ativos
- [x] Isolamento multi-tenant

### Segurança

- [x] HTTPS obrigatório em produção
- [x] Senhas com hash bcrypt
- [x] JWT com expiração curta
- [x] Rate limiting configurado
- [x] Headers de segurança (Helmet)
- [x] Validação de input (Zod)
- [x] CORS configurado por ambiente
- [ ] Pentesting realizado

### WhatsApp

- [x] Conexão via QR Code (user-initiated)
- [x] Respeito à janela de 24h
- [x] Bloqueio manual de contatos
- [x] Opt-out funcional para contatos
- [ ] Templates HSM (planejado)

---

## 8.6 Incidentes de Segurança

### Protocolo de Resposta

1. **Detecção** (< 1h)
   - Monitoramento de logs
   - Alertas automáticos
   - Relatórios de usuários

2. **Contenção** (< 4h)
   - Isolamento do sistema afetado
   - Revogação de tokens comprometidos
   - Bloqueio de IPs suspeitos

3. **Investigação** (< 24h)
   - Análise de logs
   - Identificação da causa raiz
   - Avaliação de impacto

4. **Notificação** (< 72h)
   - Usuários afetados
   - ANPD (se dados pessoais)
   - Autoridades competentes

5. **Correção**
   - Patch de vulnerabilidade
   - Atualização de procedimentos
   - Documentação do incidente

### Contatos de Emergência

| Papel | Responsabilidade |
|-------|------------------|
| **Tech Lead** | Coordenação técnica |
| **DPO** | Comunicação regulatória |
| **Jurídico** | Avaliação legal |
| **Comunicação** | Notificação a usuários |

---

[← Voltar para Índice PRD](README.md)
