# 🏗️ PR 46 — Fase 2: Estruturação Mínima do Resultado Operacional em Ingestion
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
> Esta PR continua diretamente a PR 45. Após o primeiro consumo operacional do `PdfExtractionAgent` dentro de `ingestion`, o próximo passo mínimo correto é tornar o resultado persistido explícito no domínio, sem reabrir a arquitetura aprovada e sem ampliar a fase.
>
> - persiste `extractedText` e `processedContent` no resultado final
> - mantém o fluxo atual com ajuste localizado no ponto de conclusão
> - melhora leitura, inspeção e previsibilidade do dado persistido
>
> **Este PR não introduz novo pipeline, OCR, chunking, múltiplos outputs, múltiplos agents ou nova orquestração.**

---

## 📌 Sumário

1. Síntese Executiva
2. Objetivo do PR
3. Decisão Arquitetural
4. Escopo
5. Fora de Escopo
6. Fluxo Arquitetural
7. Contratos Mínimos
8. Regras de Implementação
9. Critérios de Review
10. Critérios de Aceite
11. Conclusão

---

## 1. Síntese Executiva

A PR 45 conectou o `PdfExtractionAgent` ao fluxo operacional de `ingestion` e tornou efetivo o primeiro consumo desse processamento dentro da fase atual. Com isso, o fluxo passou a produzir, no mesmo caminho linear, o texto extraído do PDF e o conteúdo processado a partir dessa extração.

A PR 46 não cria nova etapa. Ela apenas torna esse resultado final mais explícito na persistência. Em vez de concluir a ingestion com uma saída opaca, o recorte passa a registrar de forma clara os dois artefatos já produzidos pelo fluxo atual: `extractedText` e `processedContent`.

Esse avanço melhora legibilidade operacional, facilita futuras consultas e mantém a progressão incremental da fase sem inflar arquitetura.

---

## 2. Objetivo do PR

- estruturar o `result` final da ingestion
- persistir `extractedText` e `processedContent`
- manter o processor linear e simples
- ajustar contratos mínimos no menor recorte possível
- atualizar testes aderentes ao novo shape

---

## 3. Decisão Arquitetural

A arquitetura existente é mantida integralmente. Não há novos módulos, filas, workers adicionais, pipelines paralelos ou abstrações extras.

A única decisão desta PR é representar o resultado final como objeto explícito de domínio, preservando os dados já gerados pelo fluxo atual. O ganho está na semântica e na clareza, não em expansão estrutural.

---

## 4. Escopo

- alterar o shape persistido de `result`
- incluir `extractedText`
- incluir `processedContent`
- ajustar leitura de `findById`, se aplicável
- atualizar testes do processor/service
- preservar status, fila e fluxo atual

---

## 5. Fora de Escopo

- OCR
- chunking real
- múltiplos resultados por ingestion
- classificação automática
- múltiplos agents em sequência
- observabilidade expandida
- versionamento de resultados
- refactor amplo de módulos

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark'}}%%
flowchart LR
A[Payload.content] --> B[AiService.extractPdf]
B --> C[extractedText]
C --> D[Processamento atual]
D --> E[processedContent]
E --> F[result estruturado]
F --> G[completed]
```

---

## 7. Contratos Mínimos

```ts
export type IngestionResult = {
  extractedText: string;
  processedContent: string;
};
```

```ts
result: {
  extractedText,
  processedContent,
}
```

Se já existir tipo equivalente no projeto, reutilizar e ajustar apenas o necessário.

---

## 8. Regras de Implementação

Controller permanece fino. Processor permanece simples. DAO/repositório continua restrito à persistência.

A mudança deve ficar concentrada no ponto de conclusão da ingestion e nos contratos necessários para leitura/escrita do resultado. Não preparar próximas fases dentro desta PR.

---

## 9. Critérios de Review

Validar se o novo `result` melhora a semântica sem elevar complexidade. Confirmar que a alteração ficou localizada, que o fluxo principal segue legível e que não houve overengineering.

Verificar também se testes cobrem o novo formato persistido e se consumidores atuais permanecem consistentes.

---

## 10. Critérios de Aceite

- [ ] `result` passa a ter shape explícito
- [ ] `extractedText` é persistido
- [ ] `processedContent` é persistido
- [ ] fluxo continua linear e simples
- [ ] testes estão passando
- [ ] nenhum componente estrutural extra foi criado

---

## 11. Conclusão

A PR 46 refina o valor entregue pela PR 45 sem ampliar a arquitetura. O sistema já processava conteúdo real; agora passa a representar esse resultado de forma explícita e mais útil ao domínio. O avanço permanece pequeno, revisável e coerente com a fase atual.
