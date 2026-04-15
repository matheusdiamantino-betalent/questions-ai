# 🧩 PR 34 — Fase 2: Base Local Inicial de Conhecimento no Shared AI Runtime
## Primeira ingestão vetorial mínima do Código Civil em corpus jurídico local sobre a foundation compartilhada existente

---

<div align="left">

![PR](https://img.shields.io/badge/PR-34-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-base%20local%20de%20conhecimento-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ingestao%20vetorial%20juridica%20inicial-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua a sequência funcional das PRs 29–33 ao transformar a foundation compartilhada em uma base local concreta de conhecimento. A entrega implementa a primeira ingestão vetorial real usando o Código Civil como corpus inicial, com execução isolada por módulo dedicado para evitar acoplamento indevido ao runtime HTTP principal.
>
> - preserva a arquitetura já aprovada em `shared/ai`
> - materializa ingestão real de fonte jurídica pública e estável
> - persiste chunks vetoriais no banco local
> - introduz módulo separado apenas para execução isolada do processo de ingestão
>
> **Este PR não introduz crawler genérico, múltiplas fontes, scheduler, pipeline expandido, reindexação automática ou redesign arquitetural.**

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

As PRs anteriores consolidaram o boundary compartilhado de IA, validaram consumo real em `modules/content` e introduziram capacidades internas adicionais sem inflar a arquitetura. O próximo passo natural era tornar essa foundation útil com conhecimento local persistido.

Esta PR implementa a primeira ingestão vetorial real a partir do Código Civil compilado no portal do Planalto. O fluxo busca a fonte remota, extrai texto, particiona o conteúdo, gera embeddings placeholder compatíveis com a estrutura atual e persiste os registros em `document_embeddings`.

Também foi introduzido um módulo dedicado de ingestão. A separação não cria nova arquitetura de domínio; ela apenas permite executar o processo em contexto isolado, sem carregar dependências desnecessárias de `ContentModule`, guards HTTP ou fluxos não relacionados. É uma separação pragmática para este job técnico específico.

---

## 2. Objetivo do PR

- implementar a primeira ingestão vetorial real em `shared/ai`
- usar o Código Civil como corpus jurídico inicial
- persistir conhecimento local no banco vetorial existente
- executar a ingestão via runner isolado e módulo mínimo dedicado
- manter `modules/content` sem acoplamento ao processo interno de carga

---

## 3. Decisão Arquitetural

A decisão central desta PR é manter `shared/ai` como owner das capacidades reutilizáveis e tratar a ingestão como processo técnico interno. O conhecimento continua pertencendo ao runtime compartilhado, não ao módulo consumidor.

O `LegalKnowledgeIngestionModule` foi criado para compor apenas as dependências necessárias ao runner: `HttpModule`, DAO, embeddings e service de ingestão. Isso evita reutilizar `ContentModule` como contêiner genérico de providers, reduz ruído de bootstrap e preserva responsabilidade clara entre consumo HTTP e carga técnica de base local.

Não há abertura de subplataforma de jobs nem novo boundary de negócio. Existe apenas isolamento operacional proporcional ao slice atual.

---

## 4. Escopo

- criar `LegalKnowledgeIngestionService`
- criar runner de execução manual da ingestão
- criar módulo dedicado para bootstrap isolado
- buscar HTML do Código Civil no Planalto
- extrair e normalizar texto da página
- gerar chunks com índice sequencial
- gerar embeddings placeholder de dimensão compatível
- persistir `source`, `content`, `chunkIndex` e vetor
- ajustar schema/tabela para novos metadados mínimos

---

## 5. Fora de Escopo

- múltiplas fontes jurídicas
n- crawler genérico
- scheduler recorrente
- fila de ingestão
- retries sofisticados
- deduplicação avançada
- parser universal para PDF/HTML arbitrário
- embeddings reais de provider externo
- busca semântica refinada por relevância jurídica
- interface administrativa de ingestão
- acoplamento direto com `modules/content`

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Runner] --> B[LegalKnowledgeIngestionModule]
    B --> C[Buscar Código Civil]
    C --> D[Extrair Texto]
    D --> E[Chunking]
    E --> F[Embeddings]
    F --> G[document_embeddings]
```

O fluxo permanece linear, explícito e pequeno. O módulo dedicado existe apenas para suportar esse caminho técnico com o mínimo de dependências.

---

## 7. Contratos Mínimos

Metadados persistidos neste slice:

```ts
type DocumentEmbeddingRecord = {
  id: string;
  source: string;
  content: string;
  chunkIndex: number;
}
```

Fonte inicial utilizada:

```ts
source = 'planalto-codigo-civil'
```

Nenhum contrato público novo foi exposto para consumidores HTTP.

---

## 8. Regras de Implementação

- service de ingestão concentra o fluxo principal
- DAO permanece responsável apenas por persistência e consulta
- módulo dedicado compõe dependências mínimas do runner
- sem abstrações prematuras para loaders ou providers de fonte
- sem preparar scheduler, jobs futuros ou múltiplos conectores
- implementação direta, visível e revisável
- `ContentModule` continua focado em consumo funcional HTTP

---

## 9. Critérios de Review

- a ingestão executa de ponta a ponta com sucesso
- registros são persistidos com `source` e `chunkIndex`
- módulo dedicado reduz acoplamento em vez de inflar arquitetura
- `shared/ai` permanece owner da capacidade técnica
- `ContentModule` não foi transformado em módulo genérico
- implementação está proporcional ao slice
- fluxo principal está claro e fácil de revisar

---

## 10. Critérios de Aceite

- [ ] runner executa a ingestão sem subir a aplicação HTTP completa
- [ ] Código Civil é carregado da fonte oficial definida
- [ ] texto é processado e chunkado
- [ ] embeddings são gerados no formato esperado
- [ ] chunks são persistidos em `document_embeddings`
- [ ] metadados `source` e `chunkIndex` são gravados corretamente
- [ ] arquitetura principal permanece preservada
- [ ] testes existentes continuam passando

---

## 11. Conclusão

A PR 34 transforma a foundation compartilhada em utilidade prática concreta ao introduzir a primeira base jurídica local persistida. A entrega mantém o padrão incremental do projeto: pouco ruído, escopo claro e evolução funcional real.

A criação de um módulo separado nesta etapa não expande arquitetura; apenas isola corretamente um processo técnico de ingestão. O resultado é uma base inicial utilizável, construída com simplicidade e aderência ao shape atual do projeto.

