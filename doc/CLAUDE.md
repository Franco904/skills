# e-SUS Atendimento — Guia do Agente de IA

## Visão Geral

**e-SUS Atendimento** é um aplicativo mobile Flutter para registro de atendimentos domiciliares na Estratégia de Saúde da Família (ESF) e eMulti, dentro do domínio da Atenção Primária à Saúde (APS). O aplicativo proporciona acesso instantâneo às informações de folha de rosto dos cidadãos atendidos, facilitando decisões clínicas rápidas e assertivas, com base em dados completos e atualizados. Desenvolvido para funcionar offline, o aplicativo garante que os profissionais possam registrar e consultar informações essenciais, mesmo em áreas de baixa conectividade, assegurando a continuidade e a qualidade do atendimento em qualquer cenário. Integra-se ao PEC (Prontuário Eletrônico do Cidadão) via protocolo Thrift/LEDI, com modelo compatível com a RNDS. É desenvolvido em nome do Ministério da Saúde, Governo Federal, e dividido entre duas equipes ágeis: **Candy** e **Snake**.

- **Flutter:** 3.32.0 (gerenciado via FVM — use sempre `fvm flutter`, nunca `flutter` diretamente)
- **Dart SDK:** ^3.7.0
- **Gerenciador de monorepo:** Melos

---

## Estrutura do Projeto

Monorepo com módulos independentes em `modules/`:

| Módulo | Responsabilidade |
|---|---|
| `app_atendimento` | Ponto de entrada do projeto |
| `common` | Entidades, exceções, utilitários e test utils compartilhados |
| `dependencies` | Re-exporta todas as dependências de terceiros |
| `root` | Inicialização e configuração do app |
| `authentication` | Login, termos de uso e seleção de lotação |
| `home` | Lista de Pessoas (LP) e Lista de Atendimentos (LA) |
| `atendimento` | Registro clínico (Folha de Rosto, SOAP e Dados Pessoais) |
| `identificacao` | Identificação do Cidadão antes do atendimento |
| `drawer` | Menu lateral (configurações e informações do app) |
| `feedback` | Envio de feedback do profissional para as equipes desenvolvedoras |
| `sync` | Sincronização com o PEC via Thrift/LEDI (ACL) |

Cada módulo segue a estrutura:
```
lib/module/
  domain/       # entidades, repositórios (interfaces), casos de uso, exceções, validadores
  data/         # implementações de repositórios, modelos, datasources (interfaces)
  external/     # implementações de datasources e adapters (Drift DAOs, Dio, SharedPreferences)
  presentation/ # blocs/cubits, states, views, widgets, ui_models
```

---

## Arquitetura

### Clean Architecture + DDD

A dependência flui de fora para dentro: `external → data → domain ← presentation`.

- **domain**: objetos Dart puros. Sem dependência de frameworks ou infraestrutura. Contém as regras de negócio.
- **data**: implementa as interfaces do domain. Traduz dados entre infraestrutura e domínio.
- **external**: código que encosta em bibliotecas de terceiros (Drift, Dio, SharedPreferences).
- **presentation**: BLoC/Cubit + Widgets Flutter.

Para maior contexto consulte a seção "Estrutura do Projeto" do arquivo README.md, no ponto de entrada do projeto.

### Injeção de Dependência e Roteamento

Usa **flutter_modular**. Cada módulo registra seus bindings em `<nome>_module.dart`:

```dart
class HomeModule extends Module {
  @override
  void binds(Injector i) {
    i.add<ListaAtendimentoCubit>(ListaAtendimentoCubit.new);
    // ...
  }
  @override
  void routes(RouteManager r) {
    r.child('/lista', child: (context) => const ListaAtendimentoView());
  }
}
```

### Estado e Eventos

Gerenciamento de estado com **flutter_bloc** (padrão Cubit). Estados em `presentation/states/`, eventos de UI em `presentation/ui_events/`.

### Tratamento de Erros

Usa **dartz** (`Either<DomainException, T>`) como retorno de repositórios e casos de uso:

```dart
// Repositório (interface — domain)
abstract class CidadaoRepository {
  Future<Either<BuscarCidadaoDomainException, Cidadao>> buscarPorCpf(String cpf);
}

// Exceção de domínio — sempre enum implementando DomainException
enum BuscarCidadaoDomainException implements DomainException { cidadaoNaoEncontrado, erroInesperado }
```

Nunca expor `Left`/`Right`/`Either` em descrições de testes ou nomes de variáveis voltados ao domínio.

### Banco de Dados

**Drift** (SQLite). DAOs ficam em `commons/core`, e referenciados pelos datasources locais, em `external/datasources/` de cada módulo. O banco é compartilhado entre módulos como um **Shared Kernel** — trate-o como superfície de integração, não como prova de que módulos pertencem ao mesmo contexto.

---

## Linguagem Ubíqua

**Obrigatório:** use os termos definidos em `.claude/rules/1_glossary.md` para nomear classes, métodos, variáveis, módulos e descrições de teste. Nunca invente sinônimos, abreviações não listadas ou termos técnicos da indústria (ex.: `Manager`, `Helper`, `DataService`).

Exemplos corretos: `Atendimento`, `Cidadao`, `Lotacao`, `Profissional`, `FolhaRosto`, `ListaAtendimento`.

---

## Convenções de Nomenclatura

| Artefato | Sufixo / padrão |
|---|---|
| Caso de uso | `<Acao><Contexto>Usecase` (ex: `StartAtendimentoUsecase`) |
| Repositório (interface) | `<Contexto>Repository` |
| Repositório (impl) | `<Contexto>RepositoryImpl` |
| Datasource (interface) | `<Contexto>Datasource` |
| Datasource (impl) | `<Contexto>DatasourceImpl` / DAO: `<Contexto>Dao` |
| Exceção de domínio | `<Contexto>DomainException` (enum) |
| Cubit | `<Contexto>Cubit` |
| State | `<Contexto>State` |
| View | `<Contexto>View` |
| Widget | `<Contexto>Widget` ou nome descritivo sem sufixo |
| Modelo de UI | `<Contexto>UiModel` |
| Arquivo de teste | `<arquivo_produção>_test.dart` |

---

## Testes

Consulte os guias detalhados em `.claude/rules/`:

- **Testes unitários:** `.claude/rules/4_unit_tests_guidelines.md`
- **Testes de integração:** `.claude/rules/5_integration_tests_guidelines.md`
- **Testes de apresentação (Cubit/State):** `.claude/rules/6_presentation_tests_guidelines.md`

### Comandos de Teste

```bash
# Todos os módulos
melos run test:full
melos run coverage:full

# Módulo específico
melos run test:atendimento
melos run coverage:atendimento

# Módulo individual com cobertura
cd modules/<modulo> && fvm flutter test --coverage
```

---

## Geração de Código

O projeto usa `build_runner` para gerar mocks de teste automatizado (mockito), entidades Drift e outros artefatos:

```bash
# Gerar em todos os módulos
melos run generate

# Gerar apenas o banco de dados
melos run gen:db

# Em um módulo específico
cd modules/<modulo> && dart run build_runner build --delete-conflicting-outputs
```

Arquivos gerados (`.g.dart`, `.freezed.dart`, `.gr.dart`, `.graphql.dart`, `.mocks.dart`) não devem ser editados manualmente. Estão excluídos da análise estática.

---

## Comandos Essenciais

```bash
# Instalar dependências (todos os módulos)
make pub-get

# Limpar e reinstalar
make clean

# Rodar app em debug
make run-debug

# Criar novo módulo
make create-module name=<nome>

# Rodar testes locais (script CI)
make test

# Atualizar schema GraphQL
make update-graphql-schema

# Buscar banco de dados atualizado
make get-db
```

---

## Contextos Delimitados (DDD Estratégico)

Consulte `.claude/rules/3_bounded_contexts.md` para o mapeamento completo de contextos.

Princípios a respeitar:
- **Encapsulamento entre módulos:** testes de um módulo não devem depender de lógica de negócio interna de outro módulo.
- **Shared Kernel:** o banco Drift é compartilhado. Não use isso como justificativa para misturar contextos.
- **ACL de sincronização:** o módulo `sync` é uma Camada Anticorrupção (ACL) para o protocolo LEDI/Thrift. Mudanças de versão das estruturas de dados e validações do LEDI devem ser isoladas nele.
- **`common`** contém somente artefatos genuinamente transversais (ex: `delete_atendimento_usecase`, entidades compartilhadas, utilitários de teste).

---

## Arquivos de Regras

| Arquivo | Conteúdo |
|---|---|
| `.claude/rules/1_glossary.md` | Glossário da linguagem ubíqua do domínio APS/PEC — base para nomes de classes, métodos, variáveis e testes |
| `.claude/rules/2_subdomains.md` | Classificação estratégica dos subdomínios (principal/suporte/genérico) e guia de padrões táticos DDD |
| `.claude/rules/3_bounded_contexts.md` | Fronteiras entre contextos delimitados, topologia das equipes e regras de encapsulamento |
| `.claude/rules/4_unit_tests_guidelines.md` | Padrões obrigatórios para testes unitários |
| `.claude/rules/5_integration_tests_guidelines.md` | Padrões obrigatórios para testes de integração |
| `.claude/rules/6_presentation_tests_guidelines.md` | Padrões para testes de Cubit e State na camada de apresentação |
| `.claude/rules/7_code_review_guidelines.md` | Padrões de code review: etiquetas de severidade, formato de comentário, fluxo GitHub e rastreabilidade de diretrizes |
