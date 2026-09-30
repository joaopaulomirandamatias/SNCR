# 09 — Roadmap e Backlog do MVP

## 1. Estratégia

Construir em fases, validando cedo as integrações de maior risco: SNCR, assinatura e regras regulatórias. Não deixar essas integrações para o fim.

## 2. Fase 0 — Descoberta e fundação

### Objetivos

- congelar escopo do MVP;
- validar documentação oficial vigente;
- definir arquitetura;
- criar repositório de código;
- escolher provedor de assinatura;
- preparar ambiente de treinamento SNCR.

### Entregas

- ADRs;
- threat model;
- modelo inicial de dados;
- OpenAPI interna;
- catálogo regulatório inicial;
- CI mínimo;
- ambientes local/dev.

### Critério de saída

Time consegue executar uma aplicação vazia com autenticação, tenant, banco, fila e telemetria.

## 3. Fase 1 — Fundação reaproveitada do InteroperaX

### Copiar/adaptar

- NestJS bootstrap/BFF;
- Keycloak/OIDC;
- tenant context;
- PostgreSQL + Prisma;
- RLS;
- Redis/BullMQ;
- DLQ;
- retry/circuit breaker;
- OpenTelemetry;
- correlation IDs;
- secret handling;
- Docker.

### Testes obrigatórios

- cross-tenant;
- auth/roles;
- job tenant propagation;
- DLQ;
- retry;
- trace ponta a ponta.

## 4. Fase 2 — Domínio clínico mínimo

### Implementar

- organização/unidade;
- prescritor;
- paciente;
- medicamento/substância mínimo;
- prescription aggregate;
- rascunho;
- validação;
- máquina de estados;
- audit trail.

### Fora do escopo

- prontuário completo;
- agenda;
- faturamento;
- teleconsulta;
- IA clínica.

## 5. Fase 3 — Regulatory Policy Engine

### Implementar

- catálogo de tipos de receita;
- regras versionadas;
- policy de assinatura;
- policy de SNCR;
- policy de template;
- snapshots de decisão;
- feature flags por data/ambiente.

### Testes

Tabela de decisão com dezenas de combinações conhecidas e casos inválidos.

## 6. Fase 4 — Integração SNCR treinamento

### Implementar

- `sncr-client`;
- login Gov.br;
- callback `session_id`;
- troca por token;
- armazenamento seguro;
- tratamento de 401;
- operação mínima documentada;
- timeouts;
- circuit breaker;
- contract tests;
- reconciliação.

### Critério de saída

Um prescritor de teste conclui autenticação e o sistema executa uma chamada oficial de treinamento com evidência e trace.

## 7. Fase 5 — Documentos oficiais

### Implementar

- registry de templates;
- renderer;
- geração de PDF;
- QR Code;
- hash;
- object storage;
- testes de regressão visual;
- validação automatizada do QR.

### Critério de saída

Documento gerado reproduz o modelo oficial aplicável e possui evidência de versão/hash.

## 8. Fase 6 — Assinatura digital

### Implementar

- adapter de provider;
- sandbox;
- fluxo de autorização;
- callback/webhook;
- verificação;
- evidence metadata;
- timeouts/retry/reconciliation.

### Critério de saída

Documento de teste é assinado e validado ponta a ponta pela política correta.

## 9. Fase 7 — Golden Path ponta a ponta

Missão de aceite:

```text
criar tenant
→ cadastrar/vincular prescritor
→ cadastrar paciente
→ criar prescrição
→ classificar regra
→ autenticar SNCR
→ obter/usar numeração aplicável
→ gerar documento
→ assinar
→ confirmar operação oficial
→ emitir
→ gerar QR
→ armazenar
→ entregar
→ exportar evidência
```

### Critério

A execução deve ser reproduzível no ambiente de treinamento sem intervenção manual no banco.

## 10. Fase 8 — Hardening

- pentest;
- load test;
- chaos de dependências externas;
- backup/restore;
- key rotation;
- incident simulation;
- privacy review;
- revisão jurídica;
- runbooks;
- dashboards/alertas;
- SLOs.

## 11. Fase 9 — Piloto controlado

Sugestão:

- poucos tenants;
- poucos prescritores;
- volume limitado;
- suporte próximo;
- feature flags;
- observabilidade reforçada;
- rollback operacional.

## 12. Backlog P0

### Plataforma

- [ ] monorepo novo;
- [ ] API NestJS;
- [ ] frontend Next.js;
- [ ] worker;
- [ ] PostgreSQL/Prisma;
- [ ] Redis/BullMQ;
- [ ] IAM;
- [ ] tenant/RLS;
- [ ] audit/evidence;
- [ ] OTel;
- [ ] secrets.

### Domínio

- [ ] organization;
- [ ] prescriber;
- [ ] patient;
- [ ] medication/substance;
- [ ] prescription;
- [ ] regulatory rules;
- [ ] state machine;
- [ ] idempotency.

### SNCR

- [ ] auth/login;
- [ ] callback;
- [ ] token exchange;
- [ ] token lifecycle;
- [ ] typed client;
- [ ] numbering/operation interfaces conforme API vigente;
- [ ] 401 handling;
- [ ] reconciliation.

### Documento/assinatura

- [ ] official template registry;
- [ ] PDF;
- [ ] QR;
- [ ] storage;
- [ ] signature provider;
- [ ] verification.

### Segurança

- [ ] CSP/headers;
- [ ] encryption;
- [ ] redaction;
- [ ] rate limits;
- [ ] cross-tenant tests;
- [ ] secret scan;
- [ ] backup restore.

## 13. Backlog P1

- [ ] API B2B;
- [ ] service accounts;
- [ ] webhooks;
- [ ] tenant dashboard;
- [ ] compliance dashboard;
- [ ] evidence export;
- [ ] improved medication registry;
- [ ] reconciliation UI;
- [ ] manual DLQ resolution UI;
- [ ] notification delivery;
- [ ] billing/plans.

## 14. Backlog P2

- [ ] SDK TypeScript;
- [ ] SDK Java/PHP;
- [ ] FHIR adapter se houver caso validado;
- [ ] knowledge graph regulatório;
- [ ] regras avançadas de interação medicamentosa com fonte clínica validada;
- [ ] fraude/anomaly detection;
- [ ] assistente de preenchimento;
- [ ] integrações HIS/ERP;
- [ ] possível vertical de farmácia separada.

## 15. Definition of Done para módulo crítico

Um item crítico só está concluído quando:

- implementação revisada;
- testes unitários;
- teste de integração;
- autorização/tenant testados;
- telemetria presente;
- erros redacted;
- documentação atualizada;
- migration reversível operacionalmente;
- threat model atualizado se necessário;
- nenhuma vulnerabilidade crítica aberta.

## 16. Gates para produção

### Gate regulatório

- norma vigente conferida;
- templates vigentes;
- API vigente;
- regras de assinatura revisadas.

### Gate técnico

- E2E golden path;
- idempotência;
- reconciliação;
- backup/restore;
- observabilidade.

### Gate segurança

- pentest;
- secret management;
- RLS;
- audit;
- incident response.

### Gate jurídico/LGPD

- privacy notice;
- contratos;
- papéis controlador/operador;
- bases legais;
- retenção;
- RIPD quando indicado.

## 17. Sequência recomendada de implementação

Não construir UI sofisticada antes do núcleo.

Ordem:

1. infraestrutura reaproveitada;
2. domínio;
3. policy engine;
4. SNCR training;
5. documento;
6. assinatura;
7. E2E;
8. UI refinada;
9. API B2B;
10. piloto.

## 18. Primeira milestone sugerida

**M1 — SNCR Technical Proof**

Objetivo: provar que a nova aplicação independente consegue autenticar um prescritor no fluxo oficial de treinamento, manter tenant/contexto, consumir uma operação documentada, registrar audit trail e produzir trace completo.

Essa milestone reduz o maior risco de integração antes de desenvolver o restante do produto.