# 📄 PR 93 — Correção: Separação do AiPdfService

## Isolamento do serviço de PDF em arquivo próprio para reduzir acoplamento estrutural

---

<div align="left">

![PR](https://img.shields.io/badge/PR-93-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-refactor%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-ai%20workflow-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

> [!IMPORTANT]
> Esta PR aplica uma correção estrutural pontual indicada no review do fluxo de IA.
> O objetivo é separar o `AiPdfService` do arquivo `ai.service.ts`, preservando comportamento, providers e testes, sem alterar o fluxo de extração de PDF.

---

## Sumário

1. [Síntese Executiva](#1-síntese-executiva)
2. [Objetivo do PR](#2-objetivo-do-pr)
3. [Decisão Arquitetural](#3-decisão-arquitetural)
4. [Escopo da PR](#4-escopo-da-pr)
5. [Fora de Escopo](#5-fora-de-escopo)
6. [Fluxo Arquitetural](#6-fluxo-arquitetural)
7. [Contratos Mínimos](#7-contratos-mínimos)
8. [Regras de Implementação](#8-regras-de-implementação)
9. [Critérios de Review](#9-critérios-de-review)
10. [Critérios de Aceite](#10-critérios-de-aceite)
11. [Conclusão](#11-conclusão)

---

## 1. Síntese Executiva

O arquivo `ai.service.ts` concentra duas classes de serviço com responsabilidades distintas:

```txt
AiService
AiPdfService
```

Embora isso funcione tecnicamente, a estrutura reduz clareza de leitura e mistura responsabilidades que evoluem por motivos diferentes.

Esta PR move o `AiPdfService` para um arquivo próprio, mantendo o comportamento atual e sem alterar o contrato do serviço.

---

## 2. Objetivo do PR

Separar o `AiPdfService` em arquivo dedicado:

```txt
src/shared/ai/infra/services/ai-pdf.service.ts
```

E manter o `AiService` em:

```txt
src/shared/ai/infra/services/ai.service.ts
```

Com isso, cada arquivo passa a representar uma responsabilidade principal.

---

## 3. Decisão Arquitetural

A decisão é realizar apenas uma separação física de arquivo, sem criar novas abstrações.

Antes:

```txt
ai.service.ts
├── AiService
└── AiPdfService
```

Depois:

```txt
ai.service.ts
└── AiService

ai-pdf.service.ts
└── AiPdfService
```

Essa mudança melhora organização sem alterar a topologia do módulo.

---

## 4. Escopo da PR

Incluído nesta PR:

- mover `AiPdfService` para arquivo próprio;
- ajustar imports;
- manter provider registrado no módulo atual;
- atualizar specs, se necessário;
- preservar comportamento de delegação para `PdfExtractionAgent`.

Arquivos esperados:

```txt
src/shared/ai/infra/services/ai.service.ts
src/shared/ai/infra/services/ai-pdf.service.ts
src/shared/ai/ai.module.ts ou módulo equivalente
src/__tests__/shared/ai/infra/services/ai-pdf.service.spec.ts
```

---

## 5. Fora de Escopo

Não faz parte desta PR:

- alterar `PdfExtractionAgent`;
- alterar fluxo de ingestion;
- alterar prompts;
- alterar contratos de PDF;
- alterar LangGraph;
- alterar Redis;
- alterar DAO;
- alterar orquestração multi-agent;
- remover `AiPdfService`.

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
    "nodeSpacing": 54,
    "rankSpacing": 68
  }
}}%%
flowchart LR
    A["AiService<br/>LLM + knowledge context"] --> B["ai.service.ts"]
    C["AiPdfService<br/>PDF extraction facade"] --> D["ai-pdf.service.ts"]
    C --> E["PdfExtractionAgent"]

    classDef core fill:#111827,stroke:#38bdf8,stroke-width:2px,color:#ffffff;
    classDef file fill:#0b1325,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef output fill:#052e2b,stroke:#2dd4bf,stroke-width:2px,color:#ffffff;

    class A,C core;
    class B,D file;
    class E output;
```

---

## 7. Contratos Mínimos

O contrato do `AiPdfService` permanece igual:

```ts
@Injectable()
export class AiPdfService {
  constructor(private readonly pdfExtractionAgent: PdfExtractionAgent) {}

  async execute(input: PdfExtractionAgentInput): Promise<PdfExtractionAgentOutput>
}
```

Nenhum consumidor deve perceber alteração funcional.

---

## 8. Regras de Implementação

1. Não alterar comportamento.
2. Não alterar contrato público.
3. Não introduzir nova abstração.
4. Não mover responsabilidades do `PdfExtractionAgent`.
5. Não alterar o `AiService` além da remoção do serviço secundário.
6. Ajustar imports explicitamente.
7. Preservar testes existentes.
8. Manter recorte pequeno.

---

## 9. Critérios de Review

Validar se:

- `AiService` contém apenas responsabilidade própria;
- `AiPdfService` está em arquivo dedicado;
- imports foram atualizados;
- provider continua registrado;
- specs permanecem verdes;
- não houve mudança funcional no fluxo de PDF.

---

## 10. Critérios de Aceite

A PR pode ser aceita quando:

- build passar;
- testes passarem;
- `AiPdfService` estiver isolado;
- nenhum contrato externo mudar;
- o fluxo de extração de PDF continuar delegando ao `PdfExtractionAgent`.

---

## 11. Conclusão

Esta PR aplica uma correção simples de organização estrutural.

Ao separar o `AiPdfService` do `AiService`, o código fica mais claro e mais alinhado à responsabilidade única por arquivo, sem alterar comportamento e sem expandir a arquitetura.
