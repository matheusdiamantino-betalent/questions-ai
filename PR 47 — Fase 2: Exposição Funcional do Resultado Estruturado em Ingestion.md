# 🧩 PR 47 — Fase 2: Exposição Funcional do Resultado Estruturado em Ingestion
## Retorno explícito de `extractedText` e `processedContent` na leitura da ingestion concluída

---

<div align="left">

![PR](https://img.shields.io/badge/PR-47-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-read%20contract%20ingestion-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 46. Após estruturar e persistir o resultado operacional da ingestion, o próximo passo mínimo correto é expor esse mesmo valor ao consumidor da API de leitura, sem novos endpoints e sem ampliar a arquitetura já aprovada.
>
> - retorna `extractedText` e `processedContent` no contrato de leitura
> - melhora a utilidade do endpoint de status sem alterar o fluxo assíncrono
> - mantém processor, fila, persistência e transições operacionais inalterados
> - preserva compatibilidade para registros que ainda não possuem resultado
>
> **Este PR não implementa dashboards, polling avançado, OCR, múltiplos outputs, nova fila ou nova orquestração.**

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

A PR 46 passou a tratar o resultado da ingestion de forma estruturada ao persistir `extractedText` e `processedContent` em um shape explícito de domínio. Esse passo resolveu a semântica interna do resultado, mas ainda deixava incompleto o consumo funcional da informação, já que a API de leitura seguia sem refletir esse mesmo contrato de forma direta.

A PR 47 fecha esse ciclo com o próximo passo mínimo correto: alinhar a resposta de leitura ao modelo já persistido. Com isso, o consumidor consegue consultar uma ingestion concluída e receber o resultado estruturado sem depender de interpretação implícita, transformação paralela ou expansão do fluxo operacional.

---

## 2. Objetivo do PR

- ajustar o contrato de leitura da ingestion para refletir o resultado estruturado já persistido
- retornar `result` tipado com `IngestionResult | null` na consulta por id
- expor `extractedText` e `processedContent` quando o processamento já estiver concluído
- preservar comportamento atual de status, erro e compatibilidade de leitura
- atualizar testes existentes para o novo contrato público

---

## 3. Decisão Arquitetural

Não há rediscussão de arquitetura nesta PR. O fluxo consolidado até aqui permanece o mesmo: a ingestion é criada, processada de forma assíncrona, persiste seu resultado estruturado e continua sendo consultada pelo endpoint já existente.

A decisão central é restringir a mudança à borda de leitura, alinhando contrato e retorno do service ao shape já armazenado. Isso evita camadas intermediárias desnecessárias, mantém o recorte pequeno e transforma a persistência estruturada da PR 46 em valor funcional real para quem consome a API.

---

## 4. Escopo

- atualizar `IngestionReadResponse`
- tipar `result` como `IngestionResult | null`
- ajustar `IngestionService.findById()` para refletir o contrato atualizado
- revisar a serialização do controller apenas se necessário para manter o response coerente
- atualizar testes de leitura já existentes
- preservar compatibilidade para registros sem resultado persistido

---

## 5. Fora de Escopo

- criação de novo endpoint de consulta
- websocket, SSE ou qualquer mecanismo de atualização em tempo real
- dashboard operacional ou visão agregada de ingestões
- múltiplos resultados por ingestion
- versionamento de contrato
- paginação de histórico
- OCR
- nova pipeline de agents
- qualquer mudança no processor, fila ou orquestração assíncrona

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
    A[Cliente consulta ingestion] --> B[IngestionController]
    B --> C[IngestionService.findById]
    C --> D[IngestionDao.findById]
    D --> E[result estruturado persistido]
    E --> F[response com status e result]
```

---

## 7. Contratos Mínimos

O contrato público de leitura passa a refletir explicitamente o resultado já persistido na ingestion concluída.

```ts
export type IngestionReadResponse = {
  id: string;
  status: IngestionStatus;
  result: IngestionResult | null;
  failureReason: string | null;
  createdAt: Date;
  updatedAt: Date;
};
```

Quando houver resultado disponível, a leitura deve expor o mesmo shape estrutural já adotado internamente:

```json
{
  "id": "ing-1",
  "status": "completed",
  "result": {
    "extractedText": "texto extraído",
    "processedContent": "texto processado"
  },
  "failureReason": null
}
```

Registros anteriores ou ainda não concluídos permanecem compatíveis com `result: null`.

---

## 8. Regras de Implementação

A implementação deve permanecer concentrada no contrato, no fluxo de consulta e na resposta já existente. Controller fino, service simples e DAO restrito à leitura de persistência, sem presenters, adapters, mappers dedicados ou qualquer fundação paralela para um ajuste que é apenas de exposição contratual.

Também não deve haver mudança de comportamento operacional. A PR só torna visível na leitura o dado que já foi tornado explícito e persistido anteriormente.

---

## 9. Critérios de Review

O review deve validar se o contrato público ficou coerente com a persistência atual e se a alteração permaneceu proporcional ao recorte. É importante confirmar que a leitura entrega `result` estruturado apenas quando esse valor existir, sem quebrar cenários antigos nem ampliar o fluxo além da consulta já consolidada.

Também deve ser verificado que não houve reabertura de arquitetura, criação de camadas desnecessárias ou antecipação de próximos passos fora deste slice.

---

## 10. Critérios de Aceite

- [ ] `IngestionReadResponse` reflete `IngestionResult | null`
- [ ] leitura da ingestion concluída retorna `extractedText` e `processedContent`
- [ ] registros sem resultado continuam compatíveis com `result: null`
- [ ] comportamento atual de erro e not found permanece inalterado
- [ ] testes de leitura foram atualizados e permanecem passando
- [ ] nenhum componente estrutural novo foi introduzido

---

## 11. Conclusão

A PR 47 transforma em contrato público o resultado estruturado introduzido na PR 46. O avanço é pequeno, direto e funcional: o sistema já persistia melhor esse valor e agora também o expõe de forma consistente na leitura da ingestion concluída.

Com isso, a fase continua evoluindo sem redesenho, sem expansão indevida e sem inflar a solução além do necessário. O recorte permanece claro, revisável e alinhado ao próximo passo mínimo correto da trilha.
