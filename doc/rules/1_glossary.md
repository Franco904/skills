# Glossário — Linguagem Ubíqua do Domínio e-SUS APS / PEC

> **Finalidade:** Este glossário define os termos e siglas do domínio da Atenção Primária à Saúde (APS) utilizados no aplicativo e-SUS Atendimento. Serve como referência canônica para garantir alinhamento entre código, testes, documentação e comunicação entre membros das equipes envolvidas (Linguagem Ubíqua — DDD).
>
> **Instrução para o Agente de IA:** Ao nomear classes, métodos, variáveis, módulos e testes, utilize **exatamente** os termos definidos neste glossário. Nunca adote jargões técnicos da indústria ou invente termos ambíguos, sinônimos ou abreviações não listadas aqui.
>
> **Nota sobre a coluna "Contexto Delimitado":** A linguagem ubíqua é delimitada (*bounded*) — um mesmo termo pode ter significados distintos em partes do sistema diferentes - contextos diferentes. Quando isso ocorre, o termo aparece em mais de uma linha, uma por BC. Ao nomear artefatos, sempre verifique em qual BC você está operando antes de escolher o conceito correto. Consulte `.claude/rules/2_bounded_contexts.md` para o mapeamento completo de contextos.

---

## A

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **ACS** | Autenticação / Seleção de Lotação | Agente Comunitário de Saúde. Categoria de profissional que pode estar associada a uma lotação |
| **Atendimento** | Documentação Clínica | Registro clínico de um profissional com um cidadão, com estado (rascunho / finalizado), associando SOAP, Folha de Rosto e Dados Pessoais. Armazenado no dispositivo até a sincronização com o PEC |
| **Atendimento** | Sincronização | Unidade de produção enviada ao PEC: uma FAI ou FP gerada a partir do registro clínico |

---

## C

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **CBO** | Autenticação / Seleção de Lotação | Classificação Brasileira de Ocupações. Associada à lotação do profissional; determina quais tipos de atendimento ele pode registrar |
| **CIAP-2** | Documentação Clínica — SOAP | Classificação Internacional de Atenção Primária, 2ª edição. Codifica motivos de consulta e problemas; usado em intervenções e avaliações durante o registro do SOAP |
| **CID-10** | Documentação Clínica — SOAP | Classificação Internacional de Doenças. Codifica diagnósticos médicos e odontológicos; utilizado por médicos e dentistas ao registrar problemas/condições no SOAP |
| **Cidadão** | Documentação Clínica / Identificação | Pessoa que recebe o atendimento domiciliar. Pode ter dados oriundos do PEC ou ser cadastrada manualmente pelo profissional |
| **CNS** | Identificação / Documentação Clínica | Cartão Nacional de Saúde. Identificador alternativo ao CPF; cartões iniciados por **7** são definitivos, por **8** são de treinamento |
| **CPF** | Identificação / Documentação Clínica | Cadastro de Pessoas Físicas. Identificador principal de um cidadão |

---

## D

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **DUM** | Documentação Clínica — SOAP | Data da Última Menstruação. Campo clínico registrado no SOAP para avaliação gestacional |

---

## F

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **FAI** | Sincronização | Ficha de Atendimento Individual. Gerada a partir de um atendimento clínico e enviada ao PEC |
| **Folha de Rosto** | Documentação Clínica | Contexto clínico resumido do cidadão consultado antes do atendimento: alergias/reações adversas, problemas/condições, resultados de exames e medicamentos em uso. Dados sempre atualizados pelo PEC |
| **FP** | Sincronização | Ficha de Procedimentos. Gerada quando o atendimento registra apenas procedimentos, sem consulta completa. Enviada ao PEC |

---

## I

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **Identificação** | Identificação da Pessoa | Etapa pré-atendimento em que o profissional localiza o cidadão por CPF ou CNS antes de iniciar o registro clínico |
| **IMC** | Documentação Clínica — SOAP | Índice de Massa Corpórea. Calculado automaticamente a partir do peso e altura informados durante o atendimento |
| **INE** | Autenticação / Seleção de Lotação | Identificador Nacional de Equipes. Código único da equipe de saúde à qual o profissional está vinculado na lotação |

---

## L

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **Lista de Atendimentos** | Lista e Gestão de Atendimentos | Relação de atendimentos pendentes ou em rascunho para a lotação ativa |
| **Lista de Pessoas** | Lista e Gestão de Atendimentos | Relação de cidadãos da área de cobertura da equipe |
| **Lotação** | Autenticação / Seleção de Lotação | Vínculo ativo do profissional com uma equipe de saúde e uma UBS, num período determinado. Selecionada no acesso ao sistema |

---

## P

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **PEC** | Sincronização | Prontuário Eletrônico do Cidadão. Sistema com o qual o aplicativo sincroniza dados; origem e destino das informações clínicas |
| **PEC** | Documentação Clínica | Indicador de origem: dados do cidadão foram obtidos do servidor PEC (vs. cadastro manual pelo profissional) |
| **Profissional** | Autenticação / Documentação Clínica | Usuário autenticado do aplicativo: profissional de saúde identificado por CPF, com CBO e lotação ativa |
| **Prontuário** | Documentação Clínica / Sincronização | Registro clínico do cidadão no PEC. Referência que permite ao aplicativo buscar a Folha de Rosto e os dados cadastrais do cidadão |

---

## S

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **SIGTAP** | Documentação Clínica — SOAP | Sistema de Gerenciamento da Tabela de Procedimentos do SUS. Codifica procedimentos realizados durante o atendimento |
| **SOAP** | Documentação Clínica | Método de registro clínico orientado por problemas: Subjetivo, Objetivo, Avaliação e Plano |

---

## U

| Termo / Sigla | Contexto Delimitado | Descrição |
|---|---|---|
| **UBS** | Autenticação / Seleção de Lotação | Unidade Básica de Saúde. Estabelecimento onde o profissional exerce sua lotação |

---

*Este glossário lista apenas termos do domínio amplo de APS relevantes para o domínio do app Atendimento. Demais termos foram omitidos intencionalmente.*
