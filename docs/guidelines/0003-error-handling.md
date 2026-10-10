# Tratamento de Erros

De "A Philosophy of Software Design" (Ousterhout), capítulo "Define errors out of existence":

Cada lugar que precisa tratar um erro é complexidade que vaza para a interface e se multiplica pelos chamadores. O objetivo geral é **reduzir o número de lugares onde erros precisam ser tratados**.

Isso vale para **todo** erro — exceção genuína ou caminho infeliz. Um `Result<XxxError, T>` cujo lado de erro poderia não existir também obriga N chamadores a tratá-lo.

## A escada

Aplique em ordem. Desça para o próximo passo só quando o atual não couber.

### 1. Definir fora da existência

Redefina a semântica da operação para que o caso deixe de ser erro. Some o gatilho, some o tratamento em todos os chamadores.

```kotlin
// Antes: chamador precisa tratar NOT_FOUND, embora o estado final desejado já valha
suspend fun deleteRascunho(id: Long): Result<RascunhoError, Unit>

// Depois: idempotente — "rascunho não existe" ao final, independente de ter existido antes
suspend fun deleteRascunho(id: Long)
```

> **Critério**: só redefina se o estado final redefinido estiver correto do ponto de vista de **todos** os chamadores ("o rascunho não existe" ✓). Se a redefinição depende de adivinhar intenção ("o chamador provavelmente quis X"), ela esconde bugs do chamador — desça para o próximo passo.

### 2. Mascarar

O módulo detecta e trata a falha em nível baixo, sem expô-la ao chamador. É legítimo apenas quando os três itens valem:

- **(a) Promessa da interface cumprida**: depois do handler, o chamador recebe exatamente o que a interface prometeu (via retry, fallback para cache válido, recomputação, valor padrão previsto).
- **(b) Falha observável**: a falha é registrada em log/telemetria, mesmo sem aparecer na assinatura.
- **(c) Degradação declarada**: o comentário de **interface** (convenção de documentação da linguagem, ver [0004](0004-comments.md)) descreve quando e para quê o resultado degrada. Comentário de implementação não conta — o chamador não o enxerga.

```kotlin
/**
 * Centro inicial do mapa: coordenada da UBS ou, na ausência dela, capital da UF da Lotação.
 * Sem UF resolvível, degrada para a capital federal. A origem usada é registrada em telemetria.
 */
suspend fun resolveMapaInitialCenter(cnesUbs: String, ufId: Int?): MapaCenter
```

Faltando qualquer item, não é mascaramento — é **engolir o erro**: o chamador acredita que a operação foi cumprida quando não foi. Sinais típicos: `getOrNullIfError() ?: <valor>`, `onError = { <valor> }` ou `catch { log(e) }` sem degradação declarada na interface.

### 3. Agregar

Quando o erro precisa chegar a alguém, trate muitos pontos de falha em **um único lugar**, em vez de espalhar handlers.

Dentro do aplicativo ou na fronteira com o mundo externo, o mecanismo padrão de propagação é um tipo de retorno de erro — em contraste com `try-catch-finally`, que em excesso impacta desempenho de processamento e quebra linearidade do código. O chamador de alto nível é o agregador: um único `.fold()` decide o que fazer.

```kotlin
// interface
suspend fun loadProblemasCondicoes(
    atendimentoId: Int,
): Result<ProblemaCondicaoSoapError, List<ProblemaCondicaoSoap>>

// chamador
result.fold(
    onError = { error -> emit(state.copy(message = error.mapToUiMessage())) },
    onSuccess = { atendimento -> emit(state.copy(atendimento = atendimento)) },
)
```

`throw` é aceitável apenas quando **nasce e morre dentro da mesma camada**: bibliotecas de terceiros lançam, e **um único handler agregador na borda** da camada de acesso a dados captura, registra e converte em retorno de erro. Nenhum `throw` cruza para `lógica de negócio`/`apresentação`.

```kotlin
// agregador de borda — todo acesso ao mundo externo passa por aqui
suspend fun <D> onInfraBoundary(block: () -> D): Result<AppError, D> =
    try {
        Result.Success(block())
    } catch (cancellation: CancellationException) {
        throw cancellation
    } catch (e: Exception) {
        if (e.isFatal()) throw e // ver passo 4
        ExceptionLogger.log(e)
        Result.Error(CommonError.UNEXPECTED)
    }

// acesso a dados — nenhum try/catch próprio
suspend fun getImoveis(cnes: String): Result<AppError, List<Imovel>> =
    onInfraBoundary { database.queryImoveis(cnes) }
```

#### Princípio: Não distinga o que nenhum chamador distingue

Cada tipo de erro é parte da interface. Uma hierarquia (`Timeout` / `NetworkError` / `InvalidPayload`) cujo único chamador captura o supertipo é granularidade sem consumidor: colapse-a. Vale tanto para hierarquias de `throw` dentro da camada quanto para o lado de erro do retorno (ver [Modelando o lado de erro do retorno](#modelando-o-lado-de-erro-do-retorno)).

#### Princípio: Exceções apenas para o que é excepcional

De "O Programador Pragmático" (Hunt & Thomas), uma forma de detectar quando um `throw` é usado indevidamente como fluxo de controle:

> **Teste da exclusão**: Imagine excluir o **manipulador** (`try`/`catch`). Se o código, no caminho normal, continuar completando sua tarefa principal sem nunca precisar daquele `catch`, ele era só uma rede de segurança para uma falha externa/inesperada — uso correto de exceção. Se remover o `catch` impede o código de completar sua tarefa **no fluxo normal** (ex: decidir um resultado de negócio, quebrar um laço, desviar fluxo), o `catch` era parte da lógica de controle disfarçada de tratamento de exceção — uso incorreto: volte ao passo 1 ou modele como retorno de erro.

### 4. Deixar crashar

Quando não há recuperação significativa, não trate.

> **Critério**: crashar é obrigatório quando continuar pode **produzir ou propagar estado corrompido** — um invariante de domínio foi violado. Perder a operação em andamento é preferível a persistir ou sincronizar dado errado.

- Invariantes de domínio e aplicação são verificados com as asserções da linguagem (`require`/`check` ou equivalente) e **nunca** viram retorno de erro nem são mascarados.
- O `block` do agregador de borda contém **apenas** a conversa com o mundo externo (e mapeamento mecânico do que voltou). Lógica de domínio — validação, cálculo, decisão — dentro dele teria seus bugs convertidos em `UNEXPECTED` e escondidos do monitoramento de estabilidade.

```kotlin
// BOM: borda só fala com o mundo externo; invariante fora dela
val visitas = onInfraBoundary { database.queryVisitas(imovelId) }
    .fold(
        onError = { error -> … },
        onSuccess = { visitas ->
            check(visitas.all { it.imovelId == imovelId })
            visitas.map { it.toReadModel() }
        },
    )

// RUIM: invariante violado vira "Erro ao carregar" em vez de crash
onInfraBoundary {
    val visitas = database.queryVisitas(imovelId)
    check(visitas.all { it.imovelId == imovelId })
    visitas.map { it.toReadModel() }
}
```

> **Nota: nunca capture o que sinaliza estado irrecuperável do processo ou do ambiente:**
>
> - **Erros da VM** (`OutOfMemoryError`, `StackOverflowError`): são `Throwable`, não `Exception` — nunca capture `Throwable`/`Error`. Em linguagens JVM: Cuidado com `runCatching`, que captura `Throwable` (inclusive `CancellationException`).
> - **Permissão negada pelo SO** (`SecurityException`): a permissão é verificada antes da chamada (passo 1). Se ainda assim lança, é bug no fluxo de verificação.
> - **Falhas de criptografia** (`GeneralSecurityException`, ex: tag de autenticação inválida): dado adulterado ou chave errada — continuar é operar sobre dado não confiável. Exceção: invalidação de chave prevista pela plataforma (ex: `KeyPermanentlyInvalidatedException` após troca do bloqueio de tela) é evento esperado — trate pela escada (recriar a chave).
>
> Permissão e criptografia são subtipos de `Exception`, então o `catch (e: Exception)` do agregador de borda as capturaria. O agregador as relança explicitamente:
>
> ```kotlin
> fun Exception.isFatal(): Boolean =
>     this is SecurityException ||
>         (this is GeneralSecurityException && this !is KeyPermanentlyInvalidatedException)
> ```

## Modelando o lado de erro do retorno

`<Contexto>Error` pode assumir três formas — escolha pela necessidade do chamador, não por padrão fixo:

| Forma | Quando usar | Exemplo |
|---|---|---|
| `enum class : AppError` | Caso padrão — não carrega dado associado | `CidadaoError.InvalidCpf` |
| `class : AppError` | Erro carrega dado associado ao erro | `AcessoNegadoError(recursos: List<String>)` |
| Hierarquia `sealed class` | Chamador precisa ramificar por subtipo para decidir fluxo (ex: abortar tudo vs. só um item) | `SyncFatalError` / `SyncNonFatalError` |

Pattern-matching sobre subtipo de erro dentro de um `.fold(onError = { error -> … }, …)` (`if (error is AcessoNegadoError)`) é o uso correto — a ramificação acontece sobre o valor de erro, não sobre o objeto do `throw` ou bloco `catch`.
