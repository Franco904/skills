# Protótipo de Lógica

Um único arquivo HTML autocontido — uma **demo compartilhável (shareable demo)** — que permite a qualquer pessoa operar um modelo de estado clicando em botões. Use isto quando a pergunta for sobre **lógica de negócio, transições de estado, ou formato de dados** — o tipo de coisa que parece razoável no papel, mas só se revela errada quando você a leva a casos reais.

Por ser um único arquivo sem nada para instalar, você pode entregá-lo a alguém que não seja desenvolvedor — um designer, um PO, um especialista de domínio — e deixar que sinta o modelo por conta própria. Por isso ele fala a língua deles, não a do código.

## Quando este é o formato certo

- "Não tenho certeza se essa máquina de estados lida com o caso extremo em que X e depois Y."
- "Esse modelo de dados realmente me deixa representar o caso em que...?"
- "Quero sentir como a API deveria se parecer antes de escrevê-la."
- Qualquer situação em que alguém queira **apertar botões e observar o estado mudar**.

Se a pergunta for "como isso deveria se parecer" — ramificação errada. Use [UI.md](UI.md).

## Processo

### 1. Declare a pergunta

Antes de escrever código, anote qual modelo de estado e qual pergunta você está prototipando. Um parágrafo, no topo da demo (em uma introdução visível, não apenas um comentário). Um protótipo de lógica que responde à pergunta errada é puro desperdício — deixe a pergunta explícita para que possa ser conferida depois, esteja o usuário acompanhando agora ou voltando a ela mais tarde, offline (AFK).

### 2. Isole a lógica em um módulo portátil

Coloque a lógica de fato — a parte que responde à pergunta — em um único bloco `<script>`, escrito como um módulo pequeno e puro que poderia ser retirado dali e colocado no codebase real depois. A página ao redor é descartável; esse módulo não é.

O formato certo depende da pergunta:

- **Um reducer puro** — `(state, action) => state`. Bom quando as ações são eventos discretos e o estado é um único valor.
- **Uma máquina de estados** — estados e transições explícitos. Boa quando "quais ações são sequer legais agora" faz parte da pergunta.
- **Um pequeno conjunto de funções puras** sobre um tipo de dado simples. Bom quando não há um estado atual implícito — apenas transformações.
- **Uma classe ou módulo com uma superfície de métodos clara** quando a lógica de fato possui um estado interno contínuo.

Escolha o formato que melhor se encaixa na pergunta sendo feita, *não* o que for mais fácil de conectar a uma página. Mantenha-o puro: sem DOM, sem `document`, sem handlers de botão alcançando o interior dele. A página chama o módulo; nada flui na direção contrária. É isso que torna o protótipo útil além do seu próprio ciclo de vida: uma vez respondida a pergunta, o reducer / máquina / conjunto de funções validado se transfere sozinho para o módulo real.

### 3. Construa o arquivo HTML compartilhável

Um arquivo, HTML/CSS/JS puro — sem framework, sem bundler, sem servidor, tudo inline para que abra no navegador com duplo clique e sobreviva a ser enviado por e-mail. Qualquer pessoa deveria conseguir rodá-lo apenas abrindo-o.

Escreva-o para alguém que não é desenvolvedor. Todo rótulo está na **linguagem do domínio**, não no código — botões e estado devem soar como o negócio, não como o reducer. Explique em palavras simples o que está acontecendo.

Organize-o com uma hierarquia limpa, de cima para baixo:

1. **Título e explicação de uma linha** sobre o que essa demo permite explorar (a pergunta do passo 1).
2. **Estado atual** — todo o estado relevante, renderizado como um painel legível (campos rotulados, não um despejo bruto de JSON), re-renderizado após cada clique para que a mudança fique visível. Onde ajudar quem não é desenvolvedor a acompanhar, destaque o que acabou de mudar.
3. **Botões de livre exploração (free-play)** — um botão por ação, sempre disponível, para que qualquer um possa mexer no modelo em qualquer ordem. Cada clique dispara sua ação e re-renderiza o estado.
4. **Roteiros guiados (guided walkthroughs)** — um conjunto de **cenários**, um por aba. Cada aba traz uma breve descrição em linguagem simples do cenário — a situação que ele monta e o que observar — e, abaixo dela, os **botões a apertar**, em ordem, para aquele cenário. Cada passo é um botão de verdade: clicar nele executa aquela ação e avança para o próximo passo. Iniciar um roteiro reseta para um estado inicial conhecido, para que o cenário rode da mesma forma todas as vezes.

Escolha cenários que demonstrem os casos incômodos — o caminho feliz (happy path), um caso extremo complicado, uma tentativa de algo que deveria ser ilegal — os difíceis de raciocinar no papel.

Mantenha-o bonito mas contido: tipografia limpa, espaçamento generoso, uma única cor de destaque. Sem animações, sem truques — nada que compita com o estado e os botões.

### 4. Entregue

Envie o arquivo para a pessoa, ou abra-o para ela. Ela vai clicar pelos roteiros e explorar livremente quando tiver oportunidade; os momentos interessantes são quando ela diz "espera, isso não deveria ser possível" ou "hã, eu assumi que X seria diferente" — esses são os bugs na _ideia_, que é o objetivo de tudo isso. Se ela quiser novas ações ou um novo cenário, adicione-os. Protótipos evoluem.

### 5. Capture a resposta e o protótipo

Assim que o protótipo tiver respondido à sua pergunta, capture a resposta, depois capture o protótipo da forma como a [SKILL](SKILL.md) descreve. O mapeamento específico de lógica: o reducer / máquina / conjunto de funções validado se transfere para o módulo real (a decisão, absorvida); o shell HTML segue para o branch descartável que mantém o protótipo como fonte primária — e, por ser um único arquivo autocontido, permanece trivialmente re-executável lá.

## Antipadrões

- **Não adicione testes.** Um protótipo que precisa de testes deixou de ser um protótipo.
- **Não conecte a um banco de dados real.** Use estado em memória, a menos que a pergunta seja especificamente sobre persistência.
- **Não generalize.** Nada de "e se quiséssemos suportar X mais tarde." O protótipo responde a uma pergunta.
- **Não misture a lógica com a página.** Se o módulo puro referencia o DOM, `document`, ou handlers de botão, ele deixa de ser transferível. Mantenha a página como um shell fino sobre um módulo puro.
- **Não recorra a um framework, bundler, ou servidor.** Um arquivo que o destinatário abre com duplo clique; um app React ou um servidor de dev anula o "compartilhável".
- **Não leve o shell HTML para produção.** A página é otimizada para ser clicada manualmente. O módulo de lógica por trás dela é a parte que vale a pena manter.
