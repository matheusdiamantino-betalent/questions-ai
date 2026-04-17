# 🧩 PR 55 — Fase 2: Primeiro Consumo Funcional do Orchestrator Básico entre Agents
## Adoção controlada do ponto explícito de coordenação no fluxo atual sem reabrir a composição interna dos agents

---

<div align="left">

![PR](https://img.shields.io/badge/PR-55-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-primeiro%20consumo%20orchestrator-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 54. Após introduzir a unidade explícita de coordenação do fluxo básico entre agents, o próximo passo mínimo correto é colocá-la em uso real por um consumidor da fase, mantendo a composição interna atual e sem ampliar a arquitetura aprovada.
>
> - conecta um consumidor real ao orchestrator
> - preserva a cadeia funcional existente
> - mantém contratos públicos estáveis
> - valida adoção funcional com baixo impacto estrutural
>
> **Este PR não desmonta o InitialQuestionProcessingAgent, não toca ingestion, não introduz LangGraph operacional, filas, retries ou paralelismo.**

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

A PR 54 criou o ponto explícito de coordenação do fluxo básico entre agents por meio do `AgentsFlowOrchestratorService`, mantendo intacta a composição funcional já consolidada no `InitialQuestionProcessingAgent`.

A PR 55 transforma essa base em uso efetivo. O recorte desta etapa é simples: substituir o consumo direto do fluxo anterior pelo orchestrator no boundary correto da fase, sem reabrir a cadeia interna, sem alterar contratos e sem expandir responsabilidades.

---

## 2. Objetivo do PR

- conectar um consumidor real ao `AgentsFlowOrchestratorService`
- substituir o acesso direto ao fluxo anterior pelo novo ponto de coordenação
- preservar input e output já estabilizados
- ajustar testes do consumidor impactado
- consolidar adoção funcional da orquestração mínima

---

## 3. Decisão Arquitetural

A decisão desta PR é priorizar adoção incremental sobre refactor amplo. Em vez de redistribuir os steps internos entre múltiplos pontos de execução, o sistema passa primeiro a consumir a unidade de coordenação criada na PR anterior.

Isso mantém a arquitetura aprovada, reduz risco de mudança desnecessária e transforma uma fundação estrutural em comportamento real no menor recorte útil possível.

---

## 4. Escopo

- identificar o boundary atual que consome o fluxo básico da fase
- substituir a dependência direta anterior pelo orchestrator
- manter contratos existentes em `shared/ai/model/v1`
- ajustar wiring necessário no módulo atual
- adaptar testes relacionados ao novo ponto de entrada

---

## 5. Fora de Escopo

- refactor interno do `InitialQuestionProcessingAgent`
- redistribuição step-by-step da cadeia entre services
- qualquer integração com ingestion
- LangGraph operacional
- filas, workers, retries ou paralelismo
- novos agents funcionais
- expansão contratual desnecessária

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
    A[Consumer Boundary] --> B[AgentsFlowOrchestratorService]
    B --> C[InitialQuestionProcessingAgent]
    C --> D[Existing Agent Chain]
    D --> E[Structured Output]
```

O foco desta PR é a troca controlada do ponto de consumo. A cadeia interna permanece a mesma e o ganho está na adoção funcional da coordenação explícita.

---

## 7. Contratos Mínimos

```ts
InitialQuestionProcessingInput
InitialQuestionProcessingOutput
```

Os contratos permanecem os mesmos nesta etapa. A mudança ocorre no ponto de entrada do fluxo, não na interface funcional já estabilizada.

---

## 8. Regras de Implementação

O consumidor ajustado deve apenas trocar a dependência para o orchestrator e continuar operando com os mesmos dados de entrada e saída. Nenhuma regra de negócio deve migrar para o boundary consumidor.

O orchestrator continua fino e o `InitialQuestionProcessingAgent` permanece como composição funcional atual até que exista motivo real para nova evolução arquitetural.

---

## 9. Critérios de Review

Validar se o consumo direto anterior foi corretamente substituído pelo orchestrator, se não houve regressão funcional, se os contratos permaneceram estáveis e se a mudança entrega adoção real com impacto mínimo.

Confirmar também que a PR não aproveita o ajuste para introduzir refactors paralelos fora do recorte.

---

## 10. Critérios de Aceite

- [ ] existe consumidor real usando `AgentsFlowOrchestratorService`
- [ ] dependência direta anterior foi removida do boundary ajustado
- [ ] input e output permanecem compatíveis
- [ ] testes foram adaptados com sucesso
- [ ] nenhuma alteração em ingestion
- [ ] nenhuma complexidade indevida foi adicionada

---

## 11. Conclusão

A PR 55 conclui o passo natural iniciado na PR 54: a coordenação explícita deixa de ser apenas estrutura e passa a ser parte real do fluxo da fase. O avanço é pequeno, controlado e coerente com a estratégia incremental do projeto.

Com isso, a base evolui com clareza, preservando simplicidade, contratos existentes e o limite arquitetural adequado para este momento.

