# 5. Requisitos Não Funcionais

**Versão:** 1.0.0
**Última Atualização:** 2026-01-19

[← Voltar para Índice PRD](README.md)

---

## RNF-01: Performance e Latência

**Requisito:** O sistema deve responder rapidamente às solicitações dos usuários.

**Critérios:**
- Tempo de resposta da API: **p95 < 200ms**
- Tempo de carregamento de página: **< 3 segundos**
- Operações em lote: **< 30 segundos**
- Processamento de mensagem IA: **< 10 segundos**

**Medição:**
- Logs estruturados com Pino
- Monitoramento de latência por endpoint
- Alertas para degradação de performance

**Status:** ✅ Implementado

---

## RNF-02: Escalabilidade

**Requisito:** O sistema deve suportar crescimento de usuários e dados.

**Critérios:**
- Suportar **500** usuários simultâneos por servidor
- Suportar **1 milhão** de mensagens no banco
- Suportar **50.000** contatos por empresa
- Horizontal scaling via Docker containers

**Estratégia:**
- Arquitetura stateless (exceto Redis para sessões)
- Banco de dados PostgreSQL com índices otimizados
- Redis para cache e filas (BullMQ)
- Workers background para processamento assíncrono

**Status:** ✅ Implementado

---

## RNF-03: Disponibilidade

**Requisito:** O sistema deve estar disponível para uso.

**Critérios:**
- Uptime: **99.5%** (meta inicial)
- Tempo máximo de downtime: **3.6 horas** por mês
- Recuperação de falhas: **< 15 minutos**

**Estratégia:**
- Health checks em `/health`
- Docker Compose com restart policies
- Logs centralizados para diagnóstico
- Procedures de rollback documentados

**Status:** ✅ Implementado

---

## RNF-04: Segurança

**Requisito:** O sistema deve proteger dados e prevenir acessos não autorizados.

**Critérios:**
- Autenticação via **JWT** com refresh tokens
- Dados sensíveis criptografados em trânsito (HTTPS)
- Senhas com hash bcrypt (10 rounds)
- Rate limiting: 100 requests/minuto por IP
- Proteção contra OWASP Top 10

**Implementação:**
- Helmet para headers de segurança
- CORS configurado por ambiente
- API keys para integrações
- Validação de input com Zod
- Logs de auditoria para ações sensíveis

**Status:** ✅ Implementado

---

## RNF-05: Compliance (LGPD)

**Requisito:** O sistema deve estar em conformidade com a LGPD.

**Critérios:**
- Consentimento explícito para uso de dados
- Direito ao esquecimento (exclusão de dados)
- Portabilidade de dados (exportação)
- Transparência no uso de dados

**Implementação:**
- Política de privacidade no frontend
- Endpoint para solicitação de exclusão de dados
- Logs de consentimento
- Retenção de dados definida (arquivos antigos podem ser limpos)

**Status:** 🚧 Parcialmente implementado

---

## RNF-06: Observabilidade

**Requisito:** O sistema deve ser monitorável e debuggável.

**Critérios:**
- Logs estruturados em JSON (Pino)
- Métricas de negócio e técnicas
- Request IDs para tracing
- Alertas para erros críticos

**Implementação:**
- Pino logger com níveis configuráveis
- Swagger/OpenAPI para documentação de APIs
- Logs de workers e jobs
- Métricas de tokens OpenAI consumidos

**Status:** ✅ Implementado

---

## RNF-07: Usabilidade

**Requisito:** O sistema deve ser fácil de usar para usuários não técnicos.

**Critérios:**
- Interface intuitiva (aprendizado em < 30 min)
- Responsivo (mobile-first via Tailwind)
- Feedback claro para ações do usuário (Sonner toasts)
- Carregamento com estados visuais (skeletons, spinners)

**Validação:**
- Onboarding guiado para configuração de IA
- Tooltips e labels claras
- Mensagens de erro em português

**Status:** ✅ Implementado

---

## RNF-08: Manutenibilidade

**Requisito:** O código deve ser fácil de manter e evoluir.

**Critérios:**
- Cobertura de testes: **≥ 70%** (meta)
- Código tipado com TypeScript
- Arquitetura modular (modules/services)
- CI/CD automatizado

**Práticas:**
- ESLint + Prettier para formatação
- Documentação de APIs via Swagger
- Migrations Prisma versionadas
- README com instruções de desenvolvimento

**Status:** ✅ Implementado

---

## RNF-09: Rate Limiting

**Requisito:** Proteger APIs contra abuse e garantir fair use.

**Critérios:**
- Rate limit por IP: **100** requests/minuto
- Rate limit de OpenAI: baseado em tokens contratados
- Mensagens de erro claras (HTTP 429)

**Implementação:**
- Fastify rate-limit plugin
- Tracking de token usage por empresa
- Alertas quando limite próximo

**Status:** ✅ Implementado

---

## RNF-10: Backup e Recuperação

**Requisito:** Dados devem ser recuperáveis em caso de falha.

**Critérios:**
- Backup diário do PostgreSQL
- Retenção de backups: **30** dias
- RTO (Recovery Time Objective): **1 hora**
- RPO (Recovery Point Objective): **24 horas**

**Estratégia:**
- pg_dump automatizado (cron)
- Backups em storage separado (S3/R2)
- Documentação de procedimento de restore

**Status:** 📋 Planejado (dependente de infraestrutura)

---

## RNF-11: Internacionalização (i18n)

**Requisito:** Suporte a múltiplos idiomas (futuro).

**Critérios:**
- Interface em português (atual)
- Estrutura preparada para i18n (futuro)

**Status:** 📋 Planejado para V2

---

## Matriz de RNFs

| ID | Categoria | Prioridade | Status |
|----|-----------|------------|--------|
| RNF-01 | Performance | Must Have | ✅ Implementado |
| RNF-02 | Escalabilidade | Should Have | ✅ Implementado |
| RNF-03 | Disponibilidade | Must Have | ✅ Implementado |
| RNF-04 | Segurança | Must Have | ✅ Implementado |
| RNF-05 | Compliance | Must Have | 🚧 Parcial |
| RNF-06 | Observabilidade | Should Have | ✅ Implementado |
| RNF-07 | Usabilidade | Should Have | ✅ Implementado |
| RNF-08 | Manutenibilidade | Should Have | ✅ Implementado |
| RNF-09 | Rate Limiting | Should Have | ✅ Implementado |
| RNF-10 | Backup | Must Have | 📋 Planejado |
| RNF-11 | i18n | Could Have | 📋 Planejado |

---

[← Voltar para Índice PRD](README.md)
