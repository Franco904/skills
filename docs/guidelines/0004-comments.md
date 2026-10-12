# Comentários

De "A Philosophy of Software Design" (Ousterhout), capítulos sobre comentários (12, 13, 15 e 16):

Um comentário diz ao leitor o que o código não consegue dizer. O objetivo é reduzir, para o chamador, dois sintomas de complexidade: a **carga cognitiva** (quanto ele precisa saber para usar o módulo) e os **desconhecidos desconhecidos** (o que ele precisa saber e só descobriria lendo a implementação).

## Nome antes de comentário

A escada é: primeiro o nome, depois o comentário. Se o comentário só existe para explicar o que um identificador (variável, parâmetro, função, classe) significa, renomeie o identificador e remova o comentário. Quando o conceito é de domínio, o nome vem do `GLOSSARY.md` (ver [0005](0005-naming.md)). Isso evita ambiguidade, sinônimos e suposições do chamador. O comentário fica só com o que nenhum nome curto consegue carregar de maneira concisa e óbvia.

```kotlin
// RUIM: comentário compensa o nome
// dias desde a última visita
val d: Int

// BOM
val daysSinceLastVisita: Int
```

## Comentário de interface

Usa a convenção de documentação da linguagem (KDoc, Javadoc, docstring etc.) e fica logo acima da declaração pública. Descreve a parte da interface (definida na [0002](0002-interface-design.md)) que a assinatura não mostra.

É obrigatório quando ao menos um _gatilho_ se aplica:

- significado de `null`, de um booleano ou de um valor sentinela;
- degradação ou mascaramento (ver [0003, passo 2](0003-error-handling.md#2-mascarar));
- efeito colateral não óbvio pelo nome;
- pré-condição ou garantia de que o chamador depende (idempotência, thread-safety, pureza, determinismo);
- metainformação de valor, seja de parâmetro ou de retorno (unidade de medida, precisão decimal);
- o porquê de uma escolha que surpreenderia o chamador.

Quando nenhum gatilho se aplica, a declaração fica sem comentário.

```kotlin
// BOM: o chamador conhece cada degradação sem abrir o corpo

/**
 * Situação do mapa da UF da Lotação selecionada.
 *
 * Degrada para [SituacaoMapaReadModel.ATUALIZADO] quando a UF ou o catálogo não podem ser
 * resolvidos — na dúvida, não oferece download ao Profissional. Degrada para
 * [SituacaoMapaReadModel.AUSENTE] quando a leitura do arquivo falha, induzindo novo download.
 */
suspend fun getSituacaoMapa(): SituacaoMapaReadModel

// RUIM: degradações no corpo, invisíveis ao chamador

suspend fun getSituacaoMapa(): SituacaoMapaReadModel {
    // sem UF, não oferece download de atualização
    val ufSlug = currentUfSlug() ?: return SituacaoMapaReadModel.ATUALIZADO

    // falha de leitura indica ausência e força novo download
    val hasMapFile = mapaFileDataSource.hasMapaInstalled(ufSlug).getOrNull() ?: false
    if (!hasMapFile) return SituacaoMapaReadModel.AUSENTE

    // sem catálogo encontrado, não oferece download de atualização
    val catalogoUf = getCatalogoUf(ufSlug) ?: return SituacaoMapaReadModel.ATUALIZADO
    …
}
```

## Comentário de implementação

Usa comentário de linha ou de bloco dentro do corpo, logo acima do trecho, e explica **o quê** (o propósito de um bloco, em nível mais alto que o código) ou **por quê** (garantia preservada, alternativa descartada, armadilha da biblioteca/plataforma). O contrato (significado de `null`, degradação, efeito colateral) vai para o comentário de interface, porque o chamador não lê o corpo.

```kotlin
downloadedFile.copyTo(tempFile, overwrite = true)
if (!validateIntegrity(tempFile)) return false

// renameTo() no mesmo filesystem (sempre o caso em filesDir) é rename(2), que
// substitui o destino atomicamente — nunca deixa o app sem mapa instalado.
if (!tempFile.renameTo(destinationFile)) return false
```

## Teste da exclusão

- **Exclua o comentário.** Se o chamador não perde nenhuma informação que já não estivesse na assinatura ou no código, o comentário era repetição. Remova-o.
- **Exclua o corpo.** Se, vendo só a assinatura e o comentário de interface, o chamador ainda precisaria abrir a implementação para usar o método corretamente, falta um gatilho no comentário. Isso é um desconhecido desconhecido.

```kotlin
// RUIM: repete a assinatura e omite o que o chamador precisa saber
/** Retorna o arquivo do mapa. */
suspend fun getMapFile(): File?

// BOM
/** Arquivo `.map` instalado da UF da Lotação selecionada. `null` quando a UF não pode ser resolvida. */
suspend fun getMapFile(): File?
```

Se o comentário de interface precisa descrever a implementação para ser entendido, o módulo está raso. Ver [0002](0002-interface-design.md#deep-modules-x-shallow-modules).

## Comentário primeiro

Escreva o comentário de interface junto com a assinatura, antes da implementação. Se ele não sai curto, o problema está na interface: volte à [0002](0002-interface-design.md).

## Manutenção

- **O código é a fonte da verdade.** Todo comentário, de interface ou de implementação, vive no código. Repetir a explicação no commit ou na PR é aceitável. Deixá-la só lá, não.
- **Proximidade.** O comentário fica logo acima do código objeto que ele descreve.
- **Fonte única.** Cada explicação fica em um único lugar do código. Quando não há lugar óbvio, como uma decisão que cruza módulos, ela vai para um ADR em `docs/adr/`.
- **O comentário acompanha o código.** Um diff que muda um fato documentado atualiza o comentário de interface no mesmo commit, evitando mentir aos chamadores.
- **TODO não é comentário.** Trabalho pendente vai para a lista de comportamentos da `tdd` ou para uma issue de débito técnico, nunca para o código.
- **Antes do commit**, o diff passa no Teste da exclusão: toda declaração pública com gatilho tem comentário de interface, todo trecho cujo o quê ou por quê não é óbvio tem comentário de implementação, e não há TODO.

## Red flags

| Red flag | Severidade |
|---|---|
| Comentário de interface desatualizado ([Manutenção](#manutenção)) | `[must fix]` |
| Degradação ausente ou fora do comentário de interface ([Comentário de interface](#comentário-de-interface)) | `[must fix]` |
| Outro gatilho sem comentário de interface ([Comentário de interface](#comentário-de-interface)) | `[should fix]` |
| Contrato em comentário de implementação ([Comentário de implementação](#comentário-de-implementação)) | `[should fix]` |
| Detalhe de implementação em comentário de interface ([Teste da exclusão](#teste-da-exclusão)) | `[should fix]` |
| Nome explicado por comentário ([Nome antes de comentário](#nome-antes-de-comentário)) | `[should fix]` |
| TODO no diff ([Manutenção](#manutenção)) | `[should fix]` |
| Explicação só no commit ou na PR ([Manutenção](#manutenção)) | `[should fix]` |
| Comentário que repete a assinatura ou o código ([Teste da exclusão](#teste-da-exclusão)) | `[nitpick]` |
| Comentário difícil de escrever ([Comentário primeiro](#comentário-primeiro)) | code-smell de design, arrumar ao escrever |
