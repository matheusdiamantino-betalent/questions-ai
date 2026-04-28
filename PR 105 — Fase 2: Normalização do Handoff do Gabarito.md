# 🧩 PR 105 — Fase 2: Normalização do Handoff do Gabarito

## Consolidação e previsibilidade do payload enviado ao Answer Key Agent

---

<div align="left">

![PR](https://img.shields.io/badge/PR-105-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-answer%20key%20handoff-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20implementação-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui a robustez interna do fluxo de IA após os guardrails de entrada da PR 104, normalizando o payload entregue pelo `AgentsFlowOrchestratorService` ao `AnswerKeyAgent`.
>
> - Consolida o handoff final do gabarito.
> - Remove ruído de strings vazias.
> - Preserva a precedência entre resposta explícita e fallback.
>
> **Este PR não altera o endpoint HTTP, não muda o contrato final de resposta e não introduz nova fundação arquitetural.**

---

# 📚 Sumário

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

---

# 1. Síntese Executiva

A PR 104 fortaleceu a entrada pública do endpoint de processamento de questões com validação estrutural e normalização mínima. A PR 105 avança no próximo boundary natural do fluxo: o handoff interno para geração do gabarito final.

O objetivo agora é garantir que o `AnswerKeyAgent` receba um payload coerente, normalizado e previsível, sem transferir para o agente a responsabilidade de higienizar campos produzidos pela orquestração. A mudança permanece local ao `AgentsFlowOrchestratorService`, preservando o desenho aprovado e evitando expansão arquitetural.

---

# 2. Objetivo do PR

Esta PR normaliza os campos usados na composição do payload enviado ao `AnswerKeyAgent`, com foco em remover inconsistências simples antes da execução final do gabarito.

O recorte cobre:

- normalização de `correctAnswer`;
- normalização de `fallbackCorrectAlternative`;
- normalização de `fallbackJustification`;
- normalização de `fallbackSource`;
- conversão de strings vazias para `undefined`;
- preservação da prioridade entre resposta explícita e fallback.

---

# 3. Decisão Arquitetural

A normalização permanece no `AgentsFlowOrchestratorService` porque este é o ponto de composição entre os resultados intermediários dos agentes e o handoff para o `AnswerKeyAgent`.

Essa decisão mantém o fluxo coeso: os agentes produzem seus resultados, o orchestrator consolida o payload final e o `AnswerKeyAgent` recebe dados já preparados para sua responsabilidade específica. Não há criação de helper global, adapter dedicado, schema paralelo ou refactor estrutural.

---

# 4. Escopo

Entra nesta PR:

- ajuste local na montagem do payload do `AnswerKeyAgent`;
- trim de campos textuais úteis;
- descarte de strings vazias sem valor semântico;
- preservação do contrato interno existente;
- cobertura de testes unitários no orchestrator para o handoff final.

---

# 5. Fora de Escopo

Não entra nesta PR:

- alteração do controller HTTP;
- mudança em validações de entrada pública;
- mudança no `AnswerKeyAgent`;
- alteração do contrato final de resposta;
- novo endpoint;
- persistência;
- cache, retries ou observabilidade;
- reestruturação de módulos;
- novas regras semânticas de escolha de gabarito.

---

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
    A["Resultados intermediários<br/>dos agentes"] --> B["AgentsFlowOrchestratorService<br/>compõe handoff"]
    B --> C["Normalização local<br/>trim e descarte de vazios"]
    C --> D["AnswerKeyAgent<br/>recebe payload previsível"]
    D --> E["Output final<br/>do processamento"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef outputBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

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

---

# 7. Contratos Mínimos

Contrato interno envolvido no handoff:

```ts
{
  correctAnswer?: 'A' | 'B' | 'C' | 'D' | 'E';
  fallbackCorrectAlternative?: 'A' | 'B' | 'C' | 'D' | 'E';
  fallbackJustification?: string;
  fallbackSource?: string;
}
```

Regras mínimas aplicadas:

- campos textuais são normalizados com `trim()`;
- strings vazias são convertidas para `undefined`;
- `correctAnswer` explícito mantém precedência quando disponível;
- fallback permanece como apoio quando necessário;
- o contrato final de resposta não é alterado.

---

# 8. Regras de Implementação

A implementação deve ficar concentrada no `AgentsFlowOrchestratorService`, no ponto exato em que o payload do `AnswerKeyAgent` é montado. O ajuste deve ser simples, legível e local, sem distribuir normalização entre múltiplos agentes.

O `AnswerKeyAgent` não deve receber responsabilidade adicional de saneamento do handoff. A PR também não deve introduzir abstrações globais para normalização, já que o recorte atual é pequeno e específico.

---

# 9. Critérios de Review

Validar se a PR:

- mantém a mudança concentrada no orchestrator;
- preserva a responsabilidade do `AnswerKeyAgent`;
- deixa explícita a composição do handoff;
- não altera contrato externo;
- não cria abstração prematura;
- cobre cenários relevantes em teste;
- preserva a precedência entre resposta explícita e fallback;
- mantém o recorte proporcional à fase.

---

# 10. Critérios de Aceite

- [ ] Campos textuais enviados ao `AnswerKeyAgent` são normalizados.
- [ ] Strings vazias não são propagadas como valores úteis.
- [ ] `correctAnswer` explícito prevalece quando disponível.
- [ ] `fallbackCorrectAlternative` permanece disponível quando necessário.
- [ ] `fallbackJustification` vazio vira `undefined`.
- [ ] `fallbackSource` vazio vira `undefined`.
- [ ] O contrato final de resposta permanece inalterado.
- [ ] A suíte de testes permanece verde.

---

# 11. Conclusão

A PR 105 fortalece o handoff interno para geração do gabarito final sem redesenhar o fluxo de agentes. É uma evolução pequena e coesa após a PR 104: primeiro a entrada pública foi protegida; agora o payload interno mais sensível do fluxo passa a chegar ao `AnswerKeyAgent` com maior previsibilidade.
