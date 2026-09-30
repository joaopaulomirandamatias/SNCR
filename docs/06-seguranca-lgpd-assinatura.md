# 06 — Segurança, LGPD e Assinatura Digital

## 1. Classificação de risco

A plataforma processará dados pessoais e dados de saúde. Dados referentes à saúde são dados pessoais sensíveis na LGPD e exigem controles proporcionais ao risco.

A plataforma deve ser tratada como sistema de alta criticidade por combinar:

- identidade do profissional;
- dados do paciente;
- dados clínicos;
- prescrição de medicamentos;
- assinatura digital;
- integrações governamentais;
- rastreabilidade regulatória.

## 2. Privacy by Design

Princípios:

- coletar somente dados necessários;
- definir finalidade por conjunto de dados;
- aplicar retenção explícita;
- impedir uso secundário não autorizado;
- separar logs técnicos dos dados clínicos;
- limitar privilégios;
- registrar acessos sensíveis;
- revisar contratos com operadores/suboperadores;
- documentar bases legais com jurídico/DPO.

## 3. Inventário de dados

Criar Data Inventory formal:

| Categoria | Exemplos | Sensibilidade | Controles |
|---|---|---|---|
| Identidade do prescritor | CPF, nome, conselho | pessoal | criptografia, RBAC, audit |
| Dados profissionais | CRM/CRO/CRMV, UF | pessoal/profissional | RBAC, audit |
| Paciente | identificação e contato | pessoal | criptografia, minimização |
| Dados clínicos | medicamento, posologia, prescrição | sensível/saúde | acesso restrito, audit, criptografia |
| Assinatura | certificado/metadados/hash | alta criticidade | integridade, acesso restrito |
| Token SNCR | bearer token | segredo | vault/encryption, nunca logar |
| Audit events | IDs, ação, hashes | confidencial | append-only, retenção controlada |

## 4. Bases legais e governança

Não codificar a LGPD como simples checkbox de consentimento. Em saúde, a base legal aplicável pode depender da operação, relação assistencial, obrigação regulatória e papel das partes.

Antes da produção:

- mapear controlador e operadores por fluxo;
- registrar base legal por finalidade;
- produzir aviso de privacidade;
- produzir política de retenção;
- mapear compartilhamentos;
- avaliar necessidade de RIPD;
- definir canal do titular;
- definir encarregado/DPO;
- revisar contratos com provedores de cloud, assinatura e mensageria.

## 5. Autorização

Combinar RBAC + regras contextuais.

Exemplos:

- prescritor acessa pacientes/prescrições dentro de seus vínculos autorizados;
- tenant-admin não deve automaticamente visualizar conteúdo clínico completo;
- suporte não deve possuir acesso direto ao conteúdo da receita;
- auditor/compliance possui acesso controlado e justificado;
- service account tem scopes mínimos.

## 6. Break-glass

Para cenários futuros que exijam acesso emergencial:

- justificativa obrigatória;
- MFA;
- acesso temporário;
- alerta ao compliance;
- trilha reforçada;
- revisão posterior.

Não implementar break-glass no MVP se não houver necessidade real.

## 7. Criptografia

### Em trânsito

- TLS 1.2+; preferir TLS 1.3;
- HSTS;
- cookies `Secure`;
- validação estrita de certificado.

### Em repouso

- volume/banco criptografado;
- object storage com server-side encryption;
- backups criptografados;
- segredos/token adicionalmente criptografados no nível da aplicação quando necessário.

### Chaves

- usar KMS/Vault/secret manager;
- rotação definida;
- acesso por workload identity quando disponível;
- nunca guardar chave privada de assinatura do usuário na aplicação sem solução certificada e justificativa forte.

## 8. Assinatura eletrônica

O sistema deve possuir `Signature Policy` derivada do enquadramento regulatório.

Níveis possíveis no modelo interno:

```text
NONE
SIMPLE
ADVANCED
QUALIFIED_ICP_BRASIL
```

O mapeamento real por tipo de receita deve ser mantido pelo policy engine com fonte e versão.

## 9. ICP-Brasil

Quando a regra exigir assinatura qualificada, o produto deve utilizar solução compatível com ICP-Brasil.

### Estratégia preferida

Integração com provedor de assinatura remota em vez de manipulação direta de A3 no browser.

Motivos:

- melhor UX;
- reduz complexidade de drivers/middleware;
- evita custódia local inadequada de chave;
- facilita validação/auditoria;
- suporta web/mobile.

### Build vs buy

**Não construir uma Autoridade Certificadora.** O produto integra um prestador ICP-Brasil.

Comparar provedores por:

- conformidade ICP-Brasil;
- assinatura PAdES/CAdES conforme necessidade;
- validação de certificado;
- API;
- MFA/autorização do titular;
- timestamp/carimbo de tempo se aplicável;
- SLA;
- evidências;
- preço;
- LGPD/localização;
- webhooks;
- sandbox.

## 10. Pipeline de assinatura

```text
Documento pré-assinatura
    ↓ SHA-256
Signature Request
    ↓ usuário autoriza
Provider
    ↓ documento/assinatura
Verification Service
    ↓ verifica integridade e identidade
SIGNED
```

Registrar:

- hash antes/depois conforme formato;
- provider;
- transaction ID;
- certificado e dados públicos necessários;
- política aplicada;
- timestamp;
- resultado de validação.

Não registrar:

- PIN;
- senha do certificado;
- chave privada;
- token do provedor em log.

## 11. Validação de assinatura

Antes de emissão final:

- verificar integridade;
- verificar identidade do signatário;
- confirmar que signatário é o prescritor da receita;
- verificar validade/cadeia conforme solução adotada;
- registrar evidência da validação;
- tratar revogação/status conforme padrão aplicável.

## 12. Sessão Gov.br/SNCR

- `session_id` é transitório e deve ser trocado imediatamente;
- token SNCR deve ser segregado da sessão do nosso produto;
- usar armazenamento server-side quando arquitetura oficial permitir;
- token criptografado;
- expiração e reautenticação;
- jamais enviar token SNCR para analytics, Sentry ou logs.

## 13. Segurança de aplicação

### Backend

- DTO validation;
- output encoding;
- ORM parametrizado;
- SSRF allowlist para integrações;
- rate limiting;
- proteção CSRF se cookie auth;
- CORS restrito;
- headers de segurança;
- limits de payload;
- file validation.

### Frontend

- CSP forte;
- evitar `dangerouslySetInnerHTML`;
- nenhuma credencial em bundle;
- SRI onde aplicável;
- cookies HttpOnly;
- proteção contra clickjacking.

## 14. Segurança multi-tenant

Testes obrigatórios:

- IDOR/BOLA entre tenants;
- troca de `tenant_id` em payload/header;
- jobs da fila com tenant errado;
- download de documento de outro tenant;
- prescriber vinculado a dois tenants;
- service account cross-tenant;
- admin sem permissão clínica.

## 15. Logs

### Permitido

```json
{
  "event": "prescription.workflow.step",
  "tenantId": "t_123",
  "prescriptionId": "rx_456",
  "step": "sncr.submit",
  "status": "success",
  "correlationId": "..."
}
```

### Proibido em log técnico

- nome do paciente;
- CPF completo;
- conteúdo integral da prescrição;
- token;
- session_id;
- certificado privado;
- senha/PIN.

## 16. Audit trail

Auditoria é evidência de negócio e não deve ser tratada como log descartável.

Eventos mínimos:

- login/logout;
- alteração de papel;
- leitura sensível quando necessário;
- criação/edição de rascunho relevante;
- validação regulatória;
- reserva/consumo de numeração;
- assinatura;
- emissão;
- download/compartilhamento;
- cancelamento/revogação;
- alteração de integração;
- acesso administrativo.

## 17. Retenção

Criar tabela de retenção por categoria, validada juridicamente. Nunca definir prazo arbitrário no código sem fonte.

O sistema deve suportar:

- retenção mínima obrigatória;
- legal hold;
- expurgo seguro quando permitido;
- anonimização quando adequada;
- trilha do processo de expurgo.

## 18. Backups

- criptografados;
- acesso segregado;
- cópia imutável/offline lógica;
- testes periódicos de restore;
- RPO/RTO documentado;
- não manter backups indefinidamente sem política.

## 19. Gestão de vulnerabilidades

CI:

- SAST;
- dependency scanning;
- secret scanning;
- container scanning;
- IaC scanning quando aplicável;
- SBOM.

Produção:

- patch cadence;
- CVE triage;
- pentest antes do go-live e após mudanças críticas;
- programa de disclosure futuro.

## 20. Incidentes

Runbook deve prever:

1. detecção;
2. triagem;
3. contenção;
4. preservação de evidência;
5. análise de impacto;
6. comunicação interna;
7. obrigações de comunicação externa/LGPD quando aplicáveis;
8. recuperação;
9. post-mortem;
10. ações corretivas.

## 21. Threat model inicial

Ameaças prioritárias:

- account takeover de prescritor;
- emissão fraudulenta;
- replay de requisição;
- consumo duplicado de numeração;
- roubo de token SNCR;
- SSRF/exfiltração de bearer;
- IDOR cross-tenant;
- adulteração de PDF;
- assinatura de documento diferente do mostrado;
- insider/support abuse;
- vazamento de backup;
- supply chain dependency attack;
- webhooks falsificados.

## 22. Critérios de segurança para go-live

- [ ] threat model revisado;
- [ ] pentest sem crítico/alto não mitigado;
- [ ] RLS/tenant isolation testado;
- [ ] tokens redacted;
- [ ] KMS/secrets manager ativo;
- [ ] MFA administrativo;
- [ ] assinatura verificada ponta a ponta;
- [ ] restore de backup testado;
- [ ] incident response testado;
- [ ] DPA/contratos revisados;
- [ ] política de privacidade publicada;
- [ ] inventário e retenção aprovados;
- [ ] trilha de auditoria validada.