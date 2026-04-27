# 🤖 PR 88 — Fase 2: Normalização Mínima do Input do Fluxo Avançado
## Higienização básica do payload válido antes da execução dos agents

---

<div align="left">

![PR](https://img.shields.io/badge/PR-88-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-normalizacao%20input-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR adiciona uma normalização mínima do input já válido antes da execução do fluxo avançado.
>
> - higieniza `statement` e alternativas com `trim`
> - remove alternativas vazias após normalização
> - preserva contrato, output e desenho atual do pipeline
>
> **Este PR não introduz validator global, pipe customizado, schema externo, novo agent ou redesign do fluxo avançado.**

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

---

# 1. Síntese Executiva

A PR 87 consolidou a proteção mínima da fronteira do fluxo avançado, garantindo que apenas payloads compatíveis sigam para a execução dos agents. A PR 88 mantém essa decisão e adiciona o próximo passo mínimo: normalizar o input válido antes da orquestração.

O objetivo não é ampliar validação nem sofisticar o pipeline. O recorte se limita a reduzir ruído estrutural simples em campos textuais, preservando o contrato atual e mantendo a execução dos agents sobre uma entrada mais previsível.

Essa evolução continua a linha incremental da Fase 2: primeiro proteger a entrada, depois higienizar o que já foi aceito como válido, sem reabrir arquitetura ou deslocar regra simples para camadas prematuras.

---

# 2. Objetivo do PR

Esta PR tem como objetivo aplicar uma normalização mínima no payload válido recebido pelo fluxo avançado, antes da execução dos agents.

Na prática, o PR deve:

- aplicar `trim` em `question.statement`;
- aplicar `trim` nos textos das alternativas;
- remover alternativas vazias após a normalização;
- entregar aos agents um input textual mais consistente;
- preservar o comportamento atual em cenários válidos.

---

# 3. Decisão Arquitetural

A decisão arquitetural é manter a normalização dentro do `AgentsFlowOrchestratorService`, no ponto de entrada do fluxo avançado.

Esse posicionamento evita espalhar uma regra simples para pipes, validators globais, schemas externos ou helpers prematuros. A normalização pertence ao limite operacional do fluxo avançado porque atua sobre o payload já aceito e prepara apenas a entrada que será consumida pelos agents.

A arquitetura aprovada permanece inalterada. O PR não adiciona nova camada, não cria foundation paralela e não redefine o contrato do pipeline.

---

# 4. Escopo

Entram neste PR apenas as mudanças necessárias para higienizar o input textual válido antes da execução dos agents:

- normalização de `question.statement`;
- normalização dos textos das alternativas;
- remoção de alternativas vazias após `trim`;
- aplicação da normalização antes da execução do fluxo avançado;
- testes mínimos cobrindo o comportamento de normalização;
- preservação do output final já existente.

---

# 5. Fora de Escopo

Ficam explicitamente fora deste PR:

- deduplicação de alternativas;
- validação semântica da questão;
- correção gramatical;
- reordenação de alternativas;
- enriquecimento de payload;
- alteração do contrato de resposta;
- criação de validator global;
- criação de pipe customizado;
- criação de schema externo;
- novo agent;
- redesign do fluxo avançado.

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
    A["Payload válido"] --> B["Normalização mínima"]
    B --> C["Input higienizado"]
    C --> D["Fluxo avançado"]
    D --> E["Agents"]
    E --> F["Output existente"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef outputBox fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F outputBox;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
    linkStyle 4 stroke:#9ca3af,stroke-width:2px;
```

---

# 7. Contratos Mínimos

Não há alteração estrutural no contrato de saída do fluxo avançado.

O output permanece no formato já esperado:

```ts
{
  legalSearch,
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

A mudança ocorre apenas no input interno entregue aos agents. O payload válido é higienizado antes da execução, mas o contrato externo do fluxo não é expandido.

---

# 8. Regras de Implementação

A implementação deve manter a normalização simples, explícita e localizada no `AgentsFlowOrchestratorService`.

A regra central é preparar o input antes de qualquer agent ser executado, sem deslocar a responsabilidade para uma abstração maior do que o recorte exige. O código deve permanecer fácil de revisar, sem helper global, sem schema externo, sem pipe customizado e sem acoplamento com etapas futuras.

O fluxo de sucesso existente deve ser preservado. A normalização não deve introduzir comportamento novo além da higienização textual mínima definida no escopo.

---

# 9. Critérios de Review

O review deve validar se:

- `statement` recebe `trim` corretamente;
- alternativas recebem `trim` corretamente;
- alternativas vazias após normalização são removidas;
- a normalização ocorre antes da execução dos agents;
- o output de sucesso permanece inalterado;
- o recorte continua pequeno e aderente à Fase 2;
- não houve criação de abstração prematura;
- não houve redesign do fluxo avançado;
- a PR preserva o padrão visual e textual das PRs anteriores.

---

# 10. Critérios de Aceite

- [ ] `question.statement` é normalizado antes da execução dos agents.
- [ ] Textos das alternativas são normalizados antes da execução dos agents.
- [ ] Alternativas vazias após `trim` são removidas.
- [ ] O fluxo válido preserva o comportamento funcional atual.
- [ ] O output final permanece estruturalmente inalterado.
- [ ] Não há validator global, pipe customizado ou schema externo novo.
- [ ] Testes mínimos cobrem a normalização prevista.
- [ ] A suíte permanece verde.

---

# 11. Conclusão

A PR 88 melhora a previsibilidade do fluxo avançado sem ampliar sua arquitetura.

O slice mantém a sequência natural da Fase 2: após proteger a entrada, normaliza o payload válido no ponto de orquestração. A entrega permanece pequena, revisável e limitada à higienização mínima necessária antes da execução dos agents.
