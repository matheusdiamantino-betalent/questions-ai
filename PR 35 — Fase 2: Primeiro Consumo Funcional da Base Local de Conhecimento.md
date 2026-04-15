# 🧩 PR 35 — Fase 2: Primeiro Consumo Funcional da Base Local de Conhecimento
## Validação do uso real do corpus jurídico local no fluxo compartilhado existente

---

<div align="left">

![PR](https://img.shields.io/badge/PR-35-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-primeiro%20consumo%20knowledge-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-runtime%20consumindo%20base%20local-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é a continuação direta da PR 34. Depois de validar a primeira ingestão vetorial mínima do Código Civil em `shared/ai`, o próximo passo funcional correto é consumir essa base local em um fluxo real já existente, comprovando utilidade prática do corpus persistido sem redesenhar a arquitetura.
>
> - preserva a foundation compartilhada consolidada nas PRs 29 a 34
> - reutiliza a base jurídica já persistida em `document_embeddings`
> - conecta retrieval semântico ao runtime atual por meio das portas já existentes
> - mantém `modules/content` como consumidor fino do boundary compartilhado
>
> **Este PR não introduz múltiplas bases, reranking avançado, hybrid search, planner, memory, citations formais, filtros sofisticados ou nova camada arquitetural paralela.**

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

A PR 34 concluiu o primeiro ciclo mínimo de conhecimento local ao transformar o Código Civil em corpus vetorial persistido dentro de `shared/ai`. Com isso, a aplicação passou a ter uma base jurídica interna pronta para consulta, mas ainda sem consumo funcional em um fluxo real já aprovado.

A continuação natural desse caminho não é ampliar ingestão nem abrir novas frentes arquiteturais. O próximo passo mínimo correto é comprovar que a base persistida consegue participar de uma resposta concreta no runtime compartilhado já existente. Esta PR executa exatamente esse movimento ao conectar a recuperação semântica ao fluxo atual de geração.

O ganho desta entrega é objetivo: o conhecimento persistido deixa de ser apenas um ativo técnico disponível e passa a enriquecer uma resposta real. A arquitetura aprovada permanece a mesma, o consumidor continua fino e a evolução segue pequena, funcional e revisável.

---

## 2. Objetivo do PR

- consumir a base local jurídica em um fluxo funcional já existente
- recuperar trechos semanticamente relacionados à entrada recebida
- compor contexto mínimo no runtime compartilhado antes da geração final
- manter o encapsulamento técnico de retrieval dentro de `shared/ai`
- validar utilidade prática do corpus local sem inflar o recorte

---

## 3. Decisão Arquitetural

A decisão central desta PR é reutilizar a base local por meio das capacidades já expostas pelo runtime compartilhado, evitando que módulos consumidores acessem DAO, vetores, chunks ou detalhes internos de retrieval diretamente.

`modules/content` continua dependente de `AiService` como ponto de entrada funcional. A responsabilidade de consultar embeddings, recuperar contexto e enriquecer o prompt permanece concentrada em `shared/ai`, preservando o shape arquitetural já aprovado nas PRs anteriores.

Com isso, a evolução acontece por continuidade controlada: a foundation existente é mantida, o boundary compartilhado continua dono das capacidades reutilizáveis e o novo comportamento entra sem abrir abstrações artificiais, sem nova API paralela e sem redistribuir responsabilidades.

---

## 4. Escopo

- consultar `document_embeddings` pelo fluxo já existente de busca semântica
- recuperar contexto mínimo relacionado ao input funcional recebido
- incorporar o contexto recuperado ao prompt final de geração
- manter a integração dentro de `AiService` e serviços já existentes do runtime
- preservar `modules/content` como camada fina de entrada e saída
- validar o primeiro uso funcional do corpus jurídico local criado na PR 34

---

## 5. Fora de Escopo

- ingestão de novas fontes de conhecimento
- reranking avançado ou reordenação sofisticada de resultados
- hybrid search com combinação keyword + vector
- thresholds complexos de relevância
- citations formais por trecho recuperado
- memory conversacional
- planner, agent orchestration ou fluxos multi-step expandidos
- LangGraph expandido para múltiplos passos
- cache dedicado para retrieval
- observabilidade expandida específica de search
- nova API pública exclusiva para knowledge base

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Input funcional] --> B[AiService]
    B --> C[EmbeddingsService.search]
    C --> D[document_embeddings]
    D --> E[Contexto recuperado]
    E --> F[Prompt enriquecido]
    F --> G[LangchainModel]
    G --> H[Resposta final]
```

O fluxo permanece linear, pequeno e fácil de revisar. A única evolução desta PR é a participação real da base local jurídica no caminho já existente de geração, sem expansão lateral de arquitetura.

---

## 7. Contratos Mínimos

Os contratos públicos devem permanecer enxutos. O consumidor continua informando apenas o necessário para a geração e, quando aplicável, aciona o caminho já suportado de consulta contextual dentro do runtime compartilhado.

Nenhum contrato externo deve expor vetor, score bruto, estrutura de chunk, schema de armazenamento ou detalhes internos da persistência vetorial. Esses elementos seguem internos ao boundary de `shared/ai`, preservando simplicidade para o consumidor e baixo acoplamento no fluxo funcional.

---

## 8. Regras de Implementação

A implementação deve seguir o menor caminho útil. A recuperação de contexto deve reutilizar `EmbeddingsService.search`, e a composição do prompt final deve permanecer no serviço central já existente, sem introduzir orchestrators paralelos, builders genéricos de prompt, camadas novas de retrieval ou abstrações criadas para uma evolução futura ainda não solicitada.

`modules/content` deve continuar fino e orientado ao fluxo HTTP. `shared/ai` deve seguir como owner das capacidades técnicas reutilizáveis. DAO ou repository, quando envolvidos, devem permanecer restritos à persistência e não se tornar pontos de decisão funcional no fluxo principal.

Se houver dúvida entre expandir estrutura ou manter o caminho principal mais coeso, explícito e legível, esta PR deve favorecer a segunda opção.

---

## 9. Critérios de Review

- a base local passa a ser utilizada em um fluxo funcional real
- `shared/ai` continua owner das capacidades reutilizáveis de retrieval e geração
- consumidores permanecem finos e sem acoplamento a detalhes vetoriais
- o fluxo principal segue explícito, linear e simples de revisar
- não houve abertura de arquitetura paralela ou redistribuição indevida de responsabilidades
- a implementação permaneceu proporcional ao slice atual
- a PR se posiciona como continuação clara e natural da PR 34

---

## 10. Critérios de Aceite

- [ ] consultas funcionais conseguem recuperar contexto do corpus local persistido
- [ ] o contexto recuperado participa da geração final da resposta
- [ ] `modules/content` não acessa DAO nem detalhes internos de retrieval
- [ ] a integração ocorre por serviços já existentes do runtime compartilhado
- [ ] não houve expansão indevida de escopo ou abertura de nova arquitetura
- [ ] testes existentes continuam passando

---

## 11. Conclusão

A PR 35 transforma a base local criada na PR 34 em valor funcional concreto. O conhecimento persistido deixa de ser apenas infraestrutura pronta e passa a enriquecer respostas dentro do fluxo compartilhado já aprovado, preservando a arquitetura existente e o baixo acoplamento entre consumidor e runtime.

A entrega permanece pequena, coerente e revisável: recuperar contexto útil, compor o prompt final e validar uso real do corpus local sem inflar arquitetura, escopo ou narrativa. É a continuação natural da fase e o próximo passo mínimo correto para consolidar a utilidade prática da base jurídica já ingerida.
