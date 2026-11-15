---
name: wayfinder
description: Planejar um grande bloco de trabalho — maior do que uma única sessão de agente comporta — como um mapa compartilhado de issues de decisão no seu issue-tracker, e resolvê-las uma de cada vez até que o caminho até o destino fique claro.
disable-model-invocation: true
---

Chegou uma ideia solta — grande demais para uma sessão de agente, e envolta em neblina: o caminho daqui até o **destino (destination)** ainda não está visível. Wayfinding é sobre encontrar esse caminho, não avançar direto para o destino. Esta skill traça o caminho como um **mapa compartilhado (shared map)** no issue-tracker do repositório, e então resolve suas **issues de decisão (decision issues)** — perguntas cuja resolução é uma decisão, não fatias de uma build para executar — uma de cada vez, até que a rota fique clara.

O destino varia por esforço, e nomeá-lo é o primeiro ato de traçar o mapa — ele molda cada issue. Pode ser uma spec para entregar e iterar, uma decisão a travar antes de o planejamento começar, ou uma mudança feita in loco, como uma migração de estrutura de dados. O mapa é agnóstico de domínio — trabalho de engenharia, conteúdo de curso, o que se encaixar no formato.

## Planeje, não execute

O Wayfinder é **planejamento (planning)** por padrão: cada issue resolve uma decisão, e o mapa está pronto quando o caminho está claro — nada mais a decidir antes de alguém ir e fazer a coisa. A vontade de simplesmente executar o trabalho costuma ser o sinal de que você chegou à borda do mapa e é hora de passar a bola. Um esforço pode sobrepor essa regra nas suas **Notas** — trazendo a execução para dentro do próprio mapa — mas, na ausência disso, produza decisões, não entregáveis.

## Refira-se pelo nome

Todo mapa e toda issue filha têm um **nome** — o seu título. Em tudo o que o humano lê — narração, o Decidido até o momento do mapa — refira-se a cada um por esse nome, nunca por um id nu, número, ou slug. Uma parede de `#42, #43, #44` é ilegível; nomes se leem num relance. O id e a URL não desaparecem — um nome envolve seu link — mas eles transitam _dentro_ do nome, nunca no lugar dele.

## O Mapa

O mapa é uma única issue no issue-tracker deste repositório, rotulada `wayfinder:map` — o artefato canônico. Suas decisões são issues filhas do mapa.

O mapa é um **índice**, não um repositório de conteúdo. Ele lista as decisões tomadas e aponta para as issues que guardam seu detalhe; uma decisão vive em exatamente um lugar — sua issue — então o mapa nunca a reafirma, só a resume e a linka.

**O mapa, suas issues filhas, os bloqueios e as consultas de fronteira vivem no issue-tracker (GitHub, caso não especificado)**.

### O corpo do mapa

O mapa inteiro em baixa resolução, carregado uma vez por sessão. Issues abertas **não** são listadas — elas são issues filhas abertas, encontradas por consulta.

```markdown
## Destino final

<como é chegar ao fim deste mapa — a spec, decisão, ou mudança para a qual este esforço está encontrando o caminho. Uma ou duas linhas; toda sessão se orienta por ela antes de escolher uma issue.>

## Notas

<domínio; skills que toda sessão deveria consultar; preferências fixas para este esforço>

## Decidido até o momento

<!-- o índice — uma linha por issue fechada: o suficiente para julgar relevância, depois dê zoom no link para o detalhe que a issue guarda -->

- [<título da issue fechada>](link) — <resumo de uma linha da resposta>

## Ainda não especificado

<!-- veja "Fog of war": neblina dentro do escopo que você ainda não consegue transformar em issue; amadurece conforme a fronteira avança -->

## Não escopo

<!-- veja "Não escopo": trabalho julgado além do destino; fechado, nunca amadurece -->
```

### Issues

Cada decisão é uma **issue filha** do mapa; o id da issue no issue-tracker é sua identidade. Seu corpo é a pergunta, dimensionada para caber em 40% da janela de contexto do agente:

```markdown
## Questão

<a decisão ou investigação que esta issue resolve>
```

Cada issue carrega um label `wayfinder:<type>` — um entre `research`, `prototype`, `grilling`, `task` (veja Tipos de Issue).

Uma sessão **reivindica (claim)** uma issue ao atribuí-la ao dev conduzindo o mapa, **primeiro**, antes de qualquer trabalho, para que sessões concorrentes a pulem. Esse responsável designado _é_ a reivindicação: uma issue aberta e não atribuída está sem reivindicação (unclaimed).

Bloqueio (blocking) usa a relação de dependência **nativa** do issue-tracker — essencial porque isso renderiza a fronteira _visualmente_ na própria UI do issue-tracker, para que o humano veja o que está disponível para pegar sem precisar abrir o mapa. Só um issue-tracker que não tem bloqueio nativo recorre a uma convenção no corpo do texto. Uma issue está **desbloqueada (unblocked)** quando toda issue que a bloqueia está fechada; a **fronteira (frontier)** são as issues filhas abertas, desbloqueadas e sem reivindicação — a borda do conhecido.

A resposta não faz parte do corpo — ela é registrada na resolução (veja Percorra o mapa). Ativos (assets) criados ao resolver uma issue são linkados a partir da issue, não colados nela.

## Tipos de Issue

Toda issue é ou **HITL** — human in the loop (humano no loop), conduzido _com_ um humano que fala por si mesmo — ou **AFK** — away from keyboard (longe do teclado), conduzido pelo agente sozinho. Uma issue HITL só se resolve através dessa troca ao vivo; o agente nunca substitui o lado do humano nela (um agente de grilling que responde às próprias perguntas quebrou essa regra).

- **Research** (AFK): Ler documentação, APIs de terceiros, ou recursos locais como bases de conhecimento para trazer à tona um fato do qual uma decisão depende. Resolvido por um **subagente** de research. Use quando for necessário conhecimento fora do diretório de trabalho atual.
- **Prototype** (HITL): Eleve a fidelidade da discussão criando um artefato barato, bruto e concreto para reagir a ele — um esboço, um rascunho, um stub, ou código de UI/lógica via a skill /prototype. Linka o protótipo como um ativo (asset). Use quando "como isso deveria parecer" ou "como isso deveria se comportar" for a pergunta central.
- **Grilling** (HITL): Conversa. O caso padrão. Sempre invoque a skill /grill-me.
- **Task** (HITL ou AFK): Trabalho manual que precisa acontecer antes que uma _decisão_ possa ser tomada — nada a decidir, prototipar, ou pesquisar, mas a discussão fica bloqueada até que esteja feito. Cadastrar-se em um serviço para que sua API possa ser avaliada, provisionar acesso, mover dados para que seu formato possa ser visto. Este é o único tipo que _faz_ em vez de decidir — e ele merece seu lugar por desbloquear uma decisão, não por entregar o destino. O agente conduz sozinho onde consegue (AFK); caso contrário, entrega ao humano um checklist preciso (HITL). Resolvido quando o trabalho está feito; a resposta registra o que foi feito e quaisquer fatos resultantes (local das credenciais, novas URLs, contagens de linhas) dos quais issues posteriores dependem.

## Fog of war

O mapa é _deliberadamente_ incompleto: não trace o que você ainda não consegue ver. Além das issues ativas está o **fog of war** — a visão turva de decisões e investigações que você percebe que estão a caminho mas ainda não consegue fixar, porque dependem de perguntas ainda em aberto. Resolver uma issue dissipa a neblina à frente dela, fazendo amadurecer o que agora é especificável em issues novas — uma de cada vez, até que o caminho até o destino esteja claro e não reste nenhuma issue.

A seção **Ainda não especificado** do mapa é onde essa visão turva fica registrada: a pergunta suspeitada, a área a revisitar depois. É a fronteira ainda não descoberta _rumo_ ao destino — tudo aqui está dentro do escopo, só não está nítido o bastante para virar issue. Escreva de forma tão vaga ou tão completa quanto a visão permitir; isso também funciona como uma placa de sinalização para colaboradores lendo para onde o esforço está indo.

**Neblina ou issue?** O teste é se você consegue declarar a pergunta com precisão agora — _não_ se você consegue respondê-la agora.

- **Vira issue quando** a pergunta já está nítida — mesmo que esteja bloqueada e você ainda não possa agir sobre ela.
- **Fica em Ainda não especificado quando** você ainda não consegue formulá-la com essa nitidez. Não pré-fatie a neblina em pedaços do tamanho de uma issue: ela é mais grosseira que uma issue, e um trecho pode amadurecer em várias issues, ou em nenhuma, quando a fronteira o alcançar.

**Ainda não especificado** exclui o que já foi decidido (Decidido até o momento), o que já é uma issue ativa, e o que está fora do escopo (a próxima seção).

## Não escopo

A neblina só se acumula _rumo_ ao destino. O destino fixa o escopo, então trabalho além dele está **fora do escopo (não escopo)** — não é neblina, e não pertence a **Ainda não especificado**. Ele ganha sua própria seção **Não escopo** no mapa: trabalho que você conscientemente excluiu _deste_ esforço. É o escopo, não a falta de nitidez, que o coloca aqui.

Trabalho fora do escopo nunca amadurece — a fronteira para no destino — então ele só retorna se o destino for redesenhado, e aí como um esforço novo, não uma retomada.

Declarar algo fora do escopo é um ato de definição de escopo, não um passo na rota. Quando uma issue que já existe acaba se revelando além do destino — mal-escopada ao traçar o mapa, ou exposta por uma resolução — **feche-a** (uma issue fechada está inequivocamente fora da fronteira) e deixe uma linha na seção **Não escopo**: o resumo mais o porquê de estar fora do escopo, linkando a issue fechada. Ela fica de fora do **Decidido até o momento**, que registra a rota de fato percorrida — um limite de escopo não é um passo nela.

## Invocação

Dois modos. De qualquer forma, **nunca resolva mais de uma issue por sessão** — com exceção das issues de research.

### Trace o mapa

O usuário invoca com uma ideia solta.

1. **Nomeie o destino.** Rode uma sessão de `/grill-me` para fixar para onde este mapa está encontrando o caminho — a spec, decisão, ou mudança. O destino fixa o escopo, então ele é definido primeiro.
2. **Mapeie a fronteira.** Faça grilling de novo, **em largura (breadth-first)** desta vez: espalhe-se por todo o espaço em vez de aprofundar em uma única linha, trazendo à tona as decisões em aberto e os primeiros passos já pegáveis agora. **Se isso não trouxer nenhuma neblina à tona** — o caminho até o destino já está claro, a jornada inteira pequena o bastante para uma sessão — você não precisa de um mapa. Pare e pergunte ao usuário como ele gostaria de prosseguir.
3. **Crie o mapa** (label `wayfinder:map`): Destino final e Notas preenchidos, Decidido até o momento vazio, a neblina esboçada em **Ainda não especificado**.
4. **Crie as issues que você já consegue especificar agora** como issues filhas do mapa — depois conecte as arestas de bloqueio em uma **segunda passada** (issues precisam de ids antes de poder referenciar umas às outras). Conectá-las as separa entre a fronteira e as bloqueadas; tudo que você ainda não consegue especificar fica na neblina — a seção **Ainda não especificado**.
5. **Dispare os subagentes de research.** Para cada issue `research` que você acabou de criar, suba um subagente de research para resolvê-la em paralelo, capturando suas descobertas em um branch descartável `research/<name>` com um ponteiro de contexto (context pointer) a partir da issue.
6. Pare — traçar o mapa é trabalho de uma única sessão; isso não resolve nada manualmente.

### Percorra o mapa

O usuário invoca com um mapa (URL ou número). Uma issue é **opcional** — sem uma, é você quem escolhe a próxima decisão, não o usuário.

1. Carregue o **mapa** — a visão de baixa resolução, não o corpo de cada issue.
2. Escolha a issue. Se o usuário nomeou uma, use-a. Caso contrário, pegue a primeira issue da fronteira em ordem. **Reivindique-a**: atribua-a a si mesmo antes de qualquer trabalho.
3. Resolva-a — **dê zoom conforme necessário**: busque o corpo completo de qualquer issue relacionada ou fechada sob demanda; invoque as skills que o bloco `## Notas` nomeia. Na dúvida, use `/grill-me`.
4. Registre a resolução: publique a resposta como um **comentário de resolução**, **feche** a issue, e **acrescente um ponteiro de contexto (context pointer)** ao Decidido até o momento do mapa.
5. Adicione as issues recém-surgidas (crie-depois-conecte); faça amadurecer qualquer neblina que a resposta tornou especificável, removendo cada trecho amadurecido de **Ainda não especificado** para que ele viva só como sua nova issue. Se a resposta revelar que uma issue — esta ou outra — está além do destino, **declare-a fora do escopo** em vez de resolvê-la na rota. Se a decisão invalidar outras partes do mapa, atualize ou apague essas issues.

O usuário pode rodar issues desbloqueadas em paralelo, então espere que outras sessões estejam editando o issue-tracker simultaneamente.
