# 🧩 PR 44 — Fase 2: Especialização Inicial de Resposta por Intenção nos Content Agents
## Evolução mínima do comportamento dos agents após classificação estruturada já consolidada

---

<div align="left">

![PR](https://img.shields.io/badge/PR-44-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agent%20routing-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-especializacao%20de%20resposta-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua o eixo validado nas PRs 42 e 43. Após introduzir agents básicos e estruturar a classificação de intenção, o próximo passo mínimo correto é diferenciar a preparação da resposta conforme a intenção já identificada.
>
> - preserva endpoint e contratos atuais
> - mantém `ContentService` como owner do fluxo
> - adiciona comportamento específico por agent
> - aumenta utilidade sem nova arquitetura
> - mantém mudança pequena e revisável
>
> **Este PR não adiciona planner, memória, tools externos, prompt engine genérica, múltiplas intenções ou nova API paralela.**

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

A PR 42 introduziu agents internos no boundary de `content`. A PR 43 qualificou a escolha entre eles com classificação estruturada de intenção. Com a decisão já organizada, o passo natural seguinte é fazer essa decisão produzir comportamento distinto e observável.

A PR 44 especializa a preparação da resposta dentro de cada agent. Cada intenção passa a aplicar uma transformação mínima e explícita antes da execução final do fluxo, mantendo a mesma superfície pública e sem reabrir arquitetura.

---

## 2. Objetivo do PR

- especializar resposta para intenção de resumo
- especializar resposta para intenção de guidance
- manter resposta neutra para intenção geral
- preservar contrato externo existente
- manter simplicidade operacional do fluxo

---

## 3. Decisão Arquitetural

A arquitetura atual é mantida. `ContentService` continua coordenando o fluxo, `ContentIntentClassifier` segue responsável pela leitura de intenção e `AgentRouter` permanece resolvendo o agent adequado.

A mudança desta PR ocorre apenas no comportamento interno dos agents, que deixam de ser pass-through e passam a aplicar instruções mínimas coerentes com cada intenção.

---

## 4. Escopo

- ajustar `SummaryAgent` para preparar saída orientada a resumo
- ajustar `GuidanceAgent` para preparar saída orientada a explicação ou passo a passo
- manter `GeneralAgent` como fluxo neutro
- preservar metadata atual do agent selecionado
- atualizar testes unitários e de integração local

---

## 5. Fora de Escopo

- prompt builder genérico
- templates complexos
- planner multi-step
- múltiplas intenções simultâneas
- confidence score
- memória entre execuções
- tools externas
- novo endpoint funcional
- mudança no contrato de resposta

---

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Requisição Content] --> B[ContentService]
    B --> C[IntentClassifier]
    C --> D[AgentRouter]
    D --> E[Agent Especializado]
    E --> F[AiService]
    F --> G[Resposta Final]
```

A evolução permanece interna ao módulo e reutiliza a fundação já aprovada.

---

## 7. Contratos Mínimos

Os contratos públicos permanecem os mesmos. Não há novos DTOs externos nem alteração de payload de entrada ou resposta.

O recorte desta PR altera apenas a implementação interna de `execute(input: string): Promise<string>` dos agents existentes.

---

## 8. Regras de Implementação

- manter controller fino e sem regra de negócio
- preservar `ContentService` como owner do fluxo
- agents devem aplicar transformação simples e explícita
- evitar abstrações antecipadas de prompting
- não criar runtime paralelo entre agents
- manter comportamento previsível e fácil de testar
- adicionar apenas testes proporcionais ao slice

---

## 9. Critérios de Review

- agents agora possuem comportamento claramente distinto
- fluxo principal permanece simples e legível
- contrato externo de `content` foi preservado
- não houve expansão indevida de arquitetura
- mudança permaneceu incremental e revisável
- testes cobrem especialização de comportamento

---

## 10. Critérios de Aceite

- [ ] summary produz preparação compatível com resumo
- [ ] guidance produz preparação compatível com orientação
- [ ] general mantém comportamento neutro
- [ ] endpoint atual permanece funcional sem mudanças externas
- [ ] metadata atual continua íntegra
- [ ] testes passam cobrindo novo comportamento

---

## 11. Conclusão

A PR 44 transforma a classificação introduzida na PR 43 em comportamento efetivamente diferenciado por intenção. O ganho é funcional e interno, preservando simplicidade, compatibilidade e baixo ruído para review.
