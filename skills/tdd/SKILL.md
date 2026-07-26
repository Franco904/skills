---
name: tdd
description: Desenvolvimento orientado a testes com o ciclo red-green-refactor. Use quando o usuário quiser construir funcionalidades ou corrigir bugs usando TDD, mencionar "red-green-refactor", quiser testes de unidade e/ou integração ou pedir desenvolvimento test-first.
---

# Desenvolvimento Orientado a Testes (TDD)

Filosofia e anti-padrões: ver [REFERENCE.md](REFERENCE.md).

Diretrizes de testes a serem seguidas em GREEN presentes em @docs/guidelines/0001-tests.md.

## Processo

### 1. Planejamento (contexto principal)

Ao explorar a base de código, use o glossário de domínio do projeto em @CONTEXT.md para que os nomes dos testes e o vocabulário da interface correspondam à linguagem do projeto, e respeite os ADRs – em `docs/adr` – na área que você está tocando.

Antes de escrever qualquer código:

- [ ] Confirme com o usuário quais mudanças de interface são necessárias
- [ ] Confirme com o usuário quais comportamentos testar (priorizar)
- [ ] [Projete interfaces](../../docs/guidelines/0002-interface-design.md) visando legibilidade, manutenibilidade e testabilidade — para interfaces não-triviais (novo ponto de injeção, dependência cross-seam, módulo novo), aplique **Design It Twice** antes do primeiro teste
- [ ] Liste os comportamentos a testar (não etapas de implementação)
- [ ] Obtenha aprovação do usuário sobre o plano

Se julgar necessário, pergunte: "Como deve ser a interface pública? Quais comportamentos são mais importantes de testar?"

**Você não pode testar tudo.** Confirme com o usuário exatamente quais comportamentos são mais importantes. Concentre o esforço de teste em caminhos críticos e lógica complexa, não em todos os casos extremos possíveis.

### 2. Tracer Bullet — RED e GREEN em subagentes separados

De "The Pragmatic Programmer": escreva UM teste que confirme UMA coisa sobre o sistema. Este é o seu tracer bullet: fino e demonstrável — prova que o caminho funciona de ponta a ponta.

Cada fase é delegada a um subagente dedicado para garantir isolamento de contexto. O agente que escreve o teste não sabe como a implementação ficará; o agente que implementa só vê o teste falhando.

#### Fase RED — subagente dedicado

Passe ao subagente:
- O comportamento a testar (descrição precisa, em linguagem ubíqua)
- Arquivos de contexto relevantes (interfaces, entidades, convenções do projeto)
- Restrição explícita: **escrever apenas o teste e, se a interface pública ainda não existir, criar o contrato mínimo para compilar** — **nenhuma lógica comportamental ainda**
- Comando para rodar o teste e confirmar que ele compila

O subagente termina quando o teste falhar pela razão certa. Rode o teste ao final e confirme: (1) compila, (2) falha — não por erro de sintaxe, mas porque o comportamento ainda não existe. Ele entrega: caminho do arquivo de teste, nome do teste e a mensagem de falha exata.

#### Fase GREEN — subagente dedicado

Passe ao subagente:
- O teste falhando (caminho, nome e mensagem de falha exata do RED)
- Arquivos de contexto relevantes (interfaces existentes, padrões do projeto)
- Restrição explícita: **código mínimo para o teste passar — nada além**
- Comando para rodar o teste e confirmar que ele passa

O subagente termina quando o teste estiver passando. Ele entrega: arquivos alterados.

**Teste de mutação manual (obrigatório antes de considerar o GREEN concluído):** com o teste passando, prove que ele não é vazio — mutação por mutação, sempre revertendo antes da próxima:
1. Faça uma mutação plausível no código de produção tocado (inverter condição, trocar `>=`/`>`, remover um branch/chamada, trocar um valor por outro válido).
2. Rode o teste; ele deve falhar (mutante morto). Se passar (mutante sobrevivente), o teste está fraco — reforce a asserção antes de prosseguir.
3. Reverta a mutação e confirme que o teste volta a passar.
4. Repita para os mutantes plausíveis do trecho (condições de borda, operadores de comparação, negações, valores default).

Ver [REFERENCE.md](REFERENCE.md) para exemplos.

Repita RED → GREEN (subagentes separados) para cada comportamento restante da lista.

### 3. Refatorar — subagente dedicado

Após todos os testes passarem, delegue a refatoração a um subagente com:
- Todos os arquivos alterados durante os ciclos RED/GREEN
- Os testes (como restrição: não alterá-los)
- Após o ciclo de TDD, identifique candidatos à refatoração. Procure por:
  - **Duplicação** → Extrair função/classe
  - **Métodos longos** → Dividir em auxiliares privados (manter testes na interface pública)
  - **Módulos rasos** → Combinar ou aprofundar
  - **Obsessão por primitivos** → Introduzir objetos de valor
  - **Código existente** que o novo código revela como problemático; aplique princípios SOLID onde fizer sentido
- Carregue /review-code para guiar o ciclo de feedback desta etapa

Execute os testes relevantes após cada etapa de refatoração. O subagente termina quando a refatoração estiver concluída e todos os testes passando.

**Nunca refatore enquanto estiver no RED.** Chegue ao GREEN primeiro.

---

## Checklist Por Ciclo

```
[ ] Teste descreve comportamento na linguagem ubíqua, não implementação
[ ] Teste usa apenas interface pública
[ ] Teste sobreviveria a uma refatoração interna
[ ] Código é mínimo para este teste
[ ] Nenhuma funcionalidade especulativa adicionada
[ ] Teste mata os mutantes plausíveis do trecho (teste de mutação manual no GREEN)
```
