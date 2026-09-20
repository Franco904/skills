# Formato de ADR

Os ADRs ficam em `docs/adr/` e usam numeração sequencial: `0001-slug.md`, `0002-slug.md`, etc.

Crie o diretório `docs/adr/` de forma preguiçosa — somente quando o primeiro ADR for necessário.

## Template

```md
# {Título curto da decisão}

{1 a 3 frases: qual é o contexto, o que decidimos e por quê.}
```

É só isso. Um ADR pode ser um único parágrafo. O valor está em registrar *que* uma decisão foi tomada e *por quê* — não em preencher seções.

## Seções opcionais

Inclua estas seções apenas quando agregarem valor real. A maioria dos ADRs não vai precisar delas.

- **Status** em frontmatter (`proposto | aceito | descontinuado | substituído por ADR-NNNN`) — útil quando decisões são revisitadas
- **Alternativas Consideradas** — apenas quando as alternativas rejeitadas merecem ser lembradas
- **Consequências** — apenas quando efeitos colaterais não óbvios precisam ser destacados

## Numeração

Verifique em `docs/adr/` qual é o maior número existente e incremente em um.

## Quando oferecer um ADR

Os três critérios abaixo devem ser verdadeiros:

1. **Difícil de reverter** — o custo de mudar de ideia mais tarde é significativo
2. **Surpreendente sem contexto** — um leitor futuro vai olhar para o código e se perguntar "por que diabos fizeram assim?"
3. **Resultado de uma troca real** — havia alternativas genuínas e você escolheu uma por razões específicas

Se uma decisão é fácil de reverter, pule — você simplesmente vai revertê-la. Se não é surpreendente, ninguém vai se perguntar o porquê. Se não havia alternativa real, não há nada a registrar além de "fizemos a coisa óbvia."

### O que se qualifica

- **Formato arquitetural.** "Estamos usando um monorepo." "O modelo de escrita é event-sourced, o modelo de leitura é projetado no Postgres."
- **Padrões de integração entre contextos.** "Atendimento e Sync se comunicam via eventos de domínio, não via HTTP síncrono."
- **Escolhas tecnológicas que geram lock-in.** Banco de dados, barramento de mensagens, provedor de autenticação, alvo de deploy. Não toda biblioteca — apenas as que levariam um trimestre para trocar.
- **Decisões de fronteira e escopo.** "Dados do Cliente são de propriedade do contexto Customer; outros contextos o referenciam apenas por ID." Os nãos explícitos são tão valiosos quanto os sins.
- **Desvios deliberados do caminho óbvio.** "Estamos usando SQL manual em vez de um ORM por causa de X." Qualquer coisa em que um leitor razoável assumiria o contrário. Isso impede o próximo engenheiro de "corrigir" algo que foi intencional.
- **Restrições não visíveis no código.** "Não podemos usar AWS por exigências de compliance." "Os tempos de resposta devem ser inferiores a 200ms por causa do contrato com a API parceira."
- **Alternativas rejeitadas quando a rejeição não é óbvia.** Se você considerou GraphQL e escolheu REST por razões sutis, registre — caso contrário, alguém vai sugerir GraphQL de novo em seis meses.
