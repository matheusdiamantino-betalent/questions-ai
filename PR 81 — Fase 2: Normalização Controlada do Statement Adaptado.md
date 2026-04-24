# 🤖 PR 81 — Fase 2: Normalização Controlada do Statement Adaptado

## Fortalecimento da consistência textual do enunciado final processado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-81-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-statement%20adaptado%20normalizado-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

> [!IMPORTANT]
> Esta PR desloca a continuidade da fase avançada para outro output central do pipeline. Após consolidar o `answerKey` nas PRs 77, 78, 79 e 80, o foco passa a ser a consistência textual do `adaptedStatement`, preservando o recorte incremental e a arquitetura vigente.

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

O pipeline avançado já produz um `adaptedStatement` como parte do resultado processado. Entretanto, a consistência textual desse campo pode ser reforçada no fechamento do fluxo, evitando espaços residuais, quebras desnecessárias e variações de formatação vindas das etapas intermediárias.

A PR 81 consolida uma normalização controlada do statement adaptado, elevando previsibilidade sem introduzir novos agents, novas camadas ou expansão indevida da fase 2.

---

# 2. Objetivo do PR

Fortalecer a qualidade textual do `adaptedStatement` retornado pelo pipeline avançado, garantindo que o enunciado final processado seja entregue de forma limpa, estável e previsível.

Objetivos diretos:

* normalizar o `adaptedStatement` final
* remover espaços duplicados residuais
* garantir trim consistente do enunciado processado
* preservar fallback atual quando não houver adaptação útil
* manter o contrato final já consumido pelo pipeline
* evoluir outro eixo funcional sem repetir o ciclo do `answerKey`

---

# 3. Decisão Arquitetural

A responsabilidade permanece nos componentes já existentes do fluxo avançado. A evolução acontece dentro do processamento atual do statement, sem criação de novos agents, módulos ou camadas.

Não haverá:

* novo agent
* nova camada de orchestration
* redesign do orchestrator
* IA adicional
* validação semântica do enunciado
* expansão estrutural da fase 2

A decisão é reforçar a normalização do `adaptedStatement` no ponto já responsável por sua produção ou consolidação final.

---

# 4. Escopo da PR

## Incluído

* normalização controlada do `adaptedStatement`
* remoção de espaços duplicados residuais
* trim consistente do enunciado final
* preservação do fallback já existente
* atualização proporcional dos testes unitários relacionados
* preservação integral do contrato público

## Fora de Escopo

* reescrita semântica do enunciado
* geração adicional por IA
* validação jurídica do statement
* novos agents
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
    A[Input.question.statement] --> B[StatementAdaptationAgent]
    B --> C[Adapted Statement Candidate]
    C --> D[Normalize Adapted Statement]
    D --> E[Output.adaptedStatement]
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

A evolução ocorre na qualidade textual do `adaptedStatement`, não na expansão do contrato público.

---

# 7. Estratégia de Implementação

Ordem recomendada:

1. `statement-adaptation.agent.ts`
2. `statement-adaptation.agent.spec.ts`
3. `initial-question-processing.agent.ts` (apenas se necessário)
4. `initial-question-processing.agent.spec.ts`
5. validação do contrato final
6. regressão da suíte completa

Princípio central:

> melhorar a consistência do enunciado final sem ampliar complexidade sistêmica.

---

# 8. Critérios de Review

Validar se:

* o `adaptedStatement` ficou mais limpo e previsível
* houve normalização consistente de espaços e quebras
* o fallback existente foi preservado
* o contrato final permaneceu inalterado
* o recorte permaneceu pequeno
* não houve overengineering

---

# 9. Critérios de Aceite

* `adaptedStatement` retorna normalizado
* espaços duplicados e quebras residuais são removidos
* fallback atual permanece compatível
* testes permanecem verdes
* nenhuma regressão no fluxo avançado

---

# 10. Impacto Esperado

Maior previsibilidade textual no resultado avançado, com enunciado final mais estável e adequado para consumo downstream.

O sistema evolui além do `answerKey`, fortalecendo outro campo central do output processado.

---

# 11. Conclusão

A PR 81 dá continuidade à fase avançada em um eixo diferente do ciclo anterior: a qualidade textual do statement adaptado.

Sem alterar arquitetura ou contrato público, o pipeline passa a entregar um `adaptedStatement` mais limpo, consistente e confiável.
