# Nomes

De "A Philosophy of Software Design" (Ousterhout), capítulo 14 ("Choosing Names"):

Um bom nome cria na mente do leitor uma imagem do que a entidade é. Um nome mal escolhido obriga o leitor a abrir a implementação ou o leva a supor errado. Quando o conceito é de domínio, o nome é o termo do `CONTEXT.md`. A escada nome → comentário está na [0004](0004-comments.md#nome-antes-de-comentário).

## Precisão

O nome diz o que a entidade é, sem deixar lacuna para a suposição do leitor. Um nome **vago** (`data`, `result`, `status`, `flag`) só é aceitável como índice de um loop de poucas linhas (`i`). Se nenhum nome preciso cabe, a entidade não tem um propósito único. Nesse caso, volte à [0002](0002-interface-design.md).

```kotlin
// RUIM
fun getStatus(): Int

// BOM
fun getSituacaoMapa(): SituacaoMapaReadModel
```

## Consistência

Cada conceito tem um único nome, e cada nome tem um único conceito. Dois nomes para o mesmo conceito são **sinônimos**, e isso inclui um termo diferente do usado no `CONTEXT.md`. Um nome usado para dois conceitos é **ambíguo**.

```kotlin
// RUIM: sinônimo, Pessoa e Cidadão para o mesmo conceito
suspend fun saveCidadao(cidadao: Cidadao)
suspend fun getPessoaAtendimentos(pessoaId: Long): List<Atendimento>

// BOM
suspend fun saveCidadao(cidadao: Cidadao)
suspend fun getCidadaoAtendimentos(cidadaoId: Long): List<Atendimento>
```

## Palavras extras

Toda palavra do nome acrescenta informação.

```kotlin
// RUIM
val mapaData: Mapa
class MapaServiceImpl : MapaService // única implementação

// BOM
val mapa: Mapa
```

## Red flags

| Red flag | Severidade |
|---|---|
| Nome ambíguo ([Consistência](#consistência)) | `[must fix]` |
| Nome sinônimo ([Consistência](#consistência)) | `[should fix]` |
| Nome vago ([Precisão](#precisão)) | `[should fix]` |
| Palavra extra ([Palavras extras](#palavras-extras)) | `[nitpick]` |
| Nome difícil de escolher ([Precisão](#precisão)) | code-smell de design, arrumar ao escrever |
