---
name: e2e
description: Gera e mantém testes e2e com Maestro. Use quando o QA quiser criar ou atualizar testes e2e, mencionar "Maestro", "teste ponta a ponta", critérios de aceitação de uma issue, ou descrever um fluxo de usuário para validar.
argument-hint: "Qual fluxo testar de ponta a ponta?"
---

## Processo

Antes de começar, verifique se Maestro está disponível:
```bash
maestro --version
```
Se o comando falhar, avise o usuário e pare.

### 1. Leia o contexto

Antes de gerar qualquer YAML, leia:
- `e2e/` — estado atual dos fluxos existentes
- @CONTEXT.md — glossário da linguagem ubíqua do projeto a ser respeitada nos testes

### 2. Sinalize manutenção proativa

Ao ler `e2e/`, identifique fluxos que possam estar desatualizados (seletores de UI renomeados, módulos reestruturados, fluxos que não existem mais). Liste os sinais **antes** de gerar novos fluxos.

### 3. Interprete a entrada

| Entrada | Como tratar |
|---|---|
| CAs estruturados | Um fluxo por CA relevante para e2e |
| Texto livre | Interprete o fluxo descrito e mapeie às ações Maestro |
| Mistura | Combine os dois |

### 5. Identifique nodos de layout

Identifique os widgets-alvo do fluxo e seus identificadores semânticos.

### 6. Tracer bullet primeiro (AFK)

Gere o **caminho feliz mínimo**: launch → ação principal → assert crítico. Nada mais. Escreva o arquivo diretamente em `e2e/<modulo>/<fluxo>.yaml` sem pedir aprovação prévia — o tracer bullet é barato de reescrever.

Rode o teste imediatamente:
```bash
maestro test e2e/<modulo>/<fluxo>.yaml
```

- **Passou** → apresente o resultado e ofereça iterar com edge cases.
- **Falhou** → leia o output do Maestro, corrija o seletor ou passo problemático, re-rode. Até 3 tentativas autônomas. Se ainda falhar, suba ao usuário com o output completo e o diagnóstico.

Só itere com passos adicionais, asserts de erro e edge cases após o tracer bullet passar. Uma fatia por vez.

### 7. Entrega de edge cases (HITL)

Apresente cada fatia de edge case ao usuário antes de escrever. Aguarde aprovação.

---

## Como executar

Pré-requisito: [Maestro CLI instalado](https://maestro.mobile.dev/getting-started/installing-maestro) e dispositivo/emulador conectado.

```bash
# Executar um fluxo específico
maestro test e2e/authentication/login_com_cpf.yaml

# Executar todos os fluxos
maestro test e2e/
```

---

## Comandos Maestro frequentes

| Comando | Uso |
|---|---|
| `- launchApp` | Inicia o app |
| `- tapOn:\n    id: "xxx"` | Toca em elemento pelo identificador semântico |
| `- assertVisible:\n    id: "xxx"` | Verifica que o elemento está visível na tela |
| `- assertNotVisible:\n    id: "xxx"` | Verifica que o elemento não está visível |
| `- inputText: "valor"` | Digita em campo com foco ativo |
| `- scrollDown` | Rola a tela para baixo |
| `- back` | Pressiona voltar |
| `- takeScreenshot: nome` | Captura tela com nome descritivo |
| `- waitForAnimationToEnd` | Aguarda animações antes de continuar |

Referência completa: https://docs.maestro.dev/reference
