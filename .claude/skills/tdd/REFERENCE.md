# Referência TDD

## Filosofia

**Princípio central**: Testes devem verificar comportamento através de interfaces públicas, não detalhes de implementação. O código pode mudar completamente; os testes não deveriam.

**Bons testes** são no estilo de integração: exercitam caminhos reais de código através de APIs públicas. Descrevem _o que_ o sistema faz, não _como_ faz. Um bom teste lê como uma especificação — "usuário pode finalizar compra com carrinho válido" diz exatamente qual capacidade existe. Esses testes sobrevivem a refatorações porque não se importam com a estrutura interna.

**Maus testes** estão acoplados à implementação. Eles mockam colaboradores internos, testam métodos privados ou verificam por meios externos (como consultar o banco diretamente em vez de usar a interface). O sinal de alerta: o teste quebra quando você refatora, mas o comportamento não mudou. Se você renomear uma função interna e os testes falharem, esses testes estavam testando implementação, não comportamento.

---

## Anti-Padrão: Fatias Horizontais

**NÃO escreva todos os testes primeiro e depois toda a implementação.** Isso é "fatiamento horizontal" — tratar o RED como "escrever todos os testes" e o GREEN como "escrever todo o código."

Isso produz **testes ruins**:

- Testes escritos em massa testam comportamento _imaginado_, não _real_
- Você acaba testando a _forma_ das coisas (estruturas de dados, assinaturas de funções) em vez do comportamento voltado ao usuário
- Os testes ficam insensíveis a mudanças reais — passam quando o comportamento quebra, falham quando está tudo bem
- Você vai além do que enxerga, comprometendo-se com estrutura de testes antes de entender a implementação

**Abordagem correta**: Fatias verticais via tracer bullets. Um teste → uma implementação → repita. Cada teste responde ao que você aprendeu do ciclo anterior. Como você acabou de escrever o código, sabe exatamente qual comportamento importa e como verificá-lo.

```
ERRADO (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

CERTO (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

---

## Teste de Mutação Manual em GREEN

Um teste verde não prova que o teste verifica algo — só prova que ele não quebrou com a implementação atual. O teste de mutação manual fecha essa lacuna: estraga a implementação de propósito e confirma que o teste percebe.

**Como mutar:**
- Trocar operador de comparação (`>=` → `>`, `>` → `>=`, `==` → `!=`)
- Inverter uma condição booleana (`if (!x)` → `if (x)`)
- Remover uma chamada/linha (um `add`, um `filter`, uma validação)
- Trocar um valor por outro plausível (um default, um limite, uma constante)

**Ciclo por mutante:** mutar → rodar teste → confirmar falha (mutante morto) → reverter → confirmar volta ao verde. Nunca deixe uma mutação no código entre um teste e outro — se o agente for interrompido no meio, o código de produção fica quebrado sem ninguém perceber.

**Mutante sobrevivente (teste passa mesmo mutado) é sinal de teste fraco**, não de "mutação irrelevante" — antes de descartar, pergunte: essa mutação muda comportamento observável pela interface pública? Se sim, o teste precisa de uma asserção mais específica e precisa (ex: trocar `expect(x > -1, isTrue)` por `expect(indiceEsperado, x)`).
