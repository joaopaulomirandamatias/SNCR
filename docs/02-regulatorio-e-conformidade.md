# 02 — Regulatório e Conformidade

## 1. Escopo regulatório

O projeto deve ser tratado como software de prescrição eletrônica integrado a infraestrutura regulatória da Anvisa. A implementação deve distinguir claramente:

- requisitos legais/regulatórios obrigatórios;
- requisitos técnicos publicados pela Anvisa;
- controles internos de segurança e qualidade adotados pelo produto;
- hipóteses de arquitetura ainda sujeitas a validação.

A fonte de verdade regulatória deve ser a documentação oficial vigente da Anvisa, não esta documentação isoladamente.

## 2. Base normativa e documental a acompanhar

### 2.1 RDC 1.000/2025

É a referência central da nova disciplina de receituários físicos e eletrônicos abrangidos pelo SNCR, com alterações nas regras relacionadas às Portarias SVS/MS 344/1998 e 6/1999.

Pontos que impactam diretamente o produto:

- emissão eletrônica integrada ao SNCR;
- modelos oficiais de receituários;
- regras específicas por tipo de notificação/receita;
- assinatura eletrônica aplicável;
- rastreabilidade e utilização;
- coexistência de meio físico e eletrônico.

### 2.2 RDC 1.028/2026

Prorrogou para 30/09/2026 a disponibilização das funcionalidades eletrônicas do SNCR, sem revogar as demais regras já vigentes da RDC 1.000/2025.

### 2.3 Portaria SVS/MS 344/1998

Continua essencial para classificação e controle de substâncias/medicamentos sujeitos a controle especial. O policy engine do produto deve ser parametrizado a partir da regra vigente e suas atualizações.

### 2.4 Portaria SVS/MS 6/1999

Complementa operacionalmente aspectos da Portaria 344/1998 e deve integrar o inventário normativo.

### 2.5 RDC 471/2021 e normas correlatas

Relevante para antimicrobianos e prescrições sujeitas a retenção conforme enquadramento aplicável. Não assumir que todos os receituários sujeitos a retenção têm o mesmo modelo eletrônico ou as mesmas exigências do controle especial.

## 3. Situação operacional a partir de 30/09/2026

Segundo a Anvisa, a partir de 30/09/2026 inicia-se uma nova etapa do SNCR com funcionalidades de emissão e controle eletrônico. Isso **não elimina a receita em papel** e não significa que todos os atores precisem migrar simultaneamente.

Para sistemas de prescrição, o ponto central é: o profissional continua usando sua plataforma; a plataforma integrada consome os serviços necessários do SNCR.

## 4. Tipos de receituário contemplados na etapa eletrônica

A comunicação oficial da Anvisa cita, entre outros:

- Notificação de Receita A;
- Notificação de Receita B;
- Notificação de Receita B2;
- Notificação para retinoides de uso sistêmico;
- Notificação para talidomida;
- Receita de Controle Especial;
- receitas sujeitas à retenção, conforme escopo e regras aplicáveis.

A implementação não deve codificar essa lista como conjunto fechado permanente. Deve existir catálogo versionado de tipos de receituário e feature flags regulatórias.

## 5. Numeração

Para Notificações de Receita, permanece a necessidade de solicitação/autorização da numeração perante a Vigilância Sanitária local. A Anvisa informa que as numerações destinadas à emissão eletrônica são distintas das destinadas ao meio físico.

### Requisitos de sistema

- nunca inventar numeração local;
- nunca reutilizar número já consumido;
- associar número ao prescritor/escopo aplicável;
- guardar status e origem da numeração;
- impedir corrida concorrente na alocação;
- usar transações/locks/idempotência;
- reconciliar estado local com SNCR;
- registrar evidência de solicitação, retorno, consumo e erro.

## 6. Prescritores

A Instrução de Integração API SNCR v1.0 informa que:

- o SNCR possui fluxo automatizado de cadastro de prescritores;
- é necessário acesso do profissional ao sistema pelo menos uma vez para novos usuários;
- o profissional deve possuir inscrição ativa em conselho profissional consultado pelo SNCR;
- o documento de julho/2026 cita CFM, CFMV e CFO;
- os dados precisam existir na base do SNCR para disponibilização de numerações via API.

### Implicação para onboarding

O produto deve possuir um estado explícito de elegibilidade SNCR do prescritor, por exemplo:

- `NOT_CHECKED`
- `PENDING_SNCR_ENROLLMENT`
- `ELIGIBLE`
- `INELIGIBLE_COUNCIL`
- `AUTH_REQUIRED`
- `TEMPORARY_ERROR`

O status local nunca deve ser apresentado como homologação permanente; deve possuir data, origem e expiração/revalidação.

## 7. Credenciamento da plataforma

A Instrução de Integração v1.0 de julho/2026 afirma que **o credenciamento de plataformas não será necessário**, sendo suficientes as validações do profissional e, futuramente, da organização mantenedora da plataforma.

### Decisão de projeto

- não criar um fluxo fictício de “credenciamento Anvisa” no MVP;
- preparar cadastro de `platform_organization` e metadados necessários para futura validação;
- acompanhar informes da API, pois esse ponto pode evoluir;
- registrar versão da documentação oficial usada em cada release.

## 8. Modelos eletrônicos

A página oficial de Receituário Eletrônico informa que os modelos eletrônicos disponibilizados pela Anvisa são exclusivos para uso em receituário eletrônico e **não podem ser customizados**.

### Consequências

O `Document Service` deve:

- manter templates oficiais versionados;
- impedir alteração de estrutura regulatória por tenant;
- permitir identidade visual apenas fora do documento regulatório, se juridicamente permitido;
- ter testes de regressão visual;
- registrar hash do template usado;
- registrar versão do template e da regra;
- bloquear emissão se o template obrigatório estiver desatualizado.

## 9. QR Code

A Anvisa publica o padrão para QR Code da Notificação de Receita eletrônica:

`https://sncr.anvisa.gov.br/receita/consultar?numero={numero}`

O número deve ser substituído pelo identificador correspondente no formato oficial documentado.

### Requisitos

- QR deve ser gerado a partir do número final validado;
- o conteúdo do QR deve ser determinístico;
- não inserir PII no QR além do que o padrão oficial determinar;
- testar leitura em dispositivos comuns;
- manter margem/contraste/tamanho suficientes;
- teste automatizado deve decodificar o QR gerado e comparar URL esperada.

## 10. SNCR x SNGPC

Os sistemas não são substitutos.

### SNCR

Foco em receituários: numeração, emissão eletrônica e registro/controle de utilização dentro das funções disponibilizadas.

### SNGPC

Foco na escrituração e controle da movimentação de medicamentos em farmácias e drogarias.

### Consequência arquitetural

O produto de prescrição pode integrar futuramente dados ou serviços relacionados ao SNGPC somente se houver base jurídica, necessidade de negócio e interface oficial adequada. Não misturar responsabilidades no domínio inicial.

## 11. Assinatura eletrônica

A política de assinatura deve ser decidida por tipo de receituário e enquadramento legal vigente.

Para medicamentos sujeitos a controle especial, a orientação oficial da Anvisa deve ser tratada como requisito de alta criticidade. Quando for exigida assinatura eletrônica qualificada, a implementação deve utilizar certificado ICP-Brasil e validação compatível.

### Regra de arquitetura

Nunca colocar a regra de assinatura somente na UI. O backend precisa recusar emissão incompatível com a política regulatória.

Exemplo de decisão:

```text
prescription -> classification -> prescription_type -> signature_policy
                                              -> SNCR policy
                                              -> template policy
                                              -> validity policy
```

## 12. Regras regulatórias computáveis

Criar uma camada de `Regulatory Policy Engine` com objetos versionados:

```yaml
rule_id: RX-SIGN-001
version: 2026.09
scope:
  prescription_type: NR_B
requirements:
  signature_level: QUALIFIED_ICP_BRASIL
  sncr_required: true
  official_template: NR_B_ELECTRONIC_V2
status: active
source:
  authority: ANVISA
  reference: RDC/official guidance
```

Toda decisão deve gravar:

- `rule_id`;
- versão;
- timestamp;
- entrada relevante minimizada;
- resultado;
- justificativa técnica;
- fonte normativa.

## 13. Gestão de mudança regulatória

Criar processo formal:

1. monitorar Anvisa/SNCR;
2. abrir change request regulatório;
3. classificar impacto;
4. atualizar catálogo de normas;
5. alterar regras atrás de feature flag quando necessário;
6. escrever testes;
7. revisar por jurídico/compliance;
8. homologar;
9. ativar em data programada;
10. manter regra anterior para auditoria histórica.

## 14. Requisitos para farmácias não são automaticamente requisitos do nosso produto

A Anvisa criou perfis e fluxos específicos para farmácias/drogarias no Cadastro Anvisa e SNCR. No MVP, nosso sistema será de prescrição, não de dispensação. Portanto:

- não implementar perfil `SNCR-Farmácia` como se fosse nosso;
- não registrar dispensação em nome da farmácia sem escopo próprio e autorização;
- considerar módulo de farmácia apenas em fase futura e separada.

## 15. Checklist regulatório antes da produção

- [ ] confirmar versão vigente da RDC 1.000/2025 e alterações;
- [ ] confirmar versão da API SNCR e informes recentes;
- [ ] confirmar tipos de receituário eletrônicos efetivamente disponíveis;
- [ ] confirmar regras de assinatura por tipo;
- [ ] confirmar modelos eletrônicos vigentes;
- [ ] confirmar formato do QR Code;
- [ ] confirmar ambiente de treinamento/homologação;
- [ ] confirmar comportamento de numeração;
- [ ] confirmar requisitos de organização mantenedora da plataforma;
- [ ] revisar termos de uso da API;
- [ ] revisar bases legais LGPD e contratos;
- [ ] realizar avaliação de segurança;
- [ ] executar testes ponta a ponta;
- [ ] validar contingência/indisponibilidade;
- [ ] obter parecer jurídico/compliance antes de produção.

## 16. Fontes primárias

Ver [`11-fontes-oficiais.md`](11-fontes-oficiais.md).