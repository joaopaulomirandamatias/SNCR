# 05 — Integração com a API SNCR

## 1. Fonte

Este documento usa como referência principal a **Instruções de Integração API SNCR v1.0 — JUL/2026**, publicada pela Anvisa, além dos comunicados oficiais de setembro de 2026.

> Toda implementação deve confirmar a documentação vigente antes do deploy. Endpoints, payloads e políticas podem mudar.

## 2. Ambiente de treinamento

A documentação de julho/2026 cita o ambiente:

`https://sncr-treinamento.anvisa.gov.br/`

Esse ambiente deve ser usado no processo de preparação e validação conforme instruções oficiais.

### Regra de ambiente

Nunca reutilizar:

- banco de produção;
- cookies de produção;
- tokens de produção;
- certificados/chaves reais desnecessários;
- dados reais de pacientes

no ambiente de treinamento.

## 3. Pré-condição de prescritor

A documentação informa que novos prescritores precisam realizar acesso ao SNCR para que seus dados estejam disponíveis na base e possam participar do fluxo via API. O profissional deve possuir inscrição ativa em conselho profissional suportado.

### UX proposta

No onboarding:

```text
Prescritor criado
    ↓
Verificar pré-condição SNCR
    ↓
Pendente? → orientar acesso/cadastro oficial
    ↓
Revalidar
    ↓
Elegível
```

O produto deve explicar que isso não é “cadastro no nosso sistema”, mas uma pré-condição do serviço oficial.

## 4. Credenciamento da plataforma

A versão 1.0 da instrução informa que **não será necessário credenciamento da plataforma**, e cita validação do profissional e, futuramente, da organização mantenedora.

### Implementação

Criar entidade/configuração para organização mantenedora desde já, sem inventar um processo de credenciamento inexistente.

## 5. URL base

A instrução usa como exemplo:

`https://sncr-api.apps.anvisa.gov.br`

Nunca hard-code em componentes espalhados. Usar:

```text
SNCR_BASE_URL
SNCR_ENVIRONMENT
SNCR_ALLOWED_HOSTS
```

O cliente deve validar allowlist de host para impedir exfiltração de bearer token por SSRF/configuração maliciosa.

## 6. Fluxo de autenticação Gov.br

### 6.1 Início

A instrução oficial demonstra:

```http
GET {SNCR_BASE_URL}/api/v1/auth/login?client_url={URL_FRONTEND}
```

O `client_url` deve ser a URL completa de retorno do frontend.

### 6.2 Redirecionamento

O SNCR conduz o fluxo de autenticação via Gov.br e redireciona de volta ao `client_url` com:

```text
?session_id=...
```

### 6.3 Troca por token

A instrução demonstra:

```http
GET {SNCR_BASE_URL}/api/v1/auth/token?session_id={SESSION_ID}
```

Em sucesso, a resposta inclui `access_token`.

### 6.4 Uso

Requisições protegidas usam:

```http
Authorization: Bearer {access_token}
Content-Type: application/json
```

### 6.5 Expiração

A documentação orienta tratar expiração/401 removendo a sessão e redirecionando para novo login.

## 7. Melhoria de segurança em relação ao exemplo didático

A documentação oficial mostra implementação frontend e recomenda `sessionStorage` em vez de `localStorage`. Para nosso produto, preferimos uma arquitetura ainda mais defensiva:

### Opção recomendada — BFF

```text
Browser
  ↓
Nosso BFF
  ↓ redirect
SNCR/Gov.br
  ↓ callback session_id
Nosso BFF
  ↓ troca session_id
SNCR token
  ↓
Sessão server-side criptografada
  ↓
Browser recebe somente cookie HttpOnly/Secure/SameSite
```

Benefícios:

- bearer token SNCR não fica acessível a JavaScript;
- reduz impacto de XSS;
- centraliza refresh/reautenticação;
- melhora redaction e auditoria.

Essa variação deve ser validada contra as restrições atuais de `client_url`/callback da API. Se a API exigir retorno direto ao frontend, trocar o `session_id` imediatamente e enviar o token ao backend por canal seguro, evitando persistência longa no browser.

## 8. Exemplo de rota de receita branca presente na instrução

A instrução v1.0 apresenta como exemplo:

```http
POST /api/v1/receita-branca/
Authorization: Bearer ...
Content-Type: application/json
```

Payload de exemplo oficial contém campos equivalentes a:

```json
{
  "conselho": "CRM",
  "tipo": "RET",
  "documento": "...",
  "uf": "SP",
  "cnpj": "..."
}
```

### Importante

Este exemplo **não deve ser tratado como especificação completa da API**. Antes de implementar cada operação, devemos importar/consultar a especificação técnica vigente e gerar contratos tipados.

## 9. Cliente SNCR isolado

Criar package `sncr-client`.

Estrutura sugerida:

```text
packages/sncr-client/
├─ src/
│  ├─ auth/
│  ├─ prescriptions/
│  ├─ numbering/
│  ├─ dto/
│  ├─ errors/
│  ├─ transport/
│  └─ telemetry/
└─ tests/
   ├─ contract/
   └─ fixtures/
```

Interface pública sugerida:

```ts
interface SncrClient {
  getLoginUrl(input: { clientUrl: string; state?: string }): string;
  exchangeSession(sessionId: string): Promise<SncrSession>;
  createOrRequestPrescription(input: unknown, ctx: SncrContext): Promise<unknown>;
  health(): Promise<SncrHealth>;
}
```

As interfaces finais devem refletir a API oficial, não o exemplo acima.

## 10. Sessão SNCR

Entidade lógica:

```text
SncrSession
- id
- tenant_id
- prescriber_id
- access_token_encrypted
- expires_at
- created_at
- last_used_at
- status
- environment
```

### Regras

- token criptografado;
- nunca retornar token pela API pública;
- nunca logar;
- vincular ao prescritor correto;
- expirar automaticamente;
- excluir/revogar quando possível no logout;
- limpar token após expiração;
- acesso restrito ao módulo SNCR.

## 11. CORS e domínio

A instrução v1.0 informa suporte a CORS e frontend em domínio `.br`.

Mesmo assim:

- validar o domínio final com ambiente oficial;
- não assumir wildcard CORS;
- preferir BFF;
- manter allowlist de origens;
- separar URLs de callback por ambiente.

## 12. Timeouts

Valores iniciais sugeridos, sujeitos a benchmark:

- connect timeout: 3–5s;
- response timeout: 15–30s;
- workflow timeout superior, sem manter request HTTP aberta indefinidamente.

Operações longas podem retornar `202 Accepted` e serem concluídas por workflow.

## 13. Retry

### Pode ter retry automático

- GET idempotente;
- consulta sem efeito colateral;
- operação com idempotency key oficialmente suportada;
- falha de rede antes de qualquer possibilidade de efeito, quando demonstrável.

### Não repetir automaticamente sem proteção

- emissão;
- consumo de numeração;
- registro que possa produzir efeito externo;
- qualquer POST sem garantia de idempotência.

Nesses casos: estado `UNKNOWN_EXTERNAL_RESULT` + reconciliação.

## 14. Circuit breaker

Abrir circuito em falhas repetidas e:

- bloquear novas emissões dependentes do serviço;
- preservar rascunhos;
- informar indisponibilidade sem inventar sucesso;
- iniciar reconciliação após recuperação;
- gerar alerta operacional.

## 15. Idempotência local

Cada comando de emissão deve possuir:

```text
idempotency_key
prescription_id
tenant_id
operation
payload_hash
status
external_reference
created_at
```

Se a mesma chave chegar com payload diferente: `409 Conflict`.

## 16. Reconciliação

Criar worker periódico para casos:

- timeout após envio;
- HTTP 5xx com resultado incerto;
- job reiniciado;
- aplicação caiu após resposta externa e antes do commit local;
- documento assinado mas status não finalizado.

Estados possíveis:

- `CONFIRMED`
- `NOT_FOUND`
- `RETRY_SAFE`
- `MANUAL_REVIEW`
- `EXTERNAL_INCONSISTENCY`

## 17. Erros normalizados

Exemplo interno:

```json
{
  "code": "SNCR_AUTH_EXPIRED",
  "category": "AUTHENTICATION",
  "retryable": false,
  "httpStatus": 401,
  "externalStatus": 401,
  "correlationId": "..."
}
```

Categorias:

- AUTHENTICATION;
- AUTHORIZATION;
- VALIDATION;
- RATE_LIMIT;
- CONFLICT;
- EXTERNAL_UNAVAILABLE;
- TIMEOUT;
- UNKNOWN_RESULT;
- CONTRACT_CHANGED.

## 18. Contract tests

Na CI e no ambiente de treinamento:

- validar schema de resposta;
- validar campos obrigatórios;
- validar comportamento de 401;
- validar redirecionamento/callback;
- validar payloads conhecidos;
- detectar breaking changes;
- manter fixtures anonimizadas.

## 19. Segurança de transporte

- TLS válido;
- validação de hostname;
- sem `rejectUnauthorized=false`;
- sem proxy não autorizado;
- DNS/egress controlado em produção;
- allowlist para domínios oficiais;
- redaction completa de `Authorization` e `session_id`.

## 20. Checklist para primeira integração real

- [ ] confirmar URL do ambiente atual;
- [ ] confirmar URL de produção atual;
- [ ] cadastrar/testar prescritor no ambiente de treinamento;
- [ ] implementar login;
- [ ] implementar callback;
- [ ] trocar `session_id` uma única vez;
- [ ] armazenar token com segurança;
- [ ] testar 401;
- [ ] implementar cliente tipado;
- [ ] implementar operação mínima disponível;
- [ ] gerar traces sem PII/token;
- [ ] testar timeout;
- [ ] testar queda/recovery;
- [ ] implementar reconciliação;
- [ ] validar contrato com documentação vigente;
- [ ] registrar versão da API/documentação usada no release.

## 21. Fonte oficial

Instruções de Integração API SNCR v1.0:

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/documentos-do-sncr/instrucoes-de-integracao-api-sncr-v1-0.pdf
