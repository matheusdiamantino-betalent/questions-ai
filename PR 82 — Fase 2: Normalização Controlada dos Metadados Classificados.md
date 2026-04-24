# 🤖 PR 82 — Fase 2: Normalização Controlada dos Metadados Classificados

## Fortalecimento da consistência textual dos metadados no output avançado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-82-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-metadados%20normalizados-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

> [!IMPORTANT]
> Esta PR dá continuidade à fase avançada em um novo eixo funcional. Após consolidar `answerKey` e `adaptedStatement`, o foco evolutivo passa a ser a consistência textual dos metadados classificados, preservando o recorte incremental e a arquitetura vigente.

---

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

O pipeline avançado já produz `metadata` como parte do output final. Entretanto, campos textuais classificados ainda podem carregar espaços residuais, valores vazios normalizados de forma inconsistente ou pequenas variações de formatação.

A PR 82 fortalece a previsibilidade dos metadados retornados ao consolidar uma normalização controlada dos campos classificados.

---

# 2. Objetivo do PR

Garantir que os metadados do fluxo avançado sejam retornados de forma mais limpa, estável e consistente, sem alterar o contrato público já consumido pelo pipeline.

Objetivos diretos:

* normalizar campos textuais de `metadata`
* remover espaços duplicados residuais
* transformar valores vazios em `undefined`
* preservar `year` sem nova regra de negócio
* reforçar previsibilidade do output final
* evoluir outro eixo funcional sem repetir ciclos anteriores

---

# 3. Decisão Arquitetural

A responsabilidade permanece no `ClassificationAgent`, componente já responsável por interpretar e consolidar os metadados iniciais do fluxo.

Não haverá:

* novo agent
* nova camada de orchestration
* redesign do orchestrator
* IA adicional
* novos campos de metadata
* expansão estrutural da fase 2

A decisão é fortalecer a normalização no mesmo componente já responsável pela classificação.

---

# 4. Escopo da PR

## Incluído

* normalização textual de `law`
* normalização textual de `article`
* normalização textual de `bank`
* normalização textual de `position`
* normalização textual de `organization`
* transformação de valores vazios em `undefined`
* cobertura proporcional de testes unitários
* preservação integral do contrato público

## Fora de Escopo

* reclassificação por IA
* novos campos de metadata
* enumeração rígida de valores
* validação jurídica
* redesign do pipeline
* expansão indevida da fase 2

---

# 5. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#0f172a",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#38bdf8",
    "lineColor": "#64748b",
    "secondaryColor": "#111827",
    "tertiaryColor": "#1e293b",
    "background": "#020617"
  }
}}%%
flowchart LR
    A[Question Input] --> B[ClassificationAgent]
    B --> C[Raw Metadata]
    C --> D[Normalize Metadata]
    D --> E[Output.metadata]
```

---

# 6. Contratos Mínimos

Sem alteração estrutural no output final.

```ts
{
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

A evolução ocorre na qualidade dos valores retornados em `metadata`, não na expansão do contrato público.

---

# 7. Estratégia de Implementação

Ordem recomendada:

1. `classification.agent.ts`
2. `classification.agents.spec.ts`
3. regressão da suíte completa

Princípio central:

> elevar a consistência dos metadados sem ampliar complexidade sistêmica.

---

# 8. Critérios de Review

Validar se:

* os metadados ficaram mais limpos e previsíveis
* valores vazios passam a ser `undefined`
* `year` permaneceu íntegro
* o contrato final permaneceu inalterado
* o recorte permaneceu pequeno
* não houve overengineering

---

# 9. Critérios de Aceite

* campos textuais retornam normalizados
* campos vazios não poluem o output
* `year` permanece compatível
* testes permanecem verdes
* nenhuma regressão no fluxo avançado

---

# 10. Impacto Esperado

Maior consistência prática no bloco de metadados do output avançado, reduzindo ruído estrutural para consumidores downstream.

O sistema evolui além de `answerKey` e `adaptedStatement`, fortalecendo outro eixo central do resultado processado.

---

# 11. Conclusão

A PR 82 mantém a progressão madura da fase avançada ao atacar outro componente relevante do output final: os metadados classificados.

Sem alterar arquitetura ou contrato público, o pipeline passa a entregar `metadata` mais limpo, consistente e confiável.
