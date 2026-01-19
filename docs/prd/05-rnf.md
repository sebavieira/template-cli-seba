# 5. Requisitos Não Funcionais

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

[← Voltar para Índice PRD](README.md)

---

## RNF-01: Performance e Latência

**Requisito:** O sistema deve responder rapidamente às solicitações dos usuários.

**Critérios:**
- Tempo de resposta da API: **p95 < 200ms**
- Tempo de carregamento de página: **< 3 segundos**
- Operações em lote: **< 30 segundos**

**Medição:**
- Monitoramento via APM (ex: DataDog, New Relic)
- Alertas para degradação de performance

---

## RNF-02: Escalabilidade

**Requisito:** O sistema deve suportar crescimento de usuários e dados.

**Critérios:**
- Suportar **{{USUARIOS_SIMULTANEOS}}** usuários simultâneos
- Suportar **{{REGISTROS}}** registros no banco de dados
- Horizontal scaling quando necessário

**Estratégia:**
- Arquitetura stateless
- Banco de dados com índices otimizados
- Cache para dados frequentes

---

## RNF-03: Disponibilidade

**Requisito:** O sistema deve estar disponível para uso.

**Critérios:**
- Uptime: **{{UPTIME}}** (ex: 99.9%)
- Tempo máximo de downtime: **{{MAX_DOWNTIME}}** por mês
- Recuperação de falhas: **< 15 minutos**

**Estratégia:**
- Monitoramento contínuo
- Health checks automatizados
- Procedimentos de DR documentados

---

## RNF-04: Segurança

**Requisito:** O sistema deve proteger dados e prevenir acessos não autorizados.

**Critérios:**
- Autenticação via **JWT** ou similar
- Dados sensíveis criptografados em repouso e trânsito
- Proteção contra OWASP Top 10
- Rate limiting para prevenir abuse

**Implementação:**
- HTTPS obrigatório
- Senhas com hash bcrypt (min 12 rounds)
- Tokens com expiração curta
- Logs de auditoria

---

## RNF-05: Compliance

**Requisito:** O sistema deve estar em conformidade com regulamentações aplicáveis.

**Critérios:**
{{#se aplicável}}
- **LGPD:** Consentimento, direito ao esquecimento, portabilidade
- **GDPR:** (se aplicável internacionalmente)
- **PCI-DSS:** (se processar pagamentos)
{{/se}}

**Implementação:**
- Política de privacidade clara
- Mecanismo de opt-out
- Retenção de dados definida
- Processo de exclusão de dados

---

## RNF-06: Observabilidade

**Requisito:** O sistema deve ser monitorável e debuggável.

**Critérios:**
- Logs estruturados em JSON
- Métricas de negócio e técnicas
- Tracing distribuído
- Alertas para anomalias

**Ferramentas Sugeridas:**
- Logs: ELK Stack, CloudWatch, ou Loki
- Métricas: Prometheus + Grafana
- APM: DataDog, New Relic, ou Jaeger

---

## RNF-07: Usabilidade

**Requisito:** O sistema deve ser fácil de usar.

**Critérios:**
- Interface intuitiva (sem manual)
- Responsivo (mobile-first)
- Acessibilidade WCAG 2.1 nível AA
- Feedback claro para ações do usuário

**Validação:**
- Testes de usabilidade com usuários reais
- Análise de métricas de uso

---

## RNF-08: Manutenibilidade

**Requisito:** O código deve ser fácil de manter e evoluir.

**Critérios:**
- Cobertura de testes: **≥ 80%** (unitários)
- Código documentado
- Arquitetura modular
- CI/CD automatizado

**Práticas:**
- Code review obrigatório
- Padrões de código (ESLint, Prettier)
- Documentação de APIs (OpenAPI/Swagger)

---

## RNF-09: Rate Limiting

**Requisito:** Proteger APIs contra abuse e garantir fair use.

**Critérios:**
- Rate limit por usuário: **{{LIMITE_USUARIO}}** requests/minuto
- Rate limit global: **{{LIMITE_GLOBAL}}** requests/segundo
- Mensagens de erro claras quando excedido

---

## RNF-10: Backup e Recuperação

**Requisito:** Dados devem ser recuperáveis em caso de falha.

**Critérios:**
- Backup diário do banco de dados
- Retenção de backups: **{{RETENCAO}}** dias
- RTO (Recovery Time Objective): **{{RTO}}**
- RPO (Recovery Point Objective): **{{RPO}}**

---

## Matriz de RNFs

| ID | Categoria | Prioridade | Status |
|----|-----------|------------|--------|
| RNF-01 | Performance | Must Have | 📋 |
| RNF-02 | Escalabilidade | Should Have | 📋 |
| RNF-03 | Disponibilidade | Must Have | 📋 |
| RNF-04 | Segurança | Must Have | 📋 |
| RNF-05 | Compliance | Must Have | 📋 |
| RNF-06 | Observabilidade | Should Have | 📋 |
| RNF-07 | Usabilidade | Should Have | 📋 |
| RNF-08 | Manutenibilidade | Should Have | 📋 |
| RNF-09 | Rate Limiting | Should Have | 📋 |
| RNF-10 | Backup | Must Have | 📋 |

---

[← Voltar para Índice PRD](README.md)
