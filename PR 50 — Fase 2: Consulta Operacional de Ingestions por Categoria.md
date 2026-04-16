# 🧩 PR 50 — Fase 2: Consulta Operacional de Ingestions por Categoria
## Busca mínima de ingestions utilizando classificação semântica já persistida

---

<div align="left">

![PR](https://img.shields.io/badge/PR-50-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-query%20ingestions%20por%20categoria-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 49. Após enriquecer a ingestion com classificação mínima, o próximo passo correto é transformar esse dado em capacidade consultável, permitindo busca operacional simples por categoria sem expandir a arquitetura da fase.
>
> - adiciona consulta por categoria
> - reutiliza metadados já persistidos
> - cria valor operacional imediato
> - mantém recorte pequeno e revisável
>
> **Esta PR não implementa paginação avançada, dashboard, analytics, full text search, vector search ou filtros complexos.**

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

A PR 49 introduziu classificação semântica mínima no resultado da ingestion. A partir desse ponto, o sistema passou a possuir informação categorizada persistida, porém ainda sem uma forma direta de consulta operacional orientada a esse dado.

Esta PR converte esse enriquecimento em utilidade prática ao adicionar uma listagem simples filtrável por categoria. O ganho é imediato para inspeção operacional e organização de ingestions, mantendo a evolução incremental e sem ampliar a base arquitetural já aprovada.

---

## 2. Objetivo do PR

- permitir busca de ingestions por categoria
- aceitar combinação opcional com status
- expor endpoint mínimo de listagem
- reutilizar contratos e estruturas já existentes quando possível
- validar o fluxo com testes objetivos e proporcionais ao recorte

---

## 3. Decisão Arquitetural

Não há mudança estrutural nesta PR. A evolução permanece dentro das camadas já consolidadas: controller, service e DAO.

A decisão central é adicionar capacidade consultiva mínima sobre dados já disponíveis, sem introduzir motores de busca, cache, agregadores ou novas dependências. O foco é consulta simples sobre valor que o sistema já produz e persiste.

---

## 4. Escopo

- criar endpoint `GET /ingestion`
- aceitar filtro `category`
- aceitar filtro opcional `status`
- adicionar `findMany` no DAO
- retornar lista simplificada de resultados
- atualizar testes de controller, service e DAO

---

## 5. Fora de Escopo

- paginação avançada
- ordenação avançada
- full text search
- vector search
- dashboard analítico
- exportação CSV
- filtros compostos complexos
- cache distribuído
- agregações estatísticas

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
    A[Cliente] --> B[GET /ingestion?category=x]
    B --> C[IngestionController]
    C --> D[IngestionService.findMany]
    D --> E[IngestionDao.findMany]
    E --> F[Listagem filtrada]
```

---

## 7. Contratos Mínimos

A listagem introduz filtros pequenos e um retorno objetivo, suficiente para uso operacional inicial.

```ts
export type FindIngestionsFilters = {
  category?: string;
  status?: IngestionStatus;
};

export type IngestionListItem = {
  id: string;
  status: IngestionStatus;
  category: string | null;
  createdAt: Date;
  updatedAt: Date;
};
```

Exemplo esperado:

```json
[
  {
    "id": "ing-1",
    "status": "completed",
    "category": "financeiro"
  }
]
```

---

## 8. Regras de Implementação

A implementação deve priorizar simplicidade: query direta no banco, filtros opcionais e retorno enxuto. Evitar builders complexos, repositories genéricos, abstrações prematuras ou camadas adicionais sem necessidade real.

Se nenhum filtro for informado, o comportamento pode seguir a decisão mínima adotada pelo projeto, desde que permaneça explícito, previsível e simples de revisar.

---

## 9. Critérios de Review

Validar se a consulta ficou simples, previsível e aderente ao recorte proposto. Confirmar que filtros funcionam sem inflar a complexidade do DAO e que a resposta entregue é suficiente para uso operacional inicial.

Também deve ser observado se não houve expansão indevida de escopo para funcionalidades analíticas ou mecanismos de busca fora desta etapa.

---

## 10. Critérios de Aceite

- [ ] endpoint de listagem foi criado
- [ ] filtro por categoria está funcionando
- [ ] filtro por status opcional está funcionando
- [ ] DAO suporta consulta mínima proposta
- [ ] testes foram atualizados e permanecem passando
- [ ] nenhum componente estrutural novo foi criado

---

## 11. Conclusão

A PR 50 transforma a classificação introduzida anteriormente em capacidade operacional real. O sistema passa a permitir consulta simples por categoria sobre dados já enriquecidos e persistidos.

O avanço segue incremental, útil e contido. Não há aumento artificial de complexidade, apenas uso prático de uma informação que a plataforma já passou a produzir na etapa anterior.
