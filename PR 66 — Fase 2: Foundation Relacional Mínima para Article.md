# 🔎 PR 66 — Fase 2: Expansão Controlada do ID Resolution para Entidades Essenciais
## Evolução incremental da resolução de IDs com cobertura mínima das classificações centrais do legado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-66-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-id%20resolution%20essencial-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR expande o ID Resolution sobre a fundação criada nas PRs 64 e 65, adicionando cobertura objetiva para entidades centrais do domínio sem alterar o desenho arquitetural atual.
>
> - prioriza disciplina e assunto como núcleo mínimo
> - adiciona suporte proporcional para subassunto, ano, cargo, instituição, banca e artigo
> - preserva o boundary entre `IdResolutionAgent` e `IdResolutionDao`
>
> **Não inclui fuzzy matching, catálogo local, cache Redis ou heurísticas avançadas.**

## Sumário

11. [Síntese Executiva](#1-síntese-executiva)
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

Após corrigir a base relacional (PR 64) e consolidar a normalização determinística do primeiro lookup real (PR 65), o próximo passo natural é ampliar a cobertura para entidades efetivamente usadas pelo domínio.

Esta entrega adiciona resolução real e proporcional para classificações centrais do legado, mantendo a mesma separação de responsabilidades e sem introduzir complexidade paralela.

## 2. Objetivo do PR

- expandir o lookup real para entidades essenciais
- priorizar disciplina e assunto
- suportar subassunto, ano, cargo, instituição, banca e artigo
- preservar contratos existentes
- manter evolução pequena e testável

## 3. Decisão Arquitetural

O `IdResolutionAgent` continua coordenando a resolução a partir do metadata classificado. O `IdResolutionDao` permanece como fronteira de acesso à base principal.

A evolução ocorre apenas pela ampliação de consultas objetivas e reaproveitamento da normalização já existente.

## 4. Escopo

- resolver `disciplineId`
- resolver `matterId`
- resolver `subMatterId`
- resolver `yearId`
- resolver `bankId`
- resolver `institutionId`
- resolver `jobId`
- resolver `articleId`
- ajustar testes unitários proporcionais ao slice

## 5. Fora de Escopo

- fuzzy matching
- score de confiança
- aliases extensos
- cache Redis
- catálogo local
- refactor amplo de classificação
- abstrações genéricas de matching
- observabilidade expandida

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
    A["Classification Metadata"] --> B["IdResolutionAgent"]
    B --> C["Normalize Inputs"]
    C --> D["IdResolutionDao"]
    D --> E["Main API DB"]
    E --> F["Resolved IDs"]
    F --> G["Next Agents"]

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

```ts
type ResolvedIds = {
  disciplineId?: string | null;
  matterId?: string | null;
  subMatterId?: string | null;
  yearId?: string | null;
  bankId?: string | null;
  institutionId?: string | null;
  jobId?: string | null;
  articleId?: string | null;
};
```

## 8. Regras de Implementação

- disciplina e assunto como prioridade mínima
- agent orquestra, DAO consulta
- queries pequenas e aderentes ao legado
- reaproveitar normalização determinística
- evitar overengineering

## 9. Critérios de Review

- expansão coerente com a fase
- boundary claro entre agent e DAO
- ausência de foundation paralela
- cobertura funcional restrita ao escopo
- diff revisável

## 10. Critérios de Aceite

- [ ] `disciplineId` e `matterId` resolvidos quando presentes
- [ ] demais entidades resolvidas de forma proporcional
- [ ] contratos compatíveis
- [ ] testes verdes
- [ ] sem persistência local adicional

## 11. Conclusão

A PR 66 amplia o ID Resolution para entidades essenciais do domínio de forma controlada, incremental e aderente à arquitetura já consolidada.
