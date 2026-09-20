# Definições das Etiquetas de Revisão

## `[must fix]` — Problema urgente

Requer correção antes do merge. Use quando o código:

- gera bug técnico (crash, overflow, race condition, estado inválido, memory leak);
- produz comportamento disfuncional do sistema;
- viola critério de aceitação da tarefa atual ou de outra tarefa relacionada;
- pode bloquear o trabalho de outros times;
- pode gerar regressão difícil de detectar;
- compromete consideravelmente a manutenção adaptativa ou evolutiva futura;
- viola padrão ou diretriz documentada do projeto (referencie o número da diretriz).

> Ao orientar um júnior em um `[must fix]`, sempre explique o risco concreto. Evite apenas dizer "está errado" — mostre o que pode acontecer em produção.

---

## `[should fix]` — Problema não urgente

Não causa bug nem viola critério de aceitação, mas use quando o código:

- aumenta o risco futuro de manutenção;
- reduz legibilidade;
- reduz testabilidade;
- aumenta acoplamento;
- viola padrão ou diretriz documentada do projeto.

> Ao orientar um júnior em um `[should fix]`, explique o princípio de design por trás da sugestão (coesão, inversão de dependência, testabilidade). É uma das melhores oportunidades de aprendizado.

---

## `[nitpick]` — Preferência

Sugestão estética ou de consistência. Use quando não há impacto em comportamento, arquitetura ou manutenção. A adoção é opcional e deve resultar de consenso.

> Use `[nitpick]` com moderação. Excesso obscurece os problemas reais.

---

## `[question]` — Dúvida

Use para questionar intenção, regra de negócio ou levantar um possível problema ainda não validado. Por si só, não exige correção.

Pode ser promovida a `[must fix]`, `[should fix]` ou `[nitpick]` após resolução.

> Formule perguntas de forma didática: "Você considerou o caso em que X acontece?" em vez de "Por que você fez isso?".
