# Design de interfaces

De "A Philosophy of Software Design" (Ousterhout):

Projete **módulos profundos**: encapsule bastante comportamento por trás de uma interface pequena e posicionada em um ponto de injeção claro. O comportamento complexo deve ser testável através dessa interface. Use os princípios descritos sempre que o código estiver sendo projetado ou reestruturado. O objetivo é elevar: capacidade por unidade para os chamadores, concentração de mudanças para os mantenedores e testabilidade para ambos.

> Não confundir "interface" com a palavra-chave `interface` de uma linguagem específica: muito restrito — interface aqui é uma superfície de contato que inclui todos os fatos que um chamador precisa saber para usar o módulo, sem precisar olhar a implementação: assinatura dos métodos públicos, efeitos colaterais, exceções possíveis e invariantes (ex: "esse método é idempotente", "essa chamada é thread-safe").

## Deep Modules x Shallow Modules

**Módulo profundo** = interface pequena + muita implementação

```
┌─────────────────────┐
│  Interface Pequena  │  ← Poucos métodos, parâmetros simples
├─────────────────────┤
│                     │
│                     │
│ Implementação       │  ← Lógica complexa escondida
│ Profunda            │
│                     │
└─────────────────────┘
```

**Módulo raso** = interface grande + pouca implementação (evitar)

```
┌─────────────────────────────────┐
│       Interface Grande          │  ← Muitos métodos, parâmetros complexos
├─────────────────────────────────┤
│  Implementação Fina             │  ← Apenas repassa
└─────────────────────────────────┘
```

Ao projetar interfaces, pergunte:

- Posso reduzir o número de métodos?
- Posso simplificar os parâmetros?
- Posso esconder mais complexidade internamente?

## Princípios

- **Profundidade é uma propriedade da interface, não da implementação**: Um módulo profundo pode ser internamente composto de partes pequenas (métodos privados, classes helpers, etc.) e dependências mockáveis e intercambiáveis — elas simplesmente não fazem parte da interface e não a tornam grande. O que importa é o que o chamador enxerga.
- **Teste da exclusão**: Imagine excluir o módulo. Se a complexidade desaparecer, era um pass-through: só repassava a chamada adiante sem esconder nada de fato e agregar valor – ex: use case raso entre apresentação e acesso a dados. Se a complexidade reaparece em N chamadores, ele estava sendo útil e deve ser preservado.
- **A interface é a superfície para testes**: Chamadores em código de produção e testes usam o mesmo ponto de injeção. Se você precisar testar *além* da interface (usar métodos privados, reflection), o módulo provavelmente está mal projetado.
- **YAGNI principle**: Não introduza uma abstração a menos que algo realmente varie através dela pelo ponto de injeção. Isso adiciona custo de indireção desnecessário. Ao invés disso, espere até ter o segundo caso de uso real antes de generalizar — não generalize preventivamente "porque pode precisar no futuro".

## Design de Interface para Testabilidade

Boas interfaces tornam os testes naturais:

1. **Aceite dependências, não as crie**

   ```kotlin
   // Testável
   fun processOrder(order: Order, paymentGateway: PaymentGateway) {}

   // Difícil de testar
   fun processOrder(order: Order) {
       val gateway = StripeGateway()
   }
   ```

2. **Retorne resultados, não produza efeitos colaterais**

   ```kotlin
   // Testável
   fun calculateDiscount(cart: Cart): Discount {}

   // Difícil de testar
   fun applyDiscount(cart: Cart) {
       cart.total -= discount
   }
   ```

3. **Superfície pequena**
   - Menos métodos = menos testes necessários
   - Menos parâmetros = configuração de teste mais simples
   - Busque o equilíbrio com base no valor de negócio – prefira várias interfaces específicas em vez de uma única interface genérica ("gorda")

4. **Específica por responsabilidade**
   - Prefira interfaces pequenas e específicas para que implementadores não sejam obrigados a satisfazer contratos que não fazem sentido para eles
   - Evita interfaces "gordas" – mocks e stubs não precisam implementar métodos irrelevantes só para satisfazer o contrato (complexidade acidental)

      ```kotlin
      // BOM: Cada interface é específica
      interface Authenticable {
          fun clockIn()
      }

      interface Commissionable {
          fun calculateCommission()
      }

      // RUIM: Nem todo funcionário ganha comissão
      interface Employee {
          fun clockIn()
          fun calculateCommission()
      }
      ```

## Design It Twice

Quando o design for não-trivial, use este padrão de subagentes paralelos com o objetivo de explorar interfaces alternativas para um dado módulo candidato. Baseado no princípio "Design It Twice" (Ousterhout) — sua primeira ideia dificilmente é a melhor.

Processo:

### 1. Defina o escopo do problema

Antes de spawnar subagentes, escreva uma explicação voltada ao usuário do escopo do problema para o módulo candidato escolhido:

- As restrições que qualquer nova interface precisaria satisfazer
- As dependências das quais ela dependeria
- Um código sketch ilustrativo e aproximado para fundamentar restrições — não é uma sugestão, é só uma forma de dar concretude às restrições

Mostre isso ao usuário, e em seguida prossiga imediatamente para o passo 2. O usuário lê e pensa enquanto os subagentes trabalham em paralelo.

### 2. Spawne os subagentes

Spawne 3+ subagentes em paralelo. Cada um deve produzir uma interface **radicalmente diferente** para o módulo profundo.

Faça o prompt de cada subagente com um resumo técnico separado (caminhos de arquivo, detalhes de acoplamento, natureza da dependência – local substituível x remoto externo, o que fica por trás do ponto de injeção). O resumo é independente da explicação voltada ao usuário do passo 1. Dê a cada agente uma restrição de design distinta:

- Agente 1: "Minimize a interface — mire em 1–3 entry points no máximo. Maximize a capacidade/alavanca por entry point."
- Agente 2: "Maximize a flexibilidade — suporte muitos casos de uso e extensões."
- Agente 3: "Otimize para o chamador mais comum — torne o caso default trivial."
- Agente 4 (se aplicável): "Projete favorecendo inversão de dependências para dependências cross-seam."

Inclua o vocabulário do @CONTEXT.md no resumo, para que cada subagente nomeie as coisas de forma consistente com a linguagem ubíqua do projeto.

Cada subagente produz como saída:

1. Interface (tipos, métodos, parâmetros — além de caminhos de exceções e invariantes)
2. Exemplo de uso mostrando como os chamadores a usam
3. O que a implementação esconde por trás do ponto de injeção
4. Natureza da dependência e adapters (local substituível x remoto externo)
5. Trade-offs — onde a capacidade/alavanca é alta, onde é baixa

### 3. Apresente e compare

Apresente os designs sequencialmente, para que o usuário consiga absorver cada um, e depois compare-os em prosa. Contraste por **profundidade** (capacidade da interface), **localidade** (onde as mudanças se concentram) e **facilidade de uso** pelo chamador.

Depois de comparar, forneça sua opção recomendada: qual design você acha mais forte e por quê. Se elementos de designs diferentes combinarem bem, proponha um híbrido. Seja opinativo — o usuário quer uma leitura forte, não um menu.
