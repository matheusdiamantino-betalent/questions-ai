# ⚖️ PR 51 — Fase 2: Primeira Composição Funcional Mínima com Busca Legal
## Inclusão incremental do `LegalSearchAgent` após classificação e resolução de IDs sem ampliar a pipeline operacional

---

<div align="left">

![PR](https://img.shields.io/badge/PR-51-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-composicao%20com%20busca%20legal-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 50. Após validar a composição mínima entre classificação e resolução de IDs, o próximo passo correto é enriquecer esse fluxo com a etapa de busca legal. O objetivo é conectar a questão processada à sua referência normativa de forma simples, previsível e testável, sem reabrir ingestion e sem introduzir orquestração complexa.
>
> - adiciona `LegalSearchAgent` ao fluxo já existente
> - amplia valor funcional sem inflar a arquitetura
> - mantém composição sequencial e leitura simples
> - preserva o boundary de agents como eixo principal da fase
>
> **Este PR não implementa LangGraph operacional, integração com ingestion, adaptação de enunciado, geração de gabarito, retries distribuídos ou pipeline final de questões.**

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

A PR 50 comprovou a composição funcional mínima entre `ClassificationAgent` e `IdResolutionAgent`, validando que a fase de agents básicos já consegue produzir encadeamento útil sem abrir coordenação mais sofisticada.

A PR 51 avança exatamente um passo além desse ponto. Após classificar a questão e resolver os identificadores derivados do metadata, o fluxo passa a executar a busca legal como terceira etapa sequencial. O ganho é direto: a composição deixa de apenas estruturar e identificar a questão e passa também a associá-la ao contexto normativo correspondente, sem alterar os boundaries já estabilizados.

Esse é o próximo passo mínimo correto porque adiciona valor funcional real ao fluxo atual, preservando simplicidade, previsibilidade e baixo custo de review.

---

## 2. Objetivo do PR

- incluir `LegalSearchAgent` na composição funcional atual
- preservar `ClassificationAgent` como primeira etapa do fluxo
- preservar `IdResolutionAgent` como segunda etapa do fluxo
- executar a busca legal após a resolução de IDs
- retornar output agregado com `metadata`, `ids` e `legalSearch`
- validar a cadeia completa por testes
- manter a evolução isolada da pipeline operacional de ingestion

---

## 3. Decisão Arquitetural

A arquitetura já aprovada é mantida integralmente. Esta PR não introduz orchestrator dedicado, state machine, composição genérica de steps nem nova camada de coordenação. A decisão central é evoluir o mesmo fluxo explícito da PR 50 com apenas mais uma etapa funcional, mantendo a composição como um encadeamento direto entre agents.

Com isso, a fase continua avançando por slices reais e pequenos: primeiro a composição entre classificação e resolução, agora a inclusão da busca legal sobre o resultado já estruturado. O desenho permanece visível, fácil de revisar e proporcional ao estágio atual do projeto.

---

## 4. Escopo

- evoluir o fluxo de composição atual
- injetar `LegalSearchAgent` no mesmo boundary já existente
- manter classificação e resolução de IDs como etapas anteriores inalteradas
- executar busca legal após a obtenção dos identificadores necessários
- agregar o resultado da busca ao output final da composição
- adicionar testes cobrindo o novo encadeamento
- manter providers e wiring consistentes no módulo atual

---

## 5. Fora de Escopo

- integração com `IngestionProcessor`
- integração com `ContentService`
- inclusão de `StatementNormalizationAgent` no fluxo principal
- inclusão de `AnswerKeyAgent` no fluxo principal
- LangGraph operacional
- persistência adicional
- observabilidade expandida
- retries, DLQ e coordenação distribuída
- paralelização de etapas
- pipeline final de geração de questões

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
    A[ExtractedQuestion] --> B[ClassificationAgent]
    B --> C[QuestionMetadata]
    C --> D[IdResolutionAgent]
    D --> E[ResolvedIds]
    E --> F[LegalSearchAgent]
    F --> G[LegalSearchResult]
    C --> H[Output Agregado]
    E --> H
    G --> H
```

O fluxo permanece linear e explícito. A composição recebe a questão já extraída, classifica, resolve identificadores e então executa a busca legal antes de consolidar o output final. Não há bifurcação operacional nova nem ampliação indevida do processo.

---

## 7. Contratos Mínimos

```ts
export type InitialQuestionProcessingOutput = {
  metadata: QuestionMetadata;
  ids: ResolvedIds;
  legalSearch: LegalSearchResult | null;
};
```

Os contratos anteriores permanecem válidos. Esta PR apenas amplia o output mínimo da composição para incluir `legalSearch`, sem remodelar os tipos já aprovados e sem introduzir contrato paralelo de pipeline.

---

## 8. Regras de Implementação

A composição deve continuar restrita à coordenação sequencial entre agents, com fluxo totalmente visível no ponto de uso. O papel dessa camada é apenas encadear chamadas, repassar os dados mínimos necessários entre etapas e devolver o resultado agregado da execução.

`LegalSearchAgent` deve consumir exclusivamente os dados derivados das etapas anteriores, sem absorver responsabilidades externas ao seu recorte. Não deve haver abstração genérica para steps, preparação de futuras composições, fundação paralela para orchestrator nem acoplamento com ingestion. Quando houver falha, ela deve emergir de forma simples e rastreável, preservando diagnóstica direta e manutenção barata.

---

## 9. Critérios de Review

Validar se a PR continua diretamente a 50 e se a inclusão de `LegalSearchAgent` representa apenas a próxima evolução mínima do fluxo, sem transformar a composição em mecanismo mais amplo do que o necessário.

Confirmar também que a cadeia segue clara e explícita, que o resultado agregado está consistente com o recorte, que os testes cobrem a nova etapa e que não houve expansão para ingestion, LangGraph, normalização de enunciado, geração de gabarito ou outras fases ainda não abertas.

---

## 10. Critérios de Aceite

- [ ] `LegalSearchAgent` foi integrado ao fluxo atual de composição
- [ ] classificação continua executando como primeira etapa
- [ ] resolução de IDs continua executando como segunda etapa
- [ ] busca legal executa após as etapas anteriores
- [ ] output final retorna `metadata`, `ids` e `legalSearch`
- [ ] testes cobrem a cadeia completa com a nova etapa
- [ ] nenhuma alteração indevida foi feita em ingestion
- [ ] nenhuma orquestração complexa foi adicionada

---

## 11. Conclusão

A PR 51 amplia a composição mínima validada na PR 50 com a primeira inclusão funcional de busca legal no encadeamento de agents. O ganho é objetivo: além de classificar e estruturar a questão, o fluxo passa a anexar referência normativa ao resultado processado.

A evolução permanece pequena, coesa e alinhada à fase. Não há redesenho, nem antecipação de pipeline maior. Há apenas o próximo passo funcional mínimo, incorporado com simplicidade e baixo ruído para review.
