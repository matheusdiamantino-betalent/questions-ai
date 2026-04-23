# 🤖 PR 72 — Fase 2: Consolidação da Composição Final do Answer Key
## Centralização controlada da responsabilidade da resposta final no fluxo orquestrado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-72-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-centralizacao%20do%20answer%20key%20final-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR dá continuidade direta à PR 71, removendo a duplicidade de composição do `answerKey` e consolidando a responsabilidade final em um único ponto do fluxo orquestrado.
>
> - simplifica o `InitialQuestionProcessingAgent`
> - centraliza a composição final no `AnswerKeyAgent`
> - preserva a arquitetura atual com ajustes mínimos e testes proporcionais
>
> **Este PR não cria novos agents, não redesenha o pipeline e não expande a fase além da consolidação funcional necessária.**

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

A PR 70 fez o `StatementAdaptationAgent` participar explicitamente do fluxo orquestrado. A PR 71 conectou esse resultado refinado à composição do `answerKey`. Com essa evolução concluída, surgiu um ponto claro de melhoria: a composição final da resposta ainda permanece distribuída entre etapas diferentes.

O próximo passo mínimo correto é consolidar essa responsabilidade. Esta PR simplifica o `InitialQuestionProcessingAgent`, preservando nele apenas o processamento intermediário necessário, e deixa a composição final do `answerKey` centralizada no ponto final do fluxo. O ganho é clareza operacional, menor sobreposição de responsabilidade e manutenção da progressão incremental da fase.

## 2. Objetivo do PR

- remover duplicidade de composição do `answerKey`
- simplificar o `InitialQuestionProcessingAgent`
- centralizar a composição final no `AnswerKeyAgent`
- manter o `AgentsFlowOrchestratorService` como coordenador do fechamento do fluxo
- atualizar testes proporcionais ao novo limite de responsabilidade

## 3. Decisão Arquitetural

A decisão central desta PR é consolidar responsabilidades antes de abrir novas frentes funcionais. Em vez de adicionar novos comportamentos sobre um fluxo ainda com composição distribuída, o recorte corrige a fronteira natural entre processamento intermediário e resposta final.

O `InitialQuestionProcessingAgent` continua responsável por produzir insumos do fluxo, como contexto legal e statement adaptado. O `AnswerKeyAgent` passa a ser o único responsável pela composição final do `answerKey`. O orchestrator permanece simples: recebe os insumos e coordena a etapa final sem assumir lógica de negócio indevida.

## 4. Escopo

- remover do `InitialQuestionProcessingAgent` a composição final do `answerKey`
- manter nesse agent apenas resultados intermediários necessários
- consolidar no `AnswerKeyAgent` a composição final da resposta
- ajustar o `AgentsFlowOrchestratorService` para o novo fechamento do fluxo
- revisar testes do processing agent e do orchestrator

## 5. Fora de Escopo

- novos agents
- redesign estrutural amplo
- alteração do pipeline macro
- múltiplos answer keys condicionais
- expansão de contratos públicos sem necessidade
- retry, paralelismo, observabilidade ampliada ou registry dinâmico
- nova frente funcional além da consolidação atual

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
    A["Questão extraída"] --> B["ClassificationAgent"]
    B --> C["IdResolutionAgent"]
    C --> D["InitialQuestionProcessingAgent"]
    D --> E["Insumos intermediários"]
    E --> F["AnswerKeyAgent"]
    F --> G["Answer key final único"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef step7 fill:#10243a,stroke:#38bdf8,stroke-width:2px,color:#ffffff;

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

O contrato público final permanece estável. O fluxo continua retornando `legalSearch`, `adaptedStatement`, `answerKey`, `metadata` e `ids`.

A mudança desta PR é interna: o `InitialQuestionProcessingAgent` deixa de produzir a composição final do `answerKey`, retornando apenas insumos necessários para a etapa seguinte. A montagem final fica centralizada no `AnswerKeyAgent`, sem ampliar o shape externo já aprovado.

## 8. Regras de Implementação

- manter o `InitialQuestionProcessingAgent` focado em processamento intermediário
- manter o `AnswerKeyAgent` como único responsável pela resposta final
- preservar o orchestrator como coordenador simples
- evitar novos contratos ou abstrações não exigidas pelo recorte
- manter testes objetivos sobre fronteiras de responsabilidade

## 9. Critérios de Review

- a duplicidade de composição do `answerKey` foi removida
- o `InitialQuestionProcessingAgent` ficou mais simples e coeso
- o `AnswerKeyAgent` tornou-se o único ponto de composição final
- o orchestrator continua linear e legível
- o contrato público final permaneceu estável
- não houve expansão indevida da fase

## 10. Critérios de Aceite

- [ ] o `InitialQuestionProcessingAgent` não compõe mais o `answerKey` final
- [ ] o `AnswerKeyAgent` é o único ponto de composição final da resposta
- [ ] o `AgentsFlowOrchestratorService` fecha corretamente o fluxo atualizado
- [ ] testes do processing agent e orchestrator permanecem verdes
- [ ] nenhum novo agent ou redesign estrutural foi introduzido

## 11. Conclusão

A PR 72 representa o próximo passo maduro após a PR 71: antes de seguir adicionando comportamento, o fluxo é consolidado em suas responsabilidades naturais. A resposta final passa a ter um único ponto de composição, o processamento intermediário fica mais claro e a fase continua evoluindo com simplicidade, previsibilidade e controle de escopo.
