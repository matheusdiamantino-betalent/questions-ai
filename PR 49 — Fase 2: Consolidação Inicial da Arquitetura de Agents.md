# 🧩 PR 49 — Fase 2: Consolidação Inicial da Arquitetura de Agents
## Estruturação mínima do fluxo especializado de agents sem acoplamento prematuro ao pipeline de ingestion

---

<div align="left">

![PR](https://img.shields.io/badge/PR-49-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-architecture%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-arquitetura%20inicial%20de%20agents-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR ajusta o rumo da fase após o fechamento do primeiro ciclo operacional de ingestion. O objetivo agora deixa de ser continuar enriquecendo o resultado diretamente dentro do processor e passa a ser consolidar a arquitetura dos agents que futuramente será integrada ao fluxo real.
>
> 1. encerra o ciclo inicial de ingestion como base operacional suficiente
> 2. desloca a evolução da fase para o domínio de agents
> 3. documenta e organiza o fluxo especializado de processamento
> 4. preserva desacoplamento entre pipeline atual e arquitetura futura
>
> **Esta PR não implementa orquestração completa em produção, não amplia a pipeline de ingestion e não introduz integração final com geração de questões.**

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

As PRs anteriores estabilizaram o primeiro ciclo operacional de ingestion como mecanismo de entrada, processamento inicial e exposição do resultado estruturado. Esse ciclo deixa de evoluir por enquanto como centro da fase.

Com o direcionamento técnico atual, o próximo passo correto passa a ser a consolidação do domínio de agents. A PR 49 reorganiza o foco para a arquitetura especializada que sustentará o pipeline de geração de questões, sem ainda acoplar essa estrutura ao fluxo operacional já existente de ingestion.

---

## 2. Objetivo do PR

1. consolidar o foco da fase no domínio de agents
2. estruturar os agents iniciais e suas responsabilidades
3. explicitar o fluxo macro entre os agents previstos
4. preparar o terreno para integração futura com ingestion sem forçar esse acoplamento agora
5. manter a fase incremental, legível e arquiteturalmente coerente

---

## 3. Decisão Arquitetural

A principal decisão desta PR é separar claramente duas camadas do sistema:

- **ingestion**, como frente operacional de entrada e processamento assíncrono
- **agents**, como núcleo especializado de interpretação, classificação, resolução, busca e composição de saída

Em vez de continuar embutindo inteligência incremental dentro do processor de ingestion, a evolução agora se concentra na definição da arquitetura dos agents. Essa abordagem reduz retrabalho, protege o fluxo operacional já estabilizado e evita acoplamento prematuro com uma pipeline que ainda sofrerá ajustes quando for posta em prática ponta a ponta.

---

## 4. Escopo

1. consolidar o arquivo `agents-architecture.md` como referência da fase
2. organizar melhor o boundary de `shared/ai/infra/agents`
3. explicitar os agents previstos no fluxo
4. definir responsabilidades mínimas de cada agent
5. alinhar a arquitetura proposta com o direcionamento técnico atual
6. manter ingestion fora da evolução funcional desta PR, salvo ajustes mínimos de alinhamento documental

---

## 5. Fora de Escopo

1. integrar agents reais ao `ingestion.processor`
2. criar novo worker ou nova fila
3. implementar LangGraph completo em produção
4. persistir saída final de questões neste microserviço
5. construir pipeline fim a fim de geração
6. conectar todos os agents entre si com execução real
7. ampliar o contrato HTTP de ingestion
8. introduzir taxonomias, scoring avançado ou regras de negócio finais

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{
  'background':'#0b1220',
  'primaryColor':'#111827',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#22d3ee',
  'lineColor':'#39ff14',
  'secondaryColor':'#0f172a',
  'tertiaryColor':'#111827'
}}}%%
flowchart LR
    A[PDFs recebidos] --> B[Ingestion assíncrona]
    B --> C[Extração inicial]
    C --> D[Orchestrator Agent]
    D --> E[Classification Agent]
    E --> F[ID Resolution Agent]
    F --> G[Legal Search Agent]
    G --> H[Statement Adaptation Agent]
    H --> I[Answer Key Agent]
    I --> J[Questão estruturada]
    J --> K[Serviço principal MySQL como pending]
```

A ingestion permanece como porta de entrada operacional. A inteligência da fase passa a ser organizada no fluxo de agents, que será integrado posteriormente no momento arquitetural correto.

---

## 7. Contratos Mínimos

O foco desta PR não é aprofundar contratos operacionais de ingestion, mas deixar claro o shape mínimo do domínio de agents para evolução incremental.

```ts
export type Agent<Input, Output> = {
  execute(input: Input): Promise<Output>;
};

export type ExtractedQuestion = {
  statement: string;
  alternatives: string[];
  comments?: string;
  correctAnswer?: string;
};

export type QuestionMetadata = {
  article?: string;
  law?: string;
  bank?: string;
  year?: number;
  position?: string;
  organization?: string;
};

export type ResolvedIds = {
  bankId?: string | null;
  lawId?: string | null;
  articleId?: string | null;
  yearId?: string | null;
};
```

A consolidação aqui é de direção arquitetural: os agents devem evoluir por responsabilidades claras, com contratos pequenos, isolados e compatíveis com futura composição.

---

## 8. Regras de Implementação

1. manter agents como unidades especializadas e pequenas
2. evitar acoplamento prematuro com `content` ou com o `processor` de ingestion
3. não antecipar orquestração completa se o recorte atual ainda é arquitetural
4. preservar DAO apenas para persistência e acesso a dados
5. manter clareza entre responsabilidade operacional e responsabilidade semântica
6. priorizar evolução incremental, com slices que possam ser revisados isoladamente

---

## 9. Critérios de Review

Validar se a PR realmente muda o foco da fase para agents e se deixa claro que ingestion não deve continuar recebendo inteligência incremental agora.

Também deve ser verificado se a arquitetura proposta está coerente com o fluxo esperado do produto, se o arquivo de arquitetura ficou útil como referência prática e se o recorte permaneceu pequeno, sem antecipar integração final nem sofisticar além do necessário.

---

## 10. Critérios de Aceite

- [ ] arquitetura de agents está explicitada de forma clara
- [ ] responsabilidades iniciais dos agents estão delimitadas
- [ ] ingestion permanece preservada como frente operacional
- [ ] não há acoplamento novo entre processor e pipeline futura
- [ ] documentação da fase está alinhada ao direcionamento técnico
- [ ] recorte continua incremental e revisável
- [ ] base preparada para próximas PRs focadas em agents

---

## 11. Conclusão

A PR 49 deixa de aprofundar ingestion e passa a consolidar o núcleo que realmente representa a evolução da fase: a arquitetura de agents.

Esse ajuste reduz risco de retrabalho, respeita o direcionamento técnico do time e mantém a progressão do projeto em um eixo mais consistente com o objetivo final. O avanço continua pequeno, mas agora no lugar correto: estruturação dos agents antes da integração definitiva ao fluxo operacional.
