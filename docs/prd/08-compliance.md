# 8. Compliance e Políticas

**Versão:** 1.0.0
**Última Atualização:** {{DATA}}

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
| {{DADO_1}} | {{FINALIDADE_1}} | {{BASE_1}} | {{RETENCAO_1}} |
| {{DADO_2}} | {{FINALIDADE_2}} | {{BASE_2}} | {{RETENCAO_2}} |
| {{DADO_3}} | {{FINALIDADE_3}} | {{BASE_3}} | {{RETENCAO_3}} |

---

## 8.2 Políticas por Canal (Se Aplicável)

### Email

- Compliance com CAN-SPAM / LGPD
- Opt-out em todas as mensagens
- Identificação clara do remetente
- Assunto não enganoso

### WhatsApp

- Compliance com Meta Business Policies
- Uso de templates aprovados
- Janela de 24h para respostas
- Opt-in explícito obrigatório

### SMS (Se Aplicável)

- Compliance com ANATEL
- Horário de envio respeitado
- Opt-out via PARE

---

## 8.3 Segurança de Dados

### Dados Sensíveis

| Dado | Classificação | Proteção |
|------|---------------|----------|
| Senhas | Crítico | Hash bcrypt, nunca armazenado em texto |
| Tokens | Crítico | Criptografado, expiração curta |
| Dados pessoais | Alto | Criptografia em repouso |
| Logs | Médio | Retenção limitada, sem PII |

### Controles de Acesso

- Autenticação obrigatória
- Autorização por roles
- Audit trail de acessos
- Sessões com timeout

---

## 8.4 Termos e Políticas

### Documentos Necessários

- [ ] Termos de Uso
- [ ] Política de Privacidade
- [ ] Política de Cookies (se aplicável)
- [ ] SLA (se aplicável)

### Checklist de Compliance

- [ ] Consentimento implementado
- [ ] Mecanismo de opt-out funcional
- [ ] Exportação de dados disponível
- [ ] Exclusão de dados implementada
- [ ] Política de privacidade publicada
- [ ] Logs de auditoria ativos
- [ ] Criptografia configurada
- [ ] Backup de dados ativo

---

[← Voltar para Índice PRD](README.md)
