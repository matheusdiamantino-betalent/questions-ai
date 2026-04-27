# 🤖 PR 88 — Fase 2: Normalização Mínima do Input do Fluxo Avançado

## Higienização básica do payload válido antes da execução dos agents

---

<div align="left">

![PR](https://img.shields.io/badge/PR-88-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-normalizacao%20input-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui a robustez operacional da fase avançada ao normalizar inputs válidos antes da orquestração.
>
> - higieniza texto de entrada
> - remove ruído estrutural simples
> - preserva contrato atual em cenários válidos
>
> **Este PR não introduz validator global, pipe customizado, schema externo, novo agent ou redesign do pipeline.**

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

As PRs anteriores protegeram a fronteira de entrada do fluxo avançado. O próximo passo incremental é garantir que entradas válidas cheguem aos agents em formato minimamente limpo e previsível.

A PR 88 adiciona normalização simples no `AgentsFlowOrchestratorService`, reduzindo ruído sem alterar o comportamento funcional do pipeline.

# 2. Objetivo do PR

- aplicar `trim` em `question.statement`
- aplicar `trim` nas alternativas
- remover alternativas vazias após normalização
- entregar input consistente aos agents
- preservar contrato atual em cenários válidos

# 3. Decisão Arquitetural

A normalização permanece no `AgentsFlowOrchestratorService`, na fronteira de entrada do fluxo avançado.

A decisão é manter a higienização próxima do ponto de orquestração, sem deslocar regra simples para pipes, validators globais, schemas externos ou helpers prematuros.

# 4. Escopo

- normalizar `statement`
- normalizar textos das alternativas
- remover entradas vazias após `trim`
- executar antes dos agents
- adicionar testes cobrindo a normalização
- manter output de sucesso inalterado

# 5. Fora de Escopo

- deduplicação de alternativas
- validação semântica
- correção gramatical
- reordenação de alternativas
- enriquecimento de payload
- novo validator global
- redesign do pipeline

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
    "nodeSpacing": 34,
    "rankSpacing": 44
  }
}}%%
flowchart LR
    A["Input Válido"] --> B["Normalização"]
    B --> C["Advanced Flow"]
    C --> D["Agents Execution"]
    D --> E["Output Final"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef outputBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E outputBox;
```

# 7. Contratos Mínimos

Sem alteração estrutural no output final:

```ts
{
  legalSearch,
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

A PR adiciona apenas higienização interna do input antes da execução do fluxo.

# 8. Regras de Implementação

- concentrar normalização no `AgentsFlowOrchestratorService`
- executar antes de qualquer agent
- manter transformação simples e explícita
- não adicionar schema externo ou pipe customizado
- não criar helper global prematuro
- não alterar fluxo de sucesso

# 9. Critérios de Review

- trims aplicados corretamente
- alternativas vazias removidas
- agents recebem input normalizado
- fluxo válido permanece igual
- recorte pequeno foi mantido
- não há overengineering ou reestruturação indevida

# 10. Critérios de Aceite

- [ ] `statement` é normalizado
- [ ] alternativas são normalizadas
- [ ] entradas vazias são removidas
- [ ] fluxo válido preserva o comportamento atual
- [ ] output de sucesso permanece inalterado
- [ ] suíte permanece verde

# 11. Conclusão

A PR 88 evolui a qualidade do fluxo avançado no ponto correto: a fronteira de entrada do orchestrator.

Sem ampliar arquitetura ou contrato, o pipeline passa a operar com inputs mais consistentes antes de acionar os agents.
