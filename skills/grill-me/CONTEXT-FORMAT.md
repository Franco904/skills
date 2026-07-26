# Formato do CONTEXT.md

## Estrutura

```md
# {Nome do Contexto}

{Uma ou duas frases descrevendo o que é este contexto e por que ele existe.}

**Termo**: {Uma ou duas frases descrevendo o termo}
_Evitar_: Sinonimo1, Sinonimo2
_Alias de código_: {Nomenclatura legada usada em código}

**Fatura**: Uma solicitação de pagamento enviada ao cliente após a entrega.
_Evitar_: Conta, PagamentoRequest
_Alias de código_: PedidoCliente

**Cliente**: Uma pessoa ou organização que realiza pedidos.
_Evitar_: Comprador, Conta, Perfil
```

## Regras

- Todos os termos de cada contexto em **ordem alfabética**.
- Nenhum termo abreviado, camelCase, PascalCase, snake_case ou sem espaços/acentos (exceto dentro de alias de código: ...).
- Alias de código informado **apenas se houver divergência legada**.
- **Seja opinativo.** Quando múltiplas palavras existem para o mesmo conceito, escolha a melhor e liste as demais como aliases a evitar.
- **Mantenha as definições enxutas.** No máximo uma ou duas frases. Defina o que o termo É, não o que ele faz.
- **Mostre relacionamentos no termo do lado forte.** Use nomes de termos em negrito e expresse cardinalidade quando não for óbvio.
- **Inclua apenas termos específicos ao contexto deste projeto.** Conceitos gerais de programação (timeouts, tipos de erro, padrões utilitários) não pertencem aqui, mesmo que o projeto os use extensivamente. Antes de adicionar um termo, pergunte-se: este é um conceito único a este contexto, ou um conceito geral de programação? Somente o primeiro entra.
- **Agrupe termos em subtítulos** quando clusters naturais emergirem. Se todos os termos pertencem a uma única área coesa, uma lista plana é suficiente.
- **Escreva um diálogo de exemplo.** Uma conversa entre um dev e um especialista de domínio que demonstre como os termos interagem naturalmente e esclareça os limites entre conceitos relacionados.

## Localização

Um `CONTEXT.md` na raiz do repositório.
