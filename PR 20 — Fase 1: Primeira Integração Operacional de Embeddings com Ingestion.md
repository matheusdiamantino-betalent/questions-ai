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
> Esta PR adiciona apenas a primeira integração operacional mínima entre o fluxo já existente de ingestion e a base vetorial local introduzida anteriormente.
>
> - mantém a base vetorial local já estabelecida na PR 19
> - conecta a geração de embeddings a um fluxo real já existente da aplicação
> - persiste o resultado vetorial sem expandir a arquitetura além do recorte atual
>
> **Este PR não introduz retrieval, chunking, ranking, pipeline vetorial completo, abstrações adicionais ou expansão de observabilidade.**

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

A PR 19 consolidou a fundação mínima de persistência vetorial local, mantendo o recorte restrito à estrutura necessária para armazenar embeddings sem ainda conectá-la a um fluxo real da aplicação.

Esta PR 20 dá o próximo passo mínimo correto nessa linha evolutiva: usar a base já introduzida em um fluxo existente de ingestion, conectando geração de embeddings e persistência vetorial dentro do processamento já estabelecido.

A decisão aqui não é abrir uma nova frente arquitetural, mas validar operacionalmente a primeira integração real entre:

- o fluxo já existente de ingestion
- a geração mínima de embeddings
- a persistência vetorial local já disponível

Com isso, a aplicação passa a ter um primeiro caminho funcional ponta a ponta entre conteúdo processado e registro vetorial persistido, sem introduzir retrieval, chunking, orquestração paralela ou qualquer expansão indevida da fase.

---

## 2. Objetivo do PR

Este PR tem como objetivo prático:

- integrar a geração de embeddings ao fluxo atual de ingestion
- persistir o vetor gerado na base vetorial local já introduzida
- validar um primeiro caminho operacional ponta a ponta entre processamento e armazenamento vetorial
- manter o fluxo visível, simples e aderente ao padrão já aprovado do projeto

---

## 3. Decisão Arquitetural

A decisão arquitetural desta PR é manter integralmente a base já aprovada e adicionar apenas o próximo passo funcional mínimo sobre ela.

Em termos práticos, isso significa:

- manter a fundação vetorial local introduzida na PR 19
- reutilizar o fluxo já existente de ingestion como ponto de integração
- executar a geração de embeddings dentro do fluxo já controlado da aplicação
- persistir o resultado vetorial de forma direta, sem criar camadas cosméticas ou infraestrutura paralela

Esta PR não redefine arquitetura, não reprojeta o pipeline de ingestion e não cria um subsistema vetorial separado. Ela apenas conecta duas partes já previstas no desenho incremental da fase: processamento existente e persistência vetorial mínima.

---

## 4. Escopo

Entra neste PR:

- integração mínima entre ingestion e geração de embeddings
- persistência do embedding gerado na base vetorial local
- associação operacional entre o item processado e o registro vetorial persistido
- validação do fluxo ponta a ponta dentro do caminho já existente de processamento
- manutenção do comportamento geral da aplicação fora do recorte vetorial introduzido aqui

---

## 5. Fora de Escopo

Não entra neste PR:

- retrieval semântico
- busca vetorial
- ranking ou re-ranking
- chunking de conteúdo
- pipeline vetorial completo
- múltiplos providers de embeddings
- estratégias de fallback, retry ou DLQ
- abstrações genéricas para storage vetorial
- orquestração separada para embeddings
- expansão transversal do uso de embeddings para outros módulos
- observabilidade expandida específica para o fluxo vetorial
- qualquer redesenho do fluxo principal de ingestion

Esta seção é propositalmente restritiva para preservar a natureza incremental da entrega e evitar leitura exagerada do escopo.

---

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Ingestion já persistida] --> B[Processamento existente]
    B --> C[Geração mínima de embedding]
    C --> D[Persistência vetorial local]
    D --> E[Fluxo concluído com integração real]

    classDef dark fill:#0f172a,stroke:#22d3ee,color:#e5f9ff,stroke-width:1.5px;
    class A,B,C,D,E dark;
```

Leitura do fluxo:

1. a ingestion segue entrando pelo caminho já existente
2. o processamento atual alcança o ponto mínimo em que o conteúdo pode ser transformado em embedding
3. o embedding é gerado no próprio fluxo operacional já controlado
4. o resultado é persistido na base vetorial local
5. a integração ponta a ponta passa a existir sem necessidade de expandir a arquitetura além desse acoplamento mínimo

---

## 7. Contratos Mínimos

Do ponto de vista externo, este PR não precisa inflar contratos públicos já existentes. O foco está na integração interna mínima entre processamento e persistência vetorial.

Contratos mínimos envolvidos no recorte:

- **entrada operacional do fluxo**: conteúdo textual já disponível no processamento da ingestion
- **saída mínima da geração**: vetor numérico gerado a partir do conteúdo processado
- **persistência mínima**: registro vetorial associado ao item de ingestion que originou a geração

O princípio aqui é simples:

- não inventar novos contratos externos sem necessidade
- não ampliar payloads públicos apenas para acomodar a integração interna
- introduzir somente os dados estritamente necessários para gerar e persistir o embedding

Se já houver contrato interno consolidado para o item processado, ele é mantido e apenas enriquecido no ponto mínimo necessário para a persistência vetorial.

---

## 8. Regras de Implementação

Para manter aderência ao padrão do projeto, esta PR deve seguir os seguintes guardrails:

- controller permanece fino e focado no boundary HTTP, sem absorver regra vetorial
- service/processador concentra o fluxo operacional de forma explícita e legível
- repository/DAO permanece restrito à persistência, sem decidir o fluxo de negócio
- geração de embeddings entra apenas como passo mínimo necessário dentro do processamento atual
- persistência vetorial ocorre sem wrappers paralelos ou camada genérica prematura
- nenhuma abstração deve ser criada apenas por antecipação de evolução futura
- nenhuma fase futura deve ser escondida dentro desta entrega
- o fluxo principal precisa continuar visível, direto e fácil de revisar

A regra central desta PR é: integrar de forma real, mas no menor recorte possível.

---

## 9. Critérios de Review

O review desta PR deve validar principalmente:

- se a PR realmente continua a PR 19 sem reabrir arquitetura já aprovada
- se a integração entre ingestion e persistência vetorial foi feita no menor recorte funcional possível
- se o fluxo principal continua explícito e fácil de acompanhar
- se a geração de embeddings não introduz camadas artificiais ou abstrações sem uso real
- se a persistência vetorial ficou restrita ao papel de persistência, sem assumir orquestração indevida
- se o escopo permaneceu controlado, sem embutir retrieval, chunking ou pipeline completo
- se a entrega está proporcional ao slice e pronta para revisão rápida

---

## 10. Critérios de Aceite

- [ ] A base vetorial local previamente introduzida continua sendo reutilizada sem redesign
- [ ] O fluxo de ingestion passa a acionar a geração de embedding dentro do processamento já existente
- [ ] O embedding gerado é persistido com associação operacional ao item processado
- [ ] O fluxo ponta a ponta é validável localmente sem necessidade de componentes adicionais fora do recorte
- [ ] Não foram introduzidos retrieval, chunking, ranking, retries ou abstrações genéricas além do necessário
- [ ] O fluxo principal permanece simples, explícito e aderente ao padrão já aprovado do projeto

---

## 11. Conclusão

A PR 20 representa a continuidade natural da PR 19.

Se a PR anterior consolidou a fundação mínima de persistência vetorial local, esta entrega adiciona o próximo passo funcional correto: colocar essa base em uso dentro de um fluxo real já existente da aplicação, conectando ingestion, geração de embeddings e armazenamento vetorial no menor recorte operacional possível.

O ganho desta PR é objetivo e proporcional ao slice: a aplicação passa a ter uma primeira integração vetorial real, sem redesenho, sem inflar contratos e sem antecipar fases futuras. O resultado é uma evolução pequena, funcional, revisável e coerente com o padrão técnico já consolidado no projeto.
