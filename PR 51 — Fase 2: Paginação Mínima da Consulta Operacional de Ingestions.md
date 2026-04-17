# ⚖️ PR 51 — Fase 2: Primeira Composição Funcional Mínima com Busca Legal
## Inclusão incremental do LegalSearchAgent após classificação e resolução de IDs sem ampliar a pipeline operacional

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

A PR 50 comprovou que os agents já consolidados na fase conseguem operar em cadeia mínima entre classificação e resolução de IDs. Esse passo validou composição funcional real sem abrir nova arquitetura.

A PR 51 adiciona a próxima evolução incremental: após classificar e resolver os identificadores necessários, o fluxo passa a executar a busca legal. O resultado continua pequeno e revisável, mas agora entrega contexto normativo associado à questão processada.

---

## 2. Objetivo do PR

- incluir `LegalSearchAgent` na composição existente
- manter `ClassificationAgent` como primeira etapa
- manter `IdResolutionAgent` como segunda etapa
- executar busca legal como próxima etapa sequencial
- retornar output agregado com `metadata`, `ids` e `legalSearch`
- validar a cadeia completa por testes
- preservar isolamento da pipeline operacional atual

---

## 3. Decisão Arquitetural

A arquitetura aprovada é mantida. Em vez de introduzir orchestrator dedicado ou state machine, a evolução ocorre no mesmo fluxo simples já criado na PR 50, adicionando apenas a próxima etapa funcional necessária.

A decisão preserva o princípio do projeto: evoluir por composições pequenas e reais antes de sofisticar coordenação.

---

## 4. Escopo

- evoluir o agent de composição atual
- injetar `LegalSearchAgent`
- manter etapas anteriores inalteradas
- executar busca legal após classificação e resolução
- agregar resultado da busca no output final
- adicionar testes cobrindo o novo encadeamento
- manter providers consistentes no módulo atual

---

## 5. Fora de Escopo

- integração com `IngestionProcessor`
- integração com `ContentService`
- `StatementNormalizationAgent` no fluxo principal
- `AnswerKeyAgent` no fluxo principal
- LangGraph operacional
- persistência externa nova
- observabilidade expandida
- retries e DLQ
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

---

## 7. Contratos Mínimos

```ts
export type InitialQuestionProcessingOutput = {
  metadata: QuestionMetadata;
  ids: ResolvedIds;
  legalSearch: LegalSearchResult | null;
};
```

Os contratos existentes permanecem os mesmos. Esta PR apenas amplia o output da composição para incluir o resultado mínimo da busca legal.

---

## 8. Regras de Implementação

O fluxo deve continuar explícito e sequencial. A responsabilidade da composição permanece restrita a coordenar chamadas entre agents e devolver o resultado agregado, sem absorver persistência, sem criar abstrações genéricas de pipeline e sem preparar etapas futuras.

A busca legal deve consumir somente os dados necessários produzidos nas etapas anteriores. Falhas devem emergir de forma transparente, mantendo diagnóstica simples e baixo custo de manutenção.

---

## 9. Critérios de Review

Validar se a PR continua diretamente a 50, se o recorte segue pequeno e se a inclusão de `LegalSearchAgent` realmente adiciona valor funcional sem inflar a solução.

Confirmar também que a cadeia está clara, que os testes cobrem a nova etapa e que não houve expansão indevida para ingestion, LangGraph ou pipeline maior.

---

## 10. Critérios de Aceite

- [ ] `LegalSearchAgent` foi integrado ao fluxo atual
- [ ] classificação continua executando primeiro
- [ ] resolução de IDs continua recebendo os metadados corretos
- [ ] busca legal executa após as etapas anteriores
- [ ] output final retorna `metadata`, `ids` e `legalSearch`
- [ ] testes cobrem a cadeia completa
- [ ] nenhuma alteração indevida em ingestion
- [ ] nenhuma orquestração complexa foi adicionada

---

## 11. Conclusão

A PR 51 evolui a composição mínima da PR 50 para um fluxo funcional mais útil ao domínio, conectando a questão processada ao seu contexto legal. O ganho vem por uma única etapa adicional, sem ampliar desnecessariamente a arquitetura.

O recorte permanece pequeno, coerente com a fase e alinhado ao histórico incremental do projeto.
