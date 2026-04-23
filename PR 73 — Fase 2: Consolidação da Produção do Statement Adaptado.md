# 🤖 PR 73 — Fase 2: Consolidação da Produção do Statement Adaptado
## Centralização controlada da responsabilidade do contexto textual refinado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-73-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-consolidacao%20do%20statement-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR remove duplicidade residual no pipeline avançado e define um único ponto responsável pela geração do `adaptedStatement`.
>
> - centraliza a responsabilidade textual no processamento intermediário
> - simplifica o orchestrator sem alterar o contrato final
> - mantém o fluxo atual e ajusta apenas o necessário
>
> **Este PR não cria novos agents, não redesenha a fase e não amplia escopo funcional.**

## Sumário

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

## 1. Síntese Executiva

A PR 72 consolidou a composição final do `answerKey` no fluxo orquestrado. O próximo passo natural é aplicar o mesmo princípio ao `adaptedStatement`, removendo reaplicações desnecessárias e deixando a responsabilidade textual concentrada em um único ponto. Esta PR reduz redundância interna e melhora legibilidade operacional sem alterar a saída pública do pipeline.

## 2. Objetivo do PR

- definir um único produtor do `adaptedStatement`
- simplificar o `AgentsFlowOrchestratorService`
- remover adaptação textual redundante no fluxo
- preservar a resposta final já consolidada
- atualizar testes proporcionais ao recorte

## 3. Decisão Arquitetural

A arquitetura da fase é mantida. A mudança consiste apenas em consolidar responsabilidade já existente, evitando que duas etapas distintas manipulem o mesmo artefato textual. O orchestrator permanece coordenador do fluxo, com menor carga operacional.

## 4. Escopo

- ajustar `InitialQuestionProcessingAgent` como ponto único do `adaptedStatement`
- remover reaplicação no orchestrator
- manter composição final do `answerKey`
- preservar contratos públicos atuais
- revisar testes afetados

## 5. Fora de Escopo

- novos agents avançados
- heurísticas adicionais de adaptação
- redesign completo do pipeline
- observabilidade expandida
- retry, paralelismo ou registry
- mudanças funcionais externas

## 6. Fluxo Arquitetural

```mermaid
%%{init: {"theme": "base","themeVariables": {"background": "#050b16","primaryColor": "#0b1220","primaryTextColor": "#ffffff","primaryBorderColor": "#22d3ee","lineColor": "#94a3b8","secondaryColor": "#0b1220","tertiaryColor": "#0b1220","fontFamily": "Inter, Arial, sans-serif"},"flowchart": {"htmlLabels": true,"curve": "linear","nodeSpacing": 34,"rankSpacing": 44}}}%%
flowchart LR
A[Questão Extraída] --> B[ClassificationAgent]
B --> C[IdResolutionAgent]
C --> D[InitialQuestionProcessingAgent\nproduz adaptedStatement]
D --> E[AgentsFlowOrchestratorService\ncompõe saída final]
E --> F[answerKey + metadata + ids]
classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
class A step1; class B step2; class C step3; class D step4; class E step5; class F step6;
linkStyle 0 stroke:#9ca3af,stroke-width:2px;
linkStyle 1 stroke:#9ca3af,stroke-width:2px;
linkStyle 2 stroke:#9ca3af,stroke-width:2px;
linkStyle 3 stroke:#9ca3af,stroke-width:2px;
linkStyle 4 stroke:#9ca3af,stroke-width:2px;
```

## 7. Contratos Mínimos

O contrato externo permanece o mesmo. Ajustes internos apenas reforçam que `adaptedStatement` passa a ter origem única antes da composição final da resposta.

## 8. Regras de Implementação

- manter controller fino e fluxo atual intacto
- orchestrator coordena, não recalcula texto redundante
- processamento intermediário concentra a adaptação textual
- sem abstrações novas ou fundações paralelas
- mudanças mínimas e localizadas

## 9. Critérios de Review

- existe apenas um ponto produtor do `adaptedStatement`
- orchestrator ficou mais simples
- contrato final foi preservado
- não houve expansão arquitetural
- testes cobrem a nova fronteira de responsabilidade

## 10. Critérios de Aceite

- [ ] `adaptedStatement` deixa de ser recalculado no orchestrator
- [ ] fluxo final continua retornando mesma estrutura pública
- [ ] composição do `answerKey` permanece funcional
- [ ] suíte de testes permanece verde
- [ ] recorte segue pequeno e revisável

## 11. Conclusão

A PR 73 continua a limpeza incremental do pipeline avançado. Após centralizar o `answerKey`, centraliza também a produção do `adaptedStatement`, reduzindo redundância e fortalecendo fronteiras de responsabilidade sem inflar a solução.

