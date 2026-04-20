# 🔎 PR 64 — Fase 2: Primeiro Lookup Real de IDs via Base Relacional
## Ativação inicial do match persistido no fluxo principal com recorte mínimo e previsível

---

<div align="left">

![PR](https://img.shields.io/badge/PR-64-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-primeiro%20lookup%20real-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR ativa o primeiro uso funcional da base relacional para resolução de IDs.
>
> - implementa consultas reais no DAO
> - retorna IDs persistidos quando houver match
> - mantém fallback previsível quando não houver correspondência
>
> **Este PR não introduz fuzzy matching, cache ou heurísticas avançadas.**

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

Após a introdução do boundary de resolução via DAO e da foundation relacional, o próximo passo natural é conectar o fluxo principal à persistência real. Esta entrega habilita o primeiro lookup funcional, retornando IDs reais em correspondências diretas e preservando o comportamento previsível nos casos sem match.

## 2. Objetivo do PR

- implementar queries reais no `IdResolutionDao`
- ativar lookup persistido no fluxo principal
- preservar contrato atual de saída
- manter fallback controlado para no-match
- validar a evolução com testes objetivos

## 3. Decisão Arquitetural

A arquitetura existente é mantida. O `IdResolutionAgent` continua responsável pela orquestração, enquanto o `IdResolutionDao` passa a executar consultas reais na base relacional. O recorte permanece pequeno, direto e revisável.

## 4. Escopo

- implementação de consultas reais no DAO
- primeiro match persistido para entidades suportadas nesta etapa
- retorno de IDs reais quando encontrados
- fallback atual quando não encontrados
- testes cobrindo match e ausência de match

## 5. Fora de Escopo

- fuzzy matching
- score de confiança
- aliases extensos
- cache Redis
- múltiplas estratégias paralelas
- heurísticas avançadas
- observabilidade expandida
- enriquecimento adicional

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Metadata] --> B[IdResolutionAgent]
    B --> C[IdResolutionDao]
    C --> D[(Relational DB)]
    D --> E[IDs reais ou fallback]
    E --> F[Output]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F step6;
```

## 7. Contratos Mínimos

Os contratos externos permanecem inalterados.

```ts
ResolvedIds {
  lawId?: string | null;
  articleId?: string | null;
  bankId?: string | null;
  yearId?: string | null;
}
```

## 8. Regras de Implementação

- queries simples e legíveis
- priorizar match exato
- agent sem SQL
- DAO concentrando persistência
- fallback preservado
- sem antecipar próximas fases

## 9. Critérios de Review

- lookup real encapsulado no DAO
- agent permanece coeso
- ausência de overengineering
- fallback previsível
- testes cobrem match e no-match

## 10. Critérios de Aceite

- [ ] DAO consulta base relacional
- [ ] IDs reais retornam quando houver match
- [ ] no-match mantém fallback esperado
- [ ] contrato externo permanece compatível
- [ ] suíte de testes verde

## 11. Conclusão

Esta PR conclui a transição entre foundation e uso funcional da persistência para resolução de IDs. O fluxo passa a consumir dados reais de forma controlada, incremental e aderente ao desenho da Fase 2.
