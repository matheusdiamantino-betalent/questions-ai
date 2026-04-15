# 🧩 PR 36 — Fase 2: Content Knowledge Retrieval Refinement
## Refinamento mínimo do consumo da base local no fluxo compartilhado já validado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-36-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-content%20knowledge%20refinement-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-refino%20do%20consumo%20vetorial-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é a continuação direta da PR 35. Após validar o primeiro consumo funcional da base jurídica local, o próximo passo mínimo correto é refinar a qualidade e previsibilidade do contexto recuperado no mesmo fluxo já aprovado.
>
> - preserva a foundation compartilhada consolidada nas PRs 29 a 35
> - mantém `shared/ai` como owner de retrieval e geração
> - melhora composição do prompt enriquecido com contexto já disponível
> - ajusta o consumo real em `modules/content` sem redesenho arquitetural
>
> **Este PR não introduz reranking, múltiplas fontes, hybrid search, scheduler, pipeline adicional ou nova camada paralela.**

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

A PR 34 criou a base jurídica local com ingestão vetorial mínima. A PR 35 comprovou o uso real desse corpus ao conectá-lo ao fluxo funcional de `content` por meio do runtime compartilhado.

Com o caminho principal validado, a evolução natural não é abrir novas frentes. O próximo passo mínimo correto é melhorar a utilidade do contexto já recuperado, refinando como os resultados são limitados, organizados e incorporados ao prompt final.

Esta PR executa exatamente esse ajuste. O foco permanece no fluxo existente: respostas mais úteis e previsíveis a partir da base local, sem alterar ownerships, sem ampliar arquitetura e sem inflar o recorte.

---

## 2. Objetivo do PR

- refinar o consumo de contexto vetorial no fluxo de `content`
- calibrar `knowledgeLimit` para o uso real atual
- melhorar a composição do prompt enriquecido
- aumentar a utilidade prática da resposta gerada
- preservar simplicidade e facilidade de review

---

## 3. Decisão Arquitetural

A decisão central desta PR é evoluir o comportamento de consumo sem alterar a arquitetura aprovada. `modules/content` continua consumidor fino do boundary compartilhado, enquanto `shared/ai` permanece responsável por retrieval, composição contextual e geração.

Nenhum módulo novo é introduzido. Não há acesso direto do consumidor a detalhes vetoriais, nem redistribuição de responsabilidades já estabilizadas nas PRs anteriores.

A evolução ocorre por continuidade controlada: melhorar o que já existe antes de adicionar novas capacidades.

---

## 4. Escopo

- ajustar uso de `knowledgeLimit` no fluxo real
- refinar seleção e aproveitamento do contexto recuperado
- melhorar montagem do prompt final enriquecido
- manter integração via `AiService` e serviços existentes
- preservar `modules/content` como camada fina
- elevar previsibilidade do consumo da base local

---

## 5. Fora de Escopo

- reranking avançado
- hybrid search
- múltiplas fontes documentais
- filtros ricos de metadata
- scheduler
- retry pipeline
- agents adicionais
- LangGraph expandido
- observabilidade adicional
- nova API pública de knowledge base

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Input funcional] --> B[AiService]
    B --> C[EmbeddingsService.search]
    C --> D[Contexto calibrado]
    D --> E[Prompt refinado]
    E --> F[LangchainModel]
    F --> G[Resposta final]
```

O fluxo permanece linear e conhecido. O refinamento desta PR acontece dentro do caminho já validado, tornando o uso do contexto mais consistente sem adicionar novas etapas laterais.

---

## 7. Contratos Mínimos

Os contratos públicos permanecem enxutos e compatíveis com a fase atual. O refinamento concentra-se em comportamento interno de consumo, sem exigir nova superfície contratual para módulos consumidores.

```ts
type ExecuteAiInput = {
  prompt: string;
  knowledgeQuery?: string;
  knowledgeLimit?: number;
};
```

Nenhum contrato externo deve expor score, chunks, estruturas vetoriais ou detalhes internos de busca.

---

## 8. Regras de Implementação

A implementação deve seguir o menor caminho útil. Ajustes de retrieval devem reutilizar serviços já existentes, e a composição final do prompt deve permanecer centralizada no runtime compartilhado.

`modules/content` segue fino e orientado ao fluxo funcional. `shared/ai` continua owner técnico das capacidades reutilizáveis. Não introduzir builders genéricos, orchestrators paralelos ou abstrações criadas para fases futuras.

Se houver dúvida entre sofisticar estrutura ou manter o fluxo explícito e legível, esta PR deve favorecer a segunda opção.

---

## 9. Critérios de Review

- houve melhoria real no consumo do contexto recuperado
- `shared/ai` continua owner de retrieval e geração
- consumidores seguem sem acoplamento a detalhes internos
- o fluxo principal permanece simples e linear
- não houve expansão indevida de arquitetura
- a implementação ficou proporcional ao slice
- a PR se posiciona como continuação natural da PR 35

---

## 10. Critérios de Aceite

- [ ] `content` utiliza recuperação vetorial refinada no fluxo atual
- [ ] `knowledgeLimit` está calibrado para o caso real
- [ ] o prompt final aproveita melhor o contexto retornado
- [ ] `modules/content` continua sem acesso a detalhes internos de retrieval
- [ ] nenhuma nova fundação paralela foi introduzida
- [ ] testes existentes continuam passando

---

## 11. Conclusão

A PR 36 aprofunda a utilidade prática da base local sem mudar a direção arquitetural já aprovada. Depois de validar o primeiro consumo funcional, o sistema passa a consumir melhor o contexto disponível dentro do mesmo fluxo compartilhado.

A entrega permanece pequena, objetiva e revisável: refinar recuperação, melhorar composição contextual e elevar previsibilidade da resposta sem inflar escopo, narrativa ou arquitetura.
