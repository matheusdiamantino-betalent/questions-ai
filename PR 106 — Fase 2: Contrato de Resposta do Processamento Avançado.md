# 📦 PR 106 — Fase 2: Contrato de Resposta do Processamento Avançado

## Consolidação explícita do output HTTP do endpoint de processamento de questões

---

<div align="left">

![PR](https://img.shields.io/badge/PR-106-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-response%20contract-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20implementação-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR consolida o contrato de saída HTTP do endpoint `POST /ai/questions/process`, tornando explícita a estrutura retornada pelo fluxo avançado.
>
> - Melhora previsibilidade para consumidores da API.
> - Reduz ambiguidade de integração.
> - Mantém o desenho arquitetural atual.
>
> **Este PR não altera regras internas dos agentes, não muda input e não cria novas camadas.**

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

A sequência recente fortaleceu dois pontos centrais do fluxo: entrada pública (PR 104) e handoff interno para geração de gabarito (PR 105). O próximo passo mínimo natural é explicitar o contrato público de saída do processamento avançado.

Esta PR formaliza a estrutura HTTP retornada pelo endpoint, alinhando tipagem, previsibilidade e integração externa. O comportamento funcional do pipeline é preservado; a mudança foca clareza contratual.

---

# 2. Objetivo do PR

Garantir que o endpoint exponha de forma explícita e consistente os principais artefatos produzidos pelo fluxo avançado.

O recorte cobre:

- formalização do retorno HTTP tipado;
- exposição consistente de `metadata`;
- exposição consistente de `ids`;
- exposição de `idResolutionConfidence`;
- exposição de `answerKey`;
- exposição de `adaptedStatement`;
- exposição de `legalSearch`;
- cobertura automatizada do contrato de resposta.

---

# 3. Decisão Arquitetural

A composição da resposta continua no `AgentsFlowOrchestratorService`, que já centraliza o fluxo e consolida os outputs intermediários. O controller permanece apenas como boundary HTTP, delegando execução e retornando o resultado.

A decisão evita presenters, adapters ou camadas artificiais para um recorte que depende apenas de explicitação contratual e tipagem clara.

---

# 4. Escopo

Entra nesta PR:

- tipagem explícita do response contract;
- alinhamento do retorno do controller ao output consolidado;
- validação do contrato no spec do controller;
- verificação dos campos centrais no fluxo feliz;
- documentação técnica proporcional ao slice.

---

# 5. Fora de Escopo

Não entra nesta PR:

- alteração de lógica dos agentes;
- mudança de persistência;
- novo endpoint;
- versionamento de API;
- swagger expandido;
- mudança de input;
- refactor estrutural;
- nova camada de apresentação;
- alteração semântica do pipeline.

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
    A["HTTP Request"] --> B["Controller<br/>delegação fina"]
    B --> C["AgentsFlowOrchestratorService<br/>compõe resultado"]
    C --> D["Response Contract<br/>estrutura explícita"]
    D --> E["HTTP Response"]

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

Resposta esperada:

```json
{
  "legalSearch": {},
  "adaptedStatement": "string",
  "answerKey": {
    "correctAlternative": "A",
    "justification": "string",
    "source": "string"
  },
  "metadata": {},
  "ids": {},
  "idResolutionConfidence": {}
}
```

Garantias mínimas:

- estrutura previsível e estável;
- campos centrais explicitamente acessíveis;
- retorno alinhado ao output consolidado interno;
- controller sem transformação paralela.

---

# 8. Regras de Implementação

A implementação deve priorizar clareza contratual, não redesign. O controller deve permanecer fino e o orchestrator continua dono da composição do resultado final.

Qualquer ajuste deve ser local e proporcional: alinhar tipos, retornar estrutura consolidada e validar via testes. Não introduzir wrappers, presenters ou abstrações de resposta sem necessidade concreta.

---

# 9. Critérios de Review

Validar se a PR:

- mantém controller fino;
- explicita corretamente o contrato de saída;
- preserva responsabilidade do orchestrator;
- expõe campos relevantes esperados;
- não cria expansão arquitetural;
- possui testes aderentes ao slice;
- melhora legibilidade de integração externa;
- mantém proporcionalidade entre entrega e documentação.

---

# 10. Critérios de Aceite

- [ ] Endpoint retorna a estrutura documentada.
- [ ] Response contém `answerKey`.
- [ ] Response contém `metadata`.
- [ ] Response contém `ids`.
- [ ] Response contém `idResolutionConfidence`.
- [ ] Response contém `adaptedStatement`.
- [ ] Response contém `legalSearch`.
- [ ] Tipagem esperada permanece válida.
- [ ] Testes permanecem verdes.
- [ ] Não há regressão funcional no fluxo atual.

---

# 11. Conclusão

A PR 106 fecha o ciclo natural após proteção da entrada e normalização do handoff interno: tornar explícito o contrato público de saída do processamento avançado. É uma evolução pequena, útil e revisável, mantendo o padrão incremental e sem inflar a arquitetura.
