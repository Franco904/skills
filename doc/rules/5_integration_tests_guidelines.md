# Padrões de Testes de Integração — Flutter/Dart

## Escopo

Este arquivo define os padrões obrigatórios para escrita de testes de integração
neste projeto. Aplica-se a todos os arquivos em `test/` por módulo dentro de `modules/`.

- Este nível de teste valida a funcionalidade lógica da integração do código de produção, mantido internamente, com códigos de bibliotecas de terceiros, dos quais não temos controle de evolução.
- A maior parte do código coberto por testes de integração concentra-se na integração do sistema com tecnologias externas, como armazenamento local, criptografia e serialização de objetos.
- O objetivo principal não está em testar o código externo em si (isso é de responsabilidade da biblioteca), mas sim testar o comportamento do aplicativo nos pontos em que ele interage (se integra) com essas bibliotecas.
- O escopo deste nível não engloba integrações com componentes de UI. Estas devem ser verificadas em testes de UI/widget de maneira unitário ou, mais fielmente, em testes end-to-end com ferramentas próprias para instrumentação (ex.: Appium).

---

## 1. O que testar a nível de integração em Flutter

Candidatos típicos para testes de integração locais (executados na Dart VM, sem instrumentação):

- Lógica de persistência em armazenamento local (banco de dados do Drift, preferências com SharedPreferences e Flutter Secure Storage)
- Lógica de migração de esquema (Drift): verificar que dados pré-existentes sobrevivem a migrações de **stepByStep**, especialmente em colunas com valores padrão ou renomeações de tabela.
- Lógica de conversão de Thrift/binário, JSON e outros formatos de serialização;
- Lógica de transformações criptográficas (cifragem/decifragem, hash, derivação de senha) ou de encodificação (Base64, hex).

---

## 2. Arquivo de teste

- Uma classe de teste por arquivo, correspondendo ao arquivo de produção testado.
- Nome do arquivo de teste: nome do arquivo de produção sufixado com `_test`
  (ex: `cidadao_dao_test.dart`), evidenciando de maneira clara o SUT (System Under Test).

---

## 3. Estratégia de cobertura

- **Não aplicar a estratégia "Todos os caminhos"** aqui. O foco está em verificar se, após a execução de uma operação, o estado da aplicação permanece válido perante a biblioteca integrada.
- Priorize casos que cobrem o comportamento funcional do recurso integrado, não todas as combinações lógicas internas do SUT.
- Como referência, o número de casos de teste de integração deve ser **menor** que o de testes unitários.

---

## 4. Test doubles: preferência por Fakes

- Prefira **Fakes** (implementações simples e funcionais de interfaces) para isolar dependências que não sejam a principal sob teste.
- Use Mocks apenas se for estritamente necessário verificar chamadas. Se isso ocorrer com frequência, pode indicar que o SUT tem mais de uma responsabilidade — considere indicar uma refatoração.
- **Não é necessário** verificar chamadas de outras dependências nos testes de integração. O foco é no estado resultante, não na interação entre objetos.

---

## 5. Nomenclatura e Idioma

- Descrições de teste em **português**, respeitando a linguagem do negócio.
  - Exemplo de descrição incorreta: `"Deve inserir e recuperar row"`
  - Exemplo de descrição correta: `"Deve persistir e recuperar Cidadão"`

---

## 6. Padrão de setup e validação (caixa cinza)

Use as próprias APIs da biblioteca de terceiro tanto para configurar o cenário quanto para validar os efeitos. Exemplo para um DAO:

```dart
// Arrange: inserir dados via API da própria biblioteca
await db.into(db.cidadaoTable).insert(cidadao);

// Act
final result = await cidadaoDao.findAll();

// Assert: validar estado resultante
expect(result, hasLength(1));
expect(result.first.id, equals(cidadao.id));
```

Obs: Criar uma nova instância do banco de dados em memória por caso de teste (no setUp), e fechar a conexão no tearDown. Para garantir independência, nunca compartilhar estado de banco entre casos de teste.

---

## 7. Diferenças em relação aos testes unitários

| Aspecto | Unitário | Integração |
|---|---|---|
| Isolamento | Total (sem dependências externas) | Acoplamento controlado com a biblioteca integrada |
| Cobertura | Exaustiva ("Todos os caminhos") | Funcional (estado resultante válido) |
| Test doubles | Mocks preferencialmente | Fakes preferencialmente |
| Verificação de chamadas | Sim, quando relevante | Não obrigatório |
| Volume de casos | Alto | Menor que unitários |
