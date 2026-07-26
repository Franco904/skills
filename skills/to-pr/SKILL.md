---
name: to-pr
description: Gera o corpo de uma PR seguindo `pull_request_template.md`, com ênfase em plano de QA ponta-a-ponta derivado do tracer bullet da issue. Salva como `pr-<branch>.md` na raiz do repositório. Use quando o usuário quiser abrir uma PR, gerar o corpo de uma PR ou criar o plano de QA de uma entrega.
argument-hint: "Qual o número da issue e a branch de destino? (ex: 1442 develop)"
---

## Invocação

`/to-pr <issue-number> [target-branch]`

- `<issue-number>`: número da issue GitHub. Se omitido, inferir do nome da branch (ex: `issue-1422` → #1422). Se não for possível inferir, perguntar ao usuário.
- `[target-branch]`: branch de destino da PR. Default: `develop`.

---

## Processo

### 1. Coletar contexto

```bash
git branch --show-current          # nome da branch atual
git log <target-branch>...HEAD --oneline  # commits da branch
git diff <target-branch>...HEAD --stat    # arquivos alterados
```

Buscar a issue no GitHub com `gh issue view <issue-number>`. Extrair:
- Descrição e história de usuário
- Critérios de Aceitação
- Seção "Decisões de Implementação" (tracer bullet)
- Seção "Decisões de Teste"

### 2. Seção 1 — Análise de Impactos

Verificar se o diff toca arquivos em:
- `modules/common/` ou `modules/dependencies/`
- Widgets/temas compartilhados entre módulos
- Rotas (`*_module.dart`)
- Banco de dados (migrations, DAOs compartilhados)
- Contratos de API / serialização

**Se sim:** preencher cada item do checklist do template com o que foi identificado.
**Se não:** preencher a seção com `–`.

### 3. Seção 2 — Orientações para Review

- Listar decisões de implementação não-óbvias (das "Decisões de Implementação" da issue)
- Sinalizar ADRs em `docs/adr/` seguidos ou criados nesta entrega
- Sinalizar desvios em relação aos CAs originais (se houver)
- Apontar arquivos-chave que merecem atenção extra do revisor

Só isso. Se não houver nada a destacar, preencher a seção com `–`.

### 4. Seção 3 — Plano de QA

Se a issue tiver a label **"Sem teste"**, preencher a seção com `–` e pular esta etapa.

Caso contrário, derivar o fluxo ponta-a-ponta a partir do tracer bullet da issue. Gerar como lista de bullet points, um item por cenário, no formato:

- **Cenário**: pré-condição → ação → resultado esperado

Cobrir **obrigatoriamente**:

| Categoria | O que testar |
|-----------|-------------|
| Caminho feliz | Fluxo principal completo conforme tracer bullet |
| Estado vazio | Sem dados pré-existentes ou lista vazia |
| Erro de servidor | Timeout, 4xx, 5xx — respostas inesperadas da API |
| Offline | Sem conectividade ao iniciar e ao longo do fluxo |
| Sessão expirada | Token vence durante o fluxo |
| Navegação | Back button, interrupção e retomada do fluxo |
| Entrada inválida | Formulários com dados ausentes, formatos incorretos, limites |
| Edge cases de dados | Campos opcionais ausentes, valores limítrofes, strings longas |
| Impactos laterais | Outros fluxos afetados pelas mudanças da Seção 1 (se aplicável) |

### 5. Gerar arquivo

Salvar como `pr-<branch-name>.md` na raiz do repositório, usando a estrutura exata do template @pull_request_template.md.

Cabeçalho: use "closes" para features; "fixes" para bugs. Substituir `#xyz` pelo número real da issue.

Apresentar o caminho do arquivo gerado ao usuário.
