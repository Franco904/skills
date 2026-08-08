---
name: prototype
description: Construir um protótipo descartável para responder a uma pergunta de design. Use quando o usuário quiser verificar se um modelo de estado ou lógica faz sentido, ou explorar como uma UI deveria se parecer.
---

# Protótipo

Um protótipo é **código descartável que responde a uma pergunta**. A pergunta decide o formato.

## Escolha uma ramificação

Identifique qual pergunta está sendo respondida — a partir do prompt do usuário, do código ao redor, ou perguntando diretamente, se o usuário estiver por perto:

- **"Essa lógica / esse modelo de estado faz sentido?"** → [LOGIC.md](LOGIC.md). Construa um único arquivo HTML compartilhável — botões de livre exploração mais roteiros guiados em abas — que leve a máquina de estados por casos difíceis de raciocinar no papel, e que alguém que não seja desenvolvedor consiga operar.
- **"Como isso deveria se parecer?"** → [UI.md](UI.md). Gere várias variações de UI radicalmente diferentes em uma única rota, alternáveis por um parâmetro de busca (search param) na URL e uma barra flutuante na parte inferior.

As duas ramificações produzem artefatos muito diferentes — errar essa escolha desperdiça o protótipo inteiro. Se a pergunta for genuinamente ambígua e o usuário não estiver acessível, use por padrão a ramificação que melhor combina com o código ao redor (um módulo de backend → lógica; uma página ou widget → UI) e declare essa suposição no topo do protótipo.

## Regras que se aplicam a ambas

1. **Descartável desde o primeiro dia, e claramente marcado como tal.** Localize o código do protótipo perto de onde ele será de fato usado (ao lado do módulo ou página que está prototipando) para que o contexto fique óbvio — mas nomeie-o de forma que um leitor casual perceba que é um protótipo, não produção. Para rotas de UI descartáveis, obedeça a convenção de rotas que o projeto já usa; não invente uma nova estrutura de topo.
2. **Trivial de executar.** Um protótipo de UI inicia a partir de um único comando no executor de tarefas do projeto — `pnpm <name>`, `python <path>`, `bun <path>`, etc. Uma demo de lógica é um único arquivo HTML que o usuário abre com duplo clique. De qualquer forma, iniciá-lo não deve exigir nenhum raciocínio.
3. **Sem persistência por padrão.** O estado vive em memória. Persistência é a coisa que o protótipo está _verificando_, não algo de que ele deveria depender. Se a pergunta envolver explicitamente um banco de dados, use um banco descartável (scratch DB) ou um arquivo local com um nome claro do tipo "PROTOTYPE — wipe me".
4. **Pule o polimento.** Sem testes, sem tratamento de erro além do necessário para o protótipo _rodar_, sem abstrações. O objetivo é aprender algo rápido.
5. **Exponha o estado.** Depois de cada ação (lógica) ou a cada troca de variante (UI), imprima ou renderize todo o estado relevante para que o usuário veja o que mudou.
6. **Capture quando terminar.** Incorpore qualquer decisão validada ao código real, depois capture o próprio protótipo como **fonte primária**: faça commit dele em um branch descartável, fora da main, e deixe um ponteiro de contexto (context pointer) para esse branch na issue de implementação. Capture também a resposta — o veredito e a pergunta que ele resolveu — na issue ou em um commit. A branch main mantém apenas a decisão validada.
