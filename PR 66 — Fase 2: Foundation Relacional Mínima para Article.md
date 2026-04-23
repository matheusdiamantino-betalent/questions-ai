
# 🧩 PR 65 — Fase 2: Normalização Determinística para Lookup Relacional
## Evolução incremental da fundação relacional com aumento controlado de match sem ampliar arquitetura

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
> Esta PR sucede a fundação relacional mínima introduzida na PR 64 e adiciona uma etapa simples de normalização determinística antes das consultas relacionais.
>
> - reduz falhas por variações superficiais de entrada
> - aumenta reaproveitamento de registros já existentes
> - preserva boundary, contrato externo e recorte incremental
>
> **Esta PR não introduz fuzzy matching, cache, catálogo local ou heurísticas avançadas.**

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

A PR 64 estabeleceu a base relacional mínima para a evolução do ID Resolution. Com essa fundação disponível, o próximo passo natural é melhorar a taxa de match sem ampliar responsabilidades ou introduzir novas camadas. Esta entrega adiciona somente normalização determinística da entrada antes do lookup.

## 2. Objetivo do PR

- normalizar entradas usadas nas consultas relacionais
- ampliar match para valores equivalentes
- preservar contrato atual de saída
- manter no-match previsível
- validar evolução com testes objetivos

## 3. Decisão Arquitetural

O desenho permanece inalterado. O fluxo atual continua utilizando a mesma separação de responsabilidades. A mudança adiciona apenas preparação explícita da chave de busca antes da query.

## 4. Escopo

- trim de bordas
- colapso de espaços internos
- remoção de acentos quando aplicável
- padronização determinística de entrada
- reaproveitamento das queries existentes
- testes de match normalizado e no-match

## 5. Fora de Escopo

- fuzzy matching
- aliases extensos
- score de confiança
- cache Redis
- múltiplas estratégias paralelas
- catálogo local
- novas integrações
- expansão para outros domínios

## 6. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050b16",
    "primaryColor": "#0b1220",
    "primaryTextColor": "#ffffff",
    "primaryBorderColor": "#22d3ee",
    "lineColor": "#94a3b8"
  }
}}%%
flowchart LR
    A[Metadata] --> B[Normalizer]
    B --> C[IdResolutionAgent]
    C --> D[IdResolutionDao]
    D --> E[(Base Principal)]
    E --> F[IDs resolvidos ou null]
```

## 7. Contratos Mínimos

```ts
ResolvedIds {
  disciplineId?: string | null;
  matterId?: string | null;
  subMatterId?: string | null;
  bankId?: string | null;
  articleId?: string | null;
  yearId?: string | null;
  institutionId?: string | null;
  jobId?: string | null;
}
```

## 8. Regras de Implementação

- normalização simples e explícita
- sem mover responsabilidades de camada
- sem novas abstrações
- query permanece no DAO
- comportamento previsível
- recorte pequeno

## 9. Critérios de Review

- normalização legível e coesa
- boundary mantido
- ausência de overengineering
- testes cobrindo equivalência e no-match
- sem regressão contratual

## 10. Critérios de Aceite

- [ ] entrada normalizada antes do lookup
- [ ] valores equivalentes retornam match persistido quando existir
- [ ] no-match permanece previsível
- [ ] contrato externo compatível
- [ ] suíte de testes verde

## 11. Conclusão

A PR 65 aumenta a efetividade da fundação relacional introduzida na PR 64 usando apenas padronização determinística da entrada. O ganho funcional vem sem aumento indevido de complexidade e preserva a evolução incremental da Fase 2.
