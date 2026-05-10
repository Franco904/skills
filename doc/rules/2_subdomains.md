# Subdomínios — DDD Estratégico e-SUS Atendimento

> **Finalidade:** Classificar os subdomínios do aplicativo segundo o valor de negócio que entregam (principal, suporte ou genérico). Esta classificação orienta decisões táticas de modelagem da lógica de negócio e de arquitetura: onde investir em maior complexidade de design e onde usar soluções mais simples ou de disponíveis no mercado.
>
> **Instrução para o Agente de IA:** Antes de sugerir padrões de modelagem (entidades ricas, VOs, casos de uso complexos) ou de arquitetura (Clean Architecture completa, inversão de dependências), verifique a classificação do subdomínio em questão. Aplique o padrão tático correspondente da seção "Guia de Padrões Táticos". Trate o mapeamento como heurística, não como regra absoluta — a complexidade real do código deve prevalecer sobre a categoria.

---

## Classificação dos Subdomínios

### Principal (Core)

> Diferencial competitivo do produto. Contém as regras de negócio mais complexas e exclusivas. Justifica o maior investimento em modelagem e arquitetura.

| Subdomínio | Módulo / Área | Descrição |
|---|---|---|
| **Registro SOAP** | `atendimento` — SOAP | Registro clínico orientado por problemas (Subjetivo, Objetivo, Avaliação, Plano), segundo o modelo RCOP |
| **Folha de Rosto (FR)** | `atendimento` — Folha de Rosto | Contexto clínico do Cidadão: alergias/reações adversas, problemas/condições, resultados de exames e medicamentos em uso — lido antes do atendimento |
| **Dados Pessoais (DP)** | `atendimento` — Dados Pessoais | Exibição e formatação dos dados cadastrais do Cidadão oriundos do PEC ou inserção manual pelo app |

---

### Suporte (Supporting)

> Necessário para o produto funcionar e apresenta alguma personalização de marca, mas não é o diferencial competitivo. Padrões mais simples são aceitáveis; modelagem rica só se a complexidade real exigir.

| Subdomínio | Módulo(s) |
|---|---|
| **Identificação da Pessoa (IP)** | `identificacao` |
| **Lista de Atendimentos (LA) e Lista de Pessoas (LP)** | `home` |
| **Sincronização com o PEC** | `sync` |
| **Autenticação e Login** | `authentication` |
| **Seleção de Lotação** | `authentication` |
| **Termo de Uso** | `authentication`, `root` |
| **Configurações** | `drawer` |
| **Sobre** | `drawer` |
| **Pesquisa de Satisfação** | `feedback` |

---

### Genérico (Generic)

> Resolvido por bibliotecas de mercado. Não deve receber modelagem de domínio própria. O papel do código interno é apenas encapsular a biblioteca (Adapter) para permitir substituição futura, se necessário.

| Subdomínio | Biblioteca utilizada |
|---|---|
| **Criptografia local** | `flutter_secure_storage` |
| **Log de eventos** | `firebase_analytics` |
| **Propriedades e métricas (Analytics)** | `firebase_analytics` |
| **Monitoramento de falhas** | `firebase_crashlytics` |
| **Feature flags** | `firebase_remote_config` |

---

## Guia de Padrões Táticos

A classificação do subdomínio é a principal heurística para escolher entre padrões de lógica de negócio e de arquitetura.

### Padrão de Lógica de Negócio

| Classificação | Padrão indicado | Como aplicar |
|---|---|---|
| **Principal** | **Modelo de Domínio** | Entidades ricas com comportamento, Value Objects com invariantes, exceções de domínio tipadas (enums `DomainException`), casos de uso que orquestram agregados |
| **Suporte** | **Script de Transação** ou Modelo de Domínio leve | Avalie a complexidade real: se há ramificações de negócio relevantes, prefira Modelo de Domínio; se é recuperação e persistência direta, Script de Transação é suficiente |
| **Genérico** | **Nenhum** — delegar à biblioteca | Criar um domínio próprio seria reimplementar o que a biblioteca já resolve; use apenas um Adapter para encapsulá-la |

### Padrão de Arquitetura

| Classificação | Padrão indicado | Como aplicar |
|---|---|---|
| **Principal** | **Arquitetura Concêntrica (Clean Architecture)** | Camadas completas: `domain → data → external → presentation`. Inversão de dependências obrigatória: `domain` não depende de nada externo |
| **Suporte** | **Clean Architecture** (preferencial, mantém consistência) ou Arquitetura em Camadas simples | Clean é preferível se o módulo já segue esse padrão; Camadas é aceitável para lógica muito simples de CRUD |
| **Genérico** | **Adapter simples** | Uma classe que encapsula a biblioteca. Sem camadas de domínio, sem repositórios — apenas a interface necessária para o restante do app |

---

## Heurística, não Dogma

O mapeamento acima é um ponto de partida estratégico. As seguintes situações justificam revisar ou ignorar a categoria:

- **Suporte mais complexo que o esperado:** um subdomínio classificado como suporte pode desenvolver regras de negócio sofisticadas ao longo do tempo. Se o código começa a acumular condicionais e lógica de estado relevante, eleve o padrão para Modelo de Domínio independentemente da categoria.
- **Principal mais simples que o esperado:** nem toda tela do SOAP é igualmente complexa. Um método simples de leitura dentro de um subdomínio principal não precisa de agregado — use o julgamento devido.
- **Genérico que cresce:** se uma biblioteca de mercado não atende mais ao caso de uso e lógica própria começa a surgir, o subdomínio pode ter migrado para suporte. Sinalize para revisão da classificação.

**Regra prática:** quando o código e a categoria divergirem, confie no código. A classificação existe para orientar decisões, não para engessá-las.
