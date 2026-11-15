# Formato do Registro de Aprendizado

Registros de aprendizado ficam em `./learning-records/` e usam numeração sequencial: `0001-slug.md`, `0002-slug.md`, etc. Crie o diretório de forma preguiçosa (lazy) — apenas quando o primeiro registro for escrito.

São o equivalente, no ensino, aos ADRs: capturam lições não óbvias, insights-chave e conhecimento prévio declarado que vão direcionar sessões futuras. São usados para calcular a zona de desenvolvimento proximal.

## Modelo

```md
# {Título curto do que foi aprendido ou estabelecido}

{1-3 frases: o que foi aprendido (ou qual conhecimento prévio foi estabelecido), e por que isso importa para sessões futuras.}
```

Esse é o formato completo. Um registro de aprendizado pode ser um único parágrafo. O valor está em registrar _que_ isso agora é conhecido e _por que_ isso muda o que ensinar a seguir — não em preencher seções.

## Seções opcionais

Inclua-as apenas quando agregarem valor genuíno. A maioria dos registros não vai precisar delas.

- **Status** no frontmatter (`active | superseded by LR-NNNN`) — útil quando um entendimento anterior se mostra errado e é substituído.
- **Evidência** — como o usuário demonstrou o entendimento (uma pergunta respondida, um exercício concluído, experiência prévia citada). Útil quando a afirmação puder ser revisitada.
- **Implicações** — o que isso desbloqueia ou descarta para sessões futuras. Vale a pena registrar quando não for óbvio.

## Numeração

Verifique `./learning-records/` em busca do maior número existente e incremente em um.

## Quando escrever um registro de aprendizado

Escreva um quando qualquer uma destas condições for verdadeira:

1. **O usuário demonstrou entendimento genuíno de algo não trivial** — não apenas exposição, mas evidência de que consegue usar o conceito corretamente. Isso estabelece um novo piso para o que ensinar a seguir.
2. **O usuário revelou conhecimento prévio** — "eu já sei X". Registre isso para que sessões futuras não reensinem. Registre também a _profundidade_ alegada.
3. **Um equívoco foi corrigido** — o usuário acreditava anteriormente em algo errado e agora entende por quê. São de alto valor: preveem futuros pontos de tropeço em tópicos relacionados.
4. **A missão mudou em resposta ao aprendizado** — o usuário descobriu que se importava com algo diferente do que pensava. Faça link cruzado com [[MISSION.md]] e atualize-o.

### O que _não_ qualifica

- Material que foi apenas abordado. Cobertura não é aprendizado. Espere pela evidência.
- Qualquer coisa já capturada de forma concisa em [[GLOSSARY.md]] como definição de termo. Não duplique.
- Registros de atividade sessão a sessão. Registros de aprendizado não são um diário — são insights de qualidade decisória.

## Substituição (Supersession)

Quando um registro posterior contradiz um anterior (o entendimento do usuário se aprofundou ou foi corrigido), marque o registro antigo como `Status: superseded by LR-NNNN` em vez de excluí-lo. O histórico de como o entendimento evoluiu é, em si, um sinal útil.
