# Contextos Delimitados — e-SUS Atendimento

> **Finalidade:** Mapear os contextos delimitados (Bounded Contexts) do aplicativo e-SUS Atendimento, documentar as fronteiras entre equipes e módulos e orientar decisões de modelagem que respeitem as diferenças de termos, dados e comportamentos da Linguagem Ubíqua encapsulada entre contextos.
>
> **Instrução para o Agente de IA:** Ao sugerir onde adicionar código, criar novos artefatos ou mover responsabilidades entre módulos, consulte este mapeamento. Nunca proponha cruzar fronteiras de contexto sem justificativa arquitetural explícita. Respeite a topologia das equipes (Lei de Conway).

---

## Equipes e Módulos

O projeto é desenvolvido por duas equipes ágeis que compartilham uma única base de código e um banco de dados SQLite (Drift):

| Equipe | Módulos de responsabilidade |
|---|---|
| **Candy** | `authentication`, `atendimento/dados-pessoais`, `atendimento/folha-rosto`, `sync` |
| **Snake** | `atendimento/soap`, `drawer`, `feedback`, `home` (LA e LP), `identificacao` (IP) |

> O banco SQLite/drift é um **Shared Kernel** — superfície de integração técnica, não evidência de que os módulos pertencem ao mesmo contexto de negócio.

---

## Contextos Delimitados

### 1. Autenticação e Seleção de Lotação
**Módulo:** `authentication`  
**Equipe:** Candy  
**Responsabilidade:** Login do profissional, validação de credenciais contra o PEC, seleção e fixação de lotação de trabalho.  
**Fronteira:** Produz a `Lotacao` ativa que os demais contextos consomem como dado de entrada. Nenhum outro contexto deve replicar lógica de autenticação.

---

### 2. Documentação Clínica — Identificação da Pessoa (IP)
**Módulo:** `identificacao`  
**Equipe:** Snake
**Responsabilidade:** Ponto de entrada do fluxo de atendimento. Identifica o Cidadão (por CPF ou CNS) antes do início do registro clínico. Busca dados cadastrais do PEC ou cria um cadastro manual em caso de erro de rede ou cidadão não cadastrado.  
**Fronteira:** Não contém lógica de atendimento em si — apenas a identificação que precede o atendimento. Dois casos de uso também presentes em `sync` (buscar Cidadão por CPF/CNS localmente; buscar dados da Folha de Rosto por `prontuarioId`) são **duplicados intencionais**, reflexo da topologia das equipes (Snake ↔ Candy), não de um compartilhamento de domínio.

---

### 3. Documentação Clínica — Lista e Gestão de Atendimentos
**Módulo:** `home`  
**Equipe:** Snake  
**Responsabilidade:** Lista de Atendimentos (LA) e Lista de Pessoas (LP). Ponto de entrada para iniciar, continuar ou cancelar um atendimento. Suporta rascunhos para atendimentos interrompidos.  
**Fronteira:** Não é um contexto de agendamento — é a entrada da Documentação Clínica. O `delete_atendimento_usecase` em `common` é um artefato genuinamente transversal, invocável tanto daqui quanto da tela de SOAP.

---

### 4. Documentação Clínica — Registro do Atendimento (SOAP)
**Módulo:** `atendimento`
**Equipe:** Snake (SOAP) e Candy (Folha de Rosto + Dados Pessoais)  
**Responsabilidade:** Registro clínico do atendimento segundo o método SOAP (Subjetivo, Objetivo, Avaliação, Plano). Inclui dados pessoais do Cidadão, Folha de Rosto e o registro SOAP em si. O registro do SOAP permite transitar o estado do Atendimento para finalizado, ou cancelá-lo, eliminando-o do aplicativo.
**Subdivisão interna:**
- **Folha de Rosto + Dados Pessoais** (Candy): exibição dos dados cadastrais e do contexto clínico do Cidadão (alergias, problemas/condições, exames, medicamentos). A Folha de Rosto é um contexto de *leitura* — os dados são trazidos pela sync com resolução de conflito *server-wins*.
- **SOAP** (Snake): registro dos campos clínicos do atendimento propriamente dito.

**Fronteira:** A Folha de Rosto pertence à Documentação Clínica, não a um contexto de Registros do Cidadão. O módulo `sync` trata os dados de FR como uma responsabilidade de infraestrutura/ACL.

---

### 5. Sincronização com o PEC (ACL)
**Módulo:** `sync`  
**Equipe:** Candy  
**Responsabilidade:** Sincronização bidirecional com o servidor PEC via protocolo Thrift/LEDI. Envia Fichas de Atendimento Individual (`FAI`) e Fichas de Procedimento (`FP`) e recebe dados de Cidadãos, Folha de Rosto e do Profissional autenticado.
**Padrão arquitetural:** **Camada Anticorrupção (ACL)**. Isola o modelo de domínio interno das estruturas e versões do LEDI. Qualquer mudança no contrato LEDI deve ser absorvida aqui, sem vazar para os modelos de domínio dos outros contextos.  

---

### 6. Módulo Comum (Shared Kernel)
**Módulo:** `common`  
**Equipe:** Ambos  
**Responsabilidade:** Artefatos genuinamente transversais: entidades compartilhadas, exceções base, utilitários, DAOs do banco Drift, e utilitários de teste (`test_utils/`).  
**Regra:** Só entra em `common` o que é invocado por dois ou mais contextos **sem adaptação semântica**. Lógica que pertence a um único contexto deve permanecer nele.

---

## Regras de Fronteira

1. **Testes não cruzam contextos:** testes de um módulo não importam lógica de negócio interna de outro módulo. Use apenas interfaces públicas ou fakes do `test_utils/` de `common`.
2. **ACL absorve LEDI:** mudanças de versão do protocolo Thrift/LEDI ficam confinadas ao módulo `sync`.
3. **Shared Kernel não é atalho:** adicionar algo em `common` só se justifica por uso genuinamente transversal — não por conveniência.
4. **Duplicação intencional vs. compartilhamento:** a duplicação de casos de uso entre `identificacao` e `sync` é aceitável dado o modelo de equipes. Não centralize casos de uso sem validar com os tech leads das duas equipes.
