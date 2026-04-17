# 🎼 PR 56 — Fase 2: Primeira Extração Mínima da Coordenação Sequencial para o Orchestrator
## Movimentação inicial da composição do fluxo básico para o ponto explícito de coordenação sem ampliar a arquitetura da fase

---

<div align="left">

![PR](https://img.shields.io/badge/PR-56-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-coordenacao%20sequencial%20minima-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 55. Após colocar o orchestrator em uso real como ponto de entrada do fluxo básico entre agents, o próximo passo mínimo correto é iniciar a migração da coordenação sequencial para essa unidade, em um recorte pequeno e controlado.
>
> - move a primeira parte real da coordenação para o orchestrator
> - reduz responsabilidade de composição no fluxo anterior
> - preserva contratos públicos existentes
> - mantém a evolução incremental e revisável
>
> **Este PR não reescreve toda a cadeia, não introduz LangGraph operacional, não toca ingestion, não adiciona paralelismo, retries ou abstrações genéricas de pipeline.**

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

A PR 54 introduziu o `AgentsFlowOrchestratorService` como unidade explícita de coordenação. A PR 55 colocou essa unidade em uso real no boundary da fase, preservando a cadeia interna existente.

A PR 56 inicia a próxima evolução natural: mover uma primeira parte da coordenação sequencial para o orchestrator. O objetivo não é refatorar toda a cadeia de uma vez, mas começar a transferir responsabilidade de composição no menor recorte útil possível.

---

## 2. Objetivo do PR

- mover a primeira coordenação real de steps para o orchestrator
- reduzir o papel de composição centralizado no fluxo anterior
- preservar input e output já estabilizados
- manter comportamento funcional existente
- validar a nova coordenação com cobertura dedicada

---

## 3. Decisão Arquitetural

A decisão desta PR é substituir refactor amplo por migração incremental de responsabilidade. Em vez de desmontar toda a composição atual em uma única etapa, apenas uma parte pequena e clara da sequência passa a ser coordenada pelo orchestrator.

Isso mantém previsibilidade, reduz risco de regressão e permite evolução gradual da arquitetura sem inflar o recorte atual.

---

## 4. Escopo

- extrair a coordenação inicial de steps para o orchestrator
- manter etapas restantes no fluxo atual quando necessário
- preservar contratos existentes em `shared/ai/model/v1`
- ajustar wiring do módulo apenas no necessário
- adaptar testes para a nova responsabilidade de coordenação

---

## 5. Fora de Escopo

- refactor completo do `InitialQuestionProcessingAgent`
- migração total de todos os steps em uma única PR
- LangGraph operacional
- filas, workers, retries ou paralelismo
- qualquer integração com ingestion
- novos agents funcionais
- criação de engine genérica de pipeline

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
    B --> C[Classification]
    C --> D[Id Resolution]
    D --> E[Remaining Current Flow]
    E --> F[Structured Output]
```

O diagrama representa a primeira migração real de coordenação para o orchestrator. Apenas parte da sequência é movida nesta etapa para preservar simplicidade e baixo risco.

---

## 7. Contratos Mínimos

```ts
InitialQuestionProcessingInput
InitialQuestionProcessingOutput
```

Os contratos permanecem inalterados. A evolução desta PR é interna ao fluxo de coordenação e não exige expansão contratual.

---

## 8. Regras de Implementação

O orchestrator deve assumir apenas a coordenação dos steps definidos no recorte desta PR. Nenhuma regra de negócio nova deve ser introduzida e nenhuma transformação deve ser duplicada sem necessidade.

A parte ainda não migrada pode permanecer no fluxo atual temporariamente. O critério principal é manter o fluxo visível, simples e incremental.

---

## 9. Critérios de Review

Validar se a coordenação parcial foi corretamente movida para o orchestrator, se o comportamento funcional foi preservado, se os contratos continuam estáveis e se a mudança permaneceu proporcional ao recorte.

Confirmar também que não houve tentativa de resolver toda a arquitetura em uma única etapa.

---

## 10. Critérios de Aceite

- [ ] primeira parte real da coordenação foi movida para o orchestrator
- [ ] fluxo continua funcional após a extração parcial
- [ ] contratos permanecem compatíveis
- [ ] testes refletem a nova responsabilidade
- [ ] nenhuma alteração em ingestion
- [ ] nenhuma complexidade indevida foi adicionada

---

## 11. Conclusão

A PR 56 representa a primeira coordenação real exercida pelo orchestrator além do simples ponto de entrada. O avanço é pequeno, controlado e alinhado à estratégia incremental do projeto.

Com isso, a base evolui passo a passo, preservando simplicidade, contratos existentes e clareza arquitetural sem antecipar uma reestruturação maior do que o momento exige.

