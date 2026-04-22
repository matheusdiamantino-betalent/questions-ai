# ⚖️ PR 67 — Fase 2: Primeira Resolução Real de SubMatter na Base Principal
## Ativação funcional mínima da classificação jurídica existente no legado sem fundação paralela

---

<div align="left">

![PR](https://img.shields.io/badge/PR-67-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-submatter%20lookup-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR transforma o próximo passo da Fase 2 em uma entrega funcional ancorada no legado real.
>
> - ativa a primeira resolução real de `subMatter` na base principal
> - usa taxonomia já comprovada no legado (`disciplines`, `matters`, `sub_matters`)
> - evita modelagem especulativa de entidades não comprovadas
> - preserva o recorte pequeno, revisável e diretamente útil ao pipeline atual
>
> **Este PR não cria persistência local nova, não altera contratos externos e não introduz platform foundation multi-agent.**

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

## 1. Síntese Executiva

A sequência recente da Fase 2 consolidou um boundary mais aderente ao sistema principal. A PR 63 removeu catálogo relacional local indevido. A PR 64 ativou o primeiro lookup real via base principal. A PR 65 elevou a taxa de match com normalização determinística antes da consulta. A PR 66 corrigiu o enquadramento do recorte ao eliminar premissas sobre estruturas não comprovadas no legado.

Com esse histórico estabilizado, o próximo passo correto não é abrir foundation genérica de agents nem insistir em entidades normativas sem fonte de verdade comprovada. O passo incremental correto é ativar valor funcional sobre uma taxonomia já existente no legado: `disciplines`, `matters`, `sub_matters` e sua associação com `questions`.

Esta PR faz exatamente isso ao introduzir a primeira resolução real de `subMatter` na base principal, mantendo o fluxo atual e adicionando apenas o mínimo necessário para produzir um ID juridicamente classificatório aderente ao modelo já persistido.

## 2. Objetivo do PR

- ativar a primeira resolução real de `subMatter` na base principal
- aproveitar a taxonomia jurídica já comprovada no legado
- retornar `subMatterId` a partir de classificação textual normalizada
- manter o pipeline atual sem inflar escopo estrutural
- reforçar aderência ao boundary correto da Fase 2

## 3. Decisão Arquitetural

A decisão arquitetural desta PR é fazer a próxima evolução funcional exclusivamente sobre entidades reais do legado. O domínio de classificação jurídica já expõe estrutura persistida suficiente para um passo incremental útil: `disciplines`, `matters`, `sub_matters` e `question_sub_matters`.

A escolha por `subMatter` preserva o padrão correto do projeto por quatro razões:

- evolui em cima de fonte de verdade comprovada
- evita fundação paralela e modelagem especulativa
- adiciona valor funcional imediato ao pipeline de resolução
- mantém o recorte proporcional ao estágio atual do sistema

O objetivo aqui não é resolver toda a taxonomia com heurísticas amplas, nem redesenhar a arquitetura de agents. O objetivo é validar o caminho mínimo real de lookup classificatório a partir da base principal.

## 4. Escopo

- introduzir lookup real de `subMatter` na base principal
- definir a assinatura mínima de entrada para a resolução
- integrar a resolução ao `IdResolutionAgent` apenas no ponto necessário
- adicionar ou ajustar método correspondente no `IdResolutionDao`
- retornar `subMatterId` quando houver match determinístico
- manter testes e validações proporcionais ao slice

## 5. Fora de Escopo

- criação de novas tabelas locais
- migrations
- lookup de `law` ou `article`
- fuzzy matching
- aliases
- heurísticas avançadas de desambiguação
- cache Redis
- expansão de contratos externos
- refatoração para platform foundation multi-agent
- generalização prematura de agent registry, router ou APIs genéricas

## 6. Fluxo Arquitetural

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
    A["Classificação textual normalizada"] --> B["IdResolutionAgent"]
    B --> C["IdResolutionDao"]
    C --> D["Lookup real em sub_matters"]
    D --> E["Retorno de subMatterId"]
    E --> F["Pipeline segue com classificação resolvida"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F step6;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
    linkStyle 4 stroke:#9ca3af,stroke-width:2px;
```

## 7. Contratos Mínimos

Contratos externos permanecem inalterados nesta PR.

Esta etapa introduz apenas o contrato interno mínimo necessário para a primeira resolução real de `subMatter`:

```ts
type ResolveSubMatterInput = {
  normalizedSubMatterName: string;
};
```

Caso o legado exija desambiguação mínima por contexto classificatório, a evolução pode admitir apoio de `matter` ou `discipline` em etapas futuras. Nesta PR, o foco permanece no menor contrato capaz de validar o caminho real de lookup.

Saída esperada do slice:

```ts
type ResolveSubMatterOutput = {
  subMatterId: string | null;
};
```

## 8. Regras de Implementação

- não criar persistência local nova
- não inventar entidade fora do legado comprovado
- manter `IdResolutionAgent` sem SQL
- concentrar acesso à base principal no `IdResolutionDao`
- introduzir apenas um lookup determinístico mínimo
- evitar abstrações genéricas não exigidas pelo slice
- preservar baixo ruído estrutural e facilidade de review

## 9. Critérios de Review

- a PR evolui sobre entidade real do legado
- o lookup de `subMatter` foi implementado no boundary correto
- o passo continua pequeno e tecnicamente revisável
- não há fundação paralela nem modelagem especulativa
- `IdResolutionAgent` continua enxuto
- `IdResolutionDao` concentra a consulta real
- documentação e testes permanecem proporcionais ao recorte

## 10. Critérios de Aceite

- [ ] o sistema passa a resolver `subMatterId` real na base principal
- [ ] nenhuma persistência local nova foi criada
- [ ] nenhum contrato externo foi alterado
- [ ] o fluxo atual permaneceu íntegro
- [ ] suíte de testes permanece verde
- [ ] a PR entrega valor funcional concreto sem inflar a arquitetura

## 11. Conclusão

A PR 67 segue a linha correta da Fase 2 ao trocar especulação por aderência ao legado. Em vez de abrir uma foundation genérica de agents ou insistir em entidades sem fonte de verdade comprovada, o projeto adiciona uma capacidade funcional real sobre a taxonomia jurídica já existente.

O resultado é uma entrega pequena, objetiva e útil: a primeira resolução real de `subMatter` na base principal, construída no boundary correto e sem overengineering.
