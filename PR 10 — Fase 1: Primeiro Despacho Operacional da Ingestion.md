# 🔄 PR 10 — Fase 1: Primeiro Despacho Operacional da Ingestion

## Transição mínima da operação persistida para o primeiro fluxo assíncrono controlado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-10-2563eb?style=for-the-badge\&logo=gitpullrequest\&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-initial%20dispatch%20foundation-7c3aed?style=for-the-badge\&logo=nestjs\&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge\&logo=target\&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ingestion%20dispatch-0891b2?style=for-the-badge\&logo=serverless\&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge\&logo=githubactions\&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é continuação direta das **PRs 06, 07, 08 e 09** e introduz apenas o próximo passo funcional mínimo da operação de `ingestion`:
>
> * manter a abertura persistida da operação
> * introduzir o **primeiro despacho operacional**
> * permitir que a operação deixe de existir apenas como estado criado
>
> O foco desta PR é **despacho mínimo e rastreável da operação**, sem inflar o recorte com pipeline completo, filas ricas ou orquestração antecipada.

---

## 📚 Sumário

1. [Síntese Executiva](#1-síntese-executiva)
2. [Objetivo do PR](#2-objetivo-do-pr)
3. [Decisão Arquitetural](#3-decisão-arquitetural)
4. [Escopo](#4-escopo)
5. [Fora de Escopo](#5-fora-de-escopo)
6. [Fluxo Arquitetural](#6-fluxo-arquitetural)
7. [Estrutura Envolvida](#7-estrutura-envolvida)
8. [Despacho Operacional Proposto](#8-despacho-operacional-proposto)
9. [Evolução do Estado da Ingestion](#9-evolução-do-estado-da-ingestion)
10. [Contratos Mínimos](#10-contratos-mínimos)
11. [Regras de Implementação](#11-regras-de-implementação)
12. [Critérios de Review](#12-critérios-de-review)
13. [Critérios de Aceite](#13-critérios-de-aceite)
14. [Conclusão](#14-conclusão)

---

## 1. Síntese Executiva

A progressão da Fase 1 até aqui foi:

* **PR 06** → foundation mínima de autenticação delegada
* **PR 07** → propagação do usuário autenticado até `ingestion`
* **PR 08** → persistência inicial mínima da operação
* **PR 09** → foundation mínima de Redis e database access compartilhado

A **PR 10** continua esse fluxo sem reprojetar a aplicação.

O objetivo agora é fazer a operação de `ingestion` dar o **primeiro passo operacional real após sua criação persistida**.

Em vez de existir apenas como um registro criado, a operação passa a poder ser **despachada para continuidade de processamento futuro**, ainda de forma mínima, explícita e controlada.

Esta PR **não implementa o pipeline completo**.

Ela resolve o próximo passo correto: fazer a operação sair do estado puramente persistido e entrar no **primeiro fluxo operacional rastreável da aplicação**.

---

## 2. Objetivo do PR

Introduzir o primeiro despacho operacional mínimo da operação de `ingestion`, reaproveitando a foundation consolidada na PR 09.

### Em termos práticos

Esta PR deve permitir:

* criar a operação inicial de `ingestion`
* persistir seu estado inicial
* registrar o **primeiro avanço operacional da operação**
* tornar a operação elegível para continuidade futura de processamento

### Resultado esperado

Ao final desta PR, a aplicação deve ser capaz de:

* abrir uma `ingestion` persistida
* disparar seu primeiro passo operacional mínimo
* atualizar o estado da operação para refletir esse avanço
* manter rastreabilidade básica do ciclo de abertura → despacho

---

## 3. Decisão Arquitetural

A decisão central desta PR é:

> **persistir primeiro, despachar depois, sem orquestrar o pipeline completo.**

Isso significa:

* reaproveitar a foundation mínima de persistência já consolidada
* introduzir apenas o **primeiro despacho funcional necessário**
* manter o comportamento explícito e pequeno
* não antecipar múltiplos steps, workers, filas ricas ou coordenação completa

### Princípios aplicados

* **estado operacional antes de pipeline**
* **despacho mínimo antes de orquestração**
* **sem fundação paralela**
* **sem overengineering**
* **sem implementar a próxima fase antes da hora**

---

## 4. Escopo

Esta PR inclui:

* primeiro despacho operacional mínimo da `ingestion`
* evolução do estado inicial da operação
* reaproveitamento da foundation mínima de Redis já introduzida
* preservação da rastreabilidade da operação aberta
* alinhamento do fluxo de `ingestion` ao primeiro comportamento assíncrono controlado

### Em termos de implementação

Espera-se que esta PR cubra:

* evolução do `IngestionService` para realizar o despacho inicial
* eventual atualização mínima de status da operação
* uso objetivo e mínimo da foundation de Redis já existente
* manutenção do contrato pequeno e aderente ao recorte

---

## 5. Fora de Escopo

Esta PR **não** inclui:

* BullMQ completo
* workers ricos
* múltiplos jobs
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
* abstração genérica de fila
* adapters de dispatch
* estrutura de pipeline completa

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
    G --> H[Update Ingestion Operational Status]:::database

    classDef http fill:#111827,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef auth fill:#111827,stroke:#a78bfa,color:#f5f3ff,stroke-width:2px;
    classDef controller fill:#0b1220,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef service fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef database fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
    classDef redis fill:#451a03,stroke:#f59e0b,color:#fff7ed,stroke-width:2px;
```

> [!IMPORTANT]
> Neste recorte, o despacho entra como **primeiro avanço operacional mínimo da operação**, e não como pipeline completo.
>
> A intenção é apenas permitir que a `ingestion` saia do estado “criado” e passe a ter **continuidade operacional explícita**.

---

## 7. Estrutura Envolvida

A evolução desta PR preserva a estrutura já consolidada nas PRs anteriores e reutiliza a foundation compartilhada introduzida na PR 09.

```text
src/
├── modules/
│   └── ingestion/
│       ├── ingestion.module.ts
│       ├── infra/
│       │   ├── controllers/
│       │   │   └── ingestion.controller.ts
│       │   └── services/
│       │       └── ingestion.service.ts
│       └── model/
│           └── v1/
│               └── ingestion.contracts.ts
└── shared/
    ├── config/
    │   └── environment.ts
    └── infra/
        ├── database/
        │   ├── index.ts
        │   └── generated/
        └── redis/
            ├── redis.module.ts
            └── redis.service.ts
```

### Regra importante

Esta PR **não precisa abrir estrutura nova** para representar o despacho inicial.

Ela deve, preferencialmente, **evoluir o comportamento em cima da estrutura já existente**, mantendo o recorte pequeno, direto e revisável.

---

## 8. Despacho Operacional Proposto

O despacho desta PR representa o **primeiro encaminhamento operacional mínimo da operação de `ingestion`**.

### Intenção funcional

Após a abertura da operação, a aplicação deve ser capaz de:

* identificar que a `ingestion` foi criada com sucesso
* registrar o primeiro encaminhamento operacional dessa operação
* deixar a operação pronta para evolução futura do pipeline

### Intenção arquitetural

Este despacho existe para resolver **somente** o primeiro avanço operacional da operação.

Ele não tenta modelar ainda:

* processamento completo
* múltiplos jobs
* cadeia de steps
* reprocessamento
* retry policy
* coordenação distribuída

> [!TIP]
> O valor desta PR não está em “criar a fila completa”, e sim em introduzir **o primeiro comportamento assíncrono mínimo com rastreabilidade**.

---

## 9. Evolução do Estado da Ingestion

A operação deixa de existir apenas como um registro recém-criado e passa a refletir o primeiro avanço operacional do fluxo.

### Status mínimo esperado

A evolução mínima esperada pode ser algo como:

* `created`
* `queued`

ou equivalente semântico aderente ao padrão do projeto.

### Regra importante

A modelagem de estado deve continuar **pequena e funcional**.

Esta PR não deve transformar o status da operação em uma state machine prematura.

### Objetivo do estado

O estado existe aqui para resolver **somente**:

* rastreabilidade básica da operação
* leitura clara da progressão inicial
* suporte ao próximo slice da Fase 1

---

## 10. Contratos Mínimos

Os contratos devem continuar pequenos e aderentes ao recorte.

### Exemplo de evolução mínima possível

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

* metadados ricos de job
* payloads de worker completos
* contratos de pipeline
* estruturas futuras ainda não consumidas

---

## 11. Regras de Implementação

### Ingestion

O `IngestionService` deve:

* continuar simples
* continuar recebendo dados explícitos
* persistir a operação inicial
* realizar o despacho mínimo da operação
* manter rastreabilidade do avanço operacional
* não absorver infraestrutura futura desnecessária

### Redis

O uso de Redis deve ser:

* mínimo
* explícito
* aderente à foundation já introduzida
* sem criar abstração genérica de fila
* sem criar mini-framework de dispatch

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

## 12. Critérios de Review

O review desta PR deve validar se:

* a PR 10 é continuação natural das PRs 06, 07, 08 e 09
* a `ingestion` passa a ter primeiro avanço operacional real
* o despacho foi introduzido sem overengineering
* Redis foi usado de forma mínima e aderente ao recorte
* o `IngestionService` continua simples
* a evolução de estado da operação permanece pequena e coerente
* o recorte continua pequeno, revisável e sem pipeline prematuro

---

## 13. Critérios de Aceite

Esta PR pode ser considerada aceita se:

* [ ] a operação de `ingestion` continuar sendo aberta corretamente
* [ ] a persistência inicial continuar funcionando
* [ ] existir o primeiro despacho operacional mínimo da operação
* [ ] a operação puder refletir o primeiro avanço de estado
* [ ] Redis for utilizado de forma mínima e sem abstração excessiva
* [ ] o recorte permanecer pequeno, funcional e revisável
* [ ] não houver antecipação indevida de pipeline completo

---

## 14. Conclusão

A PR 10 não tenta resolver o pipeline completo.

Ela introduz o próximo passo correto da Fase 1:

> **fazer a operação de `ingestion` sair do estado puramente persistido e passar a ter o primeiro despacho operacional mínimo, explícito e rastreável.**

Em resumo:

* **PR 06** consolidou a borda autenticada
* **PR 07** propagou a identidade até o domínio
* **PR 08** materializou a operação inicial
* **PR 09** consolidou a foundation compartilhada
* **PR 10** introduz o primeiro avanço operacional real da `ingestion`

Esta PR abre, portanto, o primeiro recorte de despacho operacional da Fase 1 — ainda pequeno, explícito, incremental e sem overengineering.
