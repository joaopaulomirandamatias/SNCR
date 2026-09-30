# 11 — Fontes Oficiais e Referências

## 1. Regra de uso

Este projeto deve usar fontes primárias da Anvisa e legislação oficial como fonte de verdade. Notícias, apresentações, vídeos e respostas de terceiros podem ajudar na interpretação, mas não substituem a norma e a documentação técnica vigente.

Sempre registrar:

- URL;
- título;
- órgão;
- data de publicação/modificação;
- versão do documento;
- data em que a equipe revisou;
- quais regras/códigos dependem da fonte.

## 2. Portal oficial do SNCR

**Sistema Nacional de Controle de Receituários — Anvisa**

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr

Uso:

- ponto de entrada oficial;
- manuais;
- modelos;
- comunicados;
- perguntas e respostas;
- atualizações.

## 3. Serviços de prescrição eletrônica

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/documentos-do-sncr

A página oficial informa que serviços de prescrição eletrônica integram suas plataformas ao SNCR e concentra material técnico, utilização de numerações e orientações.

**Revisão observada:** página modificada em 29/09/2026.

## 4. Instruções de Integração API SNCR v1.0

Landing page:

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/documentos-do-sncr/instrucoes-de-integracao-api-sncr-v1-0.pdf

Arquivo PDF oficial:

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/documentos-do-sncr/instrucoes-de-integracao-api-sncr-v1-0.pdf/@@display-file/file

**Versão:** JUL/2026 — v1.0.

Pontos confirmados no documento:

- cadastro automatizado de prescritores;
- necessidade de acesso ao SNCR para novos usuários;
- conselho ativo (documento cita CFM, CFMV e CFO);
- credenciamento prévio de plataforma não necessário segundo a versão 1.0;
- integração com login Gov.br;
- exemplo de base URL `https://sncr-api.apps.anvisa.gov.br`;
- `GET /api/v1/auth/login?client_url=...`;
- callback com `session_id`;
- `GET /api/v1/auth/token?session_id=...`;
- Bearer token nas requisições protegidas;
- exemplo de `POST /api/v1/receita-branca/`;
- suporte indicado a frontend em domínio `.br`;
- orientação para tratar 401/expiração;
- recomendação de `sessionStorage` em vez de `localStorage` no exemplo frontend.

## 5. Receituário eletrônico

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/receituario-eletronico

Pontos confirmados:

- modelos eletrônicos são específicos para o meio eletrônico;
- não é permitida a customização dos modelos oficiais;
- padrão do QR Code da Notificação de Receita:

```text
https://sncr.anvisa.gov.br/receita/consultar?numero={numero}
```

A implementação deve consultar a página novamente sempre que houver atualização de modelo.

## 6. Nova etapa em 30/09/2026

**Início de nova etapa do SNCR: o que muda a partir de 30 de setembro**

https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/2026/inicio-de-nova-etapa-do-sncr-o-que-muda-a-partir-de-30-de-setembro

Publicado em 21/09/2026.

Pontos confirmados:

- início de nova etapa em 30/09/2026;
- coexistência de receitas físicas e eletrônicas;
- adoção gradual;
- sistemas de prescrição se integram ao SNCR;
- Notificações A, B, B2, retinoides sistêmicos e talidomida contempladas na comunicação;
- numeração da Vigilância Sanitária local continua necessária para Notificações;
- numeração eletrônica é distinta da física;
- SNCR não substitui SNGPC.

## 7. Publicação da documentação técnica

**Anvisa publica documentação técnica para integração de sistemas de prescrição eletrônica ao SNCR**

https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/anvisa-publica-documentacao-tecnica-para-integracao-de-sistemas-de-prescricao-eletronica-ao-sncr

Publicado em 30/06/2026.

Confirma que o material é destinado a:

- desenvolvedores;
- empresas de software;
- plataformas de prescrição eletrônica;
- equipes de TI;
- instituições interessadas.

E que inclui:

- API;
- autenticação/comunicação;
- treinamento;
- especificações técnicas;
- modelos.

## 8. Prazos e RDC 1.028/2026

**Receituários de medicamentos controlados: Anvisa esclarece prazos e regras em vigor**

https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/2026/receituarios-de-medicamentos-controlados-anvisa-esclarece-prazos-e-regras-em-vigor

Confirma:

- RDC 1.028/2026 prorrogou as funcionalidades eletrônicas para 30/09/2026;
- não revogou as demais disposições da RDC 1.000/2025;
- novos modelos físicos já possuíam vigência própria.

## 9. Perfis de acesso SNCR

**SNCR: perfis de acesso ao sistema estão disponíveis no Cadastro Anvisa**

https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/2026/sncr-perfis-de-acesso-ao-sistema-estao-disponiveis-no-cadastro-anvisa-a-partir-desta-terca-29-9

Publicado em 29/09/2026.

É especialmente relevante para entender atores de farmácia/drogaria e Vigilâncias Sanitárias. Não confundir esses perfis com os papéis internos do nosso SaaS.

## 10. Orientação a farmácias

https://www.gov.br/anvisa/pt-br/assuntos/noticias-anvisa/2026/como-farmacias-publicas-e-privadas-devem-se-preparar-para-o-sncr

Uso:

- entender a ponta de dispensação;
- não implementar responsabilidades de farmácia no MVP sem escopo explícito;
- mapear integração futura.

## 11. Perguntas e respostas RDC 1.000/2025

https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/perguntas-e-respostas/perguntas-e-respostas-rdc-1000-2025-1-ed

Usar para interpretação operacional, incluindo coexistência de receituários, assinatura eletrônica e aplicação prática da RDC.

## 12. Normas que devem integrar o Regulatory Source Registry

Manter versões oficiais vigentes de:

- RDC Anvisa nº 1.000/2025;
- RDC Anvisa nº 1.028/2026;
- Portaria SVS/MS nº 344/1998;
- Portaria SVS/MS nº 6/1999;
- RDC nº 471/2021 e atualizações/normas substitutas aplicáveis;
- Instruções Normativas relacionadas às substâncias/medicamentos abrangidos;
- Lei nº 13.709/2018 — LGPD;
- Lei nº 14.063/2020 — assinaturas eletrônicas, conforme aplicabilidade;
- regras e documentos vigentes da ICP-Brasil para assinatura qualificada usada na solução.

A equipe jurídica/regulatória deve confirmar o conjunto completo antes do go-live.

## 13. Registry sugerido

Criar futuramente arquivo versionado no código:

```yaml
sources:
  - id: ANVISA-SNCR-INTEGRATION-1.0
    authority: ANVISA
    title: Instruções de Integração API SNCR
    version: "1.0"
    published: 2026-07
    reviewed_at: 2026-09-30
    criticality: critical
    url: "https://www.gov.br/..."
    affects:
      - sncr-auth
      - sncr-client
      - prescriber-onboarding
```

## 14. Rotina de atualização

Antes de cada release regulatório:

1. revisar página principal SNCR;
2. revisar `Documentos do SNCR`;
3. revisar `INFORMES API SNCR`;
4. verificar nova versão de manual/API;
5. verificar novos templates;
6. verificar RDC/IN publicadas;
7. criar diff de impacto;
8. atualizar regras e testes;
9. registrar aprovação.

## 15. Regra de documentação

Sempre distinguir nos documentos do projeto:

- **OFICIAL/OBRIGATÓRIO:** derivado de norma/documento oficial;
- **DECISÃO DE ARQUITETURA:** escolha nossa;
- **RECOMENDAÇÃO:** prática de engenharia;
- **A CONFIRMAR:** ponto ainda não verificado na versão vigente da API/norma.

Isso evita transformar uma escolha técnica em falsa exigência da Anvisa.