# 🔎 PR 65 — Fase 2: Normalização Determinística para Lookup Relacional
## Evolução incremental do lookup real com aumento controlado de match persistido

---

<div align="left">

![PR](https://img.shields.io/badge/PR-65-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-normalizacao%20deterministica-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui o lookup real entregue na PR 64 com uma etapa mínima de normalização antes da consulta relacional.
>
> - reduz falhas por variações superficiais de entrada
> - aumenta o reaproveitamento do match persistido
> - preserva contrato externo, fallback e arquitetura atual
>
> **Este PR não introduz fuzzy matching, cache, aliases extensos ou heurísticas avançadas.**

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

A PR 64 conectou o fluxo principal ao primeiro lookup real via base relacional, retornando IDs persistidos em correspondências diretas e mantendo fallback previsível nos casos sem match.

Com essa fundação concluída, o próximo passo mínimo correto é melhorar a taxa de correspondência sem expandir a arquitetura. Esta PR adiciona normalização determinística da entrada antes da consulta, permitindo reconhecer valores equivalentes com diferenças superficiais de formatação.

## 2. Objetivo do PR

- introduzir normalização mínima antes do lookup relacional
- ampliar matches persistidos para entradas equivalentes
- preservar contrato atual de saída
- manter fallback atual para no-match
- validar a evolução com testes objetivos

## 3. Decisão Arquitetural

A arquitetura existente é mantida. O `IdResolutionAgent` continua orquestrando o fluxo e o `IdResolutionDao` permanece responsável exclusivamente pela persistência. A nova etapa adiciona apenas preparação determinística da chave antes da consulta, sem novas camadas, sem fluxos paralelos e sem mudança de boundary.

## 4. Escopo

- normalização de valores usados no lookup
- `trim` de borda
- colapso de espaços internos
- padronização de casing
- reaproveitamento das queries reais já introduzidas na PR 64
- fallback atual quando não houver match
- testes cobrindo match direto, match normalizado e ausência de match

## 5. Fora de Escopo

- fuzzy matching
- score de confiança
- aliases extensos
- cache Redis
- múltiplas estratégias paralelas
- heurísticas avançadas
- observabilidade expandida
- alteração de contratos externos
- expansão para próximos slices

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
    A["Metadata"] --> B["Normalizer"]
    B --> C["IdResolutionAgent"]
    C --> D["IdResolutionDao"]
    D --> E["Relational DB"]
    E --> F["IDs reais ou fallback"]
    F --> G["Output"]

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

- normalização simples, explícita e determinística
- agent continua sem SQL
- DAO continua concentrando persistência
- queries reais reaproveitadas sem expansão
- fallback preservado
- sem antecipar próximos passos da fase

## 9. Critérios de Review

- normalização é pequena e legível
- boundary entre agent e DAO foi mantido
- lookup continua encapsulado no DAO
- ausência de overengineering
- fallback segue previsível
- testes cobrem match normalizado e no-match
- documentação permanece proporcional ao slice

## 10. Critérios de Aceite

- [ ] entrada é normalizada antes da consulta relacional
- [ ] variações superficiais válidas retornam IDs reais persistidos
- [ ] no-match mantém fallback esperado
- [ ] contrato externo permanece compatível
- [ ] suíte de testes verde
- [ ] agent e DAO mantêm responsabilidades atuais

## 11. Conclusão

A PR 65 expande a efetividade do lookup real introduzido na PR 64 sem ampliar a complexidade estrutural. O ganho vem de uma padronização mínima e previsível da entrada, mantendo o fluxo incremental, simples e aderente ao desenho da Fase 2.
