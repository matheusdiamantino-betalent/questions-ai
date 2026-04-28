# 🧾 PR 99 — Fase 2: Handoff Explícito para Geração do Gabarito

## Extração controlada do payload enviado ao AnswerKeyAgent

---

<div align="left">

![PR](https://img.shields.io/badge/PR-99-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-refactor%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-answerKey%20handoff-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR dá continuidade à fase 2 após o ajuste de resolução de IDs, agora focando clareza interna do fluxo entre processamento inicial e geração do gabarito.
>
> - explicita o handoff para o `AnswerKeyAgent`
> - centraliza a montagem do payload em ponto único
> - preserva comportamento funcional atual
>
> **Este PR não altera regras de negócio, não muda contrato público, não cria novos agents e não redesenha o pipeline.**

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

A PR 98 evoluiu a resolução de IDs com fallback aproximado sem expandir arquitetura. A PR 99 avança em outro ponto interno do fluxo: tornar explícita a transição entre o resultado do processamento inicial e a geração do gabarito final.

O objetivo é reduzir ruído no método `execute` do orchestrator, concentrando a composição do payload enviado ao `AnswerKeyAgent` em um único método privado, sem qualquer alteração funcional.

# 2. Objetivo do PR

- extrair a montagem do input do `AnswerKeyAgent`
- explicitar o handoff entre etapas do fluxo
- melhorar legibilidade do orchestrator
- preservar contratos atuais
- manter recorte pequeno e revisável

# 3. Decisão Arquitetural

A responsabilidade permanece no `AgentsFlowOrchestratorService`, que já coordena a sequência dos agents e o intercâmbio entre resultados parciais. A PR apenas organiza essa responsabilidade em uma fronteira interna mais clara.

Não há novo service, helper global ou mudança de ownership. O orchestrator continua sendo o ponto de composição do fluxo avançado.

# 4. Escopo

- criar método privado para montar payload do `AnswerKeyAgent`
- reutilizar dados já produzidos no fluxo atual
- simplificar o corpo do método `execute`
- ajustar testes para refletir a extração
- manter retorno final inalterado

# 5. Fora de Escopo

- mudança de prompt
- alteração de regras do gabarito
- novo agent
- novo service
- helper compartilhado
- mudança de contrato público
- persistência
- redesign do pipeline
- alteração na ordem de execução dos agents

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
    A["InitialQuestionProcessingResult"] --> B["buildAnswerKeyInput()"]
    B --> C["AnswerKeyAgent"]
    C --> D["Final Output"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef foundationBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef outputBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B foundationBox;
    class C step6;
    class D outputBox;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
```

# 7. Contratos Mínimos

Payload interno preservado:

```ts
{
  adaptedStatement,
  comments,
  correctAnswer,
  fallbackCorrectAlternative,
  justification,
  fallbackJustification,
  source,
  fallbackSource
}
```

Nenhum campo novo é introduzido. Nenhum campo atual é removido.

# 8. Regras de Implementação

- método privado pequeno e coeso
- zero mudança funcional
- manter nomes de campos atuais
- não duplicar regras
- não mover responsabilidade para outro módulo
- manter `execute` mais legível
- preservar fluxo principal

# 9. Critérios de Review

- handoff ficou explícito
- `execute` ficou mais limpo
- payload enviado ao `AnswerKeyAgent` permanece igual
- ordem de execução do fluxo permanece igual
- testes cobrem a extração sem regressão
- recorte continua pequeno e proporcional

# 10. Critérios de Aceite

- [ ] payload do `AnswerKeyAgent` é montado em método dedicado
- [ ] comportamento funcional permanece igual
- [ ] retorno final permanece igual
- [ ] ordem dos agents permanece igual
- [ ] suíte permanece verde
- [ ] nenhum novo acoplamento foi criado

# 11. Conclusão

A PR 99 mantém a progressão incremental da fase 2 ao melhorar a clareza interna do orchestrator sem alterar comportamento. O fluxo continua o mesmo, mas a fronteira entre processamento inicial e geração do gabarito passa a estar explicitamente representada e mais fácil de revisar.
