# 🧩 PR 48 — Fase 2: Consolidação do Contrato Público de Resultado em Ingestion
## Formalização explícita do shape de leitura da ingestion concluída na borda HTTP, sem alterar o fluxo operacional

---

<div align="left">

![PR](https://img.shields.io/badge/PR-48-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-http%20read%20contract%20ingestion-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 47. Após refletir o resultado estruturado no fluxo de leitura da ingestion, o próximo passo mínimo correto é consolidar esse shape como contrato público explícito na borda HTTP, sem criar novos endpoints e sem alterar o processamento interno.
>
> - formaliza a resposta pública do endpoint de leitura
> - estabiliza o retorno de `extractedText` e `processedContent`
> - melhora previsibilidade para consumidores externos
> - preserva o recorte pequeno, revisável e proporcional
>
> **Este PR não implementa novo agent, OCR, dashboard, websocket, nova fila ou nova orquestração.**

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

As PRs 46 e 47 evoluíram o fluxo de ingestion em duas etapas complementares: primeiro o resultado passou a ser persistido de forma estruturada; depois esse valor passou a ser refletido no fluxo de leitura já existente. O sistema, portanto, já processa, armazena e consulta esse conteúdo de maneira funcional.

O próximo passo mínimo correto é tornar esse retorno explicitamente contratual na borda HTTP. A PR 48 consolida a forma pública da resposta sem redesenhar o fluxo atual, reduzindo ambiguidade para consumidores e deixando mais claro o compromisso externo da API.

---

## 2. Objetivo do PR

- formalizar o contrato público de leitura da ingestion
- estabilizar o retorno de `result` estruturado no endpoint existente
- alinhar controller ao shape esperado externamente
- preservar comportamento atual de status, erro e consulta
- atualizar testes relacionados à resposta HTTP

---

## 3. Decisão Arquitetural

Não há mudança de arquitetura nesta PR. O fluxo assíncrono, fila, processor, persistência e services permanecem como já aprovados nas etapas anteriores.

A alteração fica concentrada na borda HTTP já existente. A decisão central é explicitar o contrato no ponto de entrega ao cliente, evitando dependência implícita de detalhes internos e mantendo a evolução dentro de um recorte pequeno e controlado.

---

## 4. Escopo

- revisar controller de ingestion
- explicitar response contract público
- manter `result: IngestionResult | null`
- garantir serialização consistente da resposta
- atualizar testes de controller
- validar compatibilidade para cenários pendentes, concluídos e not found

---

## 5. Fora de Escopo

- novo endpoint REST
- websocket, SSE ou polling automático
- dashboard operacional
- múltiplos resultados por ingestion
- versionamento de API
- novos agents
- pipeline multi-agent
- mudanças no processor
- mudanças na fila ou orquestração

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
    A[Cliente] --> B[GET /ingestion/:id]
    B --> C[IngestionController]
    C --> D[IngestionService.findById]
    D --> E[Contrato HTTP explícito]
    E --> F[result estruturado ou null]
```

---

## 7. Contratos Mínimos

O retorno público da leitura passa a estar explicitamente formalizado na borda HTTP, preservando o mesmo shape funcional já introduzido nas PRs anteriores.

```ts
export type IngestionHttpResponse = {
  id: string;
  status: IngestionStatus;
  result: {
    extractedText: string;
    processedContent: string;
  } | null;
  failureReason: string | null;
  createdAt: Date;
  updatedAt: Date;
};
```

Exemplo esperado:

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

Registros ainda sem resultado permanecem compatíveis com `result: null`.

---

## 8. Regras de Implementação

A mudança deve permanecer localizada no controller e no contrato público necessário. Não criar presenters, adapters, DTOs paralelos excessivos ou novas camadas quando o ajuste puder ser resolvido diretamente no fluxo já existente.

Controller fino, service simples e sem alteração do comportamento operacional. O foco desta PR é apenas tornar explícito o que já é funcional no domínio.

---

## 9. Critérios de Review

O review deve validar se o contrato público ficou claro, se a resposta HTTP permaneceu simples e se a alteração continuou proporcional ao slice proposto.

Também deve confirmar que consumidores recebem shape previsível em cenários concluídos e pendentes, sem regressão em not found e sem introdução de complexidade desnecessária.

---

## 10. Critérios de Aceite

- [ ] resposta HTTP da leitura está explicitamente contratada
- [ ] `result` estruturado é retornado de forma estável
- [ ] cenários sem resultado continuam compatíveis com `null`
- [ ] comportamento de not found permanece inalterado
- [ ] testes foram atualizados e permanecem passando
- [ ] nenhum componente estrutural novo foi criado

---

## 11. Conclusão

A PR 48 consolida na borda pública o fluxo funcional construído nas PRs 46 e 47. O resultado já era processado, persistido e consultado; agora também passa a estar formalizado como contrato externo claro e previsível.

O avanço segue incremental, útil e contido. Não há expansão de arquitetura, apenas fechamento correto de uma evolução já iniciada, mantendo a fase coerente, revisável e alinhada ao próximo passo mínimo necessário.
