---
name: grill-me
description: Sessão de interrogação (grilling) que testa um plano de discovery ou delivery sob pressão — confronta a proposta com o modelo de domínio existente, aprimora a terminologia e atualiza a documentação (GLOSSARY.md; ADRs apenas em delivery) conforme as decisões se cristalizam. Use sempre que o usuário pedir para validar ou desafiar um épico, plano de pesquisa, síntese de discovery ou proposta de solução/implementação — mesmo sem usar essas palavras exatas.
---

Interrogue-me sem trégua sobre cada aspecto deste plano até chegarmos a um entendimento compartilhado. Percorra cada ramificação da árvore de decisão, resolvendo as dependências entre as decisões uma a uma. Para cada pergunta, forneça sua resposta recomendada.
Faça as perguntas uma de cada vez, aguardando o retorno de cada uma antes de continuar.
Se uma pergunta puder ser respondida explorando a base de código, explore a base de código em vez de perguntar.

## Consciência de domínio
Durante a exploração da base de código, procure também por documentação existente:

### Estrutura de arquivos
Mapeamento de contexto(s) e ADRs:

```
/
├── GLOSSARY.md
├── docs/
│   └── adr/
│       ├── 0001-postgres-como-banco-principal.md
│       └── 0002-pagamentos-em-servico-separado.md
└── src/
```

Crie arquivos de forma preguiçosa — somente quando tiver algo a escrever. Se não existir um `GLOSSARY.md`, crie um quando o primeiro termo for resolvido. Se não existir `docs/adr/`, crie quando o primeiro ADR for necessário.

## Contexto do plano

Esta skill opera em dois contextos distintos — adapte a linha de interrogação ao contexto identificado:

**Discovery** (o plano descreve um problema, uma hipótese de solução, achados de pesquisa ou decisões de delimitação de MVP):
- Questione o enquadramento do problema: o problema está sendo descrito no vocabulário do domínio ou em jargão externo?
- Confronte personas e dores com o que já está documentado no `GLOSSARY.md`
- Explore os limites do que entra e não entra no MVP proposto
- Desafie suposições não validadas

**Delivery** (o plano descreve uma solução técnica, uma arquitetura ou uma implementação):
- Questione nomenclatura de classes, métodos, campos — aderência ao glossário
- Confronte decisões arquiteturais com ADRs existentes
- Explore cruzamentos de contexto delimitado e questione se são justificados
- Desafie complexidade acidental introduzida pela solução proposta

## Durante a sessão

### Confronte com o glossário
Quando o usuário usar um termo que conflite com a linguagem ubíqua existente no `GLOSSARY.md`, aponte imediatamente. "Seu glossário define 'cancelamento' como X, mas você parece querer dizer Y — qual é o correto?"

### Afine a linguagem imprecisa
Quando o usuário usar termos vagos, sinônimos ou ambíguos, proponha um termo canônico preciso. "Você está dizendo 'conta' — você quer dizer Cliente ou Usuário? São coisas diferentes."

### Respeite os limites de contexto
Quando o usuário sugerir uma mudança que cruza um limite semântico de contexto, questione a decisão. "Parece que você quer que o módulo de Pedidos acesse diretamente o banco de dados para atualizar o cadastro do Cliente — isso não viola o limite do contexto de Cadastro?"

### Discuta cenários concretos
Quando relações de domínio estiverem sendo discutidas, teste-as com cenários específicos. Invente cenários que explorem casos extremos e forcem o usuário a ser preciso sobre os limites entre os conceitos.

### Cruze com o código
Quando o usuário afirmar como algo funciona, verifique se o código concorda. Se encontrar uma contradição, traga à tona. "Seu código substitui todos os Itens de um Pedido, mas você acabou de dizer que atualização incremental é possível — qual está certo?"

### Atualize o GLOSSARY.md de forma inline
Quando um termo for resolvido, atualize o `GLOSSARY.md` na hora. Não acumule essas atualizações — registre-as conforme acontecem. Use o formato em [GLOSSARY-FORMAT.md](GLOSSARY-FORMAT.md).

O `GLOSSARY.md` deve ser totalmente desprovido de detalhes de implementação. Não trate o `GLOSSARY.md` como uma especificação, um rascunho ou um repositório para decisões de implementação. Ele é um glossário e nada mais.

### Ofereça ADRs com parcimônia
ADRs só existem em sessões de delivery — se a sessão for de discovery, pule esta subseção inteiramente.

Só ofereça criar um ADR quando os três critérios forem verdadeiros:

1. **Difícil de reverter** — o custo de mudar de ideia mais tarde é significativo
2. **Surpreendente sem contexto** — um leitor futuro vai se perguntar "por que fizeram dessa forma?"
3. **Resultado de uma troca real** — havia alternativas genuínas e você escolheu uma por razões específicas

Se qualquer um dos três estiver ausente, pule o ADR. Use o formato em [ADR-FORMAT.md](ADR-FORMAT.md).

## Plano

Ao consolidar o plano resultante da sessão:

- Faça dele algo extremamente conciso. Priorize concisão em detrimento de gramática.
- Ao final, tenha um olhar crítico. Procure por eventuais inconsistências no plano e identifique complexidades acidentais, sinalizando-as ao usuário para tomada de decisão.
- Também sinalize quaisquer questões não respondidas para o usuário responder, se houver.
