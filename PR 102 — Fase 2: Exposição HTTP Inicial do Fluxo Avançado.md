# 🤖 PR 102 — Fase 2: Exposição HTTP Inicial do Fluxo Avançado

## Endpoint mínimo e controlado para processamento de questões via pipeline de agents

---

<div align="left">

![PR](https://img.shields.io/badge/PR-102-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-http%20endpoint-0ea5e9?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-proposed-111827?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

> [!IMPORTANT]
> Esta PR inicia a exposição externa do fluxo avançado de IA com o menor recorte possível: um endpoint HTTP fino que delega integralmente ao pipeline já consolidado. Não altera regras internas, não cria nova orquestração e não expande arquitetura além do necessário.

---

## Sumário
- [1. Síntese Executiva](#1-síntese-executiva)
- [2. Objetivo do PR](#2-objetivo-do-pr)
- [3. Decisão Arquitetural](#3-decisão-arquitetural)
- [4. Escopo da PR](#4-escopo-da-pr)
- [5. Fora de Escopo](#5-fora-de-escopo)
- [6. Fluxo Arquitetural](#6-fluxo-arquitetural)
- [7. Contratos Mínimos](#7-contratos-mínimos)
- [8. Regras de Implementação](#8-regras-de-implementação)
- [9. Critérios de Review](#9-critérios-de-review)
- [10. Critérios de Aceite](#10-critérios-de-aceite)
- [11. Conclusão](#11-conclusão)

---

## 1. Síntese Executiva

Após a consolidação interna do pipeline (classificação, resolução de IDs, processamento inicial e gabarito), o próximo passo natural é disponibilizar uma porta de entrada externa controlada.

Esta entrega adiciona um endpoint HTTP único para consumo do fluxo avançado, reutilizando o `AgentsFlowOrchestratorService` como núcleo da execução.

---

## 2. Objetivo do PR

Disponibilizar um endpoint mínimo para receber uma questão e retornar o resultado completo já suportado pelo pipeline atual.

---

## 3. Decisão Arquitetural

O controller deve ser estritamente fino:

- receber request
- validar payload mínimo
- delegar ao orchestrator existente
- retornar resposta

Toda regra de negócio permanece nos agents e no orchestrator atual.

---

## 4. Escopo da PR

Incluído nesta PR:

- endpoint `POST /ai/questions/process`
- integração com `AgentsFlowOrchestratorService`
- contrato de entrada mínimo
- retorno de `InitialQuestionProcessingOutput`
- testes do boundary HTTP

---

## 5. Fora de Escopo

Não faz parte desta entrega:

- autenticação nova
- persistência
- filas
- múltiplos endpoints
- versionamento complexo
- swagger ornamental
- dashboard
- rate limit
- redesign do pipeline

---

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
    "nodeSpacing": 44,
    "rankSpacing": 56
  }
}}%%
flowchart LR
    A["HTTP Request"] --> B["AI Controller"]
    B --> C["AgentsFlowOrchestratorService"]
    C --> D["Classification"]
    D --> E["ID Resolution"]
    E --> F["Initial Processing"]
    F --> G["Answer Key"]
    G --> H["HTTP Response"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    class A,B,C,D,E,F,G,H step1;
```

---

## 7. Contratos Mínimos

### Request

```json
{
  "question": {
    "statement": "Texto da questão",
    "alternatives": ["A", "B", "C", "D", "E"],
    "comments": "Opcional",
    "correctAnswer": "A"
  }
}
```

### Response

Contrato já existente do fluxo avançado:

- `legalSearch`
- `adaptedStatement`
- `answerKey`
- `metadata`
- `ids`
- `idResolutionConfidence`

---

## 8. Regras de Implementação

- controller sem regra de negócio
- reutilizar contratos existentes
- sem mappers desnecessários
- sem services paralelos
- sem duplicar validações já existentes no pipeline
- código proporcional ao recorte

---

## 9. Critérios de Review

Validar se:

- endpoint está fino
- pipeline atual foi reutilizado
- response mantém contrato esperado
- não houve expansão desnecessária
- nomenclatura está coesa
- testes cobrem boundary principal

---

## 10. Critérios de Aceite

- `POST /ai/questions/process` responde corretamente
- payload inválido retorna erro adequado
- orchestrator é acionado
- response contém estrutura esperada
- specs verdes

---

## 11. Conclusão

A PR 102 inicia a camada HTTP do fluxo avançado da forma correta: pequena, objetiva e sem acoplamentos extras. Primeiro expõe, depois evolui.
