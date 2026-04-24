# 🤖 PR 82 — Fase 2: Normalização Controlada dos Metadados Classificados

## Fortalecimento da consistência textual dos metadados no output avançado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-82-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-metadados%20normalizados-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui o bloco `metadata` do resultado avançado com normalização controlada dos campos textuais já classificados, sem alterar arquitetura ou contrato público.
>
> - remove espaços residuais
> - converte valores vazios em `undefined`
> - preserva a leitura atual de `year`
>
> **Este PR não introduz novos metadados, nova classificação por IA ou redesign do pipeline.**

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

As PRs anteriores consolidaram partes centrais do output avançado, especialmente `answerKey` e `adaptedStatement`. O próximo passo mínimo é fortalecer a consistência do bloco `metadata`, já presente no contrato final e consumido pelo fluxo processado.

A PR 82 trata ruídos textuais residuais após a classificação, como espaços duplicados, strings vazias e pequenas variações de formatação. A mudança melhora a previsibilidade do output sem expandir a fase 2, sem alterar o orchestrator e sem reabrir decisões já aprovadas.

# 2. Objetivo do PR

- normalizar campos textuais de `metadata`
- remover espaços residuais e duplicados
- transformar valores vazios em `undefined`
- preservar `year` sem nova regra de negócio
- manter o contrato público inalterado

# 3. Decisão Arquitetural

A responsabilidade permanece no `ClassificationAgent`, componente já encarregado de interpretar e consolidar os metadados iniciais do fluxo.

A decisão é aplicar a normalização no mesmo ponto em que os metadados são classificados, mantendo a arquitetura vigente e evitando deslocar uma regra simples para novas camadas, helpers prematuros ou orquestração adicional.

# 4. Escopo

- normalização textual de `law`
- normalização textual de `article`
- normalização textual de `bank`
- normalização textual de `position`
- normalização textual de `organization`
- transformação de valores vazios em `undefined`
- cobertura proporcional de testes unitários

# 5. Fora de Escopo

- reclassificação por IA
- novos campos de `metadata`
- enumeração rígida de valores
- validação jurídica
- alteração do contrato público
- redesign do pipeline
- expansão estrutural da fase 2

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
    "curve": "linear",
    "nodeSpacing": 34,
    "rankSpacing": 44
  }
}}%%
flowchart LR
    A["Question Input"] --> B["ClassificationAgent"]
    B --> C["Metadata Classificado"]
    C --> D["Normalização Controlada"]
    D --> E["Output.metadata"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;
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
    class E outputBox;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
```

# 7. Contratos Mínimos

Sem alteração estrutural no output final.

```ts
{
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

A evolução incide apenas na qualidade dos valores retornados em `metadata`, preservando os campos já conhecidos pelo pipeline.

# 8. Regras de Implementação

- manter controller e orchestrator fora do recorte
- concentrar a normalização no `ClassificationAgent`
- evitar novo agent, provider, mapper ou helper prematuro
- não criar nova regra de negócio para `year`
- não preparar próximos passos dentro desta PR
- manter testes proporcionais ao comportamento adicionado

# 9. Critérios de Review

- metadados textuais retornam de forma mais limpa e previsível
- valores vazios são convertidos para `undefined`
- `year` permanece compatível com o fluxo atual
- contrato final permanece inalterado
- recorte continua pequeno e objetivo
- não há overengineering ou granularidade desnecessária

# 10. Critérios de Aceite

- [ ] campos textuais de `metadata` retornam normalizados
- [ ] valores vazios não poluem o output final
- [ ] `year` permanece compatível com o contrato atual
- [ ] testes unitários cobrem a normalização adicionada
- [ ] suíte permanece verde
- [ ] nenhuma regressão é introduzida no fluxo avançado

# 11. Conclusão

A PR 82 mantém a progressão incremental da fase 2 ao fortalecer a consistência dos metadados classificados no output avançado.

Sem ampliar arquitetura, contrato ou responsabilidade sistêmica, o pipeline passa a entregar `metadata` mais limpo e previsível, preservando o recorte pequeno e a continuidade técnica das PRs anteriores.
