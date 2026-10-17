# Segurança de IA — OWASP 2026

> **Estado:** requisitos documentais propostos; não evidenciam implementação, certificação, teste ou uso em produção. Atualizado em 17/10/2026.

## Escopo
Aplicável a LLM, RAG, embeddings, agentes, MCP, A2A, ferramentas, memória e IA assistiva. Complementa políticas existentes de segurança, LGPD, SDLC e resposta a incidentes. Nenhum prompt é considerado controle efetivo de autorização; controles obrigatórios devem residir em código, serviços, políticas, banco e infraestrutura.

## OWASP Top 10 for LLM Applications 2026
| ID | Risco | Exigência | Evidência de teste |
|---|---|---|---|
| LLM01:2026 | Prompt Injection | Entrada externa nunca autoriza ferramenta, altera policy ou redefine instruções. | Teste de prompt injection direto, indireto e multimodal. |
| LLM02:2026 | Sensitive Information Disclosure | Classificar, minimizar e aplicar ACL/tenant em prompt, retrieval, log e resposta. | Teste de vazamento de PII, segredo e cross-tenant. |
| LLM03:2026 | Excessive Agency | Capacidades mínimas por identidade/recurso; HIL para operações críticas. | Tool call não autorizada deve falhar fechado. |
| LLM04:2026 | Supply Chain | Fixar, auditar e verificar proveniência de modelos, pacotes, dados e MCP. | Dependência alterada ou não aprovada deve ser recusada. |
| LLM05:2026 | Data and Model Poisoning | Verificar origem/hash de dados, índices, modelos e memória; quarentena de fontes externas. | Poisoning não deve promover dados a fonte confiável. |
| LLM06:2026 | Unbounded Consumption | Cota de tokens, custo, chamadas, tempo, retries e concorrência por tenant. | Loop de agente e excesso de custo devem ser interrompidos. |
| LLM07:2026 | Misinformation | Exigir fontes, frescor, incerteza e revisão humana em decisões sensíveis. | Resposta sem evidência não pode virar decisão automática. |
| LLM08:2026 | Hidden Context Exposure | Não guardar segredos em prompts; proteger políticas e contexto interno. | Ataque de extração não revela contexto restrito. |
| LLM09:2026 | Vector and Embedding Weaknesses | Autorizar busca RAG na indexação, recuperação, cache e memória. | Chunks de outro tenant nunca alcançam o contexto. |
| LLM10:2026 | Improper Output Handling | Saídas do LLM são dados; validar schemas, SQL parametrizado, HTML escapado. | Payload da IA não executa script, SQL ou comando. |

**Atenção:** os IDs devem conter :2026; categorias foram renumeradas em relação a 2025.

## OWASP Top 10 for Agentic Applications 2026
| ID | Categoria | Regra adicional |
|---|---|---|
| ASI01 | Agent Goal Hijack | Conteúdo externo não muda objetivos do agente. |
| ASI02 | Tool Misuse & Exploitation | Ferramenta exige schema e autorização externa. |
| ASI03 | Identity & Privilege Abuse | Delegação escopada e credencial de curta duração. |
| ASI04 | Agentic Supply Chain Vulnerabilities | Conferir origem e integridade de plugins/MCP. |
| ASI05 | Unexpected Code Execution | Sandbox e sem shell/egress amplo por padrão. |
| ASI06 | Memory & Context Poisoning | TTL, procedência e validação de memória. |
| ASI07 | Insecure Inter-Agent Communication | Autenticação e integridade de mensagem A2A. |
| ASI08 | Cascading Failures | Timeout e circuit breaker nas cadeias. |
| ASI09 | Human-Agent Trust Exploitation | HIL real com ação, evidência e impacto. |
| ASI10 | Rogue Agents | Inventário, revogação e kill switch. |

## Controles arquiteturais obrigatórios
**SEC-AI-01 — Trust boundaries:** classificar entrada de usuário, documento, web, ferramentas e memória; não promover conteúdo recuperado a comandos privilegiados. Testar ataques diretos e indiretos.
**SEC-AI-02 — Autorização fail-closed:** PEP/PDP ou middleware devem validar identidade, tenant, propósito, recurso, operação e ferramenta fora do LLM. Aplicar RLS como defesa adicional.
**SEC-AI-03 — Ferramentas e egress:** servidores MCP/agentes A2A identificados, allowlist de capabilities, schemas validados, destinos de rede permitidos e proteção contra SSRF. Não repassar segredos a terceiros.
**SEC-AI-04 — HIL:** aprovação humana autenticada para gravação, deleção, publicação, mensagens externas, decisões regulatórias, intervenções OT ou ação de impacto. Aprovação limitada à ação, alvo, diff, custo e prazo, com registro e revogação.
**SEC-AI-05 — Tenant e RAG:** aplicar ACL em ingestão, índices, retrieval, rerank, contexto, cache, memória, embeddings e resposta; testar cross-tenant. A origem de um chunk não confere autorização.
**SEC-AI-06 — Privacidade:** minimização, retenção/eliminação, criptografia, secrets manager, auditoria de transferência para modelos externos e revisão LGPD. Não gravar prompt/trace bruto sensível por padrão.
**SEC-AI-07 — Output:** validação estrita de JSON Schema, SQL parametrizado, HTML escapado, URL allowlist, sandbox sem execução arbitrária. Nunca usar texto de LLM como código executável por padrão.
**SEC-AI-08 — Evidência e observabilidade:** registrar request/trace, agent/model version, tenant pseudonimizado, fontes, tools, policy decision, HIL, custo, timestamp e erros, sem segredos ou PII desnecessários.
**SEC-AI-09 — Resiliência e custos:** quota de tokens/chamadas/custo por tenant, timeouts, rate limits, retries limitados, idempotência, circuit breaker e fallback seguro.
**SEC-AI-10 — Supply chain:** inventário e versões de modelo, prompt, dataset, imagem, pacote, ferramenta e servidor MCP; verificação de proveniência e promoção entre ambientes com gate.
**SEC-AI-11 — Testes adversariais:** cobrir prompt injection, exfiltração, vazamento cross-tenant, poisoning de RAG/memória, tool misuse, SQL/XSS/RCE/SSRF, runaway cost e cascata, com testes reproduzíveis de regressão.
**SEC-AI-12 — Incidentes:** capacidade de desativar agente, revogar credenciais, isolar índice/tenant, preservar evidência sanitizada e escalar decisão para responsável de segurança. Aceitação de risco residual é humana.

## Gate obrigatório antes da liberação de funcionalidade de IA
- [ ] Modelo de ameaça e fronteiras de confiança documentados por componente com IDs OWASP e responsável.
- [ ] Classe, fonte, finalidade, autorização, retenção e uso de modelos externos revistos.
- [ ] Testes negativos de acesso entre tenants, ferramentas e ações críticas; backend nega por padrão.
- [ ] Red-team de injeção, memória/RAG contaminados, vazamento e output inseguro reproduzido em sandbox.
- [ ] Limites de custo, quota, timeout, fallback e trilha de auditoria verificados.
- [ ] Evidências referenciadas por commit/ambiente/data e aprovadas por segurança; rollback documentado.

**Bloqueadores:** vazamento entre tenants, segredo exposto, execução remota, privilégios indevidos e ação crítica sem autorização humana impedem liberação.

## Referências oficiais
- OWASP LLM 2026 (04/08/2026): https://github.com/GenAI-Security-Project/GenAI-LLM-Top10/blob/main/2026/README.md
- OWASP Agentic 2026 (09/12/2025): https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
- OWASP Agentic Security Initiative / MCP: https://genai.owasp.org/initiatives/agentic-security-initiative/

Esta baseline é adaptação técnica dos frameworks, não cópia de conteúdo oficial nem atestado de conformidade.

## Aplicação específica — SNCR
IA eventual é estritamente assistiva; não prescreve, assina, cancela receitas ou decide clinicamente. Profissional autorizado responde por decisões clínicas.
Não enviar dados de saúde a modelos externos sem base e salvaguardas. Tokens Gov.br, certificados de assinatura e credenciais SNCR jamais entram em prompts ou ferramentas de IA.
Normas da Anvisa devem estar versionadas e verificadas; divergência ou indisponibilidade deve bloquear automação e escalar ao humano.
Complementa [segurança e LGPD](06-seguranca-lgpd-assinatura.md) e [riscos](10-riscos-e-validacao.md).
