# 🤖 PR 62 — Fase 2: Foundation de Resolução de IDs via DAO
## Preparação controlada do boundary de persistência sem expandir o fluxo atual

---

<div align="left">

![PR](https://img.shields.io/badge/PR-62-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-id%20resolution%20foundation-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR reposiciona o escopo da etapa 62 para o próximo passo mínimo compatível com o schema atual.
>
> - introduz DAO dedicado para resolução de IDs
> - move dependência de persistência para boundary explícito
> - preserva comportamento atual com fallback previsível
>
> **Este PR não implementa match real em entidades ainda inexistentes no banco.**

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

Após a consolidação do fluxo principal com classificação e resolução sintética, o próximo passo correto é separar a responsabilidade de acesso a dados antes de introduzir match real. Como o schema atual ainda não possui tabelas de domínio para leis, artigos, bancas e anos, esta PR prepara o ponto de evolução sem inflar escopo.

## 2. Objetivo do PR

- criar `IdResolutionDao` como boundary de persistência
- refatorar `IdResolutionAgent` para depender do DAO
- manter saída atual compatível
- retornar fallback previsível enquanto não houver base relacional de referência

## 3. Decisão Arquitetural

A arquitetura existente é mantida. A mudança desta PR é apenas deslocar acesso futuro a dados para uma camada dedicada, evitando acoplamento direto do agent com consultas e preparando evolução incremental posterior.

## 4. Escopo

- criação de `id-resolution.dao.ts`
- injeção do DAO no `IdResolutionAgent`
- centralização da responsabilidade de lookup
- manutenção do comportamento atual
- testes cobrindo integração agent + dao mockado

## 5. Fora de Escopo

- match real com tabelas de domínio
- migrations novas
- fuzzy matching
- score de similaridade
- cache
- múltiplas estratégias de reconciliação
- enriquecimento de metadata

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Orchestrator] --> B[IdResolutionAgent]
    B --> C[IdResolutionDao]
    C --> D[Lookup atual ou fallback]
    D --> E[Resolved IDs]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
```

## 7. Contratos Mínimos

Os contratos externos do fluxo permanecem os mesmos nesta etapa. A mudança é interna: introdução do DAO como dependência explícita do agent.

## 8. Regras de Implementação

- agent continua responsável apenas por orquestração
- dao concentra lookup/persistência
- sem SQL espalhado em agents
- sem abstrações adicionais desnecessárias
- sem alterar contrato externo do fluxo

## 9. Critérios de Review

- recorte pequeno e claro
- responsabilidade bem posicionada entre agent e dao
- ausência de overengineering
- comportamento atual preservado
- testes objetivos e legíveis

## 10. Critérios de Aceite

- [ ] `IdResolutionDao` criado e registrado
- [ ] `IdResolutionAgent` depende do DAO
- [ ] retorno atual permanece compatível
- [ ] testes passam sem regressão

## 11. Conclusão

Esta PR não antecipa a fase seguinte. Ela organiza o boundary correto para que o match real com banco entre depois de forma simples, revisável e aderente ao desenho incremental do projeto.
