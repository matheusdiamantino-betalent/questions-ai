# 🏷️ PR 49 — Fase 2: Primeiro Classification Agent em Ingestion
## Enriquecimento semântico mínimo do resultado operacional com categoria e confiança

---

<div align="left">

![PR](https://img.shields.io/badge/PR-49-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-classification%20agent%20em%20ingestion-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 48. Após estabilizar a leitura pública da ingestion com resultado estruturado, o próximo passo mínimo correto é enriquecer esse resultado com classificação semântica simples, reutilizando o boundary de agents já criado e sem expandir a arquitetura da fase.
>
> - adiciona classificação mínima baseada em `processedContent`
> - persiste categoria e confiança de forma explícita
> - mantém processor linear e fluxo operacional intacto
> - preserva compatibilidade do contrato público já consolidado
>
> **Esta PR não implementa multi-agent orchestration, taxonomia complexa, busca avançada, UI analítica ou pipelines paralelos.**

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

As PRs anteriores consolidaram o fluxo principal da ingestion em etapas objetivas: processamento do conteúdo, persistência explícita do resultado e leitura pública consistente desse valor. A base operacional já está funcional e previsível.

O próximo avanço incremental natural é adicionar valor semântico ao conteúdo já processado. A PR 49 introduz o primeiro `ClassificationAgent`, responsável por inferir uma categoria simples e um nível de confiança a partir de `processedContent`, ampliando utilidade do resultado sem alterar a estrutura central do pipeline.

---

## 2. Objetivo do PR

- introduzir `ClassificationAgent` no boundary compartilhado de agents
- classificar `processedContent` ao final do fluxo de ingestion
- persistir `category` e `confidence` junto ao resultado final
- manter processor linear e de fácil leitura
- validar o comportamento com testes mínimos e proporcionais ao recorte

---

## 3. Decisão Arquitetural

A classificação entra como uma única etapa adicional dentro do fluxo já existente. Não há novo worker, nova fila, novo módulo ou orquestração paralela.

O processor apenas delega ao novo agent após a geração de `processedContent` e persiste o enriquecimento no mesmo resultado já utilizado pelo sistema. A decisão preserva simplicidade operacional, reaproveita a foundation criada na Fase 2 e evita expansão indevida.

---

## 4. Escopo

- criar `classification.agent.ts`
- adicionar contratos mínimos de classificação
- executar o agent após gerar `processedContent`
- persistir metadados semânticos no resultado final
- ajustar testes de processor e persistência
- manter API pública compatível com contratos anteriores

---

## 5. Fora de Escopo

- múltiplos classificadores
- taxonomia avançada
- treinamento de modelo
- feedback loop
- dashboard analítico
- busca semântica
- cache distribuído
- orchestrator de agents
- mudanças no fluxo HTTP já consolidado

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
    A[Payload.content] --> B[Extraction]
    B --> C[processedContent]
    C --> D[ClassificationAgent]
    D --> E[category + confidence]
    E --> F[result enriquecido]
    F --> G[completed]
```

---

## 7. Contratos Mínimos

O resultado da ingestion passa a aceitar um enriquecimento semântico opcional, mantendo compatibilidade com o shape já existente.

```ts
export type IngestionClassification = {
  category: string;
  confidence: number;
};

export type IngestionResult = {
  extractedText: string;
  processedContent: string;
  classification?: IngestionClassification;
};
```

Exemplo esperado:

```json
{
  "result": {
    "extractedText": "texto extraído",
    "processedContent": "texto processado",
    "classification": {
      "category": "financeiro",
      "confidence": 0.92
    }
  }
}
```

---

## 8. Regras de Implementação

A classificação deve permanecer objetiva e previsível. O agent recebe texto processado e retorna um shape simples, sem estado interno, sem dependências cruzadas e sem antecipação de cenários futuros.

A alteração deve ficar localizada no ponto de conclusão da ingestion. Não introduzir camadas extras, pipelines paralelos ou abstrações além do necessário para este slice.

---

## 9. Critérios de Review

Validar se o novo agent respeita o boundary compartilhado, se o processor continua legível e se a persistência do enriquecimento ficou clara e proporcional ao recorte.

Também deve ser confirmado que não houve expansão arquitetural além do necessário e que o contrato público permanece compatível.

---

## 10. Critérios de Aceite

- [ ] `ClassificationAgent` foi criado e testado
- [ ] classificação executa após geração de `processedContent`
- [ ] categoria é persistida no resultado
- [ ] confiança é persistida no resultado
- [ ] fluxo principal continua simples e linear
- [ ] testes permanecem passando
- [ ] nenhum componente estrutural extra foi criado

---

## 11. Conclusão

A PR 49 adiciona o primeiro ganho semântico direto sobre o pipeline já consolidado. O sistema passa a não apenas processar conteúdo, mas também qualificá-lo minimamente com categoria e confiança.

O avanço segue pequeno, incremental e coerente com a fase atual. Não há redesenho do fluxo, apenas acréscimo funcional claro sobre uma base já estabilizada.
