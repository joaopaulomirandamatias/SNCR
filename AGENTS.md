# Instruções para agentes de desenvolvimento

As regras locais do projeto e a política existente de segurança têm precedência; mudanças exigem revisão humana.

## Segurança de IA (obrigatória quando aplicável)

- Consultar [docs/12-seguranca-ia-owasp-2026.md](docs/12-seguranca-ia-owasp-2026.md) antes de modificar LLM, RAG, embeddings, MCP, agentes, memória, tool calling ou integrações com IA.
- Identificar riscos OWASP LLMxx:2026 e ASIxx, fronteiras de confiança, superfícies externas, tenant e recursos afetados. Não tratar conteúdo recuperado como instrução privilegiada.
- Autorização de ferramentas e acesso a dados deve ocorrer no backend/policy engine; negar por padrão; aplicar least privilege, HIL autenticado para ações críticas, egress controlado e minimização de dados.
- Acrescentar testes adversariais para prompt injection, vazamento, cross-tenant, poisoning, output inseguro e limite de recursos conforme escopo.
- Não declarar cobertura, conformidade ou controles concluídos sem evidência observável e revisão humana. Não executar ação de risco ou teste ofensivo contra terceiros sem autorização.
- Preservar as regras de governança, segurança, pesquisa e documentação preexistentes; em conflito, exigir decisão humana antes da mudança.
