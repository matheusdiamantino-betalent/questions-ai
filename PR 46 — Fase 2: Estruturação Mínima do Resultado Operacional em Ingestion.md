# 🧩 PR 46 — Fase 2: Estruturação Mínima do Resultado Operacional em Ingestion
## Persistência explícita de `extractedText` e `processedContent` após o primeiro consumo operacional do PDF extraction

---

<div align="left">

![PR](https://img.shields.io/badge/PR-46-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-resultado%20estruturado%20ingestion-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 45. Após realizar o primeiro consumo operacional do `PdfExtractionAgent` dentro de `ingestion`, o próximo passo mínimo correto é tornar o resultado persistido mais explícito e útil, sem criar pipeline novo, sem múltiplas etapas e sem ampliar a arquitetura da fase.
>
> - preserva `extractedText` e `processedContent` de forma explícita
> - melhora legibilidade e auditabilidade do resultado operacional
> - mantém o fluxo atual com alteração mínima no ponto de persistência
>
> **Esta PR não implementa OCR, chunking real, múltiplos outputs, classificação automática, múltiplos agents ou nova orquestração.**

---

## 📌 Sumário

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

## 1. Síntese Executiva

A PR 45 tornou o PDF extraction operacional dentro do fluxo de `ingestion`, conectando `AiService.extractPdf()` ao processamento já existente. Com isso, o sistema passou a produzir dois artefatos relevantes no mesmo fluxo: o texto extraído (`extractedText`) e o conteúdo processado (`processedContent`).

O próximo passo mínimo natural é deixar esse resultado explícito na persistência final. Em vez de armazenar apenas uma saída opaca, esta PR estrutura o `result` de forma simples e objetiva, preservando os dados essenciais produzidos pelo pipeline atual. O ganho é clareza operacional, melhor inspeção futura e evolução controlada do domínio.

---

## 2. Objetivo do PR

- estruturar o `result` final da ingestion
- persistir `extractedText` e `processedContent` de forma explícita
- manter o processor com fluxo simples e linear
- ajustar contratos mínimos apenas no necessário
- validar o novo shape com testes objetivos

---

## 3. Decisão Arquitetural

A arquitetura aprovada é mantida. Não há criação de novos módulos, filas, pipelines ou camadas intermediárias. O ajuste ocorre exclusivamente na representação do resultado final já produzido pelo fluxo atual.

A decisão central desta PR é tratar o resultado da ingestion como um objeto mínimo de domínio, em vez de uma string isolada. Isso melhora semântica e previsibilidade sem elevar complexidade estrutural.

---

## 4. Escopo

- alterar o shape persistido de `result`
- incluir `extractedText`
- incluir `processedContent`
- ajustar leitura do resultado em `findById`, se necessário
- atualizar testes relacionados ao processor e service
- preservar comportamento atual de status e fila

---

## 5. Fora de Escopo

- OCR
- chunking real de documentos
- múltiplos resultados por ingestion
- classificação semântica automática
- resumo automático adicional
- múltiplos agents em sequência
- refactor modular amplo
- dashboard ou observabilidade expandida
- versionamento de resultados

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{
  'primaryColor':'#0f172a',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#22d3ee',
  'lineColor':'#39ff14',
  'secondaryColor':'#111827',
  'tertiaryColor':'#0b1220'
}}}%%
flowchart LR
    A[Payload.content] --> B[AiService.extractPdf()]
    B --> C[extractedText]
    C --> D[LangchainModel.invoke()]
    D --> E[processedContent]
    E --> F[result estruturado]
    F --> G[ingestion completed]
```

---

## 7. Contratos Mínimos

```ts
export type IngestionResult = {
  extractedText: string;
  processedContent: string;
};
```

Persistência final esperada:

```ts
result: {
  extractedText,
  processedContent,
}
```

Se o projeto já possuir tipo equivalente, reutilizar e ajustar no menor recorte possível.

---

## 8. Regras de Implementação

A mudança deve permanecer concentrada no ponto de conclusão do processor e nos contratos mínimos necessários. O fluxo operacional não deve ser reescrito. Status, fila, logs e etapas anteriores permanecem como estão.

Não entra nesta PR qualquer preparação para múltiplos formatos, múltiplas saídas, pipelines futuros ou modelagem excessiva do domínio. O objetivo é apenas tornar explícito o resultado já existente.

---

## 9. Critérios de Review

O review deve validar se o novo `result` ficou semanticamente melhor sem aumentar complexidade. É importante confirmar que a alteração ficou localizada, que o fluxo principal permanece legível e que não houve expansão arquitetural desnecessária.

Também deve ser observado se testes cobrem o novo shape persistido e se a leitura do resultado continua consistente para consumidores atuais.

---

## 10. Critérios de Aceite

- [ ] `result` deixa de ser saída opaca e passa a shape explícito
- [ ] `extractedText` é persistido no resultado final
- [ ] `processedContent` é persistido no resultado final
- [ ] fluxo de processamento continua linear e simples
- [ ] testes relacionados estão passando
- [ ] nenhum componente estrutural extra foi criado

---

## 11. Conclusão

A PR 46 refina o valor entregue pela PR 45 sem ampliar a arquitetura. O sistema já processava conteúdo real; agora passa a representar esse resultado de forma explícita e mais útil ao domínio. O avanço continua pequeno, incremental e coerente com a fase atual.

