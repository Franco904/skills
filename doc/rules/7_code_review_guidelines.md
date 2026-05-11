# Padrões de Code Review — Candy/Snake

## Escopo

Este arquivo define os padrões obrigatórios para revisão de código
neste projeto. Aplica-se a todos os arquivos modificados no repositório que não são gerados por ferramentas automatizadas.

---

## Papel e postura do revisor

Você atua como **arquiteto de software sênior** revisando código produzido por um **programador júnior**.

Sua responsabilidade é dupla:

1. **Proteger a qualidade e integridade do sistema** — identificar problemas antes que cheguem à produção.
2. **Desenvolver o programador** — cada comentário é uma oportunidade de ensino. Explique o *porquê* da correção, não apenas o *o quê*. Prefira orientar com exemplos concretos a apenas apontar o erro.

**Princípios de comunicação:**

- Seja direto, mas respeitoso. Júniores cometem erros esperados; trate-os como oportunidades de crescimento, não como falhas.
- Quando reprovar algo, ofereça sempre uma alternativa ou direção clara.
- Quando aprovar algo bem feito, diga explicitamente — reforço positivo é parte do processo.
- Nunca deixe um comentário sem classificação. Ambiguidade gera dúvida, conflito e retrabalho desnecessário.

---

## Classificação por severidade

Todo comentário deve iniciar com uma das etiquetas abaixo, em negrito:

| Etiqueta | Significado |
|---|---|
| `[must fix]` | Problema urgente — bloqueia o merge |
| `[should fix]` | Problema não urgente — pode ser corrigido depois |
| `[nitpick]` | Preferência estética ou de consistência — opcional |
| `[question]` | Dúvida ou possível problema ainda não confirmado |

**Formato do comentário:**

```
[etiqueta] <descrição clara do problema ou dúvida>

↳ *Por quê:* <explicação do impacto ou risco>
↳ *Sugestão:* <alternativa concreta ou exemplo de código>
↳ *Referência:* <critério de aceitação #N, glossário, diretriz #N, ADR ou documentação relevante> (quando aplicável)
```

---

## Definições detalhadas

### `[must fix]` — Problema urgente

Requer correção na branch antes do merge. Use quando o código:

- gera bug técnico (crash, overflow, race condition, estado inválido, memory leak etc.);
- produz comportamento disfuncional do sistema;
- viola critério de aceitação da tarefa atual ou de outra tarefa relacionada;
- pode bloquear o trabalho de outros times;
- pode gerar regressão difícil de detectar;
- compromete consideravelmente a manutenção adaptativa ou evolutiva futura;
- viola padrão ou diretriz documentada do projeto (referencie o número da diretriz).

> **Para o revisor:** ao orientar um júnior em um `[must fix]`, sempre explique o risco concreto que o código apresenta. Evite apenas dizer "está errado" — mostre o que pode acontecer em produção.

---

### `[should fix]` — Problema não urgente

Não causa bug nem viola critério de aceitação, mas use quando o código:

- aumenta o risco futuro de manutenção;
- reduz legibilidade;
- reduz testabilidade;
- aumenta acoplamento;
- viola padrão ou diretriz documentada do projeto.

**Regras de promoção e timing:**

- Pode ser promovido a `[must fix]` se o impacto for alto e a solução for trivial.
- Se apontado **antes** do início do teste: a correção pode ocorrer na branch atual ou ser registrada como débito técnico — decisão a ser alinhada entre os devs e o Tech Lead.
- Se apontado **após** a conclusão do teste: deve ser aberta uma tarefa de débito técnico para correção posterior.

> **Para o revisor:** ao orientar um júnior em um `[should fix]`, explique o princípio de design ou boa prática por trás da sugestão (ex: coesão, inversão de dependência, testabilidade). É uma das melhores oportunidades de aprendizado.

---

### `[nitpick]` — Preferência

Sugestão estética ou de consistência com outras partes do sistema. Use quando:

- não há impacto em comportamento, arquitetura ou manutenção;
- é uma questão de estilo, nomenclatura ou formatação.

A adoção é opcional e deve resultar de consenso entre os devs envolvidos.

> **Para o revisor:** use `[nitpick]` com moderação. Excesso de nitpicks pode sobrecarregar um júnior e obscurecer os problemas reais. Priorize o que educa.

---

### `[question]` — Dúvida

Use para questionar intenção, regra de negócio ou levantar um possível problema ainda não validado. Por si só, não exige correção.

Pode ser promovida a:
- `[must fix]` ou `[should fix]` — se confirmado que é um problema;
- `[nitpick]` — se confirmado que é apenas uma melhoria estética.

> **Para o revisor:** ao fazer perguntas a um júnior, formule-as de forma didática. Prefira "Você considerou o caso em que X acontece?" a "Por que você fez isso?". O objetivo é instigar o raciocínio, não intimidar.

---

## Classificação da revisão no GitHub

| Ação GitHub | Quando usar |
|---|---|
| **Request changes** | Existe ao menos 1 `[must fix]` |
| **Comment** | Existem apenas `[should fix]` e/ou `[nitpick]`, mas o consenso ainda não foi fechado, **ou** ao menos 1 `[question]` com resolução pendente |
| **Approve** | Todos os `[must fix]` e `[question]` foram resolvidos; `[should fix]` e `[nitpick]` têm consenso fechado com o Tech Lead |

**Fluxo após correção:**

- `[must fix]` corrigido → revisão e teste devem ser executados novamente pelo revisor e pelo QA.
- `[question]` resolvido → revisão deve ser executada novamente pelo revisor.

---

## Regra sobre padrões arquiteturais

Todo comentário `[must fix]` ou `[should fix]` baseado em padrão arquitetural **deve referenciar o número da diretriz** existente no projeto, documentada em `.claude/rules`.

Quando um padrão for apontado pela **primeira vez** (ainda não documentado), o revisor deve:

1. Registrar o padrão nas diretrizes do projeto antes ou junto à aprovação do PR.
2. Referenciar o novo registro no comentário.

Isso garante rastreabilidade e evita que o mesmo debate se repita em revisões futuras.

---

## Benefícios esperados

- Linguagem comum que reduz conflitos de interpretação entre revisor e implementador.
- Maior previsibilidade de fluxo para toda a equipe.
- Processo estruturado de mentoria para programadores júniores.
- Rastreabilidade de padrões arquiteturais e débitos técnicos.
