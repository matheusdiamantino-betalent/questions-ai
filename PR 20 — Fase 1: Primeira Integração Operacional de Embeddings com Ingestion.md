# 🔄 PR 20 — Primeira Integração Operacional de Embeddings com Ingestion
## Conexão mínima entre geração de embeddings e persistência vetorial no fluxo existente

---

<div align="left">

![PR](https://img.shields.io/badge/PR-20-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-integration%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-primeira%20integracao%20vetorial-1d4ed8?style=for-the-badge&logo=databricks&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-embedding%20persistence%20flow-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR adiciona apenas a primeira integração operacional mínima entre o fluxo já existente de ingestion e a persistência de embeddings já disponível no projeto.
>
> - mantém a fundação vetorial local já introduzida anteriormente
> - conecta a geração de embedding ao fluxo real já existente de ingestion
> - persiste o embedding gerado usando o caminho mínimo já implementado no módulo de embeddings
>
> **Este PR não introduz retrieval, chunking, ranking, pipeline vetorial completo, múltiplos embeddings por ingestion, abstrações adicionais ou expansão de observabilidade.**

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

A PR 19 consolidou a fundação mínima para persistência vetorial local, deixando a base pronta para armazenar embeddings sem ainda conectá-la a um fluxo operacional real da aplicação.

A PR 20 adiciona exatamente o próximo passo mínimo correto nessa evolução: usar o fluxo já existente de ingestion como ponto de integração para gerar um embedding a partir do conteúdo processado e persisti-lo na tabela vetorial já disponível.

A mudança aqui não cria um novo subsistema nem reabre a arquitetura anterior. O que esta PR faz é validar operacionalmente o primeiro caminho ponta a ponta entre:

- ingestion já persistida e enfileirada
- processamento já existente
- geração mínima de embedding
- persistência vetorial associada ao item processado

Com isso, o sistema passa a ter a primeira integração real entre processamento e armazenamento vetorial, mantendo o recorte pequeno, explícito e revisável.

---

## 2. Objetivo do PR

Este PR tem como objetivo prático:

- integrar a geração de embeddings ao fluxo atual de processing da ingestion
- extrair o conteúdo textual do payload já persistido
- gerar um embedding mínimo a partir desse conteúdo
- persistir o embedding em `document_embeddings`
- concluir a ingestion somente após a persistência vetorial com sucesso
- manter o fluxo terminal de falha já existente quando ocorrer erro durante o processamento

---

## 3. Decisão Arquitetural

A decisão arquitetural desta PR é manter integralmente o desenho incremental já aprovado e adicionar apenas o próximo passo funcional mínimo sobre ele.

Em termos práticos, isso significa:

- manter a base vetorial local já introduzida anteriormente
- reutilizar o `IngestionProcessor` como ponto real de integração
- gerar o embedding dentro do fluxo de processamento já existente
- persistir o vetor por meio do módulo de embeddings já presente na aplicação
- associar o registro vetorial ao item processado usando o próprio `ingestion.id` como identificador persistido

A implementação não cria um módulo vetorial paralelo, não introduz orchestrator adicional e não desloca a regra principal para fora do fluxo já existente. A integração acontece diretamente no processor, com geração mínima e persistência mínima, no menor recorte possível.

---

## 4. Escopo

Entra neste PR:

- ajuste do `IngestionProcessor` para integrar geração e persistência de embedding
- extração do campo `content` a partir do payload da ingestion
- validação mínima de payload textual antes da geração
- geração mínima de embedding via `EmbeddingGeneratorService`
- persistência do embedding via `DocumentEmbeddingsService` e `DocumentEmbeddingsDao`
- uso de `ingestion.id` como `id` do registro vetorial persistido
- ajuste de wiring entre `IngestionModule` e `EmbeddingsModule`
- atualização dos testes do processor para refletir o novo fluxo
- manutenção da persistência vetorial em `document_embeddings`

---

## 5. Fora de Escopo

Não entra neste PR:

- retrieval semântico
- busca vetorial como fluxo funcional novo
- ranking ou re-ranking
- chunking de conteúdo
- múltiplos embeddings por ingestion
- pipeline vetorial completo
- provider real sofisticado para embeddings
- fallback, retry ou DLQ
- abstrações genéricas para storage vetorial
- orquestração separada para embeddings
- redesign do fluxo de ingestion
- expansão transversal do uso de embeddings para outros módulos
- observabilidade expandida específica do fluxo vetorial
- qualquer camada paralela de `vector-database` sem uso real no fluxo final

Esta seção é propositalmente restritiva para proteger o recorte implementado e evitar interpretação acima da entrega real.

---

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Ingestion já persistida e enfileirada] --> B[IngestionProcessor]
    B --> C[Extrai content do payload]
    C --> D[Gera embedding mínimo]
    D --> E[Persiste em document_embeddings]
    E --> F[Atualiza ingestion para completed]

    classDef dark fill:#0f172a,stroke:#22d3ee,color:#e5f9ff,stroke-width:1.5px;
    class A,B,C,D,E,F dark;
```

Leitura do fluxo:

1. o job já existente da ingestion é consumido normalmente
2. o processor valida a ingestion e move o status para `processing`
3. o conteúdo textual é extraído do payload persistido
4. o embedding é gerado no próprio fluxo operacional atual
5. o vetor é persistido em `document_embeddings`
6. a ingestion só então é concluída como `completed`
7. se qualquer etapa falhar após entrar em `processing`, a ingestion é marcada como `failed` com `failureReason`

---

## 7. Contratos Mínimos

Do ponto de vista externo, esta PR não infla contratos públicos já existentes. O recorte permanece concentrado em contratos internos mínimos necessários para geração e persistência do embedding.

Contratos mínimos envolvidos:

### Contrato operacional da ingestion
A ingestion continua trabalhando com payload genérico, mas o processamento desta PR passa a exigir a presença de:

```ts
{
  content: string
}
```

como shape mínimo necessário para o recorte atual.

### Saída mínima da geração
A geração de embeddings produz:

```ts
number[]
```

sem introduzir metadados adicionais, múltiplos vetores ou contratos mais ricos.

### Persistência mínima
A persistência vetorial ocorre com o shape:

```ts
{
  id: string
  content: string
  embedding: number[]
}
```

onde:

- `id` = `ingestion.id`
- `content` = conteúdo textual processado
- `embedding` = vetor numérico gerado

O princípio mantido aqui é simples:

- não ampliar contratos externos sem necessidade
- não introduzir payloads artificiais
- persistir apenas o mínimo necessário para validar a integração operacional da fase

---

## 8. Regras de Implementação

Para manter aderência ao padrão do projeto, esta PR segue os seguintes guardrails:

- controller permanece fino e fora da regra vetorial
- o `IngestionProcessor` concentra explicitamente o fluxo principal da integração
- `EmbeddingGeneratorService` existe apenas como etapa mínima de geração
- `DocumentEmbeddingsService` permanece como serviço simples de aplicação para persistência
- `DocumentEmbeddingsDao` permanece restrito à persistência e busca vetorial
- nenhuma abstração nova foi criada apenas por antecipação
- nenhum módulo paralelo de persistência vetorial foi mantido no fluxo final
- nenhuma fase futura foi embutida dentro desta entrega
- o fluxo principal continua visível, direto e fácil de revisar

A regra central desta PR é: integrar de forma real, mas no menor recorte possível e usando o caminho já consolidado no projeto.

---

## 9. Critérios de Review

O review desta PR deve validar principalmente:

- se a PR continua a base anterior sem reabrir arquitetura já aprovada
- se a integração entre ingestion e embeddings foi feita dentro do fluxo já existente
- se o `IngestionProcessor` permaneceu explícito e fácil de acompanhar
- se o payload textual mínimo foi tratado sem inflar contratos públicos
- se a geração de embedding entrou sem abstração prematura
- se a persistência vetorial ficou restrita ao `DocumentEmbeddingsService` e ao `DocumentEmbeddingsDao`
- se o `EmbeddingsModule` e o `IngestionModule` ficaram corretamente conectados
- se o escopo permaneceu controlado, sem retrieval, chunking ou pipeline vetorial expandido
- se os testes refletem o novo fluxo e os cenários de falha relevantes da integração

---

## 10. Critérios de Aceite

- [ ] O fluxo de ingestion passa a extrair `content` do payload durante o processamento
- [ ] O processor gera um embedding mínimo a partir do conteúdo processado
- [ ] O embedding gerado é persistido em `document_embeddings`
- [ ] O registro vetorial é persistido usando `ingestion.id` como identificador
- [ ] A ingestion só é marcada como `completed` após a persistência do embedding
- [ ] Erros após entrada em `processing` continuam marcando a ingestion como `failed` com `failureReason`
- [ ] `EmbeddingsModule` e `IngestionModule` ficam alinhados ao novo fluxo
- [ ] Não foram introduzidos retrieval, chunking, ranking, retries ou abstrações genéricas além do necessário

---

## 11. Conclusão

A PR 20 representa a continuidade natural da fundação vetorial já introduzida anteriormente.

Se a etapa anterior deixou pronta a base mínima de persistência vetorial local, esta entrega adiciona o próximo passo funcional correto: colocar essa base em uso dentro de um fluxo real já existente da aplicação, conectando ingestion, geração de embeddings e persistência vetorial no menor recorte operacional possível.

O ganho desta PR é objetivo e proporcional ao slice implementado: o sistema passa a ter a primeira integração vetorial ponta a ponta dentro do processamento real da ingestion, sem redesenho, sem inflar contratos e sem antecipar fases futuras. O resultado é uma evolução pequena, funcional, revisável e coerente com o padrão técnico já consolidado no projeto.
