# 🔎 PR 66 — Fase 2: Foundation Relacional Mínima para Article
## Preparação incremental da base persistida para futura resolução de artigos sem ativar novo lookup nesta etapa

---

<div align="left">

![PR](https://img.shields.io/badge/PR-66-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-foundation%20article-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR ajusta a linha evolutiva da Fase 2 introduzindo a foundation relacional mínima necessária para suportar artigos no schema atual.
>
> - adiciona persistência estruturada de `article`
> - estabelece vínculo relacional com `law`
> - prepara o terreno para lookup real em PR posterior
>
> **Este PR não ativa resolução persistida de article, não altera contratos externos e não introduz heurísticas novas.**

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

As PRs 64 e 65 consolidaram a resolução persistida para entidades já presentes no schema relacional atual, com lookup real e aumento controlado de match por normalização mínima. O próximo passo natural seria avançar para artigos, porém o schema atual ainda não possui estrutura persistida para essa entidade.

Diante disso, esta PR executa o passo incremental correto: introduzir a foundation relacional mínima de `article`, mantendo a lógica atual intacta e preparando uma futura ativação funcional em recorte separado.

## 2. Objetivo do PR

- adicionar estrutura relacional mínima para `article`
- relacionar artigos a leis já persistidas
- incluir campos estritamente necessários para lookup futuro
- atualizar geração de tipos da camada de dados
- manter fluxo atual sem mudança funcional nesta etapa

## 3. Decisão Arquitetural

A arquitetura existente é mantida. O fluxo atual de `IdResolutionAgent` e `IdResolutionDao` permanece inalterado nesta PR. A mudança ocorre somente no boundary de persistência, adicionando suporte estrutural para `article` no schema relacional.

A decisão central é separar foundation de consumo funcional: primeiro a base persistida mínima, depois o uso efetivo em PR posterior. Isso preserva o incrementalismo e evita inflar o recorte atual.

## 4. Escopo

- criação da tabela relacional de `articles`
- chave primária compatível com padrão atual
- referência de `article` para `law`
- campo de número/identificador do artigo
- timestamps no padrão do projeto quando aplicável
- atualização do Prisma schema
- regeneração dos tipos do Kysely
- testes ou validações estruturais proporcionais ao slice

## 5. Fora de Escopo

- lookup real de `article`
- alterações no `IdResolutionAgent`
- alterações no `IdResolutionDao`
- fuzzy matching
- aliases
- cache Redis
- heurísticas adicionais
- mudanças de contrato externo
- enriquecimentos além da estrutura mínima

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
    A["Schema atual"] --> B["Adicionar articles"]
    B --> C["Relacionar com laws"]
    C --> D["Regenerar tipos DB"]
    D --> E["Base pronta para PR futura"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
```

## 7. Contratos Mínimos

Contratos externos permanecem inalterados nesta PR.

A mudança esperada ocorre apenas no schema interno da camada de dados, adicionando representação tipada para `articles` no `DB`.

## 8. Regras de Implementação

- modelagem mínima e objetiva
- sem campos especulativos
- nomes aderentes ao padrão atual
- vínculo claro entre `article` e `law`
- geração de tipos consistente com o schema
- sem antecipar regras de lookup nesta etapa
- sem alterar fluxo funcional existente

## 9. Critérios de Review

- tabela de `articles` é mínima e suficiente
- relacionamento com `laws` está claro
- não houve expansão indevida de modelagem
- tipos gerados refletem o novo schema
- fluxo atual permanece intacto
- documentação está proporcional ao slice
- foundation ficou pronta para consumo futuro sem excesso

## 10. Critérios de Aceite

- [ ] schema inclui estrutura mínima de `articles`
- [ ] existe vínculo relacional com `laws`
- [ ] tipos do banco foram regenerados
- [ ] nenhuma mudança funcional foi introduzida no fluxo atual
- [ ] suíte de testes verde
- [ ] base pronta para lookup real em PR posterior

## 11. Conclusão

A PR 66 corrige a sequência evolutiva da Fase 2 ao introduzir primeiro a foundation relacional necessária para artigos antes de ativar consumo funcional. O resultado é um passo pequeno, técnico e coerente com o padrão incremental já consolidado no projeto.
