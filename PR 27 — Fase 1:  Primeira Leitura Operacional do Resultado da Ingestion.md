# 🔎 PR 27 — Primeira Leitura Operacional do Resultado da Ingestion
## Exposição mínima do resultado já persistido para fechar o ciclo funcional da ingestion

---

<div align="left">

![PR](https://img.shields.io/badge/PR-27-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-pivot%20agents-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-primeira%20leitura%20do%20resultado-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua o fluxo já consolidado de ingestion e adiciona somente a primeira leitura útil do resultado persistido.
>
> - preserva a criação e o processamento já existentes
> - preserva a persistência do output introduzida anteriormente
> - adiciona acesso mínimo ao resultado já gerado
>
> **Este PR não implementa busca avançada, listagem, filtros, paginação, retrieval semântico ou dashboard.**

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

As PRs anteriores fecharam o fluxo interno de ingestion: criação, fila, processamento, uso real de LangChain e persistência do resultado final.

Com esse ciclo operacional consolidado, o próximo passo mínimo correto é permitir a leitura do que já foi gerado. Esta PR faz essa evolução sem expandir a arquitetura e sem antecipar uma camada de consulta mais sofisticada.

O foco é simples: tornar acessível, por um ponto mínimo de leitura, o resultado já persistido pela pipeline atual.

---

## 2. Objetivo do PR

- expor leitura mínima de uma ingestion já existente
- retornar status e resultado persistido
- permitir validação funcional ponta a ponta do fluxo já entregue
- preservar o pipeline atual sem redesign estrutural
- manter o recorte pequeno, objetivo e revisável

---

## 3. Decisão Arquitetural

A decisão arquitetural desta PR é reutilizar o módulo de ingestion já existente e adicionar apenas uma operação mínima de leitura sobre dados já persistidos.

Não há nova fundação, nova camada transversal nem abstração genérica de consulta. O acesso ocorre dentro da estrutura atual do módulo, com fluxo explícito e baixo acoplamento.

Em termos práticos, esta PR apenas abre uma capacidade básica de leitura sobre algo que o sistema já produz hoje.

---

## 4. Escopo

Entra neste PR:

- endpoint mínimo de leitura por identificador
- busca da ingestion persistida
- retorno de campos operacionais essenciais
- tratamento simples para registro inexistente
- testes proporcionais ao novo comportamento

---

## 5. Fora de Escopo

Não entra neste PR:

- listagem de ingestions
- filtros
- paginação
- ordenação
- busca textual
- retrieval semântico
- ranking
- dashboard
- analytics
- múltiplos formatos de resposta
- redesign do módulo

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {"theme": "dark", "themeVariables": {"lineColor": "#22d3ee"}}}%%
flowchart LR
    A["Cliente"] --> B["GET /ingestion/:id"]
    B --> C["IngestionController"]
    C --> D["IngestionService"]
    D --> E["IngestionDao"]
    E --> F["Registro Persistido"]
    F --> G["Resposta mínima"]
```

---

## 7. Contratos Mínimos

```ts
type IngestionReadResponse = {
  id: string;
  status: string;
  result: string | null;
  failureReason: string | null;
  createdAt: Date;
  updatedAt: Date;
};
```

Os campos devem permanecer mínimos e estritamente aderentes ao que já existe no fluxo atual, sem inflar o contrato de saída.

---

## 8. Regras de Implementação

- manter controller fino e focado em HTTP
- concentrar leitura no fluxo atual do módulo
- DAO restrito à persistência
- evitar nova abstração de query sem necessidade real
- retornar apenas campos essenciais
- tratar ausência de registro de forma simples e explícita
- manter testes pequenos e objetivos
- não antecipar próximas fases nesta entrega

---

## 9. Critérios de Review

Validar principalmente:

- se a PR fecha o ciclo funcional com leitura útil do resultado
- se o recorte permaneceu pequeno e claro
- se não houve expansão indevida de arquitetura
- se a leitura reutiliza corretamente a estrutura atual
- se o contrato retornado está enxuto e útil
- se o fluxo principal continua fácil de entender

---

## 10. Critérios de Aceite

- [ ] Existe endpoint mínimo de leitura por id
- [ ] Uma ingestion existente pode ser consultada
- [ ] O retorno contém status e resultado persistido
- [ ] Registro inexistente possui tratamento adequado
- [ ] Há testes proporcionais ao slice entregue
- [ ] Não houve expansão arquitetural indevida
- [ ] O recorte permanece pequeno e revisável

---

## 11. Conclusão

A PR 27 adiciona o próximo passo natural depois da persistência do resultado: a primeira leitura operacional do que a pipeline já produz.

Com isso, o fluxo de ingestion deixa de ser apenas interno e passa a oferecer uma consulta mínima, útil e verificável, mantendo simplicidade, continuidade arquitetural e baixo ruído para review.
