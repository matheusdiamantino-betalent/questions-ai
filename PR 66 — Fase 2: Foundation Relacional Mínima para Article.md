# 🔎 PR 66 — Fase 2: Preparação do Boundary para Article na Base Principal
## Consolidação do contrato mínimo para futura resolução real de artigos sem duplicação local

---

<div align="left">

![PR](https://img.shields.io/badge/PR-66-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-boundary%20article-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR reposiciona corretamente o próximo passo da Fase 2 após a reestruturação iniciada na PR 63.
>
> - prepara o boundary para futura resolução real de `article`
> - evita duplicação local de persistência não comprovada no legado
> - consolida o contrato mínimo necessário para a próxima evolução funcional
> - preserva o fluxo atual sem ativar novo lookup nesta etapa
>
> **Este PR não cria schema local de `article`, não altera contratos externos e não ativa resolução persistida de artigo.**

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

A sequência recente da Fase 2 mudou o entendimento correto do problema. A PR 63 corrigiu o boundary ao remover catálogo relacional local indevido. A PR 64 ativou o primeiro lookup real via base principal. A PR 65 aumentou a taxa de match com normalização determinística antes da consulta.

Com esse histórico consolidado, o próximo passo para `article` não é criar uma nova tabela local no serviço de IA. O legado analisado não comprova a existência de uma entidade `articles` já estabilizada no banco local deste serviço, nem justifica duplicação de persistência. O passo incremental correto é preparar o boundary para suportar `article` a partir da base principal, formalizando o contrato mínimo necessário para evolução futura sem inflar o recorte atual.

## 2. Objetivo do PR

- consolidar o contrato mínimo necessário para futura resolução real de `article`
- preparar o boundary de consulta à base principal para suportar esse próximo passo
- explicitar a dependência do shape real do legado antes de qualquer ativação funcional
- manter o fluxo atual intacto nesta etapa
- preservar o incrementalismo sem reabrir persistência local indevida

## 3. Decisão Arquitetural

A arquitetura segue o boundary corrigido desde a PR 63: este serviço não materializa localmente catálogos ou referências que pertencem à base principal. `Law` e `Bank` já migraram para esse entendimento; `article` deve seguir a mesma linha.

A decisão central desta PR é separar **preparação de boundary** de **ativação funcional**. Antes de implementar lookup real de `article`, é necessário consolidar o contrato mínimo de consulta, baseado no shape real do legado, sem inventar estrutura local não comprovada.

Isso preserva:
- aderência ao sistema principal
- recorte pequeno e revisável
- ausência de fundação paralela
- coerência com a linha evolutiva PR 63 → PR 65

## 4. Escopo

- preparar a assinatura mínima necessária para futura resolução de `article`
- formalizar o shape de entrada e dependências mínimas do lookup futuro
- explicitar vínculo esperado entre `law` e `article` no boundary da base principal
- alinhar documentação e contratos internos ao próximo passo evolutivo
- manter `IdResolutionAgent` e `IdResolutionDao` sem ativação funcional nova nesta etapa
- incluir testes ou validações proporcionais ao slice, caso existam contratos internos novos

## 5. Fora de Escopo

- criação de tabela `articles` no banco local do serviço de IA
- migration local para `article`
- regeneração de tipos do banco local para uma entidade não consolidada
- lookup real funcional de `article`
- fuzzy matching
- aliases
- cache Redis
- heurísticas adicionais
- alteração de contratos externos
- expansão estrutural do fluxo atual

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
    A["PR 63–65 Boundary Consolidado"] --> B["Identificar shape real de article no legado"]
    B --> C["Formalizar contrato mínimo"]
    C --> D["Preparar boundary da base principal"]
    D --> E["Fluxo atual preservado"]
    E --> F["PR futura ativa lookup real de article"]

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

O que esta etapa prepara é o contrato interno mínimo para futura resolução de artigo, preservando o entendimento de que a consulta dependerá:
- de uma referência de `law` real
- de um identificador de `article` aderente ao legado
- da fonte de verdade na base principal

Shape mínimo esperado para evolução futura:

```ts
type ResolveArticleInput = {
  lawId: string;
  articleNumber: string;
};
```

Essa definição não implica ativação funcional nesta PR. Ela apenas explicita o formato mínimo que a evolução seguinte deverá respeitar.

## 8. Regras de Implementação

- não criar persistência local não comprovada
- não inventar modelagem de `article` fora do legado real
- manter agent sem SQL e sem expansão funcional nesta etapa
- manter DAO sem ativar nova consulta não suportada pelo recorte
- introduzir apenas o mínimo necessário para preparar a próxima PR
- preservar baixo ruído estrutural e facilidade de review

## 9. Critérios de Review

- a PR não reabre catálogo local indevido
- o boundary corrigido desde a PR 63 foi mantido
- o passo proposto é pequeno, técnico e coerente com a sequência evolutiva
- não há fundação paralela nem modelagem especulativa
- o contrato mínimo de `article` ficou claro para a próxima entrega
- o fluxo atual permaneceu intacto
- a documentação está proporcional ao recorte

## 10. Critérios de Aceite

- [ ] nenhuma nova persistência local de `article` foi criada indevidamente
- [ ] o contrato mínimo para futura resolução de `article` ficou explícito
- [ ] o vínculo esperado entre `law` e `article` no boundary principal ficou documentado
- [ ] nenhuma mudança funcional foi introduzida no fluxo atual
- [ ] suíte de testes permanece verde
- [ ] base preparada para PR futura de lookup real de `article`

## 11. Conclusão

A PR 66 ajusta corretamente a continuidade da Fase 2 após a reestruturação iniciada na PR 63. Em vez de criar uma foundation local incorreta para `article`, o projeto dá um passo menor e mais seguro: consolidar o boundary e o contrato mínimo necessários para a próxima evolução funcional.

O resultado é uma entrega mais aderente ao legado, ao boundary correto e ao padrão incremental já consolidado pela sequência recente de PRs.
