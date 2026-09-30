# 01 — Visão do Produto

## 1. Problema

A evolução do Sistema Nacional de Controle de Receituários (SNCR) cria uma camada nacional para controle e emissão eletrônica de receituários abrangidos pela Anvisa. Profissionais e organizações continuarão utilizando sistemas de prescrição, mas esses sistemas precisam integrar os serviços oficiais do SNCR quando utilizarem os fluxos eletrônicos abrangidos.

O produto proposto resolve três problemas simultâneos:

1. oferecer uma experiência simples e segura de prescrição para profissionais de saúde;
2. encapsular a complexidade regulatória e técnica da integração com SNCR/Gov.br/assinatura digital;
3. oferecer uma API B2B para outros sistemas de saúde que desejem consumir essa infraestrutura sem reimplementar toda a camada regulatória.

## 2. Visão

Construir uma plataforma independente de prescrição eletrônica, multi-tenant e auditável, com um núcleo regulatório versionado e integrações oficiais, capaz de operar tanto como SaaS quanto como infraestrutura B2B.

### Produto A — Prescrição SaaS

Público:

- médicos;
- odontólogos;
- médicos-veterinários;
- clínicas;
- hospitais;
- redes de saúde;
- telemedicina;
- secretarias e instituições públicas, mediante requisitos próprios de contratação e operação.

Capacidades:

- cadastro/gestão de organização;
- profissionais e vínculos;
- pacientes;
- criação e validação de prescrição;
- classificação regulatória;
- integração SNCR;
- assinatura digital;
- geração de documento oficial;
- QR Code;
- histórico;
- auditoria;
- notificações;
- API e webhooks.

### Produto B — Health Regulatory Gateway / SNCR Gateway

Público:

- ERPs clínicos;
- prontuários eletrônicos;
- HIS;
- plataformas de telemedicina;
- healthtechs;
- integradores.

Capacidades:

- autenticação federada;
- API de validação regulatória;
- integração SNCR;
- assinatura;
- documentos;
- auditoria;
- idempotência;
- webhooks;
- SDKs.

## 3. Fronteiras do produto

### O sistema É

- plataforma privada integrada ao SNCR;
- sistema próprio com código, dados e deploy independentes;
- motor de workflow regulatório;
- gerador e repositório de evidências de emissão;
- camada de integração com terceiros;
- potencial gateway B2B.

### O sistema NÃO É

- substituto do SNCR;
- substituto do SNGPC;
- autoridade sanitária;
- conselho profissional;
- certificadora ICP-Brasil;
- sistema que autoriza numeração por conta própria;
- sistema que modifica modelos oficiais da Anvisa livremente;
- agente de IA autorizado a prescrever autonomamente;
- prontuário eletrônico completo no MVP.

## 4. Atores

| Ator | Responsabilidade no produto |
|---|---|
| Prescritor | autenticar, selecionar paciente, prescrever, revisar e assinar |
| Organização/tenant | gerir profissionais, unidades, permissões e configurações |
| Administrador | gestão operacional do tenant |
| Paciente | receber/acessar documento e orientações |
| Suporte autorizado | suporte sem acesso indevido a dados clínicos |
| Compliance/DPO | governança, incidentes, solicitações LGPD e auditoria |
| Sistema externo | consumir API B2B mediante contrato e autenticação |
| SNCR/Anvisa | numeração/serviços e controle oficial conforme API |
| Gov.br | identidade/autenticação do fluxo oficial |
| Provedor ICP-Brasil | assinatura qualificada quando exigida |
| Farmácia/drogaria | consulta e registro da utilização pelos fluxos oficiais |

## 5. Jornadas prioritárias

### 5.1 Onboarding da organização

1. criar tenant;
2. cadastrar dados da organização;
3. definir unidades;
4. cadastrar administradores;
5. cadastrar/vincular profissionais;
6. configurar política de acesso;
7. configurar integrações e assinatura;
8. validar ambiente;
9. habilitar operação.

### 5.2 Onboarding do prescritor

1. criar/vincular identidade interna;
2. registrar conselho, UF e número;
3. confirmar dados profissionais;
4. executar pré-condições exigidas pelo SNCR;
5. autenticar via fluxo Gov.br quando necessário;
6. registrar evidência da validação;
7. configurar método de assinatura;
8. habilitar emissão elegível.

### 5.3 Emissão

1. selecionar/criar paciente;
2. selecionar medicamento/substância/apresentação;
3. preencher posologia e dados clínicos necessários;
4. executar regras regulatórias;
5. validar prescritor, tenant e permissões;
6. determinar tipo de receituário;
7. determinar exigência de numeração SNCR;
8. reservar/obter numeração quando aplicável;
9. montar documento conforme modelo oficial;
10. assinar conforme regra aplicável;
11. registrar emissão/serviço oficial;
12. gerar QR Code quando aplicável;
13. armazenar documento e evidências;
14. entregar ao paciente;
15. publicar evento e webhook.

### 5.4 Cancelamento/revogação

Deve existir apenas quando permitido pelo fluxo regulatório e pela API oficial. O sistema nunca deve implementar cancelamento local que deixe o estado oficial divergente.

## 6. Diferenciais pretendidos

- arquitetura regulatory-first;
- trilha de evidência por prescrição;
- policy engine regulatório versionado;
- multi-tenancy robusta;
- APIs e webhooks desde o início;
- observabilidade e correlação ponta a ponta;
- integração com assinatura remota;
- motor de validação antes da emissão;
- possibilidade de knowledge graph regulatório no futuro;
- IA apenas como assistência explicável, nunca como prescritor autônomo.

## 7. IA: escopo permitido inicialmente

Pode auxiliar em:

- preenchimento;
- identificação de inconsistências;
- explicação de regras ao prescritor;
- sumarização de histórico autorizado;
- alertas de duplicidade/interação a partir de fonte clínica validada;
- detecção de fraude/anomalia;
- suporte operacional.

Qualquer sugestão deve ser claramente identificada como apoio. A decisão e assinatura permanecem com o profissional responsável.

## 8. Métricas de produto

- taxa de emissão concluída;
- tempo mediano de emissão;
- erro por etapa de integração;
- taxa de reautenticação Gov.br;
- taxa de falha de assinatura;
- número de emissões duplicadas evitadas por idempotência;
- SLA da API;
- latência p95/p99;
- disponibilidade;
- incidentes de segurança;
- divergências entre estado local e oficial;
- chamados por 1.000 prescrições;
- taxa de sucesso de webhooks.

## 9. Decisão estratégica

A primeira implementação deve ser uma **plataforma própria de prescrição com núcleo de gateway reutilizável internamente**. Não construir o gateway e o SaaS como produtos totalmente separados no início; compartilhar domínio e infraestrutura, preservando interfaces bem definidas para separação futura.