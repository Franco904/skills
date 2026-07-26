---
name: write-a-skill
description: Cria novas skills de agente com estrutura adequada, divulgação progressiva e recursos agrupados. Use quando o usuário quiser criar uma nova skill.
---

## Processo

1. **Levantar requisitos** - perguntar ao usuário:
   - Qual tarefa/domínio a skill cobre?
   - Quais casos de uso específicos ela deve tratar?
   - Precisa de scripts executáveis ou apenas instruções?
   - Há materiais de referência para incluir?

2. **Rascunhar a skill** - criar:
   - SKILL.md com instruções concisas
   - Arquivos de referência adicionais se o conteúdo ultrapassar 500 linhas
   - Scripts utilitários se houver operações determinísticas

3. **Revisar com o usuário** - apresentar o rascunho e perguntar:
   - Isso cobre seus casos de uso?
   - Falta algo ou há algo pouco claro?
   - Alguma seção deveria ser mais ou menos detalhada?

## Estrutura da Skill

```
skill-name/
├── SKILL.md           # Instruções principais (obrigatório)
├── REFERENCE.md       # Docs detalhados (se necessário)
├── EXAMPLES.md        # Exemplos de uso (se necessário)
└── scripts/           # Scripts utilitários (se necessário)
    └── helper.js
```

## Template do SKILL.md

```md
---
name: skill-name
description: Descrição breve da capacidade. Use quando [gatilhos específicos].
---

# Nome da Skill

## Quick start

[Exemplo mínimo funcional]

## Processo

[Processos passo a passo com checklists para tarefas complexas]

## Funcionalidades avançadas

[Link para arquivos separados: Ver [REFERENCE.md](REFERENCE.md)]
```

## Requisitos da Description

A description é **a única coisa que seu agente vê** ao decidir qual skill carregar. Ela aparece no system prompt junto com todas as outras skills instaladas. O agente lê essas descriptions e escolhe a skill relevante com base na solicitação do usuário.

**Objetivo**: dar ao agente informação suficiente para saber:

1. Qual capacidade esta skill oferece
2. Quando/por que ativá-la (palavras-chave, contextos, tipos de arquivo)

**Formato**:

- Máx. 1024 chars
- Escrever na terceira pessoa
- Primeira frase: o que faz
- Segunda frase: "Use quando [gatilhos específicos]"

**Bom exemplo**:

```
Extrai texto e tabelas de arquivos PDF, preenche formulários, mescla documentos. Use quando trabalhar com arquivos PDF ou quando o usuário mencionar PDFs, formulários ou extração de documentos.
```

**Mau exemplo**:

```
Ajuda com documentos.
```

O mau exemplo não dá ao agente nenhuma forma de distinguir esta skill de outras skills de documentos.

## Quando Adicionar Scripts

Adicione scripts utilitários quando:

- A operação for determinística (validação, formatação)
- O mesmo código seria gerado repetidamente
- Erros precisam de tratamento explícito

Scripts economizam tokens e aumentam a confiabilidade em relação ao código gerado.

## Quando Dividir Arquivos

Divida em arquivos separados quando:

- O SKILL.md ultrapassar 100 linhas
- O conteúdo tiver domínios distintos (ex.: schemas de finanças vs. vendas)
- Funcionalidades avançadas forem raramente necessárias

## Checklist de Revisão

Após rascunhar, verificar:

- [ ] Description inclui gatilhos ("Use quando...")
- [ ] SKILL.md com menos de 100 linhas
- [ ] REFERENCE.md e outras referências com menos de 500 linhas
- [ ] Sem informações sensíveis ao tempo
- [ ] Terminologia consistente
- [ ] Exemplos concretos incluídos
- [ ] Referências com no máximo um nível de profundidade
