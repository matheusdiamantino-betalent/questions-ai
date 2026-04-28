# 🛡️ PR 104 — Fase 2: Payload Normalization e Guardrails de Entrada

## Validação estrutural e normalização mínima do endpoint de processamento de questões

---

<div align="left">

![PR](https://img.shields.io/badge/PR-104-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-payload%20normalization-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR adiciona um guardrail HTTP explícito no endpoint `POST /ai/questions/process`, validando a estrutura mínima do payload e normalizando campos textuais antes da execução do fluxo de agentes.
>
> - Falha cedo para entradas inválidas.
> - Reduz ruído operacional antes do handoff interno.
> - Preserva o fluxo atual do orchestrator.
>
> **Este PR não altera agentes, não muda o contrato de resposta e não introduz nova fundação arquitetural.**

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

A PR 104 continua a evolução do endpoint de processamento de questões com um recorte pequeno: proteger a entrada HTTP antes de entregar o payload ao fluxo de agentes. A mudança não redesenha o processamento, não altera o orchestrator e não antecipa novas etapas do pipeline.

O ganho direto está na previsibilidade da borda pública. Entradas estruturalmente inválidas passam a falhar cedo com `BadRequestException`, enquanto campos textuais válidos são normalizados antes da execução interna.

---

# 2. Objetivo do PR

Esta PR tem como objetivo validar e normalizar o payload recebido por `POST /ai/questions/process`, garantindo que o fluxo interno receba uma `question` minimamente consistente.

O recorte cobre:

- validação da presença de `body.question`;
- validação de `statement` como string não vazia;
- validação de `alternatives` como array entre 2 e 5 itens;
- validação de cada alternativa como texto utilizável;
- validação opcional de `comments`;
- validação de `correctAnswer` conforme o range de alternativas enviado;
- normalização textual via `trim()`.

---

# 3. Decisão Arquitetural

A validação permanece no controller por se tratar de regra de boundary HTTP. O `AgentsFlowOrchestratorService` continua responsável pelo fluxo de processamento posterior, recebendo uma entrada já saneada, sem acoplamento com detalhes da requisição.

A decisão mantém a arquitetura atual: controller fino, validação local e objetiva, sem criação de DTO complexo, pipe customizado, schema externo, adapter intermediário ou nova camada para um recorte que ainda é pequeno.

---

# 4. Escopo

Entra nesta PR:

- guardrail local no controller de processamento de questões;
- helpers mínimos para validação de estrutura e `correctAnswer`;
- normalização de `statement`, `alternatives` e `comments`;
- conversão de `comments` em branco para `undefined`;
- cobertura automatizada do fluxo feliz e dos principais casos inválidos.

---

# 5. Fora de Escopo

Não entra nesta PR:

- alteração do orchestrator;
- alteração dos agentes internos;
- mudança no contrato de resposta;
- uso de `class-validator`;
- criação de pipe customizado;
- introdução de schema runtime externo;
- refactor estrutural do módulo de IA;
- novas regras semânticas sobre conteúdo da questão.

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
    A["HTTP Request<br/>POST /ai/questions/process"] --> B["Controller<br/>valida estrutura mínima"]
    B --> C["Normalização<br/>trim e comments opcional"]
    C --> D["AgentsFlowOrchestratorService<br/>processamento interno"]
    D --> E["Resposta processada"]

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

Entrada esperada:

```json
{
  "question": {
    "statement": "Texto da questão",
    "alternatives": ["Alternativa A", "Alternativa B", "Alternativa C", "Alternativa D"],
    "comments": "Comentário opcional",
    "correctAnswer": "A"
  }
}
```

Regras mínimas aplicadas:

- `statement` deve ser string não vazia após `trim()`;
- `alternatives` deve conter de 2 a 5 itens;
- cada alternativa deve ser string não vazia após `trim()`;
- `comments`, quando informado, deve ser string;
- `comments` vazio após `trim()` é tratado como `undefined`;
- `correctAnswer` deve ser uma letra válida dentro do range de alternativas enviado.

---

# 8. Regras de Implementação

A implementação deve permanecer local ao controller, com validação simples e explícita. O objetivo é proteger a entrada pública sem espalhar regra HTTP por camadas internas e sem introduzir abstrações antes de necessidade real.

O fluxo interno deve continuar visível: validar, normalizar e delegar ao orchestrator. Nenhum comportamento futuro deve ser preparado dentro desta PR.

---

# 9. Critérios de Review

Validar se a PR:

- mantém o controller simples e legível;
- protege corretamente o boundary HTTP;
- preserva o orchestrator sem acoplamento à requisição;
- não cria camada, helper ou contrato além do necessário;
- cobre falhas estruturais relevantes em teste;
- mantém a documentação proporcional ao recorte.

---

# 10. Critérios de Aceite

- [ ] Payload sem `question` retorna `400`.
- [ ] `statement` ausente, inválido ou vazio retorna `400`.
- [ ] `alternatives` fora do intervalo de 2 a 5 retorna `400`.
- [ ] Alternativa vazia ou inválida retorna `400`.
- [ ] `comments` não string retorna `400`.
- [ ] `comments` em branco é normalizado para `undefined`.
- [ ] `correctAnswer` fora do range permitido retorna `400`.
- [ ] Payload válido é normalizado e delegado ao orchestrator.
- [ ] Suíte de testes permanece verde.

---

# 11. Conclusão

A PR 104 fortalece a entrada do endpoint de processamento de questões com validação e normalização mínimas. O recorte é pequeno, preserva a arquitetura existente e melhora a previsibilidade do fluxo sem antecipar novas camadas ou ampliar o escopo da fase.
