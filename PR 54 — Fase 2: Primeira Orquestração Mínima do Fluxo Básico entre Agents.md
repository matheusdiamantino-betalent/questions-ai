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
> Esta PR continua a PR 53. Após fechar o fluxo básico entre agents, o próximo passo mínimo correto é extrair a coordenação da cadeia para uma unidade explícita de orquestração, mantendo o foco integral na fase de agents e sem reabrir ingestion.
>
> - introduz ponto central simples de coordenação
> - preserva agents especializados já existentes
> - mantém fluxo linear, previsível e testável
> - prepara evolução futura sem ampliar escopo atual
>
> **Esta PR não integra com ingestion, não introduz LangGraph completo, não adiciona filas, retries ou paralelismo.**

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

As PRs 50 a 53 construíram progressivamente a cadeia funcional entre agents até o fechamento com answer key. O fluxo já entrega valor, porém a coordenação ainda permanece acoplada à composição direta.

A PR 54 cria uma camada mínima de orquestração para centralizar a execução da cadeia atual sem alterar responsabilidades internas dos agents. O avanço mantém a fase focada em agents e evita antecipar integração operacional.

---

## 2. Objetivo do PR

- criar orchestrator mínimo para o fluxo atual
- delegar execução sequencial da cadeia existente
- preservar contratos públicos já válidos
- manter cobertura de testes do fluxo completo
- separar coordenação de execução especializada

---

## 3. Decisão Arquitetural

Em vez de migrar diretamente para uma engine de estados ou ampliar a pipeline operacional, a coordenação nasce como serviço simples e explícito. A decisão reduz acoplamento entre composição e execução sem elevar complexidade desnecessária.

---

## 4. Escopo

- criar unidade de orquestração da fase
- consumir fluxo básico já existente
- retornar resultado final tipado
- ajustar providers necessários
- adicionar testes dedicados

---

## 5. Fora de Escopo

- IngestionProcessor
- filas e workers adicionais
- LangGraph operacional
- persistência nova
- observabilidade avançada
- retries
- paralelização
- novos agents funcionais

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
C --> D[Metadata + IDs + Legal + Statement + AnswerKey]
```

---

## 7. Contratos Mínimos

```ts
export type AgentsFlowInput = InitialQuestionProcessingInput;

export type AgentsFlowOutput = InitialQuestionProcessingOutput;
```

Sem expansão contratual nesta etapa. A mudança principal é organizacional e de coordenação.

---

## 8. Regras de Implementação

Controller inexistente nesta fase. A nova unidade deve apenas coordenar chamadas e repassar resultados. Nenhuma regra de negócio deve ser duplicada do fluxo já existente. Preferir código explícito e baixo número de abstrações.

---

## 9. Critérios de Review

Validar se a coordenação ficou clara, se não houve regressão no fluxo atual, se responsabilidades permaneceram separadas e se a PR adiciona organização real sem inflar arquitetura.

---

## 10. Critérios de Aceite

- [ ] orchestrator mínimo criado
- [ ] fluxo atual executado através da nova unidade
- [ ] resultado final preservado
- [ ] testes cobrindo coordenação
- [ ] nenhuma alteração em ingestion
- [ ] nenhuma complexidade indevida adicionada

---

## 11. Conclusão

A PR 54 abre a próxima etapa natural da Fase 2 ao introduzir coordenação explícita sobre a cadeia já consolidada. O ganho é estrutural e funcional na medida certa: melhor separação de responsabilidades, mesma simplicidade operacional e continuidade coerente do roadmap.
