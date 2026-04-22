# 🔎 PR 66 — Fase 2: Primeiro Lookup Real de Article via Base Relacional
## Fechamento incremental da resolução persistida de artigos com recorte mínimo e previsível

---

<div align="left">

![PR](https://img.shields.io/badge/PR-66-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-lookup%20real%20de%20article-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui a linha iniciada nas PRs 64 e 65 ativando o primeiro lookup real de `article`.
>
> - implementa consulta persistida de artigo via base relacional
> - retorna `articleId` real quando houver correspondência válida
> - preserva fallback atual quando não houver match
>
> **Este PR não introduz fuzzy matching, aliases, cache ou novas estratégias de resolução.**

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

A PR 64 ativou o primeiro lookup real para entidades suportadas inicialmente. A PR 65 aumentou a taxa de match persistido com normalização determinística mínima da entrada. O próximo passo natural é concluir essa trilha no ponto já previsto do fluxo: a resolução persistida de artigos.

Esta entrega implementa a primeira consulta real para `article`, utilizando o `lawId` já resolvido como dependência natural da busca. O comportamento permanece previsível, incremental e compatível com o fallback existente.

## 2. Objetivo do PR

- implementar query real para `article` no `IdResolutionDao`
- retornar `articleId` persistido quando houver match
- manter fallback atual quando não houver correspondência
- preservar dependência de `article` em relação ao `lawId` resolvido
- validar a evolução com testes objetivos

## 3. Decisão Arquitetural

A arquitetura existente é mantida. O `IdResolutionAgent` continua orquestrando o fluxo e o `IdResolutionDao` permanece como boundary de persistência. A mudança desta PR é restrita à ativação da consulta real de artigo no ponto já previsto pelo desenho atual, sem novas camadas, sem pipelines paralelos e sem alteração contratual.

## 4. Escopo

- implementação de query real para `article`
- lookup por `lawId` + `articleNumber`
- retorno de `articleId` real quando encontrado
- fallback atual quando não encontrado
- manutenção do fluxo atual de dependência entre `law` e `article`
- testes cobrindo match, no-match e guarda de dependência

## 5. Fora de Escopo

- fuzzy matching de artigo
- aliases de artigo
- score de confiança
- cache Redis
- múltiplas estratégias paralelas
- heurísticas adicionais
- alteração de contratos externos
- expansão para `year`
- mudanças fora do fluxo de `article`

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
    A["Metadata"] --> B["Law resolvida"]
    B --> C["IdResolutionAgent"]
    C --> D["IdResolutionDao"]
    D --> E["Lookup article"]
    E --> F["articleId real ou fallback"]
    F --> G["ResolvedIds"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef step7 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F step6;
    class G step7;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
    linkStyle 4 stroke:#9ca3af,stroke-width:2px;
    linkStyle 5 stroke:#9ca3af,stroke-width:2px;
```

## 7. Contratos Mínimos

Os contratos externos permanecem inalterados.

```ts
export interface ResolvedIds {
  lawId?: string | null;
  articleId?: string | null;
  bankId?: string | null;
  yearId?: string | null;
}
```

## 8. Regras de Implementação

- query simples e legível para `article`
- `IdResolutionAgent` continua sem SQL
- `IdResolutionDao` concentra persistência
- lookup de artigo depende de `lawId` resolvido
- fallback preservado
- sem antecipar próximos passos da fase

## 9. Critérios de Review

- lookup real de `article` está encapsulado no DAO
- boundary entre agent e DAO foi mantido
- ausência de overengineering
- fallback continua previsível
- dependência entre `law` e `article` permanece clara
- testes cobrem match, no-match e guarda de dependência
- documentação permanece proporcional ao slice

## 10. Critérios de Aceite

- [ ] DAO consulta base relacional para `article`
- [ ] `articleId` real retorna quando houver match válido
- [ ] no-match mantém fallback atual
- [ ] sem `lawId` resolvido não há lookup persistido de artigo
- [ ] contrato externo permanece compatível
- [ ] suíte de testes verde

## 11. Conclusão

A PR 66 conclui o próximo passo lógico da trilha iniciada nas PRs 64 e 65 ao ativar a resolução persistida de artigos no fluxo já existente. O ganho funcional é direto, pequeno e aderente ao desenho atual, sem expansão indevida de escopo ou arquitetura.
