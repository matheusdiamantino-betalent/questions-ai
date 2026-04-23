# 🤖 PR 71 — Fase 2: Primeiro Encadeamento Funcional da Resposta Final
## Consumo controlado do statement adaptado na composição subsequente do answer key

---

<div align="left">

![PR](https://img.shields.io/badge/PR-71-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-primeiro%20encadeamento%20funcional%20da%20resposta%20final-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR dá continuidade direta ao encadeamento funcional iniciado na PR 70, fazendo a composição do `answerKey` passar a consumir o `adaptedStatement` já refinado no fluxo orquestrado.
>
> - conecta a geração da resposta final ao statement já adaptado
> - preserva a arquitetura atual e os mesmos agents da fase
> - limita a mudança a composição mínima do `answerKey`, ajustes locais de processamento e testes proporcionais
>
> **Este PR não introduz novos agents, não redesenha o fluxo e não expande a fase além do encadeamento funcional já em andamento.**

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

A PR 67 abriu a frente dos agents avançados. A PR 68 consolidou funcionalmente o primeiro slice avançado. A PR 69 integrou esse ganho ao pipeline principal. A PR 70 fez o `StatementAdaptationAgent` passar a consumir, de forma explícita, o contexto já refinado. Com esse encadeamento estabilizado, o próximo passo mínimo correto é fazer a composição da resposta final deixar de operar apenas sobre o resultado anterior e passar a consumir também o statement já adaptado.

Esta PR segue exatamente nessa direção. O recorte continua pequeno e controlado: o `AnswerKeyAgent` passa a considerar o `adaptedStatement` já refinado, o `InitialQuestionProcessingAgent` é alinhado a essa composição e o `AgentsFlowOrchestratorService` consolida o novo encadeamento sem reabrir arquitetura, sem introduzir novos componentes e sem antecipar a próxima frente da fase.

## 2. Objetivo do PR

- fazer o `AnswerKeyAgent` consumir o `adaptedStatement` já refinado no fluxo
- alinhar a composição do `answerKey` ao encadeamento funcional introduzido na PR 70
- ajustar o `InitialQuestionProcessingAgent` para o novo shape mínimo de composição
- consolidar no orchestrator o fluxo com `statement adaptation` seguido de `answer key`
- validar o comportamento com testes proporcionais ao recorte

## 3. Decisão Arquitetural

A decisão central desta PR é preservar a arquitetura aprovada e continuar a fase 2 por encadeamentos funcionais pequenos e sequenciais. Depois de integrar o contexto refinado e fazê-lo participar da adaptação do statement, o passo mais coerente é fazer a resposta final consumir esse insumo já melhorado, em vez de abrir nova frente paralela ou ampliar o fluxo com múltiplas decisões ao mesmo tempo.

Com isso, a evolução permanece controlada. O `AnswerKeyAgent` não muda de responsabilidade; ele apenas passa a compor a justificativa final a partir de um input mais maduro. O `InitialQuestionProcessingAgent` continua pequeno, agora alinhado ao novo encadeamento, e o orchestrator segue como coordenador simples do fluxo existente.

## 4. Escopo

- ajustar o `AnswerKeyAgent` para consumir o `adaptedStatement`
- alinhar o `InitialQuestionProcessingAgent` ao novo encadeamento mínimo do `answerKey`
- atualizar o `AgentsFlowOrchestratorService` para consolidar a sequência com `statement adaptation` e `answer key`
- revisar os testes do `AnswerKeyAgent`, do `InitialQuestionProcessingAgent` e do orchestrator
- preservar o contrato público final do fluxo sem ampliar o shape exposto

## 5. Fora de Escopo

- introdução de novos agents além dos já existentes na fase
- redesign do orchestrator ou redistribuição estrutural ampla de responsabilidades
- múltiplos novos pontos de composição da resposta final
- expansão de contratos públicos sem necessidade imediata
- registry dinâmico, paralelismo, retry, observabilidade ampliada ou fallback expandido
- abertura de nova frente funcional fora do encadeamento já em curso

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
    D --> E["StatementAdaptationAgent"]
    E --> F["AnswerKeyAgent"]
    F --> G["Resposta final mais contextualizada"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef step7 fill:#10243a,stroke:#38bdf8,stroke-width:2px,color:#ffffff;
    classDef step8 fill:#221b2f,stroke:#f472b6,stroke-width:2px,color:#ffffff;
    classDef step9 fill:#14281d,stroke:#34d399,stroke-width:2px,color:#ffffff;
    classDef step10 fill:#2a160f,stroke:#fb7185,stroke-width:2px,color:#ffffff;
    classDef step11 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;
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

O diagrama permanece proporcional ao recorte. A novidade não é uma nova estrutura de pipeline, mas a continuação do encadeamento funcional já iniciado: o `adaptedStatement` passa a participar explicitamente da composição do `answerKey`.

## 7. Contratos Mínimos

Os contratos públicos finais permanecem os mesmos. O fluxo continua retornando `legalSearch`, `adaptedStatement`, `answerKey`, `metadata` e `ids`, sem ampliar o shape exposto nem introduzir envelopes adicionais. A mudança desta PR está concentrada na composição interna do `answerKey`, que passa a considerar o `adaptedStatement` como insumo de justificativa, preservando o contrato já consolidado do projeto.

Essa escolha protege o recorte e evita expansão desnecessária. O ganho desta PR está no encadeamento funcional, não em remodelagem de tipos compartilhados.

## 8. Regras de Implementação

- manter o `AnswerKeyAgent` pequeno e determinístico, sem ampliar sua responsabilidade além da composição do `answerKey`
- concentrar a mudança no consumo do `adaptedStatement` e na consolidação mínima do fluxo
- preservar o `AgentsFlowOrchestratorService` como coordenador simples da sequência atual
- evitar introdução de contratos futuros, abstrações extras ou múltiplas composições paralelas da resposta
- manter os testes focados na nova composição e na propagação de falhas do fluxo

## 9. Critérios de Review

- a PR mantém continuidade direta com a trilha 67 → 68 → 69 → 70
- o `AnswerKeyAgent` passou a consumir o `adaptedStatement` refinado
- o `InitialQuestionProcessingAgent` foi alinhado ao novo encadeamento sem inflar escopo
- o orchestrator consolida corretamente a sequência `statement adaptation` → `answer key`
- o contrato público final do fluxo permanece estável
- não houve redesign, novo agent ou expansão indevida da fase

## 10. Critérios de Aceite

- [ ] o `AnswerKeyAgent` consome o `adaptedStatement` no fluxo atualizado
- [ ] o `InitialQuestionProcessingAgent` permanece funcional e coerente com o novo encadeamento
- [ ] o `AgentsFlowOrchestratorService` consolida corretamente o `answerKey` final após a adaptação do statement
- [ ] os testes do `AnswerKeyAgent`, do `InitialQuestionProcessingAgent` e do orchestrator permanecem verdes
- [ ] o contrato público final do fluxo não foi ampliado sem necessidade

## 11. Conclusão

A PR 71 mantém a fase 2 na direção correta ao estender, de forma mínima e controlada, o encadeamento funcional já iniciado na PR 70. Depois de fazer o statement refinado participar do fluxo, o próximo passo coerente era fazê-lo chegar à composição da resposta final. O resultado é um avanço pequeno, revisável e aderente ao padrão do projeto: continuidade clara, escopo protegido, arquitetura preservada e nenhum ruído estrutural além do necessário.
