# 🔄 PR 08 — Fase 1: Persistência Mínima da Operação de Ingestion
## Continuação do primeiro uso funcional do usuário autenticado no fluxo de entrada

---

<div align="left">

![PR](https://img.shields.io/badge/PR-08-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-ingestion%20state-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge&logo=target&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-persist%C3%AAncia%20m%C3%ADnima-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Este PR implementa apenas o **próximo passo mínimo funcional** após a PR 07:
>
> - registrar de forma persistente a operação aberta de `ingestion`
> - manter o vínculo com o ator autenticado via `initiatedByUserId`
> - preservar um estado mínimo da operação
> - continuar o fluxo sem inflar arquitetura ou antecipar pipeline futuro
>
> **Este PR não introduz fila, processamento assíncrono, BullMQ, repository pattern, ORM, ACL, pipeline multi-step ou abstrações de persistência prematuras.**

---

## 📚 Sumário

1. [Síntese Executiva](#1-síntese-executiva)  
2. [Objetivo do PR](#2-objetivo-do-pr)  
3. [Decisão Arquitetural](#3-decisão-arquitetural)  
4. [Escopo](#4-escopo)  
5. [Fora de Escopo](#5-fora-de-escopo)  
6. [Fluxo Arquitetural](#6-fluxo-arquitetural)  
7. [Estrutura Proposta](#7-estrutura-proposta)  
8. [Contratos Mínimos](#8-contratos-mínimos)  
9. [Regras de Implementação](#9-regras-de-implementação)  
10. [Critérios de Review](#10-critérios-de-review)  
11. [Critérios de Aceite](#11-critérios-de-aceite)  
12. [Conclusão](#12-conclusão)

---

## 1. Síntese Executiva

A **PR 06** estabeleceu a foundation do **auth delegado**.  
A **PR 07** fez o primeiro uso funcional real de `request.user.id` no módulo de `ingestion`.

Esta **PR 08** continua esse fluxo com o **próximo passo mínimo e funcional**:

- a operação de `ingestion` deixa de ser apenas um retorno em memória instantâneo;
- passa a existir um **estado mínimo persistido** dentro do módulo;
- esse estado continua vinculado ao usuário autenticado que iniciou a operação.

O objetivo aqui não é “resolver ingestion”.

O objetivo é validar o próximo degrau arquitetural com a **menor solução correta possível**.

---

## 2. Objetivo do PR

Este PR existe para:

- registrar a operação criada de `ingestion`
- preservar o `id` gerado da operação
- manter `initiatedByUserId`
- permitir que a abertura da operação tenha **estado mínimo persistido**
- continuar a evolução do módulo sem expandir responsabilidade fora do recorte

---

## 3. Decisão Arquitetural

A decisão deste PR é deliberadamente simples:

- o `AuthGuard` continua autenticando na borda HTTP
- o controller continua apenas propagando `request.user.id`
- o service continua recebendo `userId` explicitamente
- a operação criada agora é **registrada em armazenamento mínimo local ao módulo**

### Princípio central

> Antes de sofisticar pipeline, persistência real, jobs ou processamento, a aplicação deve primeiro ser capaz de **abrir e manter o estado mínimo rastreável da operação**.

---

## 4. Escopo

Este PR cobre apenas:

- manter a criação do `ingestion`
- registrar a operação criada em armazenamento mínimo
- preservar `id` e `initiatedByUserId`
- manter o controller fino
- manter o service simples
- manter o módulo enxuto e revisável

---

## 5. Fora de Escopo

Este PR **não cobre**:

- persistência em PostgreSQL
- repositories
- ORM / Prisma / TypeORM
- migrations
- fila / BullMQ
- pipeline multi-step
- processamento assíncrono
- status avançado de execução
- retries
- observabilidade expandida
- ACL de publicação
- validação rica de payload
- decorators adicionais
- abstrações de storage

> [!NOTE]
> Este recorte valida apenas a existência do **estado mínimo da operação** antes de qualquer sofisticação posterior.

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1220",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5f0ff",
    "primaryBorderColor": "#22d3ee",
    "lineColor": "#38bdf8",
    "secondaryColor": "#0f172a",
    "tertiaryColor": "#111827",
    "fontFamily": "Inter, Segoe UI, Arial",
    "clusterBkg": "#0f172a",
    "clusterBorder": "#334155"
  }
}}%%

flowchart LR
    A[HTTP Request] --> B[AuthGuard]
    B --> C[IngestionController]
    C --> D[IngestionService]
    D --> E[Create Ingestion Record]
    E --> F[Persist Minimal State]
    F --> G[Return Ingestion Record]
```

### Leitura do fluxo

1. a requisição chega autenticada  
2. o `AuthGuard` resolve o usuário  
3. o controller extrai `request.user.id`  
4. o service cria a operação  
5. a operação passa a ser registrada em armazenamento mínimo  
6. o estado inicial é retornado com rastreabilidade do ator

---

## 7. Estrutura Proposta

```text
src/
└── modules/
    └── ingestion/
        ├── infra/
        │   ├── controllers/
        │   │   └── ingestion.controller.ts
        │   └── services/
        │       └── ingestion.service.ts
        ├── model/
        │   └── v1/
        │       └── ingestion.contracts.ts
        └── ingestion.module.ts
```

### Observação

A estrutura permanece propositalmente pequena.

Não há criação de `repository`, `infra/database`, `entities`, `mappers`, `factories` ou camadas paralelas neste momento.

---

## 8. Contratos Mínimos

### Input

```ts
export type CreateIngestionInput = {
  userId: number;
  payload: unknown;
};
```

### Registro mínimo da operação

```ts
export type IngestionRecord = {
  id: string;
  initiatedByUserId: number;
};
```

### Intenção desses contratos

- `CreateIngestionInput` representa a entrada mínima do caso de uso
- `IngestionRecord` representa o estado mínimo inicial da operação

Esses contratos permanecem intencionalmente pequenos para evitar antecipação indevida de pipeline futuro.

---

## 9. Regras de Implementação

A implementação deste PR deve seguir estas regras:

- manter o controller fino
- manter o service simples
- não acoplar o service à camada HTTP
- continuar propagando `userId` explicitamente
- registrar a operação criada sem criar fundação paralela
- não transformar persistência mínima em infraestrutura completa
- não antecipar banco real, repository pattern ou abstrações de storage

> [!TIP]
> Se a solução parecer “arquitetural demais” para o que este PR entrega, ela provavelmente está errada para este recorte.

---

## 10. Critérios de Review

O review deste PR deve validar se:

- a persistência mínima da operação foi introduzida corretamente
- `initiatedByUserId` continua sendo registrado
- o service continua simples e coeso
- não foi criada infraestrutura desnecessária
- o recorte continua pequeno, funcional e revisável
- o módulo não foi inflado com preparação futura

---

## 11. Critérios de Aceite

- [ ] a criação da operação de `ingestion` continuar funcional
- [ ] o `id` da operação continuar sendo gerado
- [ ] `initiatedByUserId` continuar sendo registrado
- [ ] a operação passar a existir em estado mínimo persistido
- [ ] não houver abstração prematura
- [ ] não houver expansão indevida do módulo
- [ ] o recorte permanecer pequeno, funcional e revisável

---

## 12. Conclusão

A **PR 08** não tenta sofisticar o fluxo de `ingestion`.

Ela apenas consolida o próximo passo lógico da **Fase 1**:

- primeiro a identidade foi autenticada (**PR 06**)
- depois foi propagada no fluxo (**PR 07**)
- agora a operação aberta passa a ter **estado mínimo persistido** (**PR 08**)

Essa progressão mantém o projeto:

- simples
- incremental
- rastreável
- revisável
- aderente ao desenho arquitetural validado

---

## Resultado esperado

Ao final deste PR, o módulo de `ingestion` continuará pequeno, mas deixará de ser apenas um “retorno funcional de passagem” e passará a manter o **primeiro estado mínimo da operação** dentro do fluxo autenticado.

