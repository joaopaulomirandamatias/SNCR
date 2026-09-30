# 03 — Requisitos Funcionais e Não Funcionais

## 1. Convenções

Prioridades:

- **P0**: obrigatório para MVP seguro e funcional;
- **P1**: necessário para produção inicial robusta;
- **P2**: evolução planejada.

Critérios de aceite devem ser verificáveis por teste automatizado, teste de integração ou evidência operacional.

## 2. Gestão de tenants e organizações

### TEN-001 — Criar organização/tenant [P0]
O sistema deve criar organização isolada com identificador imutável.

**Aceite:** dados de um tenant não podem ser lidos ou alterados por usuário de outro tenant.

### TEN-002 — Unidades e estabelecimentos [P1]
Uma organização pode possuir múltiplas unidades/endereço/identificadores regulatórios.

### TEN-003 — Isolamento obrigatório [P0]
Toda entidade operacional deve carregar `tenant_id` ou mecanismo equivalente de escopo. O banco deve aplicar defesa em profundidade, preferencialmente RLS onde apropriado.

### TEN-004 — Configuração por tenant [P1]
Configurações de assinatura, notificações e integrações devem ser isoladas e auditadas.

## 3. Identidade e acesso

### IAM-001 — Identidade interna [P0]
Usuários devem autenticar em IAM próprio do produto.

### IAM-002 — Integração Gov.br/SNCR [P0]
O produto deve iniciar e concluir o fluxo oficial de autenticação da API SNCR quando necessário.

### IAM-003 — RBAC [P0]
Papéis mínimos:

- platform-admin;
- tenant-admin;
- prescriber;
- compliance/auditor;
- support-restricted;
- api-client.

### IAM-004 — Privilégio mínimo [P0]
Ações administrativas e acesso a dados de saúde devem ser explicitamente autorizados.

### IAM-005 — MFA administrativo [P1]
Perfis de alto privilégio devem usar MFA no IAM interno.

### IAM-006 — Sessão e revogação [P0]
Logout, expiração, revogação e refresh devem ser tratados sem reutilização indevida de token.

## 4. Prescritores

### PRE-001 — Cadastro de prescritor [P0]
Guardar CPF/identificação necessária, conselho, UF, número profissional, nome e vínculos.

### PRE-002 — Status de elegibilidade SNCR [P0]
Registrar status, data e fonte da validação.

### PRE-003 — Múltiplos vínculos [P1]
O profissional pode atuar em mais de uma organização, mantendo escopos independentes.

### PRE-004 — Bloqueio de emissão [P0]
Não permitir emissão SNCR quando pré-condições obrigatórias não estiverem satisfeitas.

## 5. Pacientes

### PAT-001 — Cadastro mínimo [P0]
Armazenar somente dados necessários ao processo de prescrição e obrigações legais.

### PAT-002 — Deduplicação [P1]
Implementar mecanismos de detecção de duplicidade sem fusão automática destrutiva.

### PAT-003 — Direitos LGPD [P1]
Suportar busca, exportação e fluxo governado de solicitações do titular, respeitando obrigações legais de retenção.

## 6. Catálogo de medicamentos e regras

### MED-001 — Catálogo versionado [P0]
O sistema deve possuir fonte identificável para medicamentos/substâncias e respectivos metadados regulatórios.

### MED-002 — Classificação regulatória [P0]
A classificação deve ser executada no backend e resultar no tipo de receituário e políticas aplicáveis.

### MED-003 — Histórico de versão [P0]
Prescrição emitida deve continuar vinculada à versão de regra usada no momento da emissão.

### MED-004 — Atualização segura [P1]
Atualizações de catálogo devem passar por validação e não alterar silenciosamente registros históricos.

## 7. Prescrição

### RX-001 — Rascunho [P0]
Permitir criar e editar prescrição antes da emissão.

### RX-002 — Validação [P0]
Validar campos obrigatórios, elegibilidade, tipo, assinatura, modelo e regras antes da emissão.

### RX-003 — Máquina de estados [P0]
Estados e transições devem ser explícitos e impedidos quando inválidos.

### RX-004 — Imutabilidade após emissão [P0]
Dados clínicos/regulatórios de documento emitido não devem ser editados. Correção deve seguir novo fluxo permitido, cancelamento/revogação quando aplicável ou nova prescrição.

### RX-005 — Idempotência [P0]
Operações críticas de emissão devem aceitar `Idempotency-Key` e impedir emissão duplicada.

### RX-006 — Evidência [P0]
Cada emissão deve registrar eventos, hashes, versões de regra/template e IDs externos relevantes.

## 8. SNCR

### SNCR-001 — Login oficial [P0]
Implementar redirecionamento para `/api/v1/auth/login?client_url=...` conforme documentação vigente.

### SNCR-002 — Callback [P0]
Capturar `session_id` e trocá-lo imediatamente pelo token pelo endpoint documentado.

### SNCR-003 — Bearer token [P0]
Enviar token somente aos hosts SNCR permitidos e nunca registrá-lo em logs.

### SNCR-004 — Erro 401 [P0]
Invalidar sessão SNCR local e iniciar reautenticação controlada.

### SNCR-005 — API client resiliente [P0]
Timeout, retry apenas para operações seguras/idempotentes, circuit breaker e métricas.

### SNCR-006 — Reconciliation [P1]
Jobs devem identificar e tratar estados locais pendentes ou divergentes do serviço oficial.

### SNCR-007 — Ambiente [P0]
Separar configuração de treinamento/homologação e produção. Não permitir credenciais de produção em desenvolvimento.

## 9. Numeração

### NUM-001 — Aquisição oficial [P0]
Utilizar somente numeração obtida/permitida pelo processo oficial.

### NUM-002 — Reserva transacional [P0]
Impedir consumo concorrente do mesmo número.

### NUM-003 — Estados [P0]
Mínimo: `AVAILABLE`, `RESERVED`, `CONSUMED`, `RELEASED_IF_ALLOWED`, `INVALID`, `RECONCILIATION_REQUIRED`.

### NUM-004 — Auditoria [P0]
Toda transição deve registrar ator, timestamp, origem e correlation ID.

## 10. Assinatura

### SIG-001 — Policy [P0]
Determinar nível exigido de assinatura por regra regulatória.

### SIG-002 — ICP-Brasil [P0 quando exigido]
Integrar provedor compatível para assinatura qualificada.

### SIG-003 — Verificação [P0]
Validar assinatura, cadeia/certificado, integridade e identidade antes de marcar documento como assinado.

### SIG-004 — Evidências [P0]
Guardar metadados da assinatura e hash, evitando armazenar segredo/chave privada.

### SIG-005 — Falha segura [P0]
Falha de assinatura nunca pode resultar em `ISSUED`.

## 11. Documentos

### DOC-001 — Template oficial [P0]
Renderizar modelo eletrônico oficial vigente sem customização não permitida.

### DOC-002 — Versionamento [P0]
Guardar `template_version` e hash.

### DOC-003 — PDF [P0]
Gerar documento determinístico e validar conteúdo obrigatório.

### DOC-004 — QR Code [P0 quando aplicável]
Gerar URL conforme padrão oficial e testar decodificação.

### DOC-005 — Armazenamento [P0]
Armazenar objeto criptografado com chave lógica própria e controle de acesso.

## 12. Auditoria/evidência

### AUD-001 — Append-only lógico [P0]
Eventos críticos não devem ser atualizados destrutivamente.

### AUD-002 — Correlation ID [P0]
Toda requisição e job deve possuir identificador de correlação.

### AUD-003 — Ator [P0]
Registrar usuário/serviço, tenant, ação, recurso, resultado e timestamp.

### AUD-004 — Minimização [P0]
Não registrar corpo completo de receita em logs técnicos. Usar IDs, hashes e campos minimizados.

### AUD-005 — Exportação de evidência [P1]
Gerar pacote de auditoria para uma emissão sem expor segredos.

## 13. API B2B

### API-001 — REST versionada [P1]
Expor `/api/v1` com OpenAPI.

### API-002 — Service accounts [P1]
Clientes B2B devem usar credenciais próprias e escopo por tenant.

### API-003 — Idempotency-Key [P1]
Obrigatório nos comandos críticos.

### API-004 — Webhooks [P1]
Eventos assinados para `prescription.issued`, `prescription.failed`, `signature.completed` e outros necessários.

### API-005 — Anti-replay [P1]
Webhooks devem conter timestamp, event ID e assinatura verificável.

## 14. Observabilidade

### OBS-001 — Traces [P0]
Cobrir gateway → prescription → policy → workflow → connector → external call.

### OBS-002 — Métricas [P0]
Latência, taxa de erro, retries, circuit breaker, filas, DLQ, emissão, assinatura e SNCR.

### OBS-003 — Logs estruturados [P0]
JSON com redaction de PII/PHI e segredos.

### OBS-004 — Alertas [P1]
Alertar sobre falhas externas, fila crescendo, divergências e erros de emissão.

## 15. Segurança

### SEC-001 — TLS [P0]
TLS para todo tráfego externo e interno sensível.

### SEC-002 — Segredos [P0]
Nenhum segredo em código, imagem Docker ou frontend.

### SEC-003 — Criptografia em repouso [P0]
Banco, storage e backups criptografados.

### SEC-004 — Rate limiting [P0]
Proteção em login, APIs públicas e endpoints sensíveis.

### SEC-005 — SAST/Dependency scan [P1]
CI deve bloquear vulnerabilidades críticas conforme política.

### SEC-006 — Pentest [P1]
Antes de produção com dados reais.

## 16. Não funcionais

### NFR-001 — Disponibilidade [P1]
Meta inicial sugerida: 99,9% mensal para componentes próprios, excluindo indisponibilidade oficial de terceiros devidamente evidenciada.

### NFR-002 — Performance [P1]
Operações locais comuns: p95 < 500 ms quando não dependentes de serviço externo. Fluxos externos devem possuir SLO separado.

### NFR-003 — Recuperação [P1]
Definir RPO/RTO após análise de negócio. Backups devem ser testados, não apenas configurados.

### NFR-004 — Escalabilidade [P1]
Serviços stateless devem escalar horizontalmente; workers por filas devem possuir concorrência controlada.

### NFR-005 — Portabilidade [P1]
Containers sem dependência obrigatória de um único provedor cloud.

### NFR-006 — Acessibilidade [P1]
Interface web deve buscar WCAG 2.2 AA.

### NFR-007 — Timezone [P0]
Persistir timestamps em UTC e apresentar no fuso adequado. Regras de validade devem usar regra normativa, não simples cálculo visual no frontend.

## 17. Requisitos de qualidade para produção

- cobertura de testes do domínio crítico;
- testes de contrato da API SNCR;
- sandbox/fixtures sem dados reais;
- testes de concorrência de numeração;
- testes de idempotência;
- testes de autorização multi-tenant;
- testes de regressão de templates;
- teste de restauração de backup;
- teste de rotação de chaves/segredos;
- runbooks de incidente e indisponibilidade.