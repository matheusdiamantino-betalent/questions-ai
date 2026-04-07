# 🔄 PR 08 — Fase 1: Estado Inicial Mínimo da Operação de Ingestion
## Primeiro registro estruturado da operação iniciada por usuário autenticado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-08-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-estado%20inicial%20m%C3%ADnimo-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge&logo=target&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ingestion%20state-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é continuação direta da **PR 07** e **não redefine a arquitetura**.
>
> Ela introduz apenas o próximo passo funcional mínimo do fluxo:
>
> - materializar o estado inicial da operação de ingestion
> - manter vínculo com o usuário autenticado (`initiatedByUserId`)
> - explicitar o primeiro registro estruturado da operação no domínio
>
> Sem:
>
> - banco de dados
> - persistência física
> - repository pattern
> - nova camada arquitetural
> - fila
> - pipeline
> - expansão de auth
> - modelagem prematura de estados futuros

---

## 1. Sumário

- [1. Sumário](#1-sumário)
- [2. Síntese Executiva](#2-síntese-executiva)
- [3. Objetivo](#3-objetivo)
- [4. Decisão Arquitetural](#4-decisão-arquitetural)
- [5. Por que esta PR existe?](#5-por-que-esta-pr-existe)
- [6. Escopo](#6-escopo)
- [7. Fora de Escopo](#7-fora-de-escopo)
- [8. Fluxo Arquitetural](#8-fluxo-arquitetural)
- [9. Contratos](#9-contratos)
- [10. Estado Inicial Esperado](#10-estado-inicial-esperado)
- [11. Regras de Implementação](#11-regras-de-implementação)
- [12. Critérios de Review](#12-critérios-de-review)
- [13. Critérios de Aceite](#13-critérios-de-aceite)
- [14. Conclusão](#14-conclusão)

---

## 2. Síntese Executiva

A evolução arquitetural da foundation até aqui é:

- **PR 06** → autenticação delegada mínima
- **PR 07** → propagação do usuário autenticado até o domínio de `ingestion`
- **PR 08** → materialização do estado inicial mínimo da operação

A **PR 07** validou que a identidade autenticada já atravessa corretamente a borda HTTP e entra no caso de uso via:

```ts
request.user.id
```

A **PR 08** dá o próximo passo natural:

> **a operação de ingestion deixa de existir apenas como retorno parcial e passa a nascer com estado inicial mínimo explícito e consistente no domínio.**

O foco desta PR **não é sofisticar o domínio**.

O foco é apenas garantir que a abertura da operação:

- exista como registro estruturado mínimo
- tenha autoria mínima registrada
- tenha estado inicial explícito
- continue pequena, simples e revisável

---

## 3. Objetivo

Introduzir a **materialização mínima do estado inicial** da operação de `ingestion`, preservando a simplicidade validada na PR 07.

### O que esta PR precisa garantir

- a operação nasce com identificador
- a operação nasce vinculada ao usuário autenticado que a iniciou
- a operação possui estado inicial mínimo explícito
- a operação pode ser rastreada desde a abertura no domínio

### Em termos práticos

A operação criada deve preservar minimamente:

- `id`
- `status`
- `initiatedByUserId`
- `payload`
- `createdAt`
- `updatedAt`

---

## 4. Decisão Arquitetural

A decisão desta PR é propositalmente conservadora:

> **Materializar antes de sofisticar.**

### Isso significa:

- manter o **controller fino**
- manter o **service simples**
- não criar **repository pattern**
- não criar **storage abstraction**
- não criar **camadas novas “por arquitetura”**
- não implementar a próxima fase agora

### Princípio aplicado

- **usar antes de abstrair**
- **explicitar antes de persistir**
- **validar antes de generalizar**
- **não criar fundação paralela**

---

## 5. Por que esta PR existe?

Na PR 07, o fluxo passou a receber e propagar corretamente a identidade autenticada até o módulo de `ingestion`.

Esse passo foi importante porque validou que:

- o `AuthGuard` funciona no boundary HTTP
- `request.user.id` chega corretamente ao domínio
- o service já recebe `userId` explicitamente
- a autoria mínima da operação já está presente no caso de uso

Mas isso ainda deixava uma lacuna:

> a operação ainda não nascia com um **estado mínimo explícito e completo**.

Ou seja, o sistema já sabia **quem iniciou**, mas ainda não materializava corretamente **a forma mínima da operação**.

A PR 08 existe para fechar exatamente esse gap, sem inflar o domínio antes da hora.

---

## 6. Escopo

Esta PR inclui:

- materialização mínima da operação de `ingestion`
- manutenção explícita de `initiatedByUserId`
- manutenção do `payload` recebido na abertura
- definição do estado inicial mínimo da operação
- manutenção do desenho simples validado na PR 07

### Em termos de implementação

Espera-se que esta PR preserve:

- `AuthGuard` na borda
- leitura de `request.user.id` no controller
- propagação explícita de `userId` ao service
- criação do estado inicial mínimo da operação
- retorno do registro criado

---

## 7. Fora de Escopo

Esta PR **não** inclui:

- banco de dados
- persistência física
- schema de tabela
- migration
- expansão do módulo `auth`
- `CurrentUser` decorator
- request context global
- roles/scopes locais
- autorização rica
- fila / BullMQ
- pipeline assíncrono
- processing
- extraction
- classification
- publication
- retry avançado
- circuit breaker
- audit completo
- abstrações de repository “para o futuro”
- múltiplos estados sofisticados do fluxo
- orquestração de steps

> [!NOTE]
> A regra continua a mesma:
>
> **não implementar a próxima fase dentro da fase atual.**

---

## 8. Fluxo Arquitetural

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
    A[HTTP Request]:::node --> B[AuthGuard]:::auth
    B --> C[request.user.id]:::auth
    C --> D[IngestionController]:::controller
    D --> E[IngestionService]:::service
    E --> F[Create Initial Ingestion State]:::domain
    F --> G[Return Initial Ingestion Record]:::result

    classDef node fill:#0f172a,stroke:#22d3ee,color:#e5f0ff,stroke-width:2px;
    classDef auth fill:#111827,stroke:#a78bfa,color:#f5f3ff,stroke-width:2px;
    classDef controller fill:#0b1220,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef service fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef domain fill:#1e1b4b,stroke:#8b5cf6,color:#ede9fe,stroke-width:2px;
    classDef result fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
```

---

## 9. Contratos

### Input mínimo do caso de uso

```ts
export type CreateIngestionInput = {
  userId: number;
  payload: unknown;
};
```

### Estado inicial mínimo da operação

```ts
export type IngestionRecord = {
  id: string;
  status: 'created';
  initiatedByUserId: number;
  payload: unknown;
  createdAt: Date;
  updatedAt: Date;
};
```

### Intenção do contrato

Este contrato não tenta modelar o pipeline completo.

Ele modela apenas o **primeiro estado mínimo explícito da operação**.

---

## 10. Estado Inicial Esperado

O estado esperado nesta PR deve ser pequeno, explícito e suficiente para o recorte atual.

### Estado inicial esperado

```ts
{
  id: string;
  status: 'created';
  initiatedByUserId: number;
  payload: unknown;
  createdAt: Date;
  updatedAt: Date;
}
```

### O que isso valida

- a operação realmente nasceu no domínio
- a operação tem autoria mínima
- a operação tem rastreabilidade inicial
- o sistema já possui um shape mínimo consistente para evolução futura

### O que isso ainda não tenta resolver

- persistência física
- lifecycle completo
- transições de estado avançadas
- execução assíncrona
- rastreabilidade completa do pipeline
- reprocessamento
- observabilidade expandida

---

## 11. Regras de Implementação

### Controller

- deve continuar fino
- continua lendo `request.user.id`
- continua delegando ao service
- não deve absorver regra de domínio

### Service

- recebe `userId` explicitamente
- recebe `payload`
- cria o estado inicial mínimo da operação
- retorna o registro criado
- não deve assumir infraestrutura ainda não definida no projeto

### Regras gerais

- simplicidade
- clareza
- baixo acoplamento
- zero overengineering
- implementação pequena e revisável

---

## 12. Critérios de Review

O review desta PR deve validar se:

- a PR 08 é continuação clara da PR 07
- o salto entre “propagação” e “estado inicial mínimo” está correto
- `initiatedByUserId` continua sendo preservado corretamente
- o estado inicial da operação está adequado ao recorte
- o controller continua fino
- o service continua simples
- não houve expansão indevida de escopo
- não surgiram abstrações sem segundo caso real

---

## 13. Critérios de Aceite

Esta PR pode ser considerada aceita se:

- [ ] o endpoint de `ingestion` continuar protegido por `AuthGuard`
- [ ] `request.user.id` continuar acessível no controller
- [ ] `userId` continuar sendo propagado explicitamente ao service
- [ ] a operação criada passar a nascer com estado inicial mínimo explícito
- [ ] `initiatedByUserId` for preservado corretamente
- [ ] `status`, `payload`, `createdAt` e `updatedAt` forem materializados corretamente
- [ ] não houver abstração prematura
- [ ] não houver expansão indevida do auth
- [ ] o recorte permanecer pequeno, funcional e revisável

---

## 14. Conclusão

A PR 08 não tenta sofisticar o domínio de `ingestion`.

Ela apenas introduz o próximo passo correto depois da PR 07:

> **se a identidade autenticada já entra corretamente no domínio, a operação iniciada por essa identidade deve passar a nascer com estado inicial mínimo explícito e consistente.**

Em resumo:

- **PR 06** autenticou a borda
- **PR 07** propagou a identidade ao domínio
- **PR 08** materializa o estado inicial mínimo da operação

Essa PR, portanto, valida o primeiro registro estruturado da operação de entrada do pipeline — ainda pequeno, simples e fiel ao recorte da Fase 1.
