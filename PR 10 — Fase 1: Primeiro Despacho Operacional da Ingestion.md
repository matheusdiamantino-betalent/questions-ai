# 🔄 PR 10 — Fase 1: Primeiro Despacho Operacional da Ingestion
## Transição mínima da operação persistida para o primeiro fluxo assíncrono controlado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-10-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-initial%20dispatch%20foundation-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge&logo=target&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ingestion%20dispatch-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é continuação direta das **PRs 06, 07, 08 e 09** e introduz apenas o próximo passo funcional mínimo da operação de `ingestion`.
>
> - manter a abertura persistida da operação
> - introduzir o **primeiro despacho operacional mínimo**
> - permitir que a operação saia do estado puramente criado
>
> **Este PR não implementa pipeline completo, orquestração rica, múltiplos steps ou infraestrutura assíncrona expandida.**

---

## 📚 Sumário

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

A progressão da Fase 1 até aqui foi:

- **PR 06** → foundation mínima de autenticação delegada
- **PR 07** → propagação do usuário autenticado até `ingestion`
- **PR 08** → persistência inicial mínima da operação
- **PR 09** → foundation mínima de Redis e database access compartilhado

A **PR 10** continua esse fluxo sem reprojetar a aplicação.

O próximo passo correto agora é fazer a operação de `ingestion` deixar de existir apenas como um registro persistido e passar a ter seu **primeiro avanço operacional explícito**.

Esta PR resolve exatamente esse ponto: **abrir, persistir, despachar minimamente e refletir esse avanço no estado da operação**.

---

## 2. Objetivo do PR

Introduzir o primeiro despacho operacional mínimo da operação de `ingestion`, reaproveitando a foundation já consolidada na PR 09.

### Em termos práticos

Esta PR deve permitir:

- criar a operação inicial de `ingestion`
- persistir seu estado inicial
- realizar o **primeiro despacho operacional mínimo**
- atualizar o estado da operação para refletir esse avanço

### Resultado esperado

Ao final desta PR, a aplicação deve ser capaz de:

- abrir uma `ingestion` persistida
- registrar seu primeiro encaminhamento operacional
- manter rastreabilidade básica do ciclo **abertura → despacho**

---

## 3. Decisão Arquitetural

A decisão central desta PR é:

> **persistir primeiro, despachar depois, sem orquestrar pipeline.**

A arquitetura-base já foi consolidada nas PRs anteriores e permanece a mesma.

Esta PR apenas adiciona o próximo comportamento funcional mínimo necessário sobre a estrutura já existente.

### Isso significa

- reaproveitar a foundation mínima de persistência e Redis já introduzida
- introduzir apenas o primeiro despacho necessário
- manter o fluxo explícito, pequeno e revisável
- evitar antecipação de múltiplos steps, workers, retries ou coordenação rica

---

## 4. Escopo

Esta PR inclui:

- primeiro despacho operacional mínimo da `ingestion`
- evolução mínima do estado inicial da operação
- reaproveitamento da foundation de Redis já existente
- preservação da rastreabilidade da operação aberta
- evolução do comportamento do `IngestionService` sem abrir nova fundação

### Em termos de implementação

Espera-se que esta PR cubra:

- persistência da operação inicial
- despacho mínimo após a criação da operação
- atualização objetiva de status
- manutenção do contrato pequeno e aderente ao recorte

---

## 5. Fora de Escopo

Esta PR **não** inclui:

- BullMQ completo
- workers ricos
- múltiplos jobs
- retries
- DLQ
- orchestration de steps
- extraction
- classification
- resolution
- publication
- histórico rico de execução
- tabela de eventos
- state machine
- abstração genérica de fila
- adapters de dispatch
- estrutura de pipeline completa

> [!NOTE]
> A regra permanece a mesma:
>
> **não implementar a próxima fase dentro da fase atual.**

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1020",
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
    A[HTTP Request]:::http --> B[AuthGuard]:::auth
    B --> C[request.user.id]:::auth
    C --> D[IngestionController]:::controller
    D --> E[IngestionService]:::service
    E --> F[Persist Initial Ingestion State]:::database
    F --> G[Dispatch Initial Ingestion Step]:::redis
    G --> H[Update Ingestion Status]:::database

    classDef http fill:#111827,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef auth fill:#111827,stroke:#a78bfa,color:#f5f3ff,stroke-width:2px;
    classDef controller fill:#0b1220,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef service fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef database fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
    classDef redis fill:#451a03,stroke:#f59e0b,color:#fff7ed,stroke-width:2px;
```

> [!IMPORTANT]
> Neste recorte, o despacho entra apenas como **primeiro avanço operacional mínimo da operação**, e não como modelagem de pipeline completo.

---

## 7. Contratos Mínimos

Os contratos devem continuar pequenos e aderentes ao recorte.

### Evolução mínima esperada

```ts
export type IngestionStatus = 'created' | 'queued';

export type CreateIngestionInput = {
  userId: number;
  payload: unknown;
};

export type IngestionRecord = {
  id: string;
  status: IngestionStatus;
  initiatedByUserId: number;
  payload: unknown;
  createdAt: Date;
  updatedAt: Date;
};
```

### Regra importante

Esta PR não deve inflar os contratos com:

- metadados ricos de job
- payloads completos de worker
- contratos de pipeline
- estruturas futuras ainda não consumidas

---

## 8. Regras de Implementação

### Ingestion

O `IngestionService` deve:

- continuar simples
- continuar recebendo dados explícitos
- persistir a operação inicial
- realizar o despacho mínimo da operação
- manter rastreabilidade do avanço operacional
- não absorver infraestrutura futura desnecessária

### Redis

O uso de Redis deve ser:

- mínimo
- explícito
- aderente à foundation já introduzida
- sem abstração genérica de fila
- sem mini-framework de dispatch

### Database

A persistência deve:

- continuar usando o ponto compartilhado já consolidado
- manter o fluxo fácil de seguir
- preservar o estado operacional mínimo da operação

### Configuração

A configuração deve:

- permanecer centralizada no `environment.ts`
- seguir o padrão já adotado com Zod
- não espalhar leitura de `process.env`

---

## 9. Critérios de Review

O review desta PR deve validar se:

- a PR 10 é continuação natural das PRs 06, 07, 08 e 09
- a `ingestion` passa a ter primeiro avanço operacional real
- o despacho foi introduzido sem overengineering
- Redis foi usado de forma mínima e aderente ao recorte
- o `IngestionService` continua simples
- a evolução de estado da operação permanece pequena e coerente
- o recorte continua pequeno, revisável e sem pipeline prematuro

---

## 10. Critérios de Aceite

Esta PR pode ser considerada aceita se:

- [ ] a operação de `ingestion` continuar sendo aberta corretamente
- [ ] a persistência inicial continuar funcionando
- [ ] existir o primeiro despacho operacional mínimo da operação
- [ ] a operação puder refletir o primeiro avanço de estado
- [ ] Redis for utilizado de forma mínima e sem abstração excessiva
- [ ] o recorte permanecer pequeno, funcional e revisável
- [ ] não houver antecipação indevida de pipeline completo

---

## 11. Conclusão

A PR 10 introduz o próximo passo correto da Fase 1:

> **fazer a operação de `ingestion` sair do estado puramente persistido e passar a ter o primeiro despacho operacional mínimo, explícito e rastreável.**

Em resumo:

- **PR 06** consolidou a borda autenticada
- **PR 07** propagou a identidade até o domínio
- **PR 08** materializou a operação inicial
- **PR 09** consolidou a foundation compartilhada
- **PR 10** introduz o primeiro avanço operacional real da `ingestion`

Este PR mantém a linha do projeto: **slice pequeno, funcional, incremental e sem overengineering**.

