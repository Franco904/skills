# Tratamento de Erros

De "O Programador Pragmático" (Hunt & Thomas): use exceções para problemas **excepcionais** — eventos genuinamente anômalos e inesperados. Uma exceção interrompe o fluxo com uma transferência de controle não local imediata (um "goto em cascata"), o que gera um gargalo de desempenho e prejudica legibilidade e manutenibilidade quando usada para o que é, na verdade, um resultado esperado do domínio.

> **Teste da exclusão**: Imagine excluir o **manipulador** (`try`/`catch`). Se o código, no caminho normal, continuar completando sua tarefa principal sem nunca precisar daquele `catch`, ele era só uma rede de segurança para uma falha externa/inesperada — uso correto de exceção. Se remover o `catch` impede o código de completar sua tarefa **no fluxo normal** (ex: decidir um resultado de negócio, quebrar um laço, desviar fluxo), o `catch` era parte da lógica de controle disfarçada de tratamento de exceção — uso incorreto.

Usa-se um tipo de retorno de erro como mecanismo padrão de erro. As regras abaixo dizem quando `throw` também é aceitável.

## Retorno de erro nas fronteiras de camada

Todo erro de negócio esperado ou recuperável que cruza `acesso a dados → lógica de negócio → apresentação` é modelado como um retorno de erro tipado (ex: `Result<XxxDomainException, T>`), nunca `throw`. `apresentação` nunca usa `try/catch` para lidar com erro de `acesso a dados` — consome exclusivamente através da API de tratamento do tipo de retorno (ex: `.fold()`, `.match()`, ou equivalente).

```kotlin
// interface
suspend fun loadProblemasCondicoes(
    atendimentoId: Int,
): Result<ProblemaCondicaoSoapDomainException, List<ProblemaCondicaoSoap>>

// chamador
result.fold(
    onFailure = { error -> emit(state.copy(error = AtendimentoError("Erro ao carregar atendimento"))) },
    onSuccess = { atendimento -> emit(state.copy(atendimento = atendimento)) },
)
```

Se um caso de uso precisa relatar múltiplas falhas paralelas, o lado de erro pode ser uma lista (`Result<List<DomainException>, T>`) — não um motivo para usar `throw`.

## `throw` contido em uma única camada

`throw` é aceitável quando **nasce e morre dentro da mesma camada**, sem cruzar para `lógica de negócio`/`apresentação` sem antes virar retorno de erro. Os dois casos legítimos:

**1. Convertendo exceção de biblioteca de terceiros.** camada de `acesso a dados` captura o que a biblioteca lança e devolve retorno de erro na própria borda:

```kotlin
// adapter — biblioteca HTTP lança, HttpClient converte
try {
    ...
} catch (e: HttpClientException) {
    throw handleHttpException(e, url)
}

// acesso a dados — absorve e devolve retorno de erro
try {
    val response = infoRemoteDataSource.getInfo(serverUrl)
    return Success(response.info)
} catch (e: Exception) {
    exceptionLogger.logException(e)
    return Error(CommonDomainException.fromHttpResponse(e))
}
```

**2. Satisfazendo o protocolo de uma biblioteca que exige `throw` para sua própria lógica interna.** Exemplo real: transação de um banco de dados local só faz rollback se o callback lançar. `DatabaseAdapter.transaction` é o adapter que possui essa chamada de biblioteca — ele deixa o `throw` propagar *dentro* do callback e o recaptura na própria borda, devolvendo o retorno de erro para quem chamou:

```kotlin
// acesso a dados — dentro do callback de transação, throw é o protocolo do banco de dados
suspend fun saveMunicipio(municipio: Municipio) {
    val result = localDataSource.saveMunicipio(...)
    return result.fold(onFailure = { e -> throw e }, onSuccess = { id -> Success(id) })
}

// o adapter dono da chamada de biblioteca recaptura e devolve retorno de erro
suspend fun <T> transaction(block: suspend () -> T): Result<DomainException, T> {
    return try {
        val result = database.transaction { block() }
        Success(result)
    } catch (e: Exception) {
        Error(CommonDomainException.unexpectedError)
    }
}
```

Antes de replicar esse padrão, confirme que o `throw` está de fato contido — se um repositório com assinatura `throw`-based puder ser chamado fora do wrapper que recaptura a exceção, o `throw` vaza para a camada de `lógica de negócio` sem virar objeto de retorno de erro, quebrando a regra.

## Modelando o lado de erro do retorno

`<Contexto>DomainException` pode assumir três formas — escolha pela necessidade do chamador, não por padrão fixo:

| Forma | Quando usar | Exemplo |
|---|---|---|
| `enum class : DomainException` | Caso padrão — não carrega dado associado | `CidadaoDomainException.InvalidCpf` |
| `class : DomainException` | Exceção carrega dado associado ao erro | `AcessoNegadoException(recursos: List<String>)` |
| Hierarquia `sealed class` | Chamador precisa ramificar por subtipo para decidir fluxo (ex: abortar tudo vs. só um item) | `SyncFatalDomainException` / `SyncNonFatalDomainException` |

Pattern-matching sobre subtipo de exceção dentro de um `.fold()` (`if (exception is AcessoNegadoException)`) é o uso correto — a ramificação acontece sobre o valor de erro, não sobre o objeto do `throw` ou bloco `catch`.
