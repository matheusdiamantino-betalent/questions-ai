# 🤖 PR 63 — Fase 2: Foundation Relacional para Resolução de IDs
## Preparação mínima do schema de referência para viabilizar lookup real sem expandir o fluxo atual

---

<div align="left">

![PR](https://img.shields.io/badge/PR-63-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-reference%20schema%20foundation-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR recalibra corretamente o passo seguinte da PR 62 com base no schema atual do projeto.
>
> - introduz estruturas relacionais mínimas de referência
> - prepara o banco para lookup real futuro via DAO
> - preserva o fluxo atual sem alterar comportamento do agent
>
> **Este PR não implementa lookup real ainda, porque as entidades de domínio necessárias não existiam no banco até esta etapa.**

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

A PR 62 criou corretamente o boundary de persistência para resolução de IDs, mas o banco atual ainda não possui tabelas de domínio para leis, bancas, artigos ou anos. Antes de qualquer lookup real no `IdResolutionDao`, o próximo passo mínimo e aderente é introduzir a base relacional de referência que permitirá essa resolução nas próximas etapas.

## 2. Objetivo do PR

- criar estrutura relacional mínima para referências de resolução de IDs
- preparar o banco para lookup real futuro via `IdResolutionDao`
- manter `IdResolutionAgent` e `IdResolutionDao` no shape atual
- evitar expansão de fluxo antes da existência do dado de referência

## 3. Decisão Arquitetural

A arquitetura atual é preservada. Esta PR não muda o fluxo do agent nem adiciona lógica de resolução real. O objetivo é exclusivamente introduzir as entidades mínimas que faltavam no schema para que o DAO preparado na PR 62 possa, em PR seguinte, executar consultas reais com base em persistência legítima.

## 4. Escopo

- criação de tabela relacional para leis
- criação de tabela relacional para bancas
- definição mínima de colunas de identificação e nome normalizado
- geração dos tipos a partir do schema atualizado
- ajustes mínimos de wiring apenas se estritamente necessários

## 5. Fora de Escopo

- implementação de lookup real no DAO
- alteração de comportamento do `IdResolutionAgent`
- resolução real de artigo por `lawId`
- tabela de artigos, se isso inflar o recorte
- fuzzy matching
- score de similaridade
- cache
- aliases ricos
- seeds complexos
- refactor estrutural amplo

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[PR 62 - DAO Boundary] --> B[PR 63 - Reference Schema]
    B --> C[PR 64 - Real Lookup via DAO]
    C --> D[Resolved IDs Reais]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
```

## 7. Contratos Mínimos

Os contratos externos permanecem inalterados nesta etapa. A mudança é exclusivamente estrutural no banco, criando a base mínima necessária para futura resolução real de IDs. Nenhum contrato do fluxo principal é expandido neste momento.

## 8. Regras de Implementação

- schema mínimo e revisável
- nomes explícitos e simples
- incluir `normalizedName` nas entidades em que a resolução dependerá de lookup textual
- sem lógica de negócio em migration/schema
- sem antecipar estratégias de reconciliação
- sem inflar modelagem além do necessário para lookup direto
- preservar baixo ruído de review

## 9. Critérios de Review

- recorte pequeno e objetivo
- schema compatível com o próximo passo real do DAO
- ausência de overengineering
- nenhuma expansão prematura do fluxo
- modelagem simples e suficiente
- geração de tipos consistente com o padrão atual do projeto

## 10. Critérios de Aceite

- [ ] tabelas mínimas de referência criadas
- [ ] `normalizedName` disponível para lookup futuro onde necessário
- [ ] tipos gerados atualizados
- [ ] fluxo atual permanece intacto
- [ ] nenhuma camada adicional criada

## 11. Conclusão

Esta PR entrega a base estrutural que faltava para a evolução iniciada na PR 62. Em vez de forçar lookup real sem dado de referência, o projeto passa a ter o schema mínimo correto para que a próxima etapa implemente resolução real de IDs de forma simples, incremental e revisável.

