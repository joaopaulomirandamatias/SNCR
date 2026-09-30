# 08 — Reuso do InteroperaX

## 1. Decisão

O sistema SNCR será um produto **próprio, independente e autocontido**.

O InteroperaX será utilizado somente como fonte de:

- padrões arquiteturais;
- módulos maduros;
- testes;
- infraestrutura;
- mecanismos de resiliência;
- práticas de isolamento e auditoria.

A estratégia é **copiar e adaptar**, removendo conceitos específicos do InteroperaX e escrevendo testes do domínio SNCR.

## 2. O que já existe no InteroperaX e é relevante

A documentação e o código do `interoperax_workspace` já demonstram componentes úteis para esta nova plataforma:

- BFF/API em NestJS;
- autenticação/federação OIDC com Keycloak;
- propagação de tenant;
- isolamento de dados e uso de RLS no PostgreSQL em áreas maduras;
- contratos OpenAPI;
- orquestrador com BullMQ;
- retry, circuit breaker e DLQ;
- conectores HTTP;
- gestão de credenciais/segredos por tenant;
- evidence layer / decision records;
- práticas de idempotência;
- OpenTelemetry, correlation IDs e métricas;
- Docker/Compose e infraestrutura de desenvolvimento;
- testes unitários, integração e E2E em múltiplos serviços.

## 3. Mapa de reuso

| Área SNCR | Origem InteroperaX | Estratégia | Prioridade |
|---|---|---|---|
| Monorepo/TypeScript | estrutura do workspace | copiar padrão e simplificar | P0 |
| API/BFF | `services/platform-bff` | copiar esqueleto, remover domínio InteroperaX | P0 |
| OIDC/Keycloak | configuração e requisitos de federação | copiar configuração/padrões de claims | P0 |
| Tenant context | serviços BFF/orchestrator | copiar abordagem, endurecer para saúde | P0 |
| PostgreSQL/RLS | migrations/modelos tenant-aware | copiar padrão de política, reescrever tabelas | P0 |
| Orchestrator | serviço de orquestração | adaptar BullMQ/runtime para workflows de prescrição | P0 |
| DLQ | modelo/fluxos persistentes | adaptar para workflows SNCR | P0 |
| Retry/circuit breaker | orchestrator/connectors | copiar utilitários e testes | P0 |
| HTTP connector | runtime de conectores | adaptar para `sncr-client` | P0 |
| Audit/evidence | evidence layer | copiar conceito e utilitários, novo schema | P0 |
| Correlation ID | observabilidade | copiar propagação | P0 |
| OpenTelemetry | stack OTel | copiar bootstrap e instrumentação | P0 |
| Vault/segredos | secret handling por tenant | adaptar para tokens SNCR/provedores | P0 |
| Idempotência | auditoria e padrões existentes | copiar testes/padrões, tornar obrigatória | P0 |
| Docker infra | compose/containers | copiar base mínima | P1 |
| CI | pipelines/testes | copiar padrões, criar gates próprios | P1 |
| Frontend | Platform Web | reutilizar somente estrutura técnica útil | P1 |

## 4. Referências concretas no `interoperax_workspace`

Antes de copiar qualquer código, usar o commit/base atual como referência e revisar licenças internas, dependências e acoplamentos.

Arquivos/documentos já identificados como úteis:

```text
docs/handbook/ARCHITECTURE.md
docs/architecture/IX-CRA.md
docs/architecture/IX-HRA.md
docs/AUDITORIA_IDEMPOTENCIA.md
docs/INTEROPERAX_DEV_PLAN_v3.md
docs/spec/INTEROPERAX_ESPECIFICACAO_TECNICA_DETERMINISTICA_v1.1.md
docs/spec/INTEROPERAX_REQUIREMENTS_REGISTRY_*.yaml
api/openapi/orchestrator.json
```

A especificação do InteroperaX registra, entre outros pontos, uso de BullMQ, retry/DLQ, vault por tenant e observabilidade. Versões recentes do registry também descrevem `tenantId` obrigatório e RLS em modelos do orquestrador. Esses padrões são diretamente relevantes para a plataforma SNCR.

## 5. Reuso: BFF/API

### Copiar

- bootstrap NestJS;
- validação global;
- exception filters;
- interceptors de correlation ID;
- logging estruturado;
- auth guards genéricos;
- OpenAPI generation;
- health/readiness.

### Não copiar

- rotas do InteroperaX;
- DTOs de pipeline/semântica;
- regras de negócio;
- nomenclaturas de missão/agente.

### Destino

```text
apps/api/src/
├─ common/
├─ auth/
├─ tenants/
├─ prescribers/
├─ patients/
├─ prescriptions/
└─ integrations/
```

## 6. Reuso: Keycloak/OIDC

### Copiar/adaptar

- realm/client bootstrap;
- validação JWT;
- JWKS;
- logout/refresh;
- papéis;
- federation-ready architecture.

### Modificar

Claims e roles serão próprios:

```text
platform-admin
tenant-admin
prescriber
compliance-auditor
support-restricted
api-client
```

Não reutilizar roles genéricas que possam conceder privilégios indevidos.

## 7. Reuso: tenant + RLS

O InteroperaX possui evolução importante em `tenantId` e RLS nos dados operacionais. Esse desenho deve ser reaproveitado.

### Regra para SNCR

- `tenant_id NOT NULL` em toda tabela operacional compartilhada;
- políticas `ENABLE ROW LEVEL SECURITY` + `FORCE ROW LEVEL SECURITY` onde adotadas;
- `SET LOCAL app.tenant_id` dentro de transação controlada ou mecanismo equivalente;
- bypass apenas em jobs administrativos explicitamente autorizados;
- testes adversariais de cross-tenant.

### Não copiar cegamente

No SNCR existem dados de saúde; portanto o modelo deve ser mais restritivo que o mínimo necessário no InteroperaX.

## 8. Reuso: Orchestrator/BullMQ

A arquitetura do InteroperaX já usa BullMQ, workflow, retry e DLQ. Isso é um dos maiores aceleradores.

### Copiar

- bootstrap Redis/BullMQ;
- workers;
- job lifecycle;
- retry/backoff;
- circuit breaker;
- DLQ persistente;
- correlação;
- telemetria;
- testes de falha.

### Reescrever

Workflow do SNCR:

```text
validate
resolve_policy
ensure_prescriber
ensure_sncr_auth
reserve_number
render
sign
verify_signature
confirm_external
issue
notify
```

Não usar abstração genérica de pipeline quando ela atrapalhar invariantes clínicas/regulatórias.

## 9. Reuso: conectores

O InteroperaX já possui conectores HTTP e políticas de retry.

### Estratégia

Extrair apenas utilitários genéricos:

- HTTP transport;
- timeout;
- auth header injection;
- redaction;
- retry classifier;
- circuit breaker;
- telemetry;
- response mapping.

Criar do zero a semântica específica:

```text
SncrClient
SignatureProvider
MedicationSourceAdapter
NotificationAdapter
```

## 10. Reuso: audit/evidence

O conceito de `Evidence Layer` é extremamente útil.

### Reutilizar

- decisão append-only;
- correlação com workflow;
- referências a evidência;
- hash;
- actor context;
- exportabilidade.

### Adaptar

No SNCR, o evidence package será centrado na prescrição:

```text
policy decision
number allocation
SNCR requests/results
signature evidence
document hash
template version
audit timeline
```

## 11. Reuso: idempotência

O InteroperaX já possui auditorias e correções relacionadas a idempotência. Esses aprendizados devem virar regra obrigatória neste projeto.

### Copiar

- padrões de unique constraint;
- request hash;
- job deduplication;
- retry-safe design;
- análises de endpoints não idempotentes.

### Tornar mais rígido

Nenhuma operação de emissão/numeração/assinatura externa deve ser repetida automaticamente sem garantia de idempotência ou reconciliação.

## 12. Reuso: observabilidade

### Copiar

- bootstrap OpenTelemetry;
- propagation de trace/span;
- `correlationId`;
- métricas de fila;
- dashboards base;
- health endpoints.

### Alterar

Criar política de redaction específica de PHI/PII. Observabilidade do SNCR deve carregar IDs técnicos, não conteúdo clínico.

## 13. Reuso: secret management

O InteroperaX já possui conceito de vault por tenant e criptografia.

### Reutilizar o padrão, não necessariamente o schema

Segredos SNCR:

- tokens oficiais;
- client secrets quando existirem;
- credenciais de provedor de assinatura;
- webhook secrets;
- service account secrets.

Todos com:

- encryption at rest;
- rotação;
- tenant scope;
- no-log;
- acesso mínimo.

## 14. Componentes que NÃO devem ser levados

Não copiar como dependência do núcleo SNCR:

- Semantic Worker/UCM genérico;
- ontologias do domínio DETRAN/ITS;
- MCP Server/Client;
- skill executor;
- agent orchestration;
- Studio de pipelines;
- Google Sheets connector;
- conectores de bancos/fontes sem uso direto;
- UI de mapeamento semântico;
- schemas de missão;
- componentes Iris/biometria sem necessidade específica.

## 15. Semântica/Knowledge Graph: fase futura

A experiência do InteroperaX em semântica pode ser reaproveitada conceitualmente no futuro para um grafo regulatório:

```text
Medication -> contains -> Substance
Substance -> belongs_to -> RegulatoryList
RegulatoryList -> requires -> PrescriptionType
PrescriptionType -> requires -> SignaturePolicy
PrescriptionType -> uses -> OfficialTemplate
Rule -> sourced_from -> Regulation
```

Isso é evolução P2, não requisito do MVP.

## 16. Processo seguro de cópia

Para cada componente:

1. identificar arquivo/módulo no InteroperaX;
2. listar dependências transitivas;
3. separar código genérico de domínio;
4. copiar para branch do novo projeto;
5. renomear namespaces;
6. remover dependências do InteroperaX;
7. escrever testes próprios;
8. executar security review;
9. documentar origem e alterações;
10. só então integrar ao produto.

## 17. Matriz copiar vs reescrever

### Copiar com pouca alteração

- OTel bootstrap;
- correlation ID;
- health checks;
- Docker base;
- utilitários de retry/circuit breaker;
- estrutura de filas;
- logging/redaction base.

### Copiar e adaptar fortemente

- tenant context;
- RLS;
- Keycloak roles;
- vault;
- audit/evidence;
- DLQ;
- BFF.

### Reescrever do zero

- Prescription aggregate;
- patient domain;
- prescriber domain;
- policy engine regulatório;
- SNCR API domain client;
- assinatura;
- templates oficiais;
- QR Code;
- medication regulatory model.

## 18. Resultado esperado

Após o bootstrap, o projeto SNCR deve conseguir rodar sem nenhum serviço do InteroperaX:

```text
SNCR Web
SNCR API
SNCR Worker
PostgreSQL
Redis
Keycloak/IAM
Object Storage
OTel
```

As únicas dependências externas de negócio serão as explicitamente necessárias, como SNCR/Anvisa, Gov.br no fluxo oficial, provedor de assinatura e outras integrações escolhidas.