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
> Esta PR é continuação direta da PR 07 e **não redefine arquitetura**.
>
> Ela apenas introduz o próximo passo funcional mínimo:
>
> - persistir a operação de ingestion criada
> - manter vínculo com o usuário autenticado (`initiatedByUserId`)
> - preservar estado mínimo da operação
>
> Sem:
> - ORM
> - repository pattern
> - fila
> - pipeline
> - abstração de storage
> - expansão de auth

---

## 📚 Sumário

1. Síntese Executiva  
2. Objetivo  
3. Decisão Arquitetural  
4. Escopo  
5. Fora de Escopo  
6. Fluxo  
7. Contratos  
8. Regras  
9. Critérios de Review  
10. Critérios de Aceite  
11. Conclusão  

---

## 1. Síntese Executiva

- PR 06 → autenticação delegada  
- PR 07 → propagação do usuário autenticado  
- PR 08 → persistência mínima da operação  

A arquitetura permanece a mesma definida na PR 07.

---

## 2. Objetivo

Adicionar persistência mínima da operação de ingestion:

- manter `id`
- manter `initiatedByUserId`
- permitir existência de estado mínimo

---

## 3. Decisão Arquitetural

Mantém-se integralmente o desenho da PR 07:

- controller fino
- service simples
- sem nova camada
- sem infra adicional

A única mudança é registrar o estado da operação.

---

## 4. Escopo

- registrar ingestion em memória
- manter contratos existentes
- não alterar estrutura de módulos

---

## 5. Fora de Escopo

- banco de dados
- repository
- ORM
- fila / BullMQ
- pipeline
- validação avançada
- abstração de storage

---

## 6. Fluxo

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
    B --> C[IngestionController]:::controller
    C --> D[IngestionService]:::service
    D --> E[Create Ingestion Record]:::domain
    E --> F[Persist Minimal State]:::storage
    F --> G[Return Ingestion Record]:::result

    classDef node fill:#0f172a,stroke:#22d3ee,color:#e5f0ff,stroke-width:2px;
    classDef auth fill:#111827,stroke:#a78bfa,color:#f5f3ff,stroke-width:2px;
    classDef controller fill:#0b1220,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef service fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef domain fill:#1e1b4b,stroke:#8b5cf6,color:#ede9fe,stroke-width:2px;
    classDef storage fill:#082f49,stroke:#06b6d4,color:#ecfeff,stroke-width:2px;
    classDef result fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
```

---

## 7. Contratos

```ts
export type CreateIngestionInput = {
  userId: number;
  payload: unknown;
};

export type IngestionRecord = {
  id: string;
  initiatedByUserId: number;
};
```

---

## 8. Regras

- não criar abstração
- não expandir arquitetura
- manter simplicidade
- não antecipar fases

---

## 9. Critérios de Review

- persistência mínima implementada
- sem overengineering
- aderente à PR 07

---

## 10. Critérios de Aceite

- [ ] ingestion continua funcionando
- [ ] estado mínimo persistido
- [ ] initiatedByUserId preservado
- [ ] sem abstração desnecessária

---

## 11. Conclusão

PR 08 apenas evolui o fluxo validado:

auth → propagação → persistência mínima
