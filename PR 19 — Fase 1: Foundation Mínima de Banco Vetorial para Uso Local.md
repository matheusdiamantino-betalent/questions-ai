# 🔄 PR 19 — Foundation Mínima de Banco Vetorial para Uso Local
## Introdução do primeiro suporte operacional mínimo para persistência e consulta de embeddings no ambiente local

---

<div align="left">

![PR](https://img.shields.io/badge/PR-19-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-vector%20store%20foundation-7c3aed?style=for-the-badge&logo=postgresql&logoColor=white)
![Fase](https://img.shields.io/badge/fase-inicio%20do%20modulo%20de%20dados-1d4ed8?style=for-the-badge&logo=databricks&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-local%20vector%20storage-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR introduz apenas a **foundation mínima de banco vetorial para uso local**, adicionando a primeira capacidade operacional real de persistir embeddings e executar consulta básica por similaridade.
>
> Esta entrega inclui:
>
> - configuração funcional de banco com suporte a `pgvector`
> - migration versionada com extensão vetorial
> - persistência simples de registros com embedding
> - consulta básica por similaridade via SQL explícito
> - DAO e service mínimos para uso interno
> - testes reais validando persistência e busca vetorial
>
> **Esta PR não introduz pipeline completo de retrieval, workflow de agents, chunking, ranking avançado, nem abstrações genéricas de vector store.**
>
> O objetivo é abrir a base vetorial do projeto com o menor recorte funcional útil, mantendo o padrão incremental já consolidado.

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
9. [Validação Operacional Local](#9-validação-operacional-local)
10. [Consultas Mínimas](#10-consultas-mínimas)
11. [Uso Operacional Local](#11-uso-operacional-local)
12. [Critérios de Review](#12-critérios-de-review)
13. [Critérios de Aceite](#13-critérios-de-aceite)
14. [Conclusão](#14-conclusão)

---

## 1. Síntese Executiva

A PR 15 introduziu a foundation local de observabilidade.

A PR 16 integrou a aplicação com logging e tracing mínimos.

A PR 17 introduziu a correlação mínima entre logs e traces.

A PR 18 consolidou a primeira leitura operacional guiada da execução correlacionada.

Com esse bloco estabilizado, o próximo passo mínimo correto não é expandir observabilidade nem inflar o fluxo de ingestion, mas abrir o primeiro suporte operacional do módulo de dados vetoriais necessário para o uso futuro de embeddings no ambiente local.

A PR 19 faz exatamente esse avanço.

Ela não tenta resolver retrieval ponta a ponta, não introduz workflow semântico completo e não abre uma camada genérica de vector store.

Ela apenas:

- prepara o banco local com `pgvector`
- versiona a estrutura mínima de persistência vetorial
- adiciona escrita de embeddings
- adiciona leitura simples por similaridade
- valida esse fluxo com teste real contra banco

---

## 2. Objetivo do PR

Permitir que a aplicação passe a ter uma base vetorial mínima, funcional e localmente validável.

### Em termos práticos

Esta PR deve permitir:

- subir o banco com suporte a vetores
- criar a estrutura mínima de persistência vetorial
- inserir um registro com embedding
- consultar registros por similaridade
- validar esse fluxo localmente com testes reais

---

## 3. Decisão Arquitetural

A decisão desta PR é iniciar o módulo vetorial pela menor foundation útil possível.

Isso implica:

- uso de PostgreSQL com `pgvector`
- manutenção do banco já existente, sem infraestrutura paralela
- SQL explícito nas operações vetoriais
- DAO mínimo para persistência
- service mínimo para orquestração
- ausência de provider genérico de vector store
- ausência de adapters, factories ou portabilidade prematura

A base vetorial passa a ser:

> uma capacidade concreta e pequena do projeto, não uma abstração para futuros backends

---

## 4. Escopo

Esta PR inclui:

- ajuste da infraestrutura local para suporte a `pgvector`
- migration com `CREATE EXTENSION IF NOT EXISTS vector`
- criação da tabela `document_embeddings`
- atualização do schema Prisma/Kysely
- DAO mínimo com operações de:
  - inserção de embeddings
  - busca por similaridade
- service mínimo para uso interno
- testes reais de DAO e service
- validação operacional local do fluxo vetorial

---

## 5. Fora de Escopo

Esta PR **não inclui**:

- geração real de embeddings por provider externo
- pipeline completo de retrieval
- chunking de documentos
- ranking
- re-ranking
- múltiplas estratégias de busca
- filtros complexos de metadata
- múltiplos backends de vector store
- abstração genérica para troca de engine
- endpoints HTTP
- observabilidade expandida do módulo vetorial
- redesign de módulos existentes
- qualquer antecipação da próxima fase dentro deste slice

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
    A[Embedding Input]:::app --> B[DocumentEmbeddingsService]:::service
    B --> C[DocumentEmbeddingsDao]:::dao
    C --> D[PostgreSQL + pgvector]:::storage
    D --> E[Similarity Query Result]:::result

    classDef app fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef service fill:#1e1b4b,stroke:#c084fc,color:#f5f3ff,stroke-width:2px;
    classDef dao fill:#082f49,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef storage fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
    classDef result fill:#172554,stroke:#60a5fa,color:#eff6ff,stroke-width:2px;
```

---

## 7. Contratos Mínimos

Esta PR introduz o contrato mínimo da persistência vetorial.

A estrutura persistida passa a conter:

- `id`
- `content`
- `embedding`
- `createdAt`

### Tabela mínima

```sql
CREATE TABLE "document_embeddings" (
    "id" UUID NOT NULL,
    "content" TEXT NOT NULL,
    "embedding" vector(1536) NOT NULL,
    "created_at" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT "document_embeddings_pkey" PRIMARY KEY ("id")
);
```

### Regras

- não introduzir metadata rica nesta fase
- não modelar documento semântico completo
- não acoplar persistência atual a agents
- manter o shape mínimo suficiente para inserção e busca

---

## 8. Regras de Implementação

- manter o banco já existente, sem novo serviço paralelo
- usar `pgvector` por extensão nativa do PostgreSQL
- manter a migration explícita e versionada
- usar SQL explícito nas operações vetoriais
- restringir DAO à persistência e consulta
- restringir service à orquestração mínima
- evitar abstração prematura de vector store
- evitar helpers sem reutilização real
- evitar provider genérico de engine futura
- evitar expansão para retrieval ou ranking

---

## 9. Validação Operacional Local

### Passo a passo mínimo

1. subir o banco local com imagem compatível
2. aplicar migration com extensão `vector`
3. validar criação da tabela `document_embeddings`
4. inserir um registro com embedding
5. consultar embeddings por similaridade
6. confirmar retorno do registro esperado

### Validação coberta nesta PR

A validação real foi feita com:

- migration aplicada no banco local
- geração de tipos Prisma/Kysely
- teste de integração do DAO
- teste do service

---

## 10. Consultas Mínimas

### Inserção vetorial

A escrita do embedding ocorre com cast explícito para `vector`:

```sql
INSERT INTO document_embeddings (id, content, embedding)
VALUES (
  $1,
  $2,
  CAST($3 AS vector)
);
```

### Busca por similaridade

A leitura usa distância vetorial explícita:

```sql
SELECT
  id,
  content,
  1 - (embedding <=> CAST($1 AS vector)) AS similarity
FROM document_embeddings
ORDER BY embedding <=> CAST($1 AS vector)
LIMIT $2;
```

### Objetivo

- manter o fluxo visível
- evitar abstração prematura
- validar persistência e consulta reais com `pgvector`

---

## 11. Uso Operacional Local

### Subida da stack

```bash
docker compose up -d
```

### Migration

```bash
pnpm exec prisma migrate dev --schema="./src/shared/infra/database/schemas/config.prisma"
```

### Geração dos tipos

```bash
pnpm exec prisma generate --schema="./src/shared/infra/database/schemas/config.prisma"
```

### Execução dos testes

```bash
pnpm test -- src/modules/embeddings/infra/dao/document-embeddings.dao.spec.ts
pnpm test -- src/modules/embeddings/infra/services/document-embeddings.service.spec.ts
```

### Banco local

- PostgreSQL + pgvector
- porta local `5433`
- banco `dls_ia`

---

## 12. Critérios de Review

O review deve validar se:

- a PR mantém recorte pequeno e controlado
- `pgvector` foi introduzido sem infraestrutura paralela
- a migration está correta e explícita
- a tabela vetorial é mínima e suficiente
- o DAO usa SQL explícito sem abstração desnecessária
- o service permanece enxuto
- a validação local é real e reproduzível
- não há preparação escondida para fases futuras

---

## 13. Critérios de Aceite

Esta PR pode ser considerada aceita se:

- [ ] o banco local suportar `pgvector`
- [ ] a migration criar a extensão vetorial corretamente
- [ ] a tabela `document_embeddings` existir com shape mínimo
- [ ] embeddings puderem ser persistidos localmente
- [ ] embeddings puderem ser consultados por similaridade
- [ ] o DAO estiver validado por teste real
- [ ] o service mínimo estiver validado
- [ ] nenhuma abstração genérica de vector store tiver sido introduzida
- [ ] o recorte permanecer pequeno, funcional e revisável

---

## 14. Conclusão

A PR 19 introduz o próximo passo correto após a estabilização do bloco inicial de ingestion e observabilidade:

> **abrir a foundation mínima do banco vetorial para uso local com persistência e consulta real de embeddings.**

Em resumo:

- esta PR não resolve retrieval completo
- esta PR não adiciona workflow semântico avançado
- esta PR não introduz abstração de vector store
- esta PR adiciona capacidade real e mínima de persistência vetorial
- esta PR valida consulta simples por similaridade
- o ambiente continua pequeno, explícito e revisável

Esta entrega estabelece a primeira base funcional do módulo vetorial, permitindo evolução incremental controlada nas próximas fases.
