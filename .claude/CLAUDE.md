# Skills para produtividade — Guia do Agente de IA

Ver [README.md](README.md) para visão geral deste repo, resumo do catalógo, listas de classificação (HITL x AFK) e fluxograma mermaid. Este arquivo cobre apenas o que é específico de como um agente de IA deve trabalhar neste repositório.

## Commits

- **Idioma:** sempre em português.
- **Coautoria proibida:** nunca incluir `Co-Authored-By` (nem qualquer
  atribuição de autoria a IA) nas mensagens de commit.
- **Tipos de commit:** prefixe a mensagem com o tipo, ex: `fix: altura errada na dialog de ajuda em 720dp width`.
  - `feat`: nova funcionalidade ou melhoria em existente
  - `fix`: correção de bug
  - `refactor`: mesma funcionalidade, apenas melhoria de código/manutenção
  - `build`: afeta arquivos de build
  - `chore`: afeta arquivos auxiliares secundários (ex: lint, scripts)
  - `test`: testes automatizados de qualquer tipo/nível
  - `doc`: documentação markdown/html
- **Descrição:** extremamente concisa. Priorize concisão em detrimento de gramática.
- **Atomicidade:** cada commit deve conter uma única mudança logicamente coesa.
