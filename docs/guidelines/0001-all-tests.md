# Padrões Comuns de Teste

## Escopo

Aplica-se a todos os níveis de teste neste projeto: unitário, integração e apresentação.

---

## 1. Nomenclatura e Idioma

- Descrições de teste em **português**, respeitando a linguagem do negócio.
- Template obrigatório: `"Deve [resultado] quando [ação], dado que [contexto opcional]"`
- **Proibido mencionar nomes de tipos ou valores de implementação nas descrições** — nem wrappers funcionais de retorno de erro (ex: tipos genéricos de sucesso/falha), nem tipos de estado de UI (`NotProvided`, `Unavailable`, `NotApplicable`, `Provided`), nem sentinelas de enum (`Sexo.unavailable`, `IdentidadeGenero.unavailable`), nem subtipos sentinela (`UnavailableRacaCor`, `UnavailableNacionalidade`). Use o termo de negócio: "não informado", "informação indisponível", "não aplicável", "valor preenchido".

| Errado | Correto |
|---|---|
| `"Deve emitir Success(atendimentos)"` | `"Deve retornar lista de atendimentos quando carregamento for bem-sucedido"` |
| `"Deve emitir NotProvided para raça/cor quando for null"` | `"Deve exibir raça/cor como não informada quando o campo não estiver preenchido"` |
| `"Deve emitir Unavailable para sexo quando não pôde ser mapeado"` | `"Deve exibir sexo como informação indisponível quando o valor não pôde ser reconhecido pelo sistema"` |
| `"Deve emitir LoadAtendimentosDomainException.unexpectedError"` | `"Deve emitir erro de carregamento quando a tentativa falhar"` |
| `"Deve emitir ListaAtendimentoState com loading=false"` | `"Deve finalizar carregamento ao receber resposta do repositório"` |
| `"Deve inserir e recuperar row"` | `"Deve persistir e recuperar o Cidadão"` |

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

## 3. Bons e Maus Testes

**Princípio:** testes verificam comportamento através de interfaces públicas, não detalhes de implementação. O código pode mudar completamente; os testes não deveriam.

### Bons testes

- Testam comportamento que usuários/chamadores se importam
- Usam apenas a API pública do SUT
- Sobrevivem a refatorações internas
- Descrevem *o quê* o sistema faz, não *como* faz

```kotlin
// BOM: Testa comportamento observável
@Test
fun `Deve retornar lista de atendimentos quando carregamento for bem-sucedido`() = runTest {
    val atendimentos = listOf(createAtendimentoFake())
    coEvery { atendimentoRepositoryMock.loadAtendimentos(lotacaoId) } returns Success(atendimentos)

    val result = loadAtendimentosUsecase(lotacaoId)

    assertTrue(result is Success)
}
```

### Maus testes

Sinais de alerta:
- Mockando colaboradores internos da mesma camada
- Testando métodos privados
- Nome do teste descreve *como* em vez de *o quê*
- Teste quebra ao refatorar sem o comportamento ter mudado

```kotlin
// RUIM: Descrição expõe tipo de implementação
@Test
fun `Deve emitir Success(atendimentos) quando datasource não lançar exceção`() {}

// RUIM: Contorna a interface para verificar estado
@Test
fun `Deve persistir Cidadão`() = runTest {
    cidadaoDao.insert(cidadaoCompanion)
    val row = db.select(db.cidadaoTable).getSingle() // acesso direto ao DB
    assertNotNull(row)
}

// BOM: Verifica através da interface do objeto de acesso a dados
@Test
fun `Deve persistir e recuperar Cidadão`() = runTest {
    cidadaoDao.insert(cidadaoCompanion)
    val result = cidadaoDao.findById(cidadaoCompanion.id.value)
    assertEquals(cidadaoCompanion.id.value, result?.id)
}
```

---

## 4. Quando Usar Mock

Use mock apenas em **fronteiras do sistema**:

- APIs externas (serviços remotos via protocolo RPC/HTTP)
- Armazenamento (banco de dados, preferências/configurações do dispositivo, armazenamento seguro)
- Sistema de arquivos
- Tempo/aleatoriedade

Não use mock em:
- Classes e colaboradores internos da mesma camada (ex: um use case não deve mockar outro use case)
- Qualquer coisa que você controla integralmente dentro do mesmo contexto delimitado

---

## 5. Projetando para Mockabilidade

Nas fronteiras do sistema, projete interfaces fáceis de mockar:

**Use injeção de dependência** — passe dependências externas em vez de criá-las internamente:

```kotlin
// Testável
suspend fun loadAtendimentos(
    lotacaoId: Int,
    repository: AtendimentoRepository,
): Result<AtendimentoDomainException, List<Atendimento>> { ... }

// Difícil de mockar
suspend fun loadAtendimentos(
    lotacaoId: Int,
): Result<AtendimentoDomainException, List<Atendimento>> {
    val repository = AtendimentoRepositoryImpl() // dependência criada internamente
    ...
}
```

**Prefira interfaces específicas por operação** em vez de uma interface genérica com lógica condicional — cada método é mockável independentemente e o tipo de retorno é explícito por operação.
