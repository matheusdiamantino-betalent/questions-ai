# 🔄 PR 12 — Fase 1: Primeiro Fechamento Operacional da Ingestion

## Transição mínima do primeiro consumo operacional para o primeiro encerramento controlado da operação

---

<div align="left">

![PR](https://img.shields.io/badge/PR-12-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-initial%20completion%20foundation-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge&logo=target&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ingestion%20completion-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é continuação direta das **PRs 06, 07, 08, 09, 10 e 11** e introduz apenas o próximo passo funcional mínimo da operação de `ingestion`.
>
> * manter a abertura persistida da operação
> * manter o primeiro dispatch mínimo já introduzido
> * manter o primeiro consumo operacional mínimo já introduzido
> * introduzir o **primeiro fechamento operacional mínimo**
>
> **Esta PR não implementa pipeline completo, múltiplos workers, retries, DLQ, orquestração expandida ou processamento rico de domínio.**

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

* **PR 06** → foundation mínima de autenticação delegada
* **PR 07** → propagação do usuário autenticado até `ingestion`
* **PR 08** → persistência inicial mínima da operação
* **PR 09** → foundation mínima de Redis e database access compartilhado
* **PR 10** → primeiro dispatch operacional mínimo
* **PR 11** → primeiro consumo operacional mínimo

A **PR 12** continua esse fluxo sem reprojetar a aplicação.

O próximo passo correto agora é fazer a operação de `ingestion` deixar de existir apenas como item consumido e em processamento, passando a ter seu **primeiro encerramento operacional mínimo explícito**.

Esta PR resolve exatamente esse ponto: **concluir minimamente a operação já consumida, persistindo o fechamento do ciclo operacional inicial da `ingestion`**.

---

## 2. Objetivo do PR

Introduzir o primeiro fechamento operacional mínimo da operação de `ingestion`, reaproveitando exclusivamente a foundation já consolidada nas PRs anteriores.

### Em termos práticos

Esta PR deve permitir apenas:

* receber uma `ingestion` já consumida
* validar a existência da operação em processamento
* executar o **efeito mínimo final da operação**
* atualizar o estado da `ingestion` de `processing` para `completed`

### Resultado esperado

Ao final desta PR, a aplicação deve ser capaz de:

* retirar a `ingestion` do estado intermediário de processamento
* refletir seu primeiro encerramento operacional controlado
* manter rastreabilidade básica do ciclo **abertura → dispatch → consumo → fechamento mínimo**

> [!NOTE]
> O objetivo desta PR **não** é executar pipeline completo, etapas de negócio ricas ou múltiplas fases da operação.
>
> O objetivo é apenas materializar o **primeiro fechamento mínimo da operação já consumida**, com evolução explícita e pequena de estado.

---

## 3. Decisão Arquitetural

A decisão central desta PR é:

> **consumir primeiro, fechar depois, sem introduzir pipeline, worker genérico ou infraestrutura expandida de processamento.**

A arquitetura-base já foi consolidada nas PRs anteriores e permanece a mesma.

Esta PR apenas adiciona o próximo comportamento funcional mínimo necessário sobre a estrutura já existente.

### Isso significa

* reaproveitar a foundation mínima de persistência e Redis já introduzida
* reaproveitar o consumo mínimo já introduzido
* introduzir apenas o **primeiro encerramento operacional controlado**
* manter o fluxo explícito, pequeno e revisável
* evitar antecipação de múltiplos steps, retries, coordenação rica ou pipeline completo

### Boundary exato desta PR

O fechamento introduzido aqui deve ser entendido apenas como:

* resolução da operação já consumida
* validação de existência da `ingestion`
* execução do efeito mínimo final da operação
* atualização mínima de estado para `completed`

Nada além disso é objetivo desta entrega.

---

## 4. Escopo

Esta PR inclui:

* primeiro fechamento operacional mínimo da `ingestion`
* evolução mínima do estado da operação após o primeiro consumo
* reaproveitamento da foundation de Redis já existente
* preservação da rastreabilidade da operação aberta, despachada, consumida e concluída minimamente
* evolução do comportamento do módulo de ingestion sem abrir nova fundação

### Em termos de implementação

Espera-se que esta PR cubra:

* resolução do `ingestionId` já consumido
* validação da existência da operação persistida
* execução do efeito mínimo final da operação
* atualização objetiva de status de `processing` para `completed`
* manutenção do contrato pequeno e aderente ao recorte

### Unidade mínima concluída nesta PR

A unidade operacional mínima desta entrega deve continuar sendo simples:

* origem: item já consumido do Redis
* unidade operacional: `ingestionId`
* efeito persistido: atualização do status final da operação

> [!IMPORTANT]
> O recorte desta PR termina na transição controlada de `processing` para `completed`.
>
> Qualquer execução adicional de regra de negócio, pipeline de processamento ou expansão de consumo fica fora desta entrega.

---

## 5. Fora de Escopo

Esta PR **não** inclui:

* BullMQ completo
* múltiplos workers
* retries
* DLQ
* orchestration de steps
* extraction
* classification
* resolution
* publication
* histórico rico de execução
* tabela de eventos
* state machine
* abstração genérica de worker
* infraestrutura expandida de processamento
* estrutura de pipeline completa
* processamento de domínio rico após o fechamento mínimo
* handlers genéricos, processors reutilizáveis ou infraestrutura preparada para próximos consumers

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
    G --> H[Update Ingestion Status to Queued]:::database
    H --> I[Consume Queued Ingestion Id]:::consumer
    I --> J[Update Ingestion Status to Processing]:::database
    J --> K[Finalize Minimal Ingestion Execution]:::consumer
    K --> L[Update Ingestion Status to Completed]:::database

    classDef http fill:#111827,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef auth fill:#111827,stroke:#a78bfa,color:#f5f3ff,stroke-width:2px;
    classDef controller fill:#0b1220,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef service fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef database fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
    classDef redis fill:#451a03,stroke:#f59e0b,color:#fff7ed,stroke-width:2px;
    classDef consumer fill:#172554,stroke:#60a5fa,color:#eff6ff,stroke-width:2px;
```

> [!IMPORTANT]
> Neste recorte, o fechamento entra apenas como **primeira conclusão operacional controlada após o consumo**, e não como modelagem de pipeline completo.

---

## 7. Contratos Mínimos

Os contratos devem continuar pequenos e aderentes ao recorte.

### Evolução mínima esperada

```ts
export type IngestionStatus = 'created' | 'queued' | 'processing' | 'completed';

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

### Evolução de estado esperada nesta PR

```ts
created -> queued -> processing -> completed
```

### Regra importante

Esta PR não deve inflar os contratos com:

* metadados ricos de worker
* payloads completos de processamento
* contratos de pipeline
* estruturas futuras ainda não consumidas

---

## 8. Regras de Implementação

### Ingestion

O fluxo de `ingestion` deve:

* continuar simples
* continuar recebendo dados explícitos
* manter a persistência inicial da operação
* manter o dispatch mínimo já introduzido
* manter o consumo mínimo já introduzido
* realizar o fechamento mínimo da operação em processamento
* manter rastreabilidade do avanço operacional
* não absorver infraestrutura futura desnecessária

### Consumer

O consumer deve continuar sendo:

* mínimo
* explícito
* específico para este caso de uso
* aderente à foundation já introduzida
* sem abstração genérica de worker
* sem mini-framework de processamento
* sem preparação estrutural para múltiplos consumers futuros

### Efeito mínimo esperado do fechamento

O fechamento desta PR deve fazer apenas:

1. receber a operação já consumida
2. resolver o `ingestionId`
3. validar a existência da operação
4. executar o efeito mínimo final da operação
5. atualizar o status da operação para `completed`

Se fizer além disso, o recorte está expandindo indevidamente.

### Database

A persistência deve:

* continuar usando o ponto compartilhado já consolidado
* manter o fluxo fácil de seguir
* preservar o estado operacional mínimo da operação

### Configuração

A configuração deve:

* permanecer centralizada no `environment.ts`
* seguir o padrão já adotado com Zod
* não espalhar leitura de `process.env`

---

## 9. Critérios de Review

O review desta PR deve validar se:

* a PR 12 é continuação natural das PRs 06, 07, 08, 09, 10 e 11
* a `ingestion` passa a ter primeiro fechamento operacional real
* o fechamento foi introduzido sem overengineering
* Redis foi usado de forma mínima e aderente ao recorte
* o fluxo continua simples
* a evolução de estado da operação permanece pequena e coerente
* o recorte continua pequeno, revisável e sem pipeline prematuro
* o fechamento introduzido está limitado a **resolução da operação + efeito mínimo final + atualização de status**
* não há worker genérico, processor reutilizável ou fundação paralela escondida nesta entrega

---

## 10. Critérios de Aceite

Esta PR pode ser considerada aceita se:

* [ ] a operação de `ingestion` continuar sendo aberta corretamente
* [ ] o dispatch mínimo continuar funcionando
* [ ] o primeiro consumo operacional mínimo continuar funcionando
* [ ] existir o primeiro fechamento operacional mínimo da operação
* [ ] a operação puder refletir a transição de `processing` para `completed`
* [ ] Redis for utilizado de forma mínima e sem abstração excessiva
* [ ] o recorte permanecer pequeno, funcional e revisável
* [ ] não houver antecipação indevida de pipeline completo
* [ ] não houver expansão indevida para worker, processor ou infraestrutura assíncrona mais rica

---

## 11. Conclusão

A PR 12 introduz o próximo passo correto da Fase 1:

> **fazer a operação de `ingestion` sair do estado de processamento e passar a ter seu primeiro encerramento operacional controlado, explícito e rastreável.**

Em resumo:

* **PR 06** consolidou a borda autenticada
* **PR 07** propagou a identidade até o domínio
* **PR 08** materializou a operação inicial
* **PR 09** consolidou a foundation compartilhada
* **PR 10** introduziu o primeiro dispatch operacional real
* **PR 11** introduziu o primeiro consumo operacional
* **PR 12** introduz o primeiro fechamento operacional


A **PR 13** deve ser o próximo passo apenas se houver necessidade real de introduzir **erro mínimo explícito (`failed`)** ou o **primeiro efeito funcional real de domínio**, ainda sem pipeline completo.
