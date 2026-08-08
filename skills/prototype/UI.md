# Protótipo de UI

Gere **várias variações de UI radicalmente diferentes** em uma única rota, alternáveis a partir de uma barra flutuante na parte inferior. O usuário alterna entre as variantes no navegador, escolhe uma (ou rouba pedaços de cada uma), e depois descarta o resto.

Se a pergunta for sobre lógica/estado em vez de como algo se parece — ramificação errada. Use [LOGIC.md](LOGIC.md).

## Quando este é o formato certo

- "Como essa página deveria se parecer?"
- "Quero ver algumas opções para esse dashboard antes de me decidir."
- "Tente um layout diferente para a tela de configurações."
- Qualquer momento em que o usuário, de outra forma, gastaria um dia escolhendo entre três mockups vagos na cabeça.

## Dois subformatos — prefira fortemente o subformato A

Um protótipo de UI é muito mais fácil de avaliar quando está **encostado no resto do app** — header real, sidebar real, dados reais, densidade real. Uma rota descartável isolada é um vácuo: toda variante parece boa isoladamente. Use por padrão o subformato A sempre que houver uma página existente plausível para hospedar as variantes. Só recorra ao subformato B se o protótipo genuinamente não tiver um lar por perto.

### Subformato A — ajuste a uma página existente (preferido)

A rota já existe. As variantes são renderizadas **na mesma rota**, controladas por um parâmetro de busca (search param) `?variant=` na URL. A busca de dados, os params e a autenticação existentes permanecem — só a renderização muda. Esse é o padrão; escolha-o a menos que haja um motivo específico para não fazê-lo.

Se o protótipo é para algo que ainda não tem uma página, mas *naturalmente viveria dentro de uma* (uma nova seção do dashboard, um novo card na tela de configurações, um novo passo em um fluxo existente) — isso ainda é o subformato A. Monte as variantes dentro da página hospedeira.

### Subformato B — uma nova página (último recurso)

Use isto apenas quando a coisa sendo prototipada genuinamente não tiver nenhuma página existente para viver dentro — por exemplo, uma superfície de topo inteiramente nova, ou um fluxo que não pode ser embutido em nenhum lugar sensato.

Crie uma **rota descartável** seguindo a convenção de rotas que o projeto já usa — não invente uma nova estrutura de topo. Nomeie-a de forma que fique óbvio que é um protótipo (por exemplo, incluindo a palavra `prototype` no caminho ou no nome do arquivo). Mesmo padrão `?variant=`.

Antes de se comprometer com o subformato B, faça uma checagem de sanidade: será que realmente não há nenhuma página existente onde isso poderia ser embutido? Uma rota vazia esconde problemas de design que uma rota povoada exporia.

Em ambos os subformatos, a barra flutuante inferior é idêntica.

## Processo

### 1. Declare a pergunta e escolha N

Use por padrão **3 variantes**. Mais de 5 deixa de ser radicalmente diferente e passa a ser ruído — limite-se a isso.

Anote o plano em uma linha, no local do protótipo ou em um comentário no topo do arquivo:

> "Três variantes da página de configurações, alternáveis via `?variant=`, na rota `/settings` existente."

Isso funciona tanto se o usuário estiver aqui para dar feedback quanto se não estiver.

### 2. Gere variantes radicalmente diferentes

Rascunhe cada variante. Avalie cada uma segundo:

- O propósito da página e os dados a que ela tem acesso.
- A biblioteca de componentes / sistema de estilos do projeto (TailwindCSS, shadcn, MUI, CSS puro, o que for).
- Um nome de componente exportado claro, por exemplo `VariantA`, `VariantB`, `VariantC`.

As variantes precisam ser **estruturalmente diferentes** — layout diferente, hierarquia de informação diferente, affordance primária diferente, não apenas cores diferentes. Três grades de cards levemente ajustadas não é um protótipo de UI, é papel de parede. Se dois rascunhos saírem parecidos demais, refaça um com uma instrução explícita de "não use uma grade de cards".

### 3. Conecte tudo

Crie um único componente switcher na rota:

```tsx
// pseudo-code — adapt to the project's framework
const variant = searchParams.get('variant') ?? 'A';
return (
  <>
    {variant === 'A' && <VariantA {...data} />}
    {variant === 'B' && <VariantB {...data} />}
    {variant === 'C' && <VariantC {...data} />}
    <PrototypeSwitcher variants={['A','B','C']} current={variant} />
  </>
);
```

Para o subformato A (página existente): mantenha toda a busca de dados existente acima do switcher; só a subárvore renderizada muda por variante.

Para o subformato B (página nova): a rota descartável em `/prototype/<name>` monta o mesmo switcher.

### 4. Construa o switcher flutuante

Uma pequena barra de posição fixa no centro inferior da tela, com três elementos:

- **Seta para a esquerda** — cicla para a variante anterior (dá a volta ao chegar no início).
- **Rótulo da variante** — mostra a chave da variante atual e, se a variante exportar um nome, esse nome também. Ex.: `B — Sidebar layout`.
- **Seta para a direita** — cicla para a próxima (dá a volta ao chegar no fim).

Comportamento:

- Clicar em uma seta atualiza o search param da URL (use o router do framework — `router.replace` no Next, `navigate` no React Router, etc.) para que a variante seja compartilhável e estável a recarregamentos.
- Teclado: as teclas `←` e `→` também ciclam. Não intercepte as teclas de seta quando um `<input>`, `<textarea>`, ou `[contenteditable]` estiver focado.
- Visualmente distinto da página (por exemplo, uma pílula de alto contraste, sombra sutil) para que fique óbvio que não faz parte do design sendo avaliado.
- Oculto em builds de produção — condicione a `process.env.NODE_ENV !== 'production'` ou uma checagem equivalente, para que um merge acidental do protótipo não leve a barra até os usuários.

Coloque o switcher em um único componente compartilhado para que ambos os subformatos possam reutilizá-lo. Localize-o onde quer que a UI compartilhada viva no projeto.

### 5. Entregue

Exponha a URL (e as chaves `?variant=`). O usuário vai alternar entre elas quando tiver oportunidade. O feedback interessante costuma ser **"quero o header da B com a sidebar da C"** — esse é o design real que eles querem.

### 6. Capture a resposta e limpe

Assim que uma variante vencer, capture a resposta — qual variante e por quê — depois capture o protótipo da forma como a [SKILL](SKILL.md) descreve. Incorpore a vencedora ao código real e mova o resto para o branch descartável, não para a main:

- **Subformato A** — incorpore a vencedora à página existente; remova da main as variantes perdedoras e o switcher.
- **Subformato B** — promova a variante vencedora a uma rota real; remova da main a rota descartável e o switcher.

O conjunto completo de variantes é a fonte primária, então ele vai para o branch descartável, não para o lixo — componentes de variante e o switcher deixados na branch main apodrecem rápido e confundem o próximo leitor.

## Antipadrões

- **Variantes que diferem só em cor ou texto (copy).** Isso é um ajuste, não um protótipo. Variantes de verdade discordam sobre estrutura.
- **Compartilhar código demais entre as variantes.** Um `<Header>` compartilhado é aceitável; um `<Layout>` compartilhado anula o propósito. Cada variante deveria ser livre para descartar o layout.
- **Conectar variantes a mutações reais.** Protótipos somente leitura são aceitáveis. Se uma variante precisa mutar dados, aponte-a para um stub — a pergunta é "como isso deveria se parecer", não "o backend funciona".
- **Promover o protótipo direto para produção.** O código da variante foi escrito sob as restrições de um protótipo (sem testes, tratamento de erro mínimo). Reescreva-o adequadamente ao incorporá-lo.
