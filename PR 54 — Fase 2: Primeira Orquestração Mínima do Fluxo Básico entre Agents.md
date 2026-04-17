# 🧩 PR 54 — Fase 2: Primeira Orquestração Mínima do Fluxo Básico entre Agents
## Introdução de coordenação inicial da cadeia funcional sem tocar ingestion

---

<div align="left">

![PR](https://img.shields.io/badge/PR-54-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-orquestracao%20minima%20agents-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua a linha evolutiva da PR 53. Após consolidar o fluxo básico entre agents, o próximo passo mínimo correto é explicitar a coordenação da cadeia em uma unidade própria, mantendo o foco integral na fase de agents e sem reabrir ingestion.
>
> - introduz um ponto central simples de coordenação
> - preserva os agents especializados já existentes
> - mantém a execução linear, previsível e testável
>
> **Este PR não toca ingestion, não introduz LangGraph completo, não adiciona filas, retries, paralelismo ou novas responsabilidades operacionais.**

---

## 📌 Sumário

1. Síntese Executiva
2. Objetivo do PR
3. Decisão Arquitetural
4. Escopo
5. Fora de Escopo
6. Fluxo Arquitetural
7. Contratos Mínimos
8. Regras de Implementação
9. Critérios de Review
10. Critérios de Aceite
11. Conclusão

---

## 1. Síntese Executiva

As PRs anteriores da Fase 2 fecharam progressivamente o fluxo básico entre agents até a composição funcional do resultado final. Esse encadeamento já existe e já entrega valor, mas a coordenação ainda permanece acoplada ao ponto de composição direta da cadeia.

A PR 54 introduz a primeira orquestração mínima desse fluxo, extraindo a sequência atual para uma unidade explícita e simples de coordenação. O recorte permanece estritamente na fase de agents, preserva os contratos já válidos e melhora a leitura do fluxo sem ampliar a arquitetura aprovada.

---

## 2. Objetivo do PR

- criar uma unidade mínima de orquestração para o fluxo básico atual
- delegar a execução sequencial da cadeia já existente
- preservar os contratos públicos já consolidados
- manter o fluxo completo coberto por testes
- separar coordenação de execução especializada sem redistribuir regra de negócio

---

## 3. Decisão Arquitetural

A decisão desta PR é introduzir uma unidade explícita de orquestração, mas ainda em formato simples e linear. Em vez de antecipar engine de estados, pipeline operacional ou integração com ingestion, a coordenação passa a existir como um ponto central enxuto que apenas organiza a sequência já validada.

Com isso, a arquitetura aprovada é mantida. Os agents continuam responsáveis pelo trabalho especializado que já executam, enquanto a nova unidade assume apenas a visibilidade e a organização do fluxo básico entre eles. O ganho aqui é clareza estrutural no recorte certo, sem inflar a solução.

---

## 4. Escopo

- criar a unidade mínima de orquestração da fase
- consumir o fluxo básico já existente entre agents
- retornar o resultado final tipado da cadeia atual
- ajustar providers e composição apenas no necessário para o novo ponto de coordenação
- adicionar ou adaptar testes focados na coordenação explícita do fluxo

---

## 5. Fora de Escopo

- qualquer integração com ingestion
- LangGraph operacional ou grafo de estados completo
- filas, workers, retries ou paralelismo
- nova persistência ou alteração de storage
- observabilidade expandida
- criação de novos agents funcionais
- redistribuição indevida de responsabilidades já estabilizadas

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
    A[Question Input] --> B[AgentsFlowOrchestrator]
    B --> C[InitialQuestionProcessingAgent]
    C --> D[Classification]
    D --> E[Id Resolution]
    E --> F[Legal Search]
    F --> G[Statement Adaptation]
    G --> H[Answer Key]
    H --> I[Structured Output]
```

O diagrama representa apenas o fluxo principal desta PR: a coordenação deixa de ficar implícita na composição direta e passa a existir em um ponto central visível, sem alterar a natureza linear da cadeia.

---

## 7. Contratos Mínimos

```ts
export type AgentsFlowInput = InitialQuestionProcessingInput;

export type AgentsFlowOutput = InitialQuestionProcessingOutput;
```

Nesta etapa, a mudança principal é de coordenação. O contrato funcional do fluxo permanece o mesmo, evitando expansão indevida de modelagem ou reabertura de interface já estabilizada.

---

## 8. Regras de Implementação

A nova unidade deve ser apenas um ponto explícito de coordenação. Ela não deve absorver regra de negócio dos agents, duplicar transformação de dados nem introduzir abstrações auxiliares que não sejam exigidas pelo recorte. O fluxo precisa continuar visível, linear e fácil de revisar.

A implementação deve manter controller inexistente nesta fase, service simples, composição direta e responsabilidade especializada preservada em cada agent. Qualquer mudança além disso foge do objetivo desta PR.

---

## 9. Critérios de Review

Validar se a nova orquestração realmente melhora a clareza do fluxo sem alterar o comportamento funcional já consolidado. Confirmar que os agents permanecem com responsabilidades especializadas, que a coordenação não introduz acoplamento desnecessário e que não houve expansão arquitetural além do recorte.

Também é importante verificar se o fluxo ficou mais explícito sem exigir navegação excessiva e se a documentação permanece proporcional a uma mudança pequena e revisável.

---

## 10. Critérios de Aceite

- [ ] existe uma unidade mínima de orquestração para o fluxo básico entre agents
- [ ] o fluxo atual passa a ser executado por essa nova unidade
- [ ] o resultado final da cadeia permanece preservado
- [ ] os contratos públicos continuam estáveis nesta etapa
- [ ] há testes cobrindo a coordenação explícita do fluxo
- [ ] ingestion permanece intocado
- [ ] nenhuma complexidade indevida foi adicionada

---

## 11. Conclusão

A PR 54 representa a continuação natural da Fase 2 após o fechamento do fluxo básico entre agents. Em vez de ampliar a arquitetura ou antecipar integração operacional, ela apenas explicita a coordenação da cadeia atual no menor recorte útil.

O resultado é um fluxo mais claro e melhor posicionado para revisão e evolução incremental, preservando a simplicidade da implementação, os contratos já estabilizados e o limite arquitetural já aprovado para esta fase.
