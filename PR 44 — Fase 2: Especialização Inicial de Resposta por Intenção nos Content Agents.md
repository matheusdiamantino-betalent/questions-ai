# 🧩 PR 44 — Fase 2: Execução Compartilhada Inicial do PDF Extraction Agent
## Primeiro consumo controlado do agent já introduzido na fase, sem ampliar a arquitetura existente

---

<div align="left">

![PR](https://img.shields.io/badge/PR-44-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-execucao%20compartilhada%20pdf%20agent-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 43. Após introduzir o primeiro agent funcional da fase, o próximo passo mínimo correto é permitir seu consumo controlado dentro do boundary compartilhado de IA, sem criar pipeline ou coordenação expandida.
>
> - adiciona ponto mínimo de uso do `PdfExtractionAgent`
> - preserva o contrato já aprovado
> - mantém a execução simples e explícita
> - evita nova foundation ou abstração prematura
>
> **Este PR não implementa router de agents, múltiplos executores, integração com `content`, `ClassificationAgent`, `SearchAgent` ou pipeline encadeado.**

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

A PR 42 consolidou o contrato base de agents em `shared/ai`. A PR 43 validou essa foundation com o primeiro agent concreto, o `PdfExtractionAgent`. Com isso resolvido, a progressão natural da fase é permitir que esse agent seja consumido por um ponto compartilhado de execução, ainda sem introduzir orquestração entre múltiplos agentes.

Esta PR entrega esse passo mínimo. O foco é transformar um agent isolado em capacidade reutilizável dentro do boundary compartilhado, mantendo leitura simples, baixo ruído e escopo pequeno.

---

## 2. Objetivo do PR

- adicionar serviço mínimo para executar `PdfExtractionAgent`
- centralizar o consumo inicial do agent em `shared/ai`
- manter fluxo explícito e simples
- validar comportamento com testes unitários
- avançar a fase sem ampliar arquitetura

---

## 3. Decisão Arquitetural

A arquitetura existente é mantida. Esta PR não cria registry, router, factory ou composition root expandido. A decisão central é apenas introduzir um ponto mínimo de consumo do agent já existente, permitindo reutilização controlada sem alterar os contratos da fase.

O fluxo permanece linear: entrada recebida, delegação para o agent, retorno do resultado.

---

## 4. Escopo

- adicionar serviço compartilhado de execução do PDF extraction
- injetar/instanciar `PdfExtractionAgent` de forma simples
- expor método mínimo de uso
- adicionar testes unitários do fluxo principal
- preservar estrutura atual da fase

---

## 5. Fora de Escopo

- múltiplos agents no mesmo fluxo
- roteamento por intenção
- registry dinâmico
- factory de agents
- integração com `content`
- classification/search/id resolution
- observabilidade expandida
- retries, filas ou pipeline

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{
  'primaryColor':'#0f172a',
  'primaryTextColor':'#e5e7eb',
  'primaryBorderColor':'#22d3ee',
  'lineColor':'#39ff14'
}}}%%
flowchart LR
    A["Input"] --> B["Ai Service"]
    B --> C["PdfExtractionAgent"]
    C --> D["Extracted Text"]
```

---

## 7. Contratos Mínimos

Os contratos existentes permanecem os mesmos. Esta PR apenas reutiliza:

```ts
Agent<TInput, TOutput>
PdfExtractionInput
PdfExtractionOutput
```

Sem novos contratos globais para este slice.

---

## 8. Regras de Implementação

- service fino e orientado ao fluxo principal
- agent mantém responsabilidade de execução
- sem abstrações adicionais
- sem preparar próximos agents
- sem alterar contrato base
- testes objetivos e proporcionais ao recorte

---

## 9. Critérios de Review

- o consumo do agent foi centralizado sem inflar a arquitetura
- o fluxo principal está claro
- não houve acoplamento com domínio
- contratos existentes foram preservados
- testes cobrem a execução principal
- a PR permanece pequena e revisável

---

## 10. Critérios de Aceite

- [ ] existe serviço mínimo consumindo `PdfExtractionAgent`
- [ ] o serviço retorna o output esperado do agent
- [ ] testes unitários estão passando
- [ ] nenhum componente extra desnecessário foi introduzido
- [ ] suíte relacionada permanece verde

---

## 11. Conclusão

A PR 44 continua a Fase 2 com o próximo passo mínimo após a criação do primeiro agent concreto: seu consumo compartilhado e controlado. O avanço preserva a arquitetura aprovada, mantém simplicidade e prepara a trilha sem antecipar coordenação maior entre agents.
