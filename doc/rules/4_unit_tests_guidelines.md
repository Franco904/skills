# Padrões de Testes Unitários — Flutter/Dart

## Escopo

Este arquivo define os padrões obrigatórios para escrita de testes unitários
neste projeto. Aplica-se a todos os arquivos em `test/` por módulo dentro de `modules/`.

- Este nível de teste valida a funcionalidade lógica das menores unidades da aplicação (ex.: classes, métodos estáticos e *funções top-level*), independente da interação delas com outras unidades do projeto.
- A maior parte do código coberto por testes unitários concentra-se na lógica de negócio do sistema.

---

## 1. Arquivo de teste

- Uma classe de teste por arquivo, correspondendo ao arquivo de produção testado.
- Nome do arquivo de teste: nome do arquivo de produção sufixado com `_test`
  (ex: `cidadao_repository_test.dart`), evidenciando de maneira clara o SUT (System Under Test).

---

## 2. Organização Interna da Classe de Teste

Seguir esta ordem obrigatória nas declarações:

1. Variáveis de dependências mockadas (sufixo `Mock`, ex: `cidadaoRepositoryMock`)
2. Variável do SUT
3. Constantes e atributos de configuração
4. Métodos `setUp` e `tearDown` (se necessários)
5. Métodos auxiliares privados (reuso de lógica)
6. Blocos `group` por método público testado

---

## 3. Estrutura de Groups e Casos de Teste

- Cada método público do SUT deve ter um `group()` próprio.
- Não criar um `group()` englobando toda a classe — apenas por método público.
- Padrão de nome do group: `"nomeMetodo |"`
- Cada método de teste representa um caminho possível no fluxo do método,
  seguindo a estratégia **"Todos os caminhos"**.
- Evitar casos de borda muito improváveis ou sem valor de negócio.

---

## 4. Nomenclatura e Idioma

- Descrições de teste em **português**, respeitando a linguagem do negócio.
- Template obrigatório:
  `"Deve [resultado] quando [ação], dado que [contexto opcional]"`
- Basear-se em scripts Gherkin da especificação quando fornecidos.

---

## 5. Valores de Entrada

- Sempre que possível, usar valores aleatórios como entrada nos testes para evitar acoplamento a valores fixos.
- **Regra de escolha:**
  - Para valores com semântica de domínio (nomes, datas, endereços, textos), use o pacote **`faker`** — os dados gerados são mais legíveis e expressivos do ponto de vista do domínio:
    ```dart
    final faker = Faker();
    final nomeCidadao = faker.person.name();
    final endereco = faker.address.streetAddress();
    ```
  - Para identificadores numéricos simples (IDs, índices), use `dart:math` (`Random`):
    ```dart
    final randomId = Random().nextInt(9999);
    ```

---

## 6. DDD (Linguagem de negócio, Lógica de negócio e Arquitetura)

### 6.1 Linguagem ubíqua

- Nomes de classes de teste, variáveis, métodos e descrições devem usar
os termos do domínio (ex: `Atendimento`, `CidadaoRepository`).
- Evitar termos genéricos como `DataService`, `Manager`, `Helper`, bem como termos que evidenciem tecnologias usadas na implementação, como `Left`, `Right`, `Json`, `Drift`, `Strategy`, `Abstract Factory`.

### 6.2 Fronteiras lógicas bem definidas

- Testes de **domínio** (entidades, VOs): sem dependência de frameworks de
  infraestrutura. Use objetos puros Dart.
- Testes de **aplicação** e **apresentação**: todas as dependências
  (interfaces de repositórios) devem ser mockadas. Nunca acesse infraestrutura real.
- Testes de **infraestrutura** (data/external): todas as dependências
  (interfaces de datasources e adapters) devem ser mockadas. Testes de bancos de dados (arquivos DAO) devem ser delegados aos testes de integração.
- Testes de um módulo não devem depender de lógica de negócio/domínio internas de outro módulo, pois isso viola o princípio de do encapsulamento. Os módulos são delimitados pela ocorrência de conflitos semânticos da linguagem ubíqua do domínio ou, na sua ausência, pela funcionalidade de tela. Desse modo, os contextos delimitados são independentes e seus limites devem ser respeitados nos testes.

### 6.3 Aggregates / Value Objects — invariantes de negócio

Casos de teste para VOs, em lógica de negócio complexa, devem cobrir as regras que garantem validade do objeto.
Exemplos: CPF inválido lança exceção de domínio, valor negativo rejeitado.

O mesmo para testes de agregado, validando regras com granularidade mais ampla que VO.

### 6.4 Exceções de domínio como contrato

Em caminhos de falha, validar o **tipo** da exceção de domínio lançada
(ex: `SessionExpiredSyncStepDomainException`, `StartAtendimentoUseCaseDomainException.lotacaoException`), não apenas
que uma exceção genérica foi lançada.

### 6.5 Separação: lógica de domínio × orquestração

- Lógica de domínio pura (entidades/VOs) → testes próprios, isolados.
- Use cases testam a **orquestração** dessa lógica, não a lógica em si.
- Não duplicar testes de regra de negócio nos testes de use case.

### 6.6 Separação: CQRS — formato de teste de consulta vs comando

Os casos de uso (use cases) de consulta (leitura) e os casos de uso de comando (escrita) têm obrigações de teste diferentes:

- **Consulta**:
   - verificar se o filtro/parâmetros corretos foram passados ​​para a fonte de dados;
   - verificar se o modelo de projeção está mapeado corretamente;
   - verificar a lógica de transformação e cálculo, se houver;
   - verificar se os caminhos de falha retornam a exceção de domínio correta;
   - nunca verificar se uma operação de gravação foi chamada.
- **Comando**:
   - verificar a lógica de validação de invariantes de negócio no agregado, se houver;
   - verificar a lógica de transição de estado, se houver;
   - verificar a lógica de emissão de eventos de domínio, se houver;
   - verificar se as operações de gravação corretas foram chamadas nos repositórios (verify(repo).save(...));
   - verificar os argumentos passados nas operações de gravação nos repositórios após a construção do agregado que envolve regras
  de negócio.
   - verificar se os caminhos de falha retornam a exceção de domínio correta.

---

## 7. Utilitários de teste

- Consultar os arquivos de utilitários de teste em `modules/common/test/test_utils/` para evitar duplicação de lógica de montagem de objetos complexos.
- Se houver necessidade e oportunidade, criar novos métodos auxiliares compartilhados para favorecer o reuso de código.

---

### 8. Execução e Cobertura de teste

- Ao final da implementação, deve ser verificado que todos os caminhos relevantes dos métodos públicos foram cobertos executando `fvm flutter test` com a flag `--coverage` e analisando o relatório de cobertura `lcov.info` dos módulos impactados.
