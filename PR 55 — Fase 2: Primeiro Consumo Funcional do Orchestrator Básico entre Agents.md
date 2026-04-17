# # 🎛️ PR 55 — Fase 2: Primeiro Consumo Funcional do Orchestrator Básico entre Agents
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
> Esta PR continua diretamente a PR 54. Após introduzir o ponto explícito de coordenação do fluxo básico entre agents, o próximo passo mínimo correto é colocá-lo em uso por um consumidor real da fase, preservando a composição interna atual e sem ampliar a arquitetura já aprovada.
>
> - conecta um consumidor real ao `AgentsFlowOrchestratorService`
> - preserva a cadeia funcional existente
> - mantém contratos públicos estáveis
> - valida a adoção funcional com baixo impacto estrutural
>
> **Este PR não desmonta o `InitialQuestionProcessingAgent`, não toca ingestion, não introduz LangGraph operacional, filas, retries ou paralelismo.**

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

A PR 54 introduziu o `AgentsFlowOrchestratorService` como ponto explícito de coordenação do fluxo básico entre agents, sem alterar a composição funcional já consolidada no `InitialQuestionProcessingAgent`. A arquitetura da fase permaneceu a mesma, mas passou a contar com uma unidade mais clara de entrada para coordenação.

A PR 55 é a continuação natural desse movimento. Em vez de manter o orchestrator apenas como estrutura disponível, esta etapa o coloca em uso real no boundary correto, substituindo o consumo direto anterior pelo novo ponto de coordenação. O ganho aqui não está em expandir comportamento, mas em consolidar adoção funcional mínima com risco baixo, contratos estáveis e continuidade arquitetural explícita.

---

## 2. Objetivo do PR

- conectar um consumidor real ao `AgentsFlowOrchestratorService`
- substituir o consumo direto anterior pelo orchestrator
- preservar os contratos de entrada e saída já estabilizados
- ajustar o wiring necessário no módulo impactado
- adaptar os testes do consumidor ao novo ponto de entrada

---

## 3. Decisão Arquitetural

A decisão central desta PR é adotar o orchestrator antes de reestruturar qualquer composição interna. A arquitetura aprovada é mantida e o recorte se limita a trocar o ponto de consumo do fluxo atual por uma coordenação explícita já introduzida na etapa anterior.

Com isso, o projeto transforma a base criada na PR 54 em uso funcional real sem abrir refactor paralelo, sem redistribuir responsabilidades entre services e sem antecipar uma orquestração mais sofisticada do que a fase pede agora.

---

## 4. Escopo

- identificar o boundary atual que consome o fluxo básico entre agents
- substituir a dependência direta anterior pelo `AgentsFlowOrchestratorService`
- manter os contratos existentes em `shared/ai/model/v1`
- ajustar o wiring do módulo que passa a consumir o orchestrator
- adaptar os testes relacionados ao novo ponto de entrada

---

## 5. Fora de Escopo

- refactor interno do `InitialQuestionProcessingAgent`
- redistribuição step-by-step da cadeia entre múltiplos services
- criação de novos agents funcionais
- alterações em ingestion
- LangGraph operacional
- filas, workers, retries ou paralelismo
- expansão contratual além do necessário para o consumo atual

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

O fluxo permanece simples. A mudança desta PR está no ponto explícito de entrada: o consumidor deixa de acessar diretamente a composição anterior e passa a consumir o orchestrator, enquanto a cadeia interna continua preservada.

---

## 7. Contratos Mínimos

```ts
InitialQuestionProcessingInput
InitialQuestionProcessingOutput
```

Os contratos permanecem os mesmos nesta etapa. A mudança ocorre no ponto de coordenação consumido pelo boundary, e não na interface funcional já estabilizada entre entrada e saída.

---

## 8. Regras de Implementação

O boundary ajustado deve apenas consumir o `AgentsFlowOrchestratorService` e continuar operando com os mesmos dados de entrada e saída. Nenhuma regra de negócio adicional deve ser deslocada para controller, service consumidor ou módulo apenas para justificar a troca do ponto de entrada.

O orchestrator deve permanecer fino, exercendo coordenação explícita sem absorver responsabilidades que ainda pertencem à composição atual. O `InitialQuestionProcessingAgent` continua como núcleo funcional existente até que uma próxima evolução real exija outra reorganização, e não antes disso.

---

## 9. Critérios de Review

Validar se o consumidor real da fase passou a usar o `AgentsFlowOrchestratorService`, se a dependência direta anterior foi removida do boundary ajustado e se o fluxo continua funcional sem alteração contratual indevida.

Também deve ser verificado se a PR manteve o recorte pequeno, se evitou refactors paralelos e se a adoção do orchestrator ocorreu como continuidade controlada da PR 54, sem inflar arquitetura ou espalhar responsabilidades novas pelo módulo.

---

## 10. Critérios de Aceite

- [ ] existe consumidor real usando `AgentsFlowOrchestratorService`
- [ ] o consumo direto anterior foi removido do boundary ajustado
- [ ] input e output permanecem compatíveis com o fluxo atual
- [ ] o wiring necessário foi ajustado sem expansão arquitetural paralela
- [ ] os testes do consumidor impactado foram adaptados com sucesso
- [ ] não houve alteração em ingestion
- [ ] nenhuma complexidade indevida foi adicionada ao fluxo

---

## 11. Conclusão

A PR 55 converte em uso real a coordenação explícita introduzida na PR 54. O avanço é pequeno, direto e coerente com a linha incremental do projeto: um novo ponto de entrada passa a ser efetivamente consumido sem reabrir a composição interna já aprovada.

Com isso, a fase evolui com mais clareza operacional, preservando simplicidade, contratos estáveis e controle de escopo. O resultado é uma adoção funcional mínima, revisável e alinhada ao padrão arquitetural já consolidado no Questions-IA.
