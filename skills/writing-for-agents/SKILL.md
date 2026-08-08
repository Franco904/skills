---
name: writing-for-agents
description: Escrever documentos para agentes. Use ao criar ou editar skills, ou ao modificar o AGENTS.md ou CLAUDE.md.
---

Referência para escrever qualquer documento consumido por um agente — uma skill, um `AGENTS.md` / `CLAUDE.md`, um doc alcançado por um ponteiro. A embalagem difere; a escrita, não: as mesmas alavancas (levers) tornam cada um previsível — o agente seguindo o mesmo _processo_ a cada execução, não produzindo o mesmo resultado.

Quando o documento que você está escrevendo é uma skill, leia [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md) para frontmatter, a escolha de invocação e skills roteadoras (router skills).

## Ponteiros de contexto

Um **ponteiro de contexto (context pointer)** é uma referência mantida no contexto do agente que nomeia algum material fora do contexto e codifica a condição para alcançá-lo. A description de uma skill é um exemplo; uma linha no `AGENTS.md` que nomeia um doc é o mesmo tipo de objeto. É a _formulação_ do ponteiro, não seu alvo, que decide quando o agente alcança o material — e com que confiabilidade. Um alvo indispensável atrás de um ponteiro mal formulado é um bug de variância: primeiro refine a formulação, e só inclua o material diretamente (inline) se o refinamento falhar.

Um ponteiro cumpre duas funções — declarar o que é o material, e listar as **ramificações (branches)** que devem disparar seu alcance (uma ramificação é um caso distinto que o documento trata, de modo que execuções diferentes percorrem caminhos diferentes por ele). Cada palavra de um ponteiro sempre carregado custa a cada turno, por isso ele merece uma poda ainda mais rigorosa do que o corpo do texto:

- **Coloque a leading word (palavra-guia) no início** — é no ponteiro que ela realiza seu trabalho de gatilho.
- **Um gatilho por ramificação.** Sinônimos que apenas renomeiam uma única ramificação são uma ramificação escrita duas vezes; colapse-os e mantenha apenas ramificações genuinamente distintas.
- **Corte qualquer identidade que o corpo do texto já carrega.**

## As duas cargas

Todo documento e ponteiro que você adiciona gasta uma de duas cargas:

- **Carga de contexto (context load)** — o custo do material sempre carregado sobre a janela de contexto (context window) do agente: uma linha do `AGENTS.md`, a description de uma skill, qualquer coisa que permaneça no contexto a cada turno, gastando tokens e atenção independentemente de disparar ou não.
- **Carga cognitiva (cognitive load)** — o custo sobre o humano: quais documentos existem e quando recorrer a cada um. O humano é o índice. Não é um custo a minimizar — é o preço da agência humana; gaste-a onde o julgamento humano importa, remova-a onde não importa.

Material alcançado apenas por meio de um ponteiro escapa da carga de contexto ao preço da própria linha do ponteiro; material sem ponteiro algum recai inteiramente sobre a carga cognitiva.

## Hierarquia de informação

Um documento é construído a partir de dois tipos de conteúdo — **passos (steps)** (as ações ordenadas que o agente executa) e **referência (reference)** (definições, regras, fatos consultados sob demanda) — que se misturam livremente: só passos (uma receita), só referência (as regras de uma review, esta própria skill), ou os dois. A decisão central é onde cada trecho se posiciona na **hierarquia de informação (information hierarchy)**, uma escada ordenada por quão imediatamente o agente precisa do material:

1. **Passo no arquivo (in-file step)** — o nível primário: o que o agente faz, em ordem.
2. **Referência no arquivo (in-file reference)** — consultada sob demanda. Frequentemente um conjunto legitimamente plano de itens equivalentes (todas as regras de uma review em um único degrau) — um arranjo válido, não um sinal de problema.
3. **Referência revelada (disclosed reference)** — empurrada para um arquivo separado, alcançada por um ponteiro de contexto, carregada somente quando o ponteiro dispara. Vai de um arquivo irmão na mesma pasta até referência totalmente externa, que pode viver em qualquer lugar e ser apontada por qualquer documento.

Empurre pouco demais para baixo e o topo inchará; empurre demais e você esconderá material que o agente realmente precisa. Essa tensão é a decisão inteira.

**Divulgação progressiva (progressive disclosure)** é o movimento de descer a escada — para fora do arquivo principal e atrás de um ponteiro — para que o topo permaneça legível. Não é primariamente uma otimização de tokens: é como a hierarquia é protegida. Ramificação é o teste de divulgação mais limpo: mantenha inline o que toda ramificação precisa, e empurre atrás de um ponteiro o que só algumas ramificações alcançam. Quando um documento tem passos, deixar no arquivo uma referência que deveria ter sido revelada enterra esses passos em meio a ela — o agente notar e segui-los deixa de ser garantido e vira uma questão de sorte a cada execução. Isso não é só um problema de legibilidade: introduz variância, porque afeta o quão consistente é o comportamento do agente entre uma execução e outra.

**Co-localização (co-location)** é a companheira dentro do arquivo: enquanto a escada decide _quão embaixo_ um trecho fica, a co-localização decide _o que fica ao lado dele_ uma vez ali. Mantenha a definição, as regras e as ressalvas de um conceito sob um único título em vez de espalhadas, para que ler uma parte traga seus vizinhos junto. O teste: o documento deve se ler como documentação escrita para o agente — material agrupado se lê assim; material espalhado, não. (é diferente de duplicação: esta repete um único significado em dois lugares; o espalhamento fragmenta um único significado em muitos.)

**Alastramento (sprawl)** é o modo de falha aqui: um documento simplesmente longo demais, mesmo quando cada linha é única. A atenção se dilui pelo excesso, e cada linha extra é mais uma a manter relevante. A solução é a escada: revele referência atrás de ponteiros, e divida por ramificação ou sequência para que cada caminho carregue apenas o que precisa.

## Passos e critérios de conclusão

Todo passo termina em um **critério de conclusão (completion criterion)** — a condição que informa ao agente que o trabalho está feito. Duas propriedades o tornam uma alavanca:

- **Clareza (clarity)** — o agente consegue distinguir feito de não-feito? Um limite vago ("compreensão alcançada") convida à **conclusão prematura (premature completion)**: encerrar o passo antes que ele esteja genuinamente concluído, com a atenção escorregando para _estar concluído_. Os passos visíveis ainda à frente — os **passos pós-conclusão (post-completion steps)** — fornecem a tração; a clareza do critério é a resistência. Defenda-se nesta ordem: **refine o limite primeiro** (local e barato); só se ele for irredutivelmente vago _e_ você observar a pressa, esconda os passos posteriores dividindo a sequência — e esconder só funciona através de uma fronteira de contexto real (uma passagem de bastão ou o despacho a um subagente; uma chamada inline deixa os passos posteriores no contexto e não resolve nada).
- **Exigência (demand)** — quanto o critério requer. "Todo modelo modificado contabilizado" força um trabalho minucioso onde "produza uma lista de mudanças" não força. A exigência impulsiona o **trabalho braçal (legwork)** — a escavação que o agente faz dentro do trabalho, latente na formulação em vez de escrita como um passo próprio — e ela não está presa a passos: "toda regra aplicada" vincula um corpo de referência plana da mesma forma que "todo passo concluído" vincula uma sequência, e é assim que um documento só de referência ainda carrega uma barra de exaustividade.

Os critérios mais fortes são, ao mesmo tempo, verificáveis e exaustivos.

## Quando dividir

Dividir um documento em dois gasta uma das duas cargas, então só divida quando o corte compensar:

- **Por sequência** — divida uma sucessão de passos onde os passos pós-conclusão tentam o agente a apressar o passo à sua frente. Mantê-los fora de vista impulsiona mais trabalho braçal na tarefa atual. Cuidado com o inverso: fundir sequências expõe os passos posteriores de cada passo ao que vem depois, convidando à conclusão prematura.
- **Por invocação** — específico de skills: veja [`SKILL-MECHANICS.md`](SKILL-MECHANICS.md).

## Leading words (palavras-guia)

Uma **leading word (palavra-guia)** é um conceito compacto que já vive no pretraining do modelo, com o qual o agente pensa enquanto executa o documento (_lesson_, _fog of war_, _tracer bullets_). Repetida como um token, nunca como uma frase, ela acumula uma definição distribuída e ancora toda uma região de comportamento no menor número de tokens, ao recrutar priors que o modelo já possui. Cunhar a sua própria funciona se você a definir com clareza, mas uma palavra inventada não recruta prior algum — você paga em tokens de definição o que uma palavra pré-treinada dá de graça; recorra primeiro a uma palavra já existente.

Ela ancora duas vezes. No corpo do texto, _execução_: o agente recorre ao mesmo comportamento toda vez que a palavra aparece, e dentro de referência plana ela foca a atenção em uma classe de coisa a procurar. No ponteiro, _invocação_: quando a mesma palavra vive nos seus prompts, nos seus docs e no seu codebase, o agente vincula essa linguagem compartilhada ao material e o alcança de forma mais confiável.

Cace oportunidades de refatorar com leading words. Uma tríade soletrada em três lugares, um ponteiro gastando uma frase inteira para apontar para uma única ideia — cada uma dessas é uma passagem implorando para colapsar em um único token:

- "fast, deterministic, low-overhead" → _tight_ (um loop _tight_).
- "a loop you believe in" → _red_ — um crivo (gate) vago vira um estado binário observável (o loop fica _red_ no bug, ou não).

Você ganha duas vezes: menos tokens, e um gancho mais afiado para o agente pendurar seu raciocínio. Assuma que todo documento carrega reformulações que leading words aposentariam — vá encontrá-las.

**Negação (negation)** é o modo de falha ao lado dessa alavanca: guiar por proibição arrasta o comportamento proibido para o contexto e o torna _mais_ disponível, não menos. _Don't think of an elephant_, e o elefante é tudo o que resta; a negação é um modificador fraco que o conceito fortemente ativado atropela, de modo que a proibição meio que se lê como uma instrução para fazer a coisa. Formule o **positivo** — declare o comportamento-alvo ("write one-line comments") para que o comportamento banido nunca seja sequer mencionado. Uma proibição só merece seu lugar como um guardrail rígido que você não consegue formular positivamente; mesmo assim, combine-a com o alvo positivo para que a atenção pouse sobre o que fazer.

## Poda (Pruning)

- Mantenha cada significado em uma **fonte única da verdade (single source of truth)**: um único lugar autoritativo, de modo que mudar o comportamento seja uma edição em um só lugar. **Duplicação (duplication)** — o mesmo significado em mais de um lugar — custa manutenção e tokens, e infla a proeminência de um significado na escada além do seu posto real. (O inverso acidental de uma leading word, que repete um token de propósito, nunca o significado.)
- O **ambiente (environment)** também é uma fonte da verdade — scripts do `package.json`, arquivos de config, o layout dos diretórios, a saída de `--help` — e um documento que o reafirma é um **cache**: uma cópia de uma consulta, que só justifica sua carga quando a consulta é cara. Faça cache do que o agente não consegue encontrar apenas olhando: a convenção não escrita, a razão por trás de uma escolha, a pegadinha que nenhuma config confessa. Deixe as consultas de um arquivo, um comando só, para o ambiente, onde elas não podem ficar desatualizadas.
- Verifique cada linha quanto à **relevância (relevance)**: ela ainda tem relação com o que o documento faz? Uma linha perde relevância por nunca ter relação com a tarefa (mera exposição, ou uma ramificação que deveria ter sido revelada) ou por ficar obsoleta conforme o comportamento ou o mundo que ela descreve muda. Documentos mais curtos são mais fáceis de manter relevantes. Sem uma disciplina de poda, o destino padrão é o **sedimento (sediment)**: camadas obsoletas que se acumulam porque adicionar parece seguro e remover parece arriscado, até que seja preciso perfurar através delas para encontrar o que ainda está vivo.
- Cace **no-ops** frase por frase: uma instrução que o modelo já obedece por padrão paga carga para não dizer nada. O teste — isso muda o comportamento em relação ao padrão? — é relativo ao modelo, não ao leitor: duas pessoas discordando sobre um no-op estão discordando sobre o padrão, e isso se resolve executando o documento, não debatendo. Quando uma frase falha no teste, apague a frase inteira em vez de aparar palavras dela. O teste também avalia leading words: uma palavra fraca demais para superar o padrão (_be thorough_ quando o agente já é meio thorough) é um no-op, e o conserto é uma palavra mais forte (_relentless_), não uma técnica diferente.
