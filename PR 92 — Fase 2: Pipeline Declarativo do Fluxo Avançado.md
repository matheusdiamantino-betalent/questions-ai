# 🤖 PR 92 — Fase 2: Pipeline Declarativo do Fluxo Avançado

## Sequência explícita de etapas com execução previsível no orchestrator

---

<div align="left">

![PR](https://img.shields.io/badge/PR-92-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-pipeline%20declarativo-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR organiza a sequência do fluxo avançado em um pipeline declarativo local ao orchestrator, mantendo a execução simples e previsível.
>
> - torna a ordem das etapas explícita
> - reutiliza o contexto compartilhado da execução
> - preserva contrato externo atual
>
> **Este PR não introduz workflow engine, DAG, paralelismo, plugin system, novo agent ou redesign do pipeline.**

## Sumário

1. [Síntese Executiva](#1-síntese-executiva)
2. [Objetivo do PR](#2-objetivo-do-pr)
3. [Decisão Arquitetural](#3-decisão-arquitetural)
4. [Escopo](#4-escopo)
5. [Fora de Escopo](#5-fora-de-escopo)
6. [Fluxo Arquitetural](#6-fluxo-arquitetural)
7. [Contratos Mínimos](#7-contratos-mínimos)
8. [Regras de Implementação](#8-regras-de-implementação)
9. [Critérios de Review](#9-critérios-de-review)
10. [Critérios de Aceite](#10-critérios-de-aceite)
11. [Conclusão](#11-conclusão)

# 1. Síntese Executiva

Após a introdução do `AgentsExecutionContext`, o próximo passo mínimo é deixar a sequência de execução do fluxo avançado mais clara sem alterar sua lógica funcional.

A PR 92 substitui chamadas rígidas e dispersas por uma lista declarativa de etapas executadas pelo `AgentsFlowOrchestratorService`, preservando a ordem atual e mantendo o pipeline local, simples e revisável.

# 2. Objetivo do PR

- declarar etapas do pipeline em estrutura única
- tornar a ordem de execução explícita
- reduzir repetição no orchestrator
- reutilizar `AgentsExecutionContext`
- preservar contrato externo atual

# 3. Decisão Arquitetural

A execução continua centralizada no `AgentsFlowOrchestratorService`, porém passa a ser guiada por uma coleção simples de etapas com `name` e `execute`.

A decisão melhora a legibilidade da sequência sem introduzir engine externa, DAG, paralelismo, plugin system ou abstrações desproporcionais ao recorte.

# 4. Escopo

- criar coleção declarativa de etapas
- iterar steps no orchestrator
- manter ordem funcional existente
- reutilizar contexto compartilhado
- manter output de sucesso inalterado
- adicionar testes objetivos do pipeline declarado

# 5. Fora de Escopo

- workflow engine
- DAG execution
- paralelismo
- plugin system
- filas
- state machine
- alteração da response pública

# 6. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050b16",
    "primaryColor": "#0b1220",
    "primaryTextColor": "#ffffff",
    "primaryBorderColor": "#22d3ee",
    "lineColor": "#94a3b8",
    "secondaryColor": "#0b1220",
    "tertiaryColor": "#0b1220",
    "fontFamily": "Inter, Arial, sans-serif"
  },
  "flowchart": {
    "htmlLabels": true,
    "curve": "linear",
    "nodeSpacing": 54,
    "rankSpacing": 64
  }
}}%%
flowchart TD
    A["<div style='min-width:190px; padding:8px 12px;'>Criar Contexto<br/>de Execução</div>"]
    B["<div style='min-width:190px; padding:8px 12px;'>Carregar Steps<br/>Declarados</div>"]
    C["<div style='min-width:180px; padding:8px 12px;'>Executar<br/>Step Atual</div>"]
    D["<div style='min-width:190px; padding:8px 12px;'>Atualizar<br/>Contexto</div>"]
    E{"<div style='min-width:170px; padding:8px 12px;'>Ainda há<br/>Steps?</div>"}
    F["<div style='min-width:180px; padding:8px 12px;'>Output<br/>Final</div>"]

    A --> B
    B --> C
    C --> D
    D --> E
    E -->|Sim| C
    E -->|Não| F

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef decision fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef outputBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step2;
    class E decision;
    class F outputBox;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
    linkStyle 4 stroke:#22d3ee,stroke-width:2px,stroke-dasharray: 5 5;
    linkStyle 5 stroke:#84cc16,stroke-width:2px;
```

# 7. Contratos Mínimos

Estrutura interna proposta:

```ts
type FlowStep = {
  name: string
  execute: (context: AgentsExecutionContext) => Promise<void>
}
```

Sem alteração estrutural no output final existente:

```ts
{
  legalSearch,
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

# 8. Regras de Implementação

- manter steps simples e locais ao orchestrator
- preservar ordem funcional existente
- reutilizar o contexto compartilhado
- atualizar contexto de forma explícita
- evitar abstrações genéricas excessivas
- não alterar interfaces públicas sem necessidade

# 9. Critérios de Review

- ordem funcional atual foi preservada
- steps ficaram explícitos e legíveis
- contexto compartilhado foi reutilizado
- output público permanece igual
- não há workflow engine, DAG ou plugin system
- recorte segue pequeno e aderente

# 10. Critérios de Aceite

- [ ] pipeline declarado foi introduzido
- [ ] steps executam na ordem esperada
- [ ] contexto compartilhado foi reutilizado
- [ ] output público permanece inalterado
- [ ] suíte permanece verde
- [ ] recorte pequeno foi mantido

# 11. Conclusão

A PR 92 melhora a definição da sequência do fluxo avançado sem ampliar arquitetura. O pipeline passa a operar com etapas explícitas e previsíveis, preservando contrato externo, ordem funcional e simplicidade de review.
