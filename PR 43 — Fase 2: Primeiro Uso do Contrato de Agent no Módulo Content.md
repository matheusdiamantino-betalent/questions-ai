# 🧩 PR 43 — Fase 2: PDF Extraction Agent Inicial em Shared
## Primeiro agent genérico funcional da trilha, aplicando o contrato compartilhado já consolidado na fase

---

<div align="left">

![PR](https://img.shields.io/badge/PR-43-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-pdf%20extraction%20agent-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 42. Depois de consolidar o contrato compartilhado de agents em `shared`, o próximo passo mínimo correto da fase é aplicar esse contrato em um comportamento real, ainda fora de qualquer módulo de domínio.
>
> - implementa o primeiro agent genérico funcional da trilha
> - valida o contrato compartilhado com uma execução concreta e simples
> - mantém o recorte isolado em `shared/ai`
> - preserva a ordem incremental prevista para a Fase 2
>
> **Este PR não implementa roteamento entre agents, pipeline encadeado, integração com `content`, `ClassificationAgent`, `IdResolutionAgent` ou `SearchAgent`.**

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

A PR 42 abriu a Fase 2 com a foundation mínima de agents genéricos em `shared`, definindo o contrato comum e o ponto de partida arquitetural da trilha. Com essa base já aprovada, a progressão natural agora é parar de discutir estrutura e começar a validar comportamento real sobre o mesmo boundary.

Esta PR faz exatamente esse avanço mínimo. O foco é introduzir o `PdfExtractionAgent` como primeira implementação concreta do contrato compartilhado, mantendo a solução simples, genérica e sem qualquer acoplamento com fluxo de domínio. O ganho aqui não é expandir a arquitetura, mas confirmar que a base da PR anterior já sustenta um agent funcional pequeno e revisável.

---

## 2. Objetivo do PR

- criar o primeiro agent funcional da Fase 2 dentro de `shared/ai`
- implementar `PdfExtractionAgent` aderente ao contrato `Agent`
- receber uma entrada mínima e retornar o conteúdo textual extraído
- validar uso prático da foundation da PR 42 sem ampliar a fase

---

## 3. Decisão Arquitetural

A decisão arquitetural da PR 42 é preservada integralmente. Esta PR não redefine a estrutura de agents, não reabre contratos e não introduz uma nova camada de orquestração. O que ela faz é apenas aplicar o contrato já consolidado em uma implementação concreta, ainda restrita ao boundary compartilhado.

O `PdfExtractionAgent` permanece como unidade simples, com responsabilidade única e fluxo explícito. A implementação não antecipa composition root de agents, coordenação entre múltiplos executores ou estratégias paralelas. A escolha aqui é validar o primeiro comportamento funcional com o menor número possível de movimentos novos.

---

## 4. Escopo

- adicionar `PdfExtractionAgent` no espaço compartilhado de IA
- implementar o método `execute` conforme o contrato comum de agent
- definir input e output mínimos para o recorte de extração
- cobrir o comportamento principal com testes unitários
- manter a implementação genérica, simples e desacoplada de domínio

---

## 5. Fora de Escopo

- classificação de conteúdo
- resolução de identificadores
- busca externa ou enriquecimento
- integração com o módulo `content`
- pipeline ou encadeamento entre agents
- fallback entre estratégias de extração
- OCR, parsing avançado ou tratamento expandido de PDF
- observabilidade específica, cache ou infraestrutura paralela
- qualquer preparação estrutural para os próximos agents além do necessário ao slice atual

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{
  'primaryColor':'#0f172a',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#22d3ee',
  'lineColor':'#39ff14',
  'secondaryColor':'#111827',
  'tertiaryColor':'#1f2937',
  'nodeBorder':'#22d3ee'
}}}%%
flowchart LR
    A["PDF Input"] --> B["PdfExtractionAgent"]
    B --> C["Extracted Text"]
```

O fluxo desta PR permanece propositalmente curto: uma entrada mínima é recebida pelo agent e devolvida como conteúdo textual extraído. Não existe coordenação com outros agents nem composição de etapas além desse comportamento central.

---

## 7. Contratos Mínimos

O contrato compartilhado da PR 42 permanece o mesmo. Esta PR apenas introduz os tipos mínimos necessários para a primeira implementação concreta do fluxo.

```ts
export interface Agent<TInput = unknown, TOutput = unknown> {
  execute(input: TInput): Promise<TOutput>;
}

export type PdfExtractionInput = {
  content: string;
};

export type PdfExtractionOutput = {
  text: string;
};
```

A modelagem continua deliberadamente pequena. O slice pede apenas uma entrada objetiva e uma saída textual igualmente direta, sem expansão prematura de metadados, estados intermediários ou estratégias múltiplas.

---

## 8. Regras de Implementação

A implementação deve manter o mesmo critério de simplicidade estabelecido na fase. O `PdfExtractionAgent` deve concentrar apenas a lógica mínima necessária para cumprir o contrato, sem tentar servir como base indireta para outros agents ainda não introduzidos. Se houver service auxiliar, ele precisa continuar proporcional ao recorte e visivelmente subordinado ao fluxo principal do agent.

Também é importante preservar o boundary correto: esta PR segue em `shared`, não toca módulo de domínio e não prepara integração futura dentro do mesmo diff. O objetivo é um agent único, compreensível em leitura rápida, com testes objetivos e sem granularidade arquitetural desnecessária.

---

## 9. Critérios de Review

- o agent foi adicionado no boundary compartilhado já definido pela PR 42
- a implementação respeita o contrato comum sem alterar sua forma
- o fluxo principal permanece simples, explícito e sem orquestração paralela
- input e output estão proporcionais ao recorte
- não houve acoplamento com `content` ou qualquer módulo de domínio
- os testes cobrem o comportamento principal sem inflar a solução
- a PR continua pequena, incremental e alinhada à ordem funcional da fase

---

## 10. Critérios de Aceite

- [ ] existe `PdfExtractionAgent` implementando o contrato `Agent`
- [ ] o método `execute` recebe o input mínimo definido para o slice
- [ ] o agent retorna conteúdo textual extraído no output esperado
- [ ] os contratos mínimos do recorte estão claros e estáveis
- [ ] testes unitários do fluxo principal estão passando
- [ ] a suíte relacionada permanece verde sem expansão indevida de escopo

---

## 11. Conclusão

A PR 43 avança a Fase 2 no ponto exato em que a PR 42 parou: sai da foundation compartilhada e entra no primeiro comportamento funcional concreto, sem mudar a arquitetura já aprovada e sem antecipar etapas seguintes. O resultado é um slice pequeno, útil para validar o contrato em execução real e suficientemente contido para revisão rápida.

