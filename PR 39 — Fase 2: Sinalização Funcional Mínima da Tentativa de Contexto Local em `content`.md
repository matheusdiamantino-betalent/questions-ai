# 🧩 PR 39 — Fase 2: Sinalização Funcional Mínima da Tentativa de Contexto Local em `content`
## Clareza adicional do fluxo contextual no boundary centralizador já aprovado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-39-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-context%20signaling-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-content%20metadata%20funcional-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui o eixo funcional das PRs 37 e 38 ao tornar o comportamento contextual mais explícito no retorno de `content`. Depois de consolidar o uso mínimo da base local e calibrar quando a busca deve ocorrer, o próximo passo natural é comunicar se houve tentativa de uso do contexto local.
>
> - preserva `content` como boundary funcional centralizador
> - adiciona sinalização mínima sobre tentativa contextual
> - melhora previsibilidade do fluxo sem expor detalhes internos
> - mantém o recorte pequeno, interno e revisável
>
> **Este PR não expõe documentos, scores, chunks, tracing, observabilidade expandida, dashboard operacional ou nova API pública.**

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

As PRs 37 e 38 consolidaram o uso funcional da base local em `content` e calibraram o disparo contextual para evitar consultas desnecessárias.

O próximo passo útil não é sofisticar retrieval, e sim tornar o comportamento atual mais explícito. Esta PR adiciona uma sinalização mínima para indicar quando houve tentativa de uso do contexto local, mesmo que nenhum contexto útil tenha sido incluído.

O resultado esperado é mais previsibilidade funcional com impacto estrutural mínimo.

---

## 2. Objetivo do PR

- informar no retorno funcional quando houve tentativa de busca contextual
- distinguir ausência de tentativa de ausência de contexto útil
- melhorar a clareza operacional do fluxo de `content`
- preservar contrato enxuto e sem detalhes internos
- evoluir sem expandir a arquitetura atual

---

## 3. Decisão Arquitetural

A decisão central desta PR é manter a sinalização no boundary funcional já existente. Em vez de criar endpoints administrativos, logs públicos ou estruturas paralelas, o próprio retorno de `content` comunica o estado mínimo do fluxo contextual.

`AiService` continua responsável pela execução contextual. `ContentService` continua owner do caso de uso funcional. O contrato externo evolui apenas no necessário para melhorar o entendimento do comportamento real.

---

## 4. Escopo

- adicionar flag mínima de tentativa contextual no metadata funcional
- preservar campos já existentes do retorno atual
- manter o comportamento atual de execução
- ajustar testes proporcionais ao slice
- evitar qualquer vazamento de detalhe interno

---

## 5. Fora de Escopo

- lista de documentos utilizados
- scores de similaridade
- chunks retornados ao cliente
- tracing distribuído
- observabilidade expandida
- dashboard administrativo
- nova rota pública
- múltiplas fontes documentais
- reranking
- estratégias híbridas de busca

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Requisição funcional] --> B[ContentController]
    B --> C[ContentService]
    C --> D{Tentou contexto?}
    D -- Não --> E[Execução direta]
    D -- Sim --> F[Busca e composição contextual]
    E --> G[Resposta com metadata]
    F --> G
```

O fluxo público permanece centralizado em `content`. A diferença desta PR está na clareza adicional do retorno funcional.

---

## 7. Contratos Mínimos

A evolução do contrato deve permanecer pequena e explícita.

```ts
type ContentExecuteOutput = {
  output: string;
  metadata: {
    knowledgeQuery: string;
    knowledgeLimit: number;
    attemptedKnowledgeContext: boolean;
    includedKnowledgeContext: boolean;
  };
};
```

Nenhum campo adicional deve expor documentos internos ou detalhes de retrieval.

---

## 8. Regras de Implementação

A implementação deve reutilizar o fluxo atual. A nova sinalização deve ser derivada do comportamento já existente, sem duplicar chamadas, sem logs artificiais e sem criar novas camadas.

Se houve query útil enviada ao fluxo contextual, a tentativa pode ser marcada como verdadeira. Se não houve query útil, a tentativa permanece falsa. A composição final deve continuar simples e direta.

---

## 9. Critérios de Review

- `content` permanece como boundary funcional centralizador
- metadata comunica se houve tentativa contextual
- ausência de tentativa e ausência de contexto útil ficam distinguíveis
- nenhum detalhe interno adicional vazou no contrato externo
- o ajuste ficou pequeno e proporcional ao slice
- a PR continua naturalmente a sequência das PRs 37 e 38

---

## 10. Critérios de Aceite

- [ ] retorno funcional inclui `attemptedKnowledgeContext`
- [ ] comportamento atual de execução foi preservado
- [ ] tentativa contextual reflete o fluxo real executado
- [ ] contrato externo continua enxuto
- [ ] testes afetados continuam passando
- [ ] nenhuma nova superfície pública foi criada

---

## 11. Conclusão

A PR 39 adiciona clareza funcional ao eixo já construído nas PRs anteriores. Sem ampliar a arquitetura nem expor detalhes internos, o sistema passa a comunicar melhor quando tentou utilizar a base local no fluxo de `content`.

É uma evolução pequena, útil e revisável: mais previsibilidade para o comportamento atual, mantendo intacto o boundary centralizador já aprovado.
