---
name: pr
description: Gera o corpo de um PR seguindo o template do projeto (ou o `pull_request_template.md` desta skill), com ênfase em plano de QA ponta-a-ponta derivado do tracer bullet da issue — ou do diff, quando não há issue. Salva como `pr-<branch>.md` na raiz do repositório. Use quando o usuário quiser abrir um PR, gerar o corpo de um PR ou criar o plano de QA de uma entrega.
argument-hint: "Número da issue (opcional) e branch de destino? (ex: 1442 main)"
---

## Invocação

`/pr [issue-number] [target-branch]`

- `[issue-number]`: número da issue GitHub. Se omitido, inferir do nome da branch (ex:
  `issue-1422` → #1422). Se não for possível inferir, perguntar ao usuário; se o usuário disser que não há issue, seguir sem issue (ver abaixo).
- `[target-branch]`: branch de destino do PR. Default: branch padrão do repositório (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`).
- Se o usuário pedir para **editar um PR existente** (ex.: "edita o PR 6097"), buscar o corpo atual com `gh pr view <n> --json body,baseRefName` e usar a base real do PR como `target-branch`.

**Sem issue:** pule `gh issue view`, derive o plano de QA do diff e dos commits, e remova do template as linhas `closes`/`fixes`.

---

## Concisão (obrigatório)

O PR é lido por revisor e QA com pouco tempo. Seja **extremamente conciso** em todas as seções (Descrição, Escopo, Revisão, QA); priorize concisão em detrimento de gramática.

- **Descrição:** causa (se bug) em 1–3 frases + correção em até 5 bullets. Sem listar exaustivamente cada método/arquivo modificado ou removido.
- **Revisão:** só o que o revisor não inferiria do diff (arquivo-chave, mudança de comportamento). Tipicamente 1–5 bullets.
- **QA:** um bullet de uma linha por cenário, agrupando cenários parecidos (ex.: "GPS desligado / permissão negada"). Tipicamente 3–6 bullets. Pré-condição comum vai numa linha antes da lista, não repetida em cada cenário.
- Não justifique ausência de testes, nem detalhe nomes internos, nem repita o que já está no diff.

---

## Processo

### 1. Coletar contexto

```bash
git branch --show-current                        # nome da branch atual
git log <target-branch>...HEAD --oneline         # commits da branch
git diff <target-branch>...HEAD --stat           # arquivos alterados
```

Buscar a issue no GitHub com `gh issue view <issue-number>` (quando houver). Extrair:

- Descrição e história de usuário
- Critérios de Aceitação
- Seção "Decisões de Implementação" (tracer bullet)
- Seção "Decisões de Teste"

### 2. Seção Escopo

Responder às duas perguntas do template comparando os critérios de aceite da issue (sem issue, os commits) com o diff. Só preencher o checklist "Caso a resposta seja sim…" quando alguma resposta for sim; caso contrário, deixar apenas as respostas.

### 3. Seção Orientações para revisão

- Listar decisões de implementação não-óbvias (das "Decisões de Implementação" da issue, quando houver)
- Sinalizar ADRs em `docs/adr/` seguidos ou criados nesta entrega
- Sinalizar desvios em relação aos CAs originais (se houver)
- Apontar arquivos-chave que merecem atenção extra do revisor

Só isso. Se não houver nada a destacar, preencher a seção com `–`.

### 4. Seção Orientações para QA

Se a issue orientar que não haverá teste, ou o diff não afetar o código de produção (ex: só docs/ ou scripts/), preencher a seção com `–` (ou uma linha dizendo que não há impacto para QA) e pular esta etapa.

Caso contrário, derivar o fluxo ponta-a-ponta a partir do tracer bullet da issue (ou, sem issue, do diff). Gerar como lista de bullet points (não usar tabela), um item por cenário, no formato:

- **Cenário**: pré-condição → ação → resultado esperado

Use as categorias abaixo como checklist — inclua só as que o diff realmente afeta; omita (sem mencionar) as que não se aplicam:

| Categoria             | O que testar                                                    |
|-----------------------|-----------------------------------------------------------------|
| Caminho feliz         | Fluxo principal completo conforme tracer bullet                 |
| Estado vazio          | Sem dados pré-existentes ou lista vazia                         |
| Erro de comunicação   | Falha na comunicação com serviço externo (timeout, erro de rede)|
| Dados inválidos       | Formulários com campos ausentes, formatos incorretos, limites   |
| Navegação             | Botão voltar, interrupção e retomada do fluxo                   |
| Sessão expirada       | Usuário não autenticado ou sessão inválida                      |
| Edge cases de dados   | Campos opcionais ausentes, valores limítrofes, strings longas   |
| Impactos laterais     | Outros fluxos afetados pelas mudanças do diff (se aplicável)    |

### 5. Gerar arquivo

Salvar como `pr-<branch-name>.md` na raiz do repositório (trocar `/` por `-` no nome da branch), usando a estrutura exata do template. Fonte do template, em ordem:

1. `.github/pull_request_template.md` do repositório, se existir
2. [pull_request_template.md](pull_request_template.md) desta skill

Cabeçalho (primeiras linhas do template): use "closes" para features; "fixes" para bugs; substitua `#xyz` pelo número real da issue. Sem issue, remova as duas linhas.

Apresentar o caminho do arquivo gerado ao usuário. Só atualizar o PR no GitHub (`gh pr edit <n> --body-file <arquivo>`) quando o usuário pedir.
