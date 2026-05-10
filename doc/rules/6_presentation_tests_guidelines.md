# Padrões de Testes de Apresentação — Flutter/Dart

## Escopo

Este arquivo define os padrões para escrita de testes da camada de apresentação neste projeto.
Aplica-se a arquivos em `test/module/presentation/` dentro de cada módulo em `modules/`.

- Este nível valida o comportamento dos **Cubits** (orquestração de estado) e dos **States** (mapeamento de entidades para modelos de UI), de forma isolada do Flutter framework.
- Os testes são executados na Dart VM, **sem instrumentação de UI** (`testWidgets`). Testes que envolvam renderização de widgets devem ser delegados a ferramentas de end-to-end (ex.: Appium).
- O foco está em: transições de estado do Cubit, sequências de emissão, mapeamento de entidades para campos de UI, e lógica de formatação de dados.

> **Nota para os tech leads:** Este guia reflete os padrões identificados no código existente. Revisem e ajustem especialmente as seções de: escolha entre `bloc_test` e `stream`/`expectLater`, uso de `mocktail` vs `mockito`, e tratamento de `tearDown` em cubits. Estão marcados com **[REVISAR]**.

---

## 1. Organização de Arquivos

| Tipo de teste | Localização |
|---|---|
| Testes de Cubit | `test/module/presentation/blocs/` ou `test/module/presentation/cubits/` |
| Testes de State | `test/module/presentation/states/` |

Nome do arquivo: sufixo `_test` sobre o arquivo de produção testado
(ex.: `lista_atendimento_cubit_test.dart`, `dados_pessoais_state_test.dart`).

---

## 2. Testes de Cubit

### 2.1 Organização interna

Seguir a mesma ordem obrigatória dos testes unitários:

1. Variáveis de mocks (sufixo `Mock`, ex.: `atendimentoRepositoryMock`)
2. Variável do SUT
3. Constantes e atributos de configuração
4. Métodos `setUp` e `tearDown` (se necessários)
5. Métodos auxiliares privados (reuso de lógica)
6. Blocos `group` por método público testado

### 2.2 Biblioteca de asserção de stream

O projeto usa dois estilos para verificar emissões de estado. **[REVISAR]**

**Estilo 1 — `bloc_test` (declarativo):** preferencial quando o Cubit recebe dependências mockadas e a sequência de estados pode ser declarada antecipadamente.

```dart
blocTest<ListaAtendimentoCubit, ListaAtendimentoState>(
  'Deve emitir lista de atendimentos agrupados quando carregamento for bem-sucedido',
  setUp: () {
    when(() => repositoryMock.loadAtendimentos(lotacaoId))
        .thenAnswer((_) => Stream.value(Right(atendimentos)));
  },
  build: () => ListaAtendimentoCubit(repositoryMock, mockDeleteUseCase, mockPreferences),
  act: (cubit) => cubit.init(),
  expect: () => [
    isA<ListaAtendimentoState>()
        .having((s) => s.loading, 'loading', false)
        .having((s) => s.atendimentosAgrupados, 'atendimentosAgrupados', isNotEmpty),
  ],
  verify: (_) {
    verify(() => repositoryMock.loadAtendimentos(lotacaoId)).called(1);
  },
);
```

**Estilo 2 — `cubit.stream` + `expectLater`:** preferencial quando o Cubit não tem dependências mockadas ou a sequência precisa ser verificada de forma imperativa.

```dart
test('Deve emitir estado de cidadão do PEC ao inicializar com Cidadão PEC', () async {
  final cidadaoPec = createCidadaoPecFake();

  final expectation = expectLater(
    cubit.stream,
    emits(isA<DadosPessoaisLoadedState>()
        .having((s) => s, 'tipo', isA<DadosPessoaisCidadaoPecState>())),
  );

  cubit.init(cidadao: cidadaoPec);

  await expectation;
});
```

Para sequências ordenadas de múltiplos estados, use `emitsInOrder`:

```dart
final expectation = expectLater(
  cubit.stream,
  emitsInOrder([
    isA<LoginLoading>(),
    isA<LoginSuccess>().having((s) => s.nextRoute, 'nextRoute', Routes.home.path),
  ]),
);
```

### 2.3 Teardown de Cubits **[REVISAR]**

Em alguns testes existentes, `cubit.close()` é chamado no `tearDown`. Em outros, não é.
Definir aqui a política do projeto:

- Se o Cubit escuta streams ou timers, fechar no `tearDown` é obrigatório para evitar vazamentos.
- Para Cubits simples sem assinaturas internas, é opcional mas recomendado.

### 2.4 Verificação de dependências

Ao usar `bloc_test`, use o bloco `verify` para confirmar chamadas esperadas nas dependências mockadas.
Ao usar o estilo imperativo, use `verify(...)` / `verifyNever(...)` após o `await`.

---

## 3. Testes de State

States que fazem mapeamento de entidades de domínio para modelos de UI devem ter testes próprios.
Estes testes verificam:

- **Mapeamento de campos:** label e value de cada campo de UI resultante da entidade.
- **Formatação de dados:** datas, telefones, endereços, textos compostos.
- **Campos condicionais:** campos que dependem do valor de outros campos (ex.: etnia condicional a raça/cor).
- **Ordem dos campos:** a lista `fields` exposta pelo state.

```dart
test('Deve mapear corretamente os campos do Cidadão do PEC para os valores de apresentação', () {
  final cidadao = CidadaoPec(
    id: 1,
    nome: 'Carlos Pereira',
    cpfCnsStatus: CpfPresent('11122233344'),
    // ...
  );

  final state = DadosPessoaisCidadaoPecState.fromEntity(cidadao, now: LazyDateTime.of(2026, 2, 28));

  expect((state.nome.value as Provided).value, 'Carlos Pereira');
  expect(state.racaCor.label, 'Raça/cor');
});
```

Estados de apresentação testados como objetos puros **não precisam de mocks** — use objetos Dart diretamente ou os fakes de `modules/common/test/test_utils/`.

---

## 4. Bibliotecas de Mock **[REVISAR]**

O projeto usa tanto `mockito` (com `@GenerateMocks`) quanto `mocktail` (sem geração de código).
Definir aqui a preferência para novos testes de apresentação:

- `mockito`: requer `@GenerateMocks` + `build_runner`, gera um arquivo `.mocks.dart`. Mais verboso, mas com checagem de tipos em tempo de compilação.
- `mocktail`: sem geração de código, setup mais rápido com `when(() => ...)`. Requer `registerFallbackValue` para tipos não-nulos.

---

## 5. Nomenclatura e Idioma

- Descrições em **português**, respeitando a linguagem do negócio.
- Template: `"Deve [resultado] quando [ação], dado que [contexto opcional]"`
- Não usar nomes de classes de estado, enums de exceção ou wrappers `Either` nas descrições.

| Errado | Correto |
|---|---|
| `"Deve emitir Right(atendimentos)"` | `"Deve emitir lista de atendimentos agrupados quando carregamento for bem-sucedido"` |
| `"Deve emitir LoadAtendimentosDomainException.unexpectedError"` | `"Deve emitir erro de carregamento quando repositório falhar"` |
| `"Deve emitir ListaAtendimentoState com loading=false"` | `"Deve finalizar carregamento ao receber resposta do repositório"` |

---

## 6. Diferenças em relação aos testes unitários

| Aspecto | Unitário (domínio/use case) | Apresentação (Cubit/State) |
|---|---|---|
| SUT | Entidades, VOs, use cases | Cubits, States |
| Framework de estado | Nenhum | `bloc_test`, `cubit.stream` |
| Mocks | Mockito/Mocktail para repositórios | Mockito/Mocktail para repositórios e datasources locais |
| Objetos de domínio | Construídos diretamente | Via fakes de `test_utils/` |
| Verificação de chamadas | Sim (comandos) | Sim, via bloco `verify` ou `verify(...)` |

---

## 7. Ausência de testes de widget Flutter

- Este projeto **não possui testes de widget Flutter** (`testWidgets`) na suite atual.
- Testes de renderização de UI devem ser cobertos por testes end-to-end com instrumentação adequada (ex.: Appium).
- Não adicione `testWidgets` na suite de testes unitários/integração — eles demandam um ambiente Flutter completo e aumentam o tempo de execução sem o isolamento adequado.
