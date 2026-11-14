---
name: tdd
description: Desenvolvimento orientado a testes com o ciclo red-green-refactor. Use quando o usuário quiser construir funcionalidades ou corrigir bugs usando TDD, mencionar "red-green-refactor", quiser testes de unidade e/ou integração ou pedir desenvolvimento test-first.
---

# Desenvolvimento Orientado a Testes (TDD)

Filosofia e anti-padrões: ver [REFERENCE.md](REFERENCE.md).

Diretrizes de testes, lidas uma vez no início do primeiro RED: em `docs/guidelines/`, as de slug `all-tests` (`NNNN-all-tests.md`) e as do tipo de teste a escrever — `unit-tests`, `integration-tests` ou `presentation-tests`. Localize por glob (`*-<slug>.md`), nunca pelo número; diretriz ausente é ignorada em silêncio.

Fluxo: planejamento interativo (§1) → execução AFK (§2–§6). Todo o trabalho roda no contexto principal, que carrega glossário, ADRs e diretrizes e projeta a interface pública com esse contexto inteiro. O único subagente é a revisão independente (§3).

## Processo

### 1. Planejamento (interativo)

Ao explorar a base de código, use o glossário de domínio do projeto em @GLOSSARY.md para que os nomes dos testes e o vocabulário da interface correspondam à linguagem do projeto, e respeite os ADRs – em `docs/adr` – na área que você está tocando.

Antes de escrever qualquer código:

- [ ] Confirme com o usuário quais mudanças de interface são necessárias
- [ ] Confirme com o usuário quais comportamentos testar (priorizar)
- [ ] Projete interfaces (diretriz `interface-design` em `docs/guidelines/`, se existir) visando legibilidade, manutenibilidade e testabilidade — para interfaces não-triviais (novo ponto de injeção, dependência cross-seam, módulo novo), aplique **Design It Twice**, e escreva o comentário de interface (diretriz `comments`, seção "Comentário primeiro", se existir) antes do primeiro teste
- [ ] Liste os comportamentos a testar (não etapas de implementação)
- [ ] Obtenha aprovação do usuário sobre o plano

Se julgar necessário, pergunte: "Como deve ser a interface pública? Quais comportamentos são mais importantes de testar?"

**Plano:**

- Faça do plano algo extremamente conciso. Priorize concisão em detrimento de gramática.
- Ao final, tenha um olhar crítico. Procure por eventuais inconsistências no plano e identifique complexidades acidentais, sinalizando-as ao usuário para tomada de decisão.
- Também sinalize quaisquer questões não respondidas para o usuário responder, se houver.

**Você não pode testar tudo.** Confirme com o usuário exatamente quais comportamentos são mais importantes. Concentre o esforço de teste em caminhos críticos e lógica complexa, não em todos os casos extremos possíveis.

**Aprovado o plano, o fluxo é AFK:** não consulte o usuário até a mensagem final (§6). Decisões em aberto viram pendências naquela mensagem.

### 2. Tracer Bullet — RED → GREEN

De "The Pragmatic Programmer": escreva UM teste que confirme UMA coisa sobre o sistema. Este é o seu tracer bullet: fino e demonstrável — prova que o caminho funciona de ponta a ponta.

Para cada comportamento da lista, uma fatia vertical por vez:

**RED** — escreva o teste. Se a interface pública ainda não existir, crie o contrato mínimo para compilar, **sem lógica comportamental**. Rode o teste e confirme: (1) compila, (2) falha — não por erro de sintaxe, mas porque o comportamento ainda não existe. Registre a mensagem de falha exata.

**GREEN** — escreva o código mínimo para o teste passar, nada além. Rode o teste e confirme que passa.

**Nunca refatore enquanto estiver no RED.** Chegue ao GREEN primeiro.

**Regras de parada:**
- RED que não falha pela razão certa após 2 tentativas, ou GREEN que não passa após 3: pare esse comportamento, reverta edições parciais até um estado que compila e siga para os comportamentos independentes. Os dependentes dele são pulados.
- Todo comportamento parado ou pulado entra nas pendências (§6).

### 3. Revisão independente — subagente

Com todos os comportamentos em GREEN, despache **um** subagente que carrega `/review-code` **sem fixed point** (trabalho não commitado). Instrua-o a:
- revisar o diff completo, produção **e** testes, a partir da raiz do projeto (caminho absoluto, informado no prompt; comandos git com `git -C <raiz>`);
- apenas ler: nenhuma edição de arquivo;
- devolver os comentários ao agente principal no formato do `review-code`.

O contexto novo remove o viés de quem escreveu o código. Mantenha o prompt curto: o subagente descobre diretrizes e glossário por conta própria.

### 4. Refatorar

O agente principal aplica a refatoração, guiado pelos comentários da §3 e por estes candidatos:
  - **Duplicação** → Extrair função/classe
  - **Métodos longos** → Dividir em auxiliares privados (manter testes na interface pública)
  - **Módulos rasos** → Combinar ou aprofundar
  - **Obsessão por primitivos** → Introduzir objetos de valor
  - **Nome vago, sinônimo ou ambíguo** → Renomear (diretriz `naming`, se existir)
  - **Código existente** que o novo código revela como problemático; aplique princípios SOLID onde fizer sentido

**Triagem dos comentários:**

| Etiqueta       | Ação                                                                                                                                                              |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `[must fix]`   | Aplique                                                                                                                                                           |
| `[should fix]` | Aplique, se dentro do diff do ciclo; fora do diff vira pendência                                                                                                  |
| `[nitpick]`    | Aplique se trivial e sem risco (renome, KDoc); o restante vira pendência                                                                                          |
| `[question]`   | Resolva consultando código, `GLOSSARY.md` e ADRs e trate o resultado como `[should fix]`; sem resposta, não aplique e registre a suposição mantida como pendência |

Comentário que contradiz a lista de comportamentos aprovada na §1 não é aplicado e vira pendência.

Os testes não são alterados nesta etapa. Execute os testes relevantes após cada etapa de refatoração; ao fim, todos passando.

### 5. Teste de mutação manual

Sobre o código **já refatorado**, restrito aos trechos tocados, prove que os testes não são vazios — mutação por mutação, sempre revertendo antes da próxima:
1. Faça uma mutação plausível no código de produção (inverter condição, trocar `>=`/`>`, remover um branch/chamada, trocar um valor por outro válido).
2. Rode o teste; ele deve falhar (mutante morto). Se passar (mutante sobrevivente), o teste está fraco — reforce a asserção. No máximo 2 reforços por mutante; se ainda sobreviver, registre nas pendências e siga.
3. Reverta a mutação e confirme que o teste volta a passar.
4. Repita para os mutantes plausíveis do trecho (condições de borda, operadores de comparação, negações, valores default).

Considerações:
- Aplique e reverta cada mutação com a ferramenta de edição de arquivos; mutante que não compila é substituído por outro compilável.
- O reforço de asserção não volta à revisão da §3. Ver [REFERENCE.md](REFERENCE.md) para exemplos.

### 6. Mensagem final

Resuma o que foi feito e termine com a seção **Pendências para você**, com estes itens (omita os vazios):
- `[question]` sem resposta, com a suposição mantida
- comentários da revisão que contradizem a §1
- `[should fix]` fora do diff e `[nitpick]` não aplicados
- comportamentos parados ou pulados, com o motivo
- mutantes sobreviventes após reforços

Não faça commit: peça permissão ao usuário.

---

## Checklist Por Ciclo

```
[ ] Teste descreve comportamento na linguagem ubíqua, não implementação
[ ] Teste usa apenas interface pública
[ ] Teste sobreviveria a uma refatoração interna
[ ] Código é mínimo para este teste
[ ] Nenhuma funcionalidade especulativa adicionada
[ ] Comentários da revisão triados (§4) e testes verdes após a refatoração
[ ] Teste mata os mutantes plausíveis do trecho refatorado (§5)
[ ] Diff passa no Teste da exclusão da diretriz `comments` (interface e implementação)
```
