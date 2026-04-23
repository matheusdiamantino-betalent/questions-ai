# 🤖 PR 74 — Fase 2: Consolidação Funcional do Contexto Jurídico no Resultado Final

## Priorização controlada da base legal na composição avançada da resposta processada

---

<div align="left">

![PR](https://img.shields.io/badge/PR-74-2563eb?style=for-the-badge\&logo=gitpullrequest\&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge\&logo=nestjs\&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge\&logo=dependabot\&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-contexto%20juridico%20funcional-0891b2?style=for-the-badge\&logo=serverless\&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge\&logo=githubactions\&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR muda o foco da limpeza estrutural para ganho funcional dentro dos agents avançados.
>
> * fortalece o uso do contexto jurídico já disponível no pipeline
> * prioriza base legal encontrada na composição da resposta final
> * mantém o fluxo atual sem novos components
>
> **Este PR não cria novos agents, não redesenha a fase e não amplia o pipeline existente.**

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

As PRs anteriores consolidaram a arquitetura operacional dos agents avançados, removeram redundâncias e estabilizaram fronteiras de responsabilidade. O próximo passo natural é converter essa base estável em valor funcional perceptível. Esta PR reforça o papel do contexto jurídico encontrado pelo `LegalSearchAgent`, fazendo com que a base legal influencie de forma mais clara a justificativa final e a qualidade da resposta processada.

## 2. Objetivo do PR

* priorizar contexto jurídico válido na composição final do `answerKey`
* fortalecer coerência entre justificativa e base legal encontrada
* melhorar fallback entre fonte jurídica e adaptação textual
* elevar valor funcional do pipeline já existente
* manter mudanças pequenas e proporcionais ao slice

## 3. Decisão Arquitetural

A arquitetura permanece intacta. Em vez de introduzir novos componentes, a decisão é extrair mais valor dos agents já integrados. O foco sai da reorganização interna e passa para o aproveitamento funcional do contexto disponível.

## 4. Escopo

* ajustar composição do `AnswerKeyAgent` para valorizar contexto jurídico
* revisar `InitialQuestionProcessingAgent` para priorização explícita da base legal
* alinhar fallback quando não houver contexto jurídico utilizável
* manter contrato público atual da resposta final
* atualizar testes dos agents impactados

## 5. Fora de Escopo

* novos agents avançados
* redesign do orchestrator
* registry dinâmico
* ranking semântico complexo
* cache adicional
* paralelismo ou retry engine
* alterações externas de API

## 6. Fluxo Arquitetural

```mermaid
%%{init: {"theme": "base","themeVariables": {"background": "#050b16","primaryColor": "#0b1220","primaryTextColor": "#ffffff","primaryBorderColor": "#22d3ee","lineColor": "#94a3b8","secondaryColor": "#0b1220","tertiaryColor": "#0b1220","fontFamily": "Inter, Arial, sans-serif"},"flowchart": {"htmlLabels": true,"curve": "linear","nodeSpacing": 34,"rankSpacing": 44}}}%%
flowchart LR
A[Questão Extraída] --> B[ClassificationAgent]
B --> C[IdResolutionAgent]
C --> D[LegalSearchAgent]
D --> E[Contexto Jurídico Priorizado]
E --> F[AnswerKeyAgent]
F --> G[Resposta Final Mais Robusta]
classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
class A step1; class B step2; class C step3; class D step4; class E step5; class F,G step6;
linkStyle 0 stroke:#9ca3af,stroke-width:2px;
linkStyle 1 stroke:#9ca3af,stroke-width:2px;
linkStyle 2 stroke:#9ca3af,stroke-width:2px;
linkStyle 3 stroke:#9ca3af,stroke-width:2px;
linkStyle 4 stroke:#9ca3af,stroke-width:2px;
linkStyle 5 stroke:#9ca3af,stroke-width:2px;
```

## 7. Contratos Mínimos

O shape externo permanece estável. Ajustes internos apenas refinam prioridade e composição das informações já presentes no fluxo, sem inflar contratos ou introduzir novos campos desnecessários.

## 8. Regras de Implementação

* priorizar base legal quando houver conteúdo utilizável
* fallback simples quando não houver contexto jurídico válido
* evitar duplicidade textual na justificativa
* preservar simplicidade dos agents atuais
* limitar mudanças aos pontos de valor real

## 9. Critérios de Review

* contexto jurídico influencia melhor o resultado final
* fallback permanece claro e previsível
* contrato público não foi expandido
* recorte segue pequeno e incremental
* testes cobrem cenários com e sem base legal

## 10. Critérios de Aceite

* [ ] justificativa final prioriza contexto jurídico quando disponível
* [ ] fallback textual continua funcional sem base legal
* [ ] resposta final mantém mesmo contrato externo
* [ ] testes permanecem verdes
* [ ] nenhum novo agent foi criado

## 11. Conclusão

A PR 74 representa a transição correta entre estabilização estrutural e ganho funcional. Após consolidar o pipeline, o foco passa a ser qualidade de resultado. O contexto jurídico deixa de ser apenas dado auxiliar e passa a gerar valor direto na resposta final, mantendo simplicidade, continuidade e controle de escopo.
