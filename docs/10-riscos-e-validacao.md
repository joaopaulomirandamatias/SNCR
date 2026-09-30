# 10 — Riscos e Plano de Validação

## 1. Objetivo

Mapear riscos que podem impedir a operação segura, correta ou regulatoriamente aderente da plataforma e definir como reduzi-los antes do go-live.

## 2. Matriz de risco inicial

| Risco | Prob. | Impacto | Mitigação |
|---|---:|---:|---|
| Mudança de API SNCR | média | alta | cliente isolado, contract tests, monitoramento oficial, versionamento |
| Mudança regulatória | alta | alta | policy engine versionado, feature flags, processo de change regulatório |
| Emissão duplicada | média | crítica | idempotência, locks, unique constraints, reconciliação |
| Consumo incorreto de numeração | média | crítica | transação, reserva explícita, auditoria, testes de concorrência |
| Token SNCR vazado | baixa/média | crítica | BFF, encryption, redaction, allowlist egress, CSP |
| Cross-tenant data leak | baixa | crítica | tenant context, RLS, testes adversariais |
| Documento fora do modelo Anvisa | média | alta | template registry, regression visual, hash/version |
| Assinatura inválida | média | crítica | provider homologado, verificação independente, fail closed |
| SNCR indisponível | média | alta | circuit breaker, estados pendentes, UX clara, reconciliação |
| Resultado externo desconhecido após timeout | média | alta | `UNKNOWN_RESULT`, reconciliação, proibir retry cego |
| Catálogo/regra regulatória incorreta | média | crítica | fonte oficial, dupla revisão, snapshots, testes de decisão |
| Acesso indevido por suporte/admin | média | alta | least privilege, segregation of duties, audit |
| Backup vazado | baixa | crítica | encryption, access control, retention, restore process |
| Dependência comprometida | média | alta | lockfile, SCA, SBOM, pinning, patch process |
| Provedor ICP indisponível | média | alta | SLA, adapter, fallback futuro, workflow pendente |
| Divergência local vs SNCR | média | alta | evidence, reconciliation worker, dashboard operacional |

## 3. Riscos regulatórios

### 3.1 Tratar orientação antiga como vigente

Mitigação:

- `regulatory_sources` versionado;
- data de última revisão;
- release bloqueado se fonte crítica estiver sem revisão;
- acompanhamento da página oficial e informes API.

### 3.2 Hard-code de regras

Mitigação:

- regras versionadas em tabela/DSL controlada;
- effective dates;
- testes;
- revisão por responsável regulatório.

### 3.3 Customização de modelo oficial

Mitigação:

- template não editável por tenant;
- apenas templates oficiais aprovados no registry;
- visual regression.

## 4. Riscos de integração SNCR

### 4.1 Callback manipulado

Mitigação:

- validar origem/fluxo;
- não confiar em `session_id` além da troca oficial;
- troca imediata;
- nunca reutilizar;
- proteger callback contra CSRF/session fixation conforme desenho final.

### 4.2 Token enviado ao host errado

Mitigação:

- HTTP client dedicado;
- base URL allowlisted;
- sem redirects automáticos para host externo quando Authorization estiver presente;
- egress policy.

### 4.3 Retry gera efeito duplicado

Mitigação:

- classificador de operações;
- idempotency keys;
- `UNKNOWN_EXTERNAL_RESULT`;
- consulta/reconciliação antes de repetir.

## 5. Riscos de assinatura

### 5.1 Assinar documento diferente do exibido

Mitigação:

- hash do documento exibido;
- assinatura vinculada ao hash;
- verificar hash após callback;
- invalidar se documento mudar.

### 5.2 Signatário diferente do prescritor

Mitigação:

- comparar identidade/certificado com prescritor;
- fail closed;
- evidence record.

### 5.3 Certificado inválido/revogado

Mitigação:

- validação compatível com provider/padrão;
- guardar resultado e timestamp;
- política de validação documentada.

## 6. Plano de testes

### Unitários

- state machine;
- policy engine;
- classification;
- idempotency service;
- URL/QR builder;
- signature policy.

### Integração

- PostgreSQL/RLS;
- Redis/BullMQ;
- object storage;
- Keycloak;
- provider signature sandbox;
- SNCR training.

### Contract

- schemas SNCR;
- error mapping;
- callbacks;
- provider signature webhooks.

### E2E

- onboarding;
- draft;
- emission golden path;
- invalid prescriber;
- token expired;
- SNCR timeout;
- signature rejected;
- duplicate click;
- cross-tenant denial.

### Segurança

- OWASP ASVS/Top 10 aplicável;
- BOLA/IDOR;
- SSRF;
- CSRF;
- XSS;
- injection;
- session fixation;
- token leakage;
- privilege escalation;
- tenant isolation.

## 7. Testes de falha obrigatórios

Executar intencionalmente:

1. SNCR retorna 500;
2. SNCR demora além do timeout;
3. resposta chega mas processo cai antes do commit;
4. worker reinicia no meio da emissão;
5. Redis reinicia;
6. callback de assinatura chega duas vezes;
7. assinatura é recusada;
8. storage indisponível;
9. banco faz failover;
10. token expira durante workflow.

Resultado esperado: nunca produzir `ISSUED` sem evidência suficiente e nunca repetir efeito crítico cegamente.

## 8. Teste de concorrência

Cenários:

- 10 requests simultâneos com mesma `Idempotency-Key`;
- 10 requests simultâneos com keys diferentes para mesma receita;
- 2 workers no mesmo job;
- disputa pelo mesmo número;
- callback durante reconciliação.

Invariantes:

- uma única emissão efetiva;
- uma única alocação do número;
- audit trail consistente.

## 9. Teste de tenant isolation

Criar Tenant A e Tenant B.

Tentar:

- ler paciente B com token A;
- baixar PDF B;
- consultar prescription B;
- alterar ID no path;
- injetar `tenant_id` no body;
- executar job A referenciando resource B;
- usar service account A em API B.

Todos devem falhar sem vazar existência/dados além do necessário.

## 10. Validação de documento

Para cada template:

- conteúdo obrigatório;
- posição/layout;
- tamanho de página;
- fontes incorporadas quando necessário;
- QR Code decodificável;
- URL exata;
- hash;
- assinatura visível/criptográfica quando aplicável;
- comparação visual golden master.

## 11. Validação regulatória

Criar planilha/tabela de decisão com casos aprovados por especialista:

```text
medication/substance
regulatory classification
expected prescription type
expected signature level
expected SNCR behavior
expected template
validity/quantity constraints when applicable
source/version
```

Transformar cada linha em teste automatizado.

## 12. Validação de observabilidade

Em uma emissão de teste, deve ser possível responder:

- qual request iniciou a emissão?
- qual tenant?
- qual prescription ID?
- qual regra foi aplicada?
- qual job processou?
- qual chamada SNCR ocorreu?
- qual foi o resultado?
- qual assinatura ocorreu?
- qual documento foi produzido?

Sem expor PII/PHI desnecessária.

## 13. SLO/alertas iniciais

Alertar quando:

- taxa de erro SNCR exceder limiar;
- 401 aumentar abruptamente;
- circuit breaker abrir;
- DLQ > 0;
- `RECONCILIATION_REQUIRED` crescer;
- fila ficar atrasada;
- assinatura falhar acima do baseline;
- audit pipeline falhar;
- storage/banco degradar.

## 14. Go-live checklist

### Regulação

- [ ] fontes revisadas na semana do go-live;
- [ ] regras aprovadas;
- [ ] templates aprovados;
- [ ] integração de produção confirmada.

### Segurança

- [ ] pentest;
- [ ] MFA;
- [ ] secrets manager;
- [ ] encryption;
- [ ] RLS;
- [ ] token redaction;
- [ ] backup restore;
- [ ] incident drill.

### Operação

- [ ] dashboards;
- [ ] alerts;
- [ ] runbooks;
- [ ] on-call;
- [ ] reconciliation dashboard;
- [ ] DLQ process.

### Produto

- [ ] onboarding validado;
- [ ] golden path;
- [ ] mensagens de erro compreensíveis;
- [ ] suporte treinado;
- [ ] termos/privacy publicados.

## 15. Critério de bloqueio

Não fazer produção se existir qualquer uma destas condições:

- dúvida sobre regra de assinatura de um fluxo habilitado;
- template não validado;
- emissão duplicável em teste;
- cross-tenant vulnerability;
- token em logs;
- reconciliação inexistente para operação com resultado incerto;
- backup não restaurável;
- vulnerabilidade crítica/alta explorável sem mitigação;
- integração de treinamento não validada.