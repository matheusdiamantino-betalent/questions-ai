# 🔎 PR 65 — Fase 2: Normalização Determinística para Lookup Relacional
## Aumento controlado da taxa de match persistido com padronização mínima e previsível

---

<div align="left">

![PR](https://img.shields.io/badge/PR-65-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-normalizacao%20deterministica%20para%20lookup-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui o lookup relacional introduzido na PR 64 com uma etapa mínima de normalização determinística antes da consulta persistida.
>
> - padroniza a chave de entrada antes do acesso relacional
> - aumenta o match em variações superficiais previsíveis
> - preserva contrato externo, fallback atual e desenho já aprovado
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

A PR 64 estabeleceu o primeiro lookup real via base relacional no fluxo de resolução. Com essa fundação ativa, o próximo passo mínimo correto é reduzir falhas causadas por variações superficiais de entrada sem reabrir arquitetura, sem alterar o contrato exposto e sem ampliar a responsabilidade dos componentes já definidos.

Esta PR adiciona uma normalização determinística e pequena antes da consulta persistida. O ganho esperado é aumentar a taxa de match em casos semanticamente equivalentes, mantendo previsibilidade operacional, fallback já existente e o mesmo desenho incremental da Fase 2.

## 2. Objetivo do PR

- introduzir normalização determinística antes do lookup relacional
- ampliar matches persistidos para entradas equivalentes com variação superficial
- preservar o contrato externo de resolução já adotado
- manter o fallback atual nos cenários sem correspondência
- validar o comportamento com testes objetivos e proporcionais ao slice

## 3. Decisão Arquitetural

A arquitetura aprovada permanece a mesma. O `IdResolutionAgent` continua responsável pela orquestração do fluxo e o `IdResolutionDao` segue como boundary de persistência. A mudança desta PR é restrita à padronização mínima da chave de entrada antes da consulta, sem novas camadas, sem estratégias paralelas e sem introdução de inteligência adicional fora do recorte.

A decisão central é tratar diferenças superficiais como problema de preparação determinística da entrada, e não como motivação para expandir a arquitetura. Com isso, a evolução permanece pequena, legível e coerente com a progressão iniciada na PR 64.

## 4. Escopo

- normalização mínima da entrada usada no lookup relacional
- aplicação de `trim` e colapso de espaços internos quando necessário
- padronização determinística de casing conforme a estratégia adotada
- reaproveitamento do fluxo atual de resolução persistida
- manutenção do fallback existente para cenários sem match
- testes cobrindo match direto, match após normalização e no-match

## 5. Fora de Escopo

- fuzzy matching
- score ou ranking de confiança
- aliases extensos ou dicionários auxiliares
- cache Redis
- múltiplas estratégias de lookup em paralelo
- heurísticas probabilísticas
- observabilidade expandida
- alteração de contrato externo
- expansão do fluxo para próximos passos da fase

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
    A["Metadata extraída"] --> B["Normalização determinística"]
    B --> C["IdResolutionAgent"]
    C --> D["IdResolutionDao"]
    D --> E["Lookup relacional"]
    E --> F["IDs reais ou fallback"]
    F --> G["ResolvedIds"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;
    classDef decision fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef successBox fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef failureBox fill:#2a160f,stroke:#fb7185,stroke-width:2px,color:#ffffff;
    classDef outputBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;
    classDef foundationBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;
    classDef coreBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F successBox;
    class G outputBox;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
    linkStyle 4 stroke:#9ca3af,stroke-width:2px;
    linkStyle 5 stroke:#9ca3af,stroke-width:2px;
```

## 7. Contratos Mínimos

Os contratos externos permanecem inalterados. Esta PR atua somente na preparação da chave de entrada antes do lookup, sem ampliar o shape de saída já consumido pelo fluxo.

```ts
export interface ResolvedIds {
  lawId?: string | null;
  articleId?: string | null;
  bankId?: string | null;
  yearId?: string | null;
}
```

## 8. Regras de Implementação

A implementação deve manter o mesmo princípio das PRs anteriores: fluxo visível, responsabilidade clara e mínima cerimônia. A normalização precisa ser explícita, determinística e pequena, evitando qualquer regra ambígua que dificulte leitura ou revisão.

O `IdResolutionAgent` deve continuar sem acoplamento de persistência e sem carregar lógica de consulta relacional. O `IdResolutionDao` permanece concentrando acesso a dados. Esta PR não deve antecipar fuzzy matching, cache, enriquecimento, aliases ou qualquer preparação estrutural para passos ainda não incluídos na fase atual.

## 9. Critérios de Review

- a normalização introduzida é pequena, legível e previsível
- o lookup relacional continua encapsulado no DAO
- o agent permanece coeso e sem responsabilidades extras
- o contrato externo segue compatível
- o fallback atual foi preservado
- não há overengineering, camadas novas ou preparação indevida de próximos passos
- a documentação permanece proporcional ao recorte entregue

## 10. Critérios de Aceite

- [ ] a entrada usada no lookup é normalizada antes da consulta persistida
- [ ] variações superficiais válidas passam a retornar os mesmos IDs reais
- [ ] cenários sem correspondência continuam respeitando o fallback atual
- [ ] o contrato externo de `ResolvedIds` permanece compatível
- [ ] a suíte de testes cobre match direto, match normalizado e no-match
- [ ] a implementação mantém o boundary atual entre agent e DAO

## 11. Conclusão

A PR 65 adiciona o próximo passo mínimo sobre o lookup relacional estabelecido na PR 64: aumentar a efetividade de match com padronização determinística da entrada, sem alterar o desenho arquitetural e sem inflar a solução. O resultado é uma evolução pequena, previsível e revisável, alinhada ao padrão de continuidade controlada da Fase 2.
