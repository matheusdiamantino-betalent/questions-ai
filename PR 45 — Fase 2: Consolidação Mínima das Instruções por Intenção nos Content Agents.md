# 🧩 PR 45 — Fase 2: Consolidação Mínima das Instruções por Intenção nos Content Agents
## Organização interna da preparação por intent sem introduzir nova camada de prompting

---

<div align="left">

![PR](https://img.shields.io/badge/PR-45-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agent%20routing-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-instrucoes%20por%20intencao-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua a evolução das PRs 42, 43 e 44. Após introduzir agents, classificar intenção e especializar o comportamento inicial, o próximo passo mínimo correto é consolidar as instruções por intenção de forma explícita e coesa dentro do módulo.
>
> - preserva endpoint e contratos atuais
> - mantém ContentService como owner do fluxo
> - mantém agents como boundary de comportamento
> - reduz acoplamento de textos inline
> - mantém mudança pequena e revisável
>
> **Este PR não adiciona prompt engine genérica, registry de templates, múltiplas intenções, tools externos ou nova arquitetura paralela.**

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

A PR 42 introduziu os content agents como boundary inicial de comportamento no módulo. A PR 43 adicionou a classificação mínima de intenção. A PR 44 conectou essa classificação ao roteamento e tornou o comportamento inicial mais observável no fluxo.

A PR 45 mantém essa linha sem reabrir a arquitetura aprovada. O recorte agora é consolidar as instruções por intenção de forma explícita, local e previsível, reduzindo strings inline dispersas e preservando a mesma superfície pública do módulo.

Esse é o próximo passo mínimo correto porque melhora clareza e manutenção exatamente no ponto em que a especialização já existe, sem introduzir nova camada de prompting, sem generalizar prematuramente e sem ampliar o escopo funcional da fase.

---

## 2. Objetivo do PR

- consolidar instruções de `summary` em estrutura mínima explícita
- consolidar instruções de `guidance` em estrutura mínima explícita
- manter `general` como comportamento neutro
- padronizar a preparação de conteúdo dentro dos agents
- preservar o fluxo atual sem impacto externo

---

## 3. Decisão Arquitetural

A arquitetura atual é mantida. `ContentService` continua como owner do fluxo, `ContentIntentClassifier` continua responsável por classificar a intenção e `AgentRouter` continua resolvendo o agent adequado para cada caso.

A decisão central desta PR é apenas organizar a origem das instruções já usadas pelos agents. Em vez de manter textos inline dispersos na implementação, cada comportamento passa a consumir instruções locais explícitas, próximas ao próprio boundary, mantendo simplicidade e evitando a introdução de uma camada genérica que ainda não se justifica.

---

## 4. Escopo

- extrair instruções textuais para constantes locais ou contratos mínimos internos do módulo
- manter `SummaryAgent`, `GuidanceAgent` e `GeneralAgent` existentes
- padronizar a preparação de conteúdo entre os agents
- ajustar testes unitários conforme a organização interna
- preservar metadata e contrato atual do retorno

---

## 5. Fora de Escopo

- prompt builder genérico
- registry central de templates
- strategy factory
- múltiplas intenções simultâneas
- planner multi-step
- memória entre execuções
- tools externas
- novo endpoint
- alteração no contrato de resposta

---

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Requisição Content] --> B[ContentService]
    B --> C[ContentIntentClassifier]
    C --> D[AgentRouter]
    D --> E[ContentAgent + Instrução Local]
    E --> F[AiService]
    F --> G[Resposta Final]
```

O fluxo principal permanece o mesmo já aprovado na fase. A mudança desta PR fica restrita à preparação interna do conteúdo dentro do boundary dos agents, sem alterar entrada, roteamento macro ou formato de saída.

---

## 7. Contratos Mínimos

Os contratos públicos permanecem inalterados. Não há novos DTOs externos, não há alteração do payload de entrada e não há mudança no contrato de resposta retornado pelo módulo.

Quando houver estrutura interna nova para apoiar a organização das instruções, ela deve permanecer mínima, local ao módulo e estritamente voltada à composição do texto preparado pelos agents.

---

## 8. Regras de Implementação

- manter controller fino e sem regra de negócio
- preservar `ContentService` como owner explícito do fluxo
- manter instruções próximas ao agent responsável
- evitar abstrações antecipadas de prompting
- não criar runtime paralelo, engine genérica ou registry desnecessário
- manter comportamento previsível, legível e fácil de testar
- limitar a implementação ao recorte organizacional desta PR

---

## 9. Critérios de Review

- as instruções por intenção ficaram explícitas e legíveis
- o fluxo principal permaneceu simples e estável
- o contrato externo do módulo `content` foi preservado
- não houve expansão indevida de arquitetura
- a mudança permaneceu incremental, pequena e revisável
- os testes continuam cobrindo o comportamento esperado sem acoplamento desnecessário à forma interna

---

## 10. Critérios de Aceite

- [ ] `summary` utiliza instrução explícita consolidada
- [ ] `guidance` utiliza instrução explícita consolidada
- [ ] `general` permanece com comportamento neutro
- [ ] a preparação de conteúdo segue padronizada entre os agents
- [ ] o endpoint atual permanece funcional sem mudanças externas
- [ ] os testes passam sem regressão

---

## 11. Conclusão

A PR 45 consolida internamente a especialização introduzida nas PRs anteriores sem alterar a superfície pública do módulo. O ganho aqui é de clareza, previsibilidade e organização local da preparação por intenção.

O recorte permanece pequeno, funcional e coerente com a fase atual. A arquitetura segue a mesma, o fluxo continua simples e a mudança evita tanto dispersão textual quanto a introdução prematura de uma camada de prompting mais ampla.
