# Mecânica das skills

A ramificação específica de skills de [`writing-for-agents`](SKILL.md): o que muda quando o documento é uma skill — frontmatter, a escolha de invocação e skills roteadoras (router skills). Todo o resto sobre como escrevê-lo referencia `SKILL.md`.

## Invocação

Duas escolhas, que trocam entre as duas cargas:

- Uma skill **invocada pelo modelo (model-invoked)** mantém uma `description`, de modo que o agente pode dispará-la autonomamente — e outras skills podem alcançá-la. Você ainda pode digitar o nome dela: a invocação pelo modelo sempre _inclui_ o alcance pelo usuário; uma description só faz adicionar descoberta pelo agente, nunca remove a do humano. A description é o ponteiro de contexto de mais alto nível da skill, forçado a permanecer carregado o tempo todo — carga de contexto permanente em troca de descobribilidade. Uma skill invocada pelo modelo cujo conteúdo é só referência também é um lar para referência compartilhada: outra skill pode invocá-la, de modo que referência necessária a várias skills vive em um só lugar. Mecânica: omita `disable-model-invocation`, e escreva uma description voltada ao modelo que carregue as ramificações de gatilho (as regras de escrita de ponteiros em `SKILL.md` se aplicam integralmente).
- Uma skill **invocada pelo usuário (user-invoked)** retira a description do alcance do agente: só o humano digitando o nome dela pode invocá-la, e nenhuma outra skill pode. Carga de contexto zero, mas ela gasta carga cognitiva — você é o índice que precisa lembrar que ela existe. Mecânica: defina `disable-model-invocation: true`; a `description` passa a ser voltada ao humano — um resumo de uma linha, sem as listas de gatilho.

Escolha invocação pelo modelo apenas quando o agente precisar alcançar a skill sozinho, ou quando outra skill precisar. Se ela só dispara na mão, torne-a invocada pelo usuário e não pague carga de contexto.

Referência compartilhada de que duas skills invocadas pelo usuário precisam não pode viver em nenhuma das duas — sem descriptions, nenhuma consegue disparar a outra. Empurre-a para um arquivo simples fora do sistema de skills: referência externa que qualquer skill pode apontar.

## Divisão por invocação

O corte por invocação da divisão (o corte por sequência vive em `SKILL.md`): separe uma skill invocada pelo modelo quando você tiver uma leading word distinta que deveria dispará-la por conta própria — uma palavra de gatilho que você de fato usa nos seus prompts — ou quando outra skill precisar alcançá-la. Você paga carga de contexto pela nova description sempre carregada, então esse alcance independente precisa valer a pena.

## Skills roteadoras (Router skills)

Quando skills invocadas pelo usuário se multiplicam além do que você consegue lembrar, essa carga cognitiva acumulada é curada por uma **skill roteadora (router skill)**: uma única skill invocada pelo usuário que nomeia as demais e quando recorrer a cada uma, de modo que o humano tenha uma skill para lembrar em vez de muitas. Ela só pode apontar, nunca disparar as outras: skills invocadas pelo usuário não têm description, então nada além do humano consegue alcançá-las.
