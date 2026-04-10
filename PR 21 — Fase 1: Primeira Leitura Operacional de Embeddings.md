# 🔄 PR 21 — Primeira Leitura Operacional de Embeddings
## Introdução do primeiro retrieval vetorial mínimo sobre a persistência já integrada ao fluxo existente

---

<div align="left">

![PR](https://img.shields.io/badge/PR-21-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-retrieval%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-primeira%20leitura%20vetorial-1d4ed8?style=for-the-badge&logo=databricks&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-minimal%20vector%20search-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR adiciona apenas a primeira leitura operacional mínima sobre a persistência vetorial já integrada anteriormente ao fluxo existente.
>
> - mantém a fundação vetorial local já introduzida anteriormente
> - mantém a integração mínima de escrita entre ingestion e embeddings já validada na etapa anterior
> - adiciona o primeiro caminho mínimo de leitura vetorial a partir de uma query textual
>
> **Este PR não introduz re-ranking, chunking, retrieval híbrido, filtros sofisticados, paginação expandida, múltiplas estratégias de busca, pipeline de busca completo ou abstrações adicionais de search.**

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

A PR 20 consolidou a primeira integração operacional mínima entre ingestion, geração de embeddings e persistência vetorial, fazendo com que o sistema deixasse de ter apenas a fundação local e passasse a gravar embeddings dentro de um fluxo real da aplicação.

A PR 21 adiciona exatamente o próximo passo mínimo correto nessa evolução: usar a persistência vetorial já validada para executar a primeira leitura operacional mínima a partir de uma query textual, fechando o primeiro ciclo básico de uso entre escrita e leitura vetorial.

A mudança aqui não cria um subsistema de busca completo nem reabre a arquitetura aprovada anteriormente. O que esta PR faz é validar operacionalmente o primeiro caminho ponta a ponta entre:

- entrada textual mínima para consulta
- geração de embedding da query
- busca por similaridade sobre a base vetorial já persistida
- retorno mínimo dos registros mais próximos

Com isso, o sistema passa a ter o primeiro uso real da persistência vetorial para leitura, mantendo o recorte pequeno, explícito e revisável.

---

## 2. Objetivo do PR

Este PR tem como objetivo prático:

- introduzir a primeira consulta vetorial mínima sobre `document_embeddings`
- receber uma query textual simples como entrada
- gerar o embedding correspondente à query
- executar busca por similaridade usando a base vetorial já persistida
- retornar os registros mais próximos em um formato mínimo e direto
- validar que a persistência introduzida anteriormente agora pode ser utilizada em um fluxo real de leitura

---

## 3. Decisão Arquitetural

A decisão arquitetural desta PR é manter integralmente o desenho incremental já aprovado e adicionar apenas o próximo passo funcional mínimo sobre ele.

Em termos práticos, isso significa:

- manter a base vetorial local já introduzida anteriormente
- manter a integração de escrita introduzida na PR anterior sem redesenho do fluxo existente
- reutilizar o `EmbeddingGeneratorService` para geração do embedding da query
- reutilizar o módulo de embeddings já existente como ponto de entrada do retrieval mínimo
- adicionar apenas o caminho mínimo necessário para busca por similaridade no `DocumentEmbeddingsDao`

A implementação não cria camada paralela de search, não introduz engine de retrieval separada e não embute ranking sofisticado dentro desta entrega. A leitura acontece sobre a base já existente, no menor recorte possível, apenas para validar o primeiro uso operacional da persistência vetorial.

---

## 4. Escopo

Entra neste PR:

- introdução do primeiro fluxo mínimo de query vetorial
- recebimento de uma `query` textual simples como entrada
- geração de embedding da query via `EmbeddingGeneratorService`
- adição de método mínimo de busca por similaridade no `DocumentEmbeddingsDao`
- retorno mínimo dos registros mais próximos persistidos em `document_embeddings`
- conexão mínima do fluxo de leitura ao módulo de embeddings já existente
- cobertura de testes para o novo fluxo de busca vetorial mínima
- preservação integral do fluxo de escrita introduzido anteriormente

---

## 5. Fora de Escopo

Não entra neste PR:

- re-ranking
- retrieval híbrido
- chunking de documentos
- busca com filtros compostos
- paginação sofisticada
- múltiplas estratégias de similaridade
- múltiplos embeddings por documento
- composição com outros módulos de domínio
- API pública inflada para busca semântica
- cache de busca
- observabilidade expandida específica para retrieval
- abstrações genéricas de search engine
- pipeline completo de retrieval
- qualquer expansão arquitetural além do primeiro uso mínimo da leitura vetorial

Esta seção é propositalmente restritiva para proteger o recorte implementado e evitar interpretação acima da entrega real.

---

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Query textual simples] --> B[EmbeddingGeneratorService]
    B --> C[Gera embedding da query]
    C --> D[DocumentEmbeddingsDao]
    D --> E[Busca similaridade em document_embeddings]
    E --> F[Retorna resultados mínimos ordenados]

    classDef dark fill:#0f172a,stroke:#22d3ee,color:#e5f9ff,stroke-width:1.5px;
    class A,B,C,D,E,F dark;
```

Leitura do fluxo:

1. uma query textual simples entra no fluxo de leitura
2. o embedding da query é gerado usando o serviço mínimo já existente
3. o vetor gerado é enviado para a camada de persistência vetorial
4. a busca por similaridade é executada em `document_embeddings`
5. os registros mais próximos são retornados no formato mínimo definido para esta fase

---

## 7. Contratos Mínimos

Do ponto de vista externo, esta PR evita inflar contratos públicos. O recorte permanece concentrado em contratos mínimos necessários para validar a primeira leitura vetorial operacional.

Contratos mínimos envolvidos:

### Entrada mínima da busca
A leitura vetorial passa a exigir apenas:

```ts
{
  query: string
}
```

como shape mínimo necessário para o recorte atual.

### Geração mínima da query
A geração do embedding da consulta continua produzindo:

```ts
number[]
```

sem adicionar metadados extras, múltiplos vetores ou estratégias alternativas.

### Saída mínima da leitura
A leitura vetorial retorna uma coleção simples de resultados compatível com o uso atual do sistema, por exemplo:

```ts
Array<{
  id: string
  content: string
  similarity?: number
}>
```

onde:

- `id` representa o identificador persistido do documento vetorial
- `content` representa o conteúdo textual associado ao embedding
- `similarity` pode ser retornado apenas se já fizer parte do caminho mínimo implementado

O princípio mantido aqui é simples:

- não inflar contrato de resposta sem necessidade
- não introduzir estrutura rica de retrieval antes da hora
- retornar apenas o mínimo necessário para validar o uso real da leitura vetorial

---

## 8. Regras de Implementação

Para manter aderência ao padrão do projeto, esta PR segue os seguintes guardrails:

- o fluxo de leitura permanece simples, direto e explícito
- `EmbeddingGeneratorService` é reutilizado sem camada paralela de geração
- `DocumentEmbeddingsDao` concentra apenas a operação mínima de busca vetorial
- nenhum orchestrator novo foi criado para compor o retrieval
- nenhuma abstraction layer de search foi criada por antecipação
- nenhuma fase futura foi embutida nesta entrega
- o fluxo principal continua visível, pequeno e fácil de revisar
- a leitura vetorial é introduzida como uso mínimo real da base já existente

A regra central desta PR é: usar o que já foi persistido, mas no menor recorte possível e sem transformar esta etapa em um pipeline completo de busca semântica.

---

## 9. Critérios de Review

O review desta PR deve validar principalmente:

- se a PR continua a base anterior sem reabrir arquitetura já aprovada
- se a leitura vetorial foi introduzida como continuação natural da persistência já existente
- se o fluxo da query permaneceu explícito e fácil de acompanhar
- se `EmbeddingGeneratorService` foi reutilizado sem abstração prematura
- se a busca por similaridade ficou restrita ao `DocumentEmbeddingsDao`
- se o retorno permaneceu mínimo, sem inflar contrato de retrieval
- se o módulo de embeddings permaneceu simples e coerente com o slice
- se o escopo permaneceu controlado, sem re-ranking, chunking ou pipeline expandido
- se os testes refletem o novo fluxo mínimo de leitura vetorial

---

## 10. Critérios de Aceite

- [ ] O sistema passa a aceitar uma `query` textual simples para consulta vetorial
- [ ] O embedding da query é gerado usando o `EmbeddingGeneratorService`
- [ ] A busca por similaridade é executada sobre `document_embeddings`
- [ ] O fluxo retorna os registros mais próximos em formato mínimo
- [ ] O caminho de leitura reutiliza a base vetorial já persistida anteriormente
- [ ] Não foram introduzidos re-ranking, chunking, retrieval híbrido ou abstrações genéricas além do necessário
- [ ] O módulo de embeddings permanece pequeno, explícito e revisável
- [ ] Os testes cobrem o novo fluxo mínimo de leitura vetorial

---

## 11. Conclusão

A PR 21 representa a continuidade natural da persistência vetorial já introduzida e conectada ao fluxo real da aplicação nas etapas anteriores.

Se a etapa anterior validou a escrita ponta a ponta entre ingestion, geração de embedding e persistência vetorial, esta entrega adiciona o próximo passo funcional correto: usar a base já persistida para executar a primeira leitura vetorial mínima a partir de uma query textual simples.

O ganho desta PR é objetivo e proporcional ao slice implementado: o sistema deixa de ter apenas write-path vetorial e passa a validar também o primeiro read-path real sobre a estrutura já existente, sem redesenho, sem inflar contratos e sem antecipar fases futuras. O resultado é uma evolução pequena, funcional, revisável e coerente com o padrão técnico já consolidado no projeto.
