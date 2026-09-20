---
name: wayfinder
description: Planejar um grande bloco de trabalho — maior do que uma única sessão de agente comporta — como um mapa compartilhado de tickets de decisão no seu rastreador de issues, e resolvê-los um de cada vez até que o caminho até o destino fique claro.
disable-model-invocation: true
---

Chegou uma ideia solta — grande demais para uma sessão de agente, e envolta em neblina: o caminho daqui até o **destino (destination)** ainda não está visível. Wayfinding é sobre encontrar esse caminho, não avançar direto para o destino. Esta skill traça o caminho como um **mapa compartilhado (shared map)** no rastreador de issues do repositório, e então resolve seus **tickets de decisão (decision tickets)** — perguntas cuja resolução é uma decisão, não fatias de uma build para executar — um de cada vez, até que a rota fique clara.

O destino varia por esforço, e nomeá-lo é o primeiro ato de traçar o mapa — ele molda cada ticket. Pode ser uma spec para entregar e iterar, uma decisão a travar antes de o planejamento começar, ou uma mudança feita in loco, como uma migração de estrutura de dados. O mapa é agnóstico de domínio — trabalho de engenharia, conteúdo de curso, o que se encaixar no formato.

## Planeje, não execute

O Wayfinder é **planejamento (planning)** por padrão: cada ticket resolve uma decisão, e o mapa está pronto quando o caminho está claro — nada mais a decidir antes de alguém ir e fazer a coisa. A vontade de simplesmente executar o trabalho costuma ser o sinal de que você chegou à borda do mapa e é hora de passar a bola. Um esforço pode sobrepor essa regra nas suas **Notas** — trazendo a execução para dentro do próprio mapa — mas, na ausência disso, produza decisões, não entregáveis.

## Refira-se pelo nome

Todo mapa e ticket é uma issue, então tem um **nome** — o seu título. Em tudo o que o humano lê — narração, o Decidido até o momento do mapa — refira-se a ele por esse nome, nunca por um id nu, número, ou slug. Uma parede de `#42, #43, #44` é ilegível; nomes se leem num relance. O id e a URL não desaparecem — um nome envolve seu link — mas eles viajam _dentro_ do nome, nunca no lugar dele.

## O Mapa

O mapa é uma única issue no rastreador de issues deste repositório, rotulada `wayfinder:map` — o artefato canônico. Seus tickets são issues filhas do mapa.

O mapa é um **índice**, não um repositório de conteúdo. Ele lista as decisões tomadas e aponta para os tickets que guardam seu detalhe; uma decisão vive em exatamente um lugar — seu ticket — então o mapa nunca a reafirma, só a resume e a linka.

**Onde o mapa, seus tickets filhos, os bloqueios e as consultas de fronteira vivem fisicamente é específico do rastreador.** O rastreador de issues deveria ter sido informado a você. Se nenhum rastreador foi informado, use por padrão o rastreador local em markdown.

### O corpo do mapa

O mapa inteiro em baixa resolução, carregado uma vez por sessão. Tickets abertos **não** são listados — eles são issues filhas abertas, encontradas por consulta.

```markdown
## Destino final

<como é chegar ao fim deste mapa — a spec, decisão, ou mudança para a qual este esforço está encontrando o caminho. Uma ou duas linhas; toda sessão se orienta por ela antes de escolher um ticket.>

## Notas

<domínio; skills que toda sessão deveria consultar; preferências fixas para este esforço>

## Decidido até o momento

<!-- o índice — uma linha por ticket fechado: o suficiente para julgar relevância, depois dê zoom no link para o detalhe que o ticket guarda -->

- [<título do ticket fechado>](link) — <resumo de uma linha da resposta>

## Ainda não especificado

<!-- veja "Fog of war": neblina dentro do escopo que você ainda não consegue transformar em ticket; amadurece conforme a fronteira avança -->

## Não escopo

<!-- veja "Não escopo": trabalho julgado além do destino; fechado, nunca amadurece -->
```

### Tickets

Cada ticket é uma **issue filha (child issue)** do mapa; o id de issue do rastreador é sua identidade. Seu corpo é a pergunta, dimensionada para caber em 40% da janela de contexto do agente:

```markdown
## Questão

<a decisão ou investigação que este ticket resolve>
```

Cada ticket carrega um label `wayfinder:<type>` — um entre `research`, `prototype`, `grilling`, `task` (veja Tipos de Ticket).

Uma sessão **reivindica (claim)** um ticket ao atribuí-lo ao dev conduzindo o mapa, **primeiro**, antes de qualquer trabalho, para que sessões concorrentes o pulem. Esse responsável designado _é_ a reivindicação: um ticket aberto e não atribuído está sem reivindicação (unclaimed).

Bloqueio (blocking) usa a relação de dependência **nativa** do rastreador — essencial porque isso renderiza a fronteira _visualmente_ na própria UI do rastreador, para que o humano veja o que está disponível para pegar sem precisar abrir o mapa. Só um rastreador que não tem bloqueio nativo recorre a uma convenção no corpo do texto. Um ticket está **desbloqueado (unblocked)** quando todo ticket que o bloqueia está fechado; a **fronteira (frontier)** são as issues filhas abertas, desbloqueadas e sem reivindicação — a borda do conhecido.

A resposta não faz parte do corpo — ela é registrada na resolução (veja Percorra o mapa). Ativos (assets) criados ao resolver um ticket são linkados a partir da issue, não colados nela.

## Tipos de Ticket

Todo ticket é ou **HITL** — human in the loop (humano no loop), conduzido _com_ um humano que fala por si mesmo — ou **AFK** — away from keyboard (longe do teclado), conduzido pelo agente sozinho. Um ticket HITL só se resolve através dessa troca ao vivo; o agente nunca substitui o lado do humano nela (um agente de grilling que responde às próprias perguntas quebrou essa regra).

- **Research** (AFK): Ler documentação, APIs de terceiros, ou recursos locais como bases de conhecimento para trazer à tona um fato do qual uma decisão depende. Resolvido por um **subagente** `/research`. Use quando for necessário conhecimento fora do diretório de trabalho atual.
- **Prototype** (HITL): Eleve a fidelidade da discussão criando um artefato barato, bruto e concreto para reagir a ele — um esboço, um rascunho, um stub, ou código de UI/lógica via a skill /prototype. Linka o protótipo como um ativo (asset). Use quando "como isso deveria parecer" ou "como isso deveria se comportar" for a pergunta central.
- **Grilling** (HITL): Conversa. O caso padrão. Sempre invoque a skill /grill-me.
- **Task** (HITL ou AFK): Trabalho manual que precisa acontecer antes que uma _decisão_ possa ser tomada — nada a decidir, prototipar, ou pesquisar, mas a discussão fica bloqueada até que esteja feito. Cadastrar-se em um serviço para que sua API possa ser avaliada, provisionar acesso, mover dados para que seu formato possa ser visto. Este é o único tipo que _faz_ em vez de decidir — e ele merece seu lugar por desbloquear uma decisão, não por entregar o destino. O agente conduz sozinho onde consegue (AFK); caso contrário, entrega ao humano um checklist preciso (HITL). Resolvido quando o trabalho está feito; a resposta registra o que foi feito e quaisquer fatos resultantes (local das credenciais, novas URLs, contagens de linhas) dos quais tickets posteriores dependem.

## Fog of war

O mapa é _deliberadamente_ incompleto: não trace o que você ainda não consegue ver. Além dos tickets ativos está o **fog of war** — a visão turva de decisões e investigações que você percebe que estão a caminho mas ainda não consegue fixar, porque dependem de perguntas ainda em aberto. Resolver um ticket dissipa a neblina à frente dele, fazendo amadurecer o que agora é especificável em tickets novos — um de cada vez, até que o caminho até o destino esteja claro e não reste nenhum ticket.

A seção **Ainda não especificado** do mapa é onde essa visão turva fica registrada: a pergunta suspeitada, a área a revisitar depois. É a fronteira ainda não descoberta _rumo_ ao destino — tudo aqui está dentro do escopo, só não está nítido o bastante para virar ticket. Escreva de forma tão vaga ou tão completa quanto a visão permitir; isso também funciona como uma placa de sinalização para colaboradores lendo para onde o esforço está indo.

**Neblina ou ticket?** O teste é se você consegue declarar a pergunta com precisão agora — _não_ se você consegue respondê-la agora.

- **Vira ticket quando** a pergunta já está nítida — mesmo que esteja bloqueada e você ainda não possa agir sobre ela.
- **Fica em Ainda não especificado quando** você ainda não consegue formulá-la com essa nitidez. Não pré-fatie a neblina em pedaços do tamanho de um ticket: ela é mais grosseira que um ticket, e um trecho pode amadurecer em vários tickets, ou em nenhum, quando a fronteira o alcançar.

**Ainda não especificado** exclui o que já foi decidido (Decidido até o momento), o que já é um ticket ativo, e o que está fora do escopo (a próxima seção).

## Não escopo

A neblina só se acumula _rumo_ ao destino. O destino fixa o escopo, então trabalho além dele está **fora do escopo (não escopo)** — não é neblina, e não pertence a **Ainda não especificado**. Ele ganha sua própria seção **Não escopo** no mapa: trabalho que você conscientemente excluiu _deste_ esforço. É o escopo, não a falta de nitidez, que o coloca aqui.

Trabalho fora do escopo nunca amadurece — a fronteira para no destino — então ele só retorna se o destino for redesenhado, e aí como um esforço novo, não uma retomada.

Declarar algo fora do escopo é um ato de definição de escopo, não um passo na rota. Quando um ticket que já existe acaba se revelando além do destino — mal-escopado ao traçar o mapa, ou exposto por uma resolução — **feche-o** (um ticket fechado está inequivocamente fora da fronteira) e deixe uma linha na seção **Não escopo**: o resumo mais o porquê de estar fora do escopo, linkando o ticket fechado. Ele fica de fora do **Decidido até o momento**, que registra a rota de fato percorrida — um limite de escopo não é um passo nela.

## Invocação

Dois modos. De qualquer forma, **nunca resolva mais de um ticket por sessão** — com exceção dos tickets de research.

### Trace o mapa

O usuário invoca com uma ideia solta.

1. **Nomeie o destino.** Rode uma sessão de `/grill-me` para fixar para onde este mapa está encontrando o caminho — a spec, decisão, ou mudança. O destino fixa o escopo, então ele é definido primeiro.
2. **Mapeie a fronteira.** Faça grilling de novo, **em largura (breadth-first)** desta vez: espalhe-se por todo o espaço em vez de aprofundar em uma única linha, trazendo à tona as decisões em aberto e os primeiros passos já pegáveis agora. **Se isso não trouxer nenhuma neblina à tona** — o caminho até o destino já está claro, a jornada inteira pequena o bastante para uma sessão — você não precisa de um mapa. Pare e pergunte ao usuário como ele gostaria de prosseguir.
3. **Crie o mapa** (label `wayfinder:map`): Destino final e Notas preenchidos, Decidido até o momento vazio, a neblina esboçada em **Ainda não especificado**.
4. **Crie os tickets que você já consegue especificar agora** como issues filhas do mapa — depois conecte as arestas de bloqueio em uma **segunda passada** (issues precisam de ids antes de poder referenciar umas às outras). Conectá-las as separa entre a fronteira e as bloqueadas; tudo que você ainda não consegue especificar fica na neblina — a seção **Ainda não especificado**.
5. **Dispare os subagentes de research.** Para cada ticket `research` que você acabou de criar, suba um subagente `/research` para resolvê-lo em paralelo, capturando suas descobertas em um branch descartável `research/<name>` com um ponteiro de contexto (context pointer) a partir do ticket.
6. Pare — traçar o mapa é trabalho de uma única sessão; isso não resolve nada manualmente.

### Percorra o mapa

O usuário invoca com um mapa (URL ou número). Um ticket é **opcional** — sem um, é você quem escolhe a próxima decisão, não o usuário.

1. Carregue o **mapa** — a visão de baixa resolução, não o corpo de cada ticket.
2. Escolha o ticket. Se o usuário nomeou um, use-o. Caso contrário, pegue o primeiro ticket da fronteira em ordem. **Reivindique-o**: atribua-o a si mesmo antes de qualquer trabalho.
3. Resolva-o — **dê zoom conforme necessário**: busque o corpo completo de qualquer ticket relacionado ou fechado sob demanda; invoque as skills que o bloco `## Notas` nomeia. Na dúvida, use `/grill-me`.
4. Registre a resolução: publique a resposta como um **comentário de resolução**, **feche** a issue, e **acrescente um ponteiro de contexto (context pointer)** ao Decidido até o momento do mapa.
5. Adicione os tickets recém-surgidos (crie-depois-conecte); faça amadurecer qualquer neblina que a resposta tornou especificável, removendo cada trecho amadurecido de **Ainda não especificado** para que ele viva só como seu novo ticket. Se a resposta revelar que um ticket — este ou outro — está além do destino, **declare-o fora do escopo** em vez de resolvê-lo na rota. Se a decisão invalidar outras partes do mapa, atualize ou apague esses tickets.

O usuário pode rodar tickets desbloqueados em paralelo, então espere que outras sessões estejam editando o rastreador simultaneamente.
