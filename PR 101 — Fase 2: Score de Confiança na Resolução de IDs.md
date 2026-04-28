# 🧾 PR 101 — Fase 2: Score de Confiança na Resolução de IDs

## Introdução de confiança explícita no fallback semântico da resolução de IDs

---

<div align="left">

![PR](https://img.shields.io/badge/PR-101-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-id%20confidence-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui a sequência após os guardrails e consolidações anteriores, adicionando um indicador explícito de confiança na resolução de IDs.
>
> - mantém match exato como prioridade
> - mantém fallback semântico/fuzzy quando necessário
> - explicita confiança da decisão tomada
> - preserva fluxo atual e contratos externos quando possível
>
> **Este PR não cria novo agent, não altera pipeline e não redesenha a arquitetura.**

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

Após introduzir match exato prioritário e fallback fuzzy para a resolução de IDs, o próximo avanço funcional é tornar a decisão observável.

A PR 101 adiciona um score de confiança para cada resolução realizada, permitindo distinguir matches fortes, matches aproximados e ausência de confiança suficiente.

O objetivo é elevar previsibilidade sem aumentar complexidade estrutural.

# 2. Objetivo do PR

- adicionar score de confiança na resolução de IDs
- explicitar qualidade do match encontrado
- manter match exato como melhor caminho
- manter fallback fuzzy existente
- facilitar futuras decisões baseadas em confiança

# 3. Decisão Arquitetural

O score permanece local ao fluxo de resolução de IDs.

Não será criada nova camada de ranking global. O cálculo fica encapsulado no ponto onde a decisão já acontece hoje, preservando baixo acoplamento e recorte pequeno.

# 4. Escopo

- calcular score para match exato
- calcular score para fallback fuzzy
- retornar score junto da resolução interna
- permitir `null` quando não houver candidato confiável
- ajustar testes do fluxo de resolução

# 5. Fora de Escopo

- novo agent
- alteração de API pública externa
- banco de dados
- embeddings
- vector store
- fila
- redesign do orchestrator
- múltiplos thresholds dinâmicos por domínio

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
    "curve": "linear"
  }
}}%%
flowchart LR
    A["Metadata Normalizado"] --> B["Exact Match"]
    B --> C{"Encontrou?"}
    C -- Sim --> D["ID + score 1.00"]
    C -- Não --> E["Fallback Fuzzy"]
    E --> F{"Threshold ok?"}
    F -- Sim --> G["ID + score calculado"]
    F -- Não --> H["null"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef decision fill:#3f3f46,stroke:#facc15,stroke-width:2px,color:#ffffff;
    classDef successBox fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef failureBox fill:#2a0f0f,stroke:#ef4444,stroke-width:2px,color:#ffffff;

    class A,B,E step1;
    class C,F decision;
    class D,G successBox;
    class H failureBox;
```

# 7. Contratos Mínimos

Exemplo interno:

```ts
{
  id: "123",
  confidence: 1
}
```

Ou:

```ts
{
  id: "456",
  confidence: 0.82
}
```

Ou ausência de match:

```ts
null
```

# 8. Regras de Implementação

- score 1.00 para match exato
- score proporcional para fallback fuzzy
- manter comportamento atual de resolução
- não expandir escopo para observabilidade global
- testes cobrindo exato, fuzzy e ausência de match

# 9. Critérios de Review

- score está coerente com o tipo de match
- exact match continua prioritário
- fallback continua controlado
- recorte permanece pequeno
- sem novas dependências
- sem regressão funcional

# 10. Critérios de Aceite

- [ ] exact match retorna score máximo
- [ ] fuzzy match retorna score intermediário
- [ ] ausência de match confiável retorna null
- [ ] testes verdes
- [ ] sem alteração indevida do fluxo atual

# 11. Conclusão

A PR 101 adiciona uma camada útil de inteligência operacional à resolução de IDs: além de resolver, o sistema passa a indicar quão confiável foi a decisão. É evolução funcional real, localizada e proporcional ao estágio atual do projeto.
