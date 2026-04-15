# 🧩 PR 40 — Fase 2: Consolidação do Status Contextual Funcional em `content`
## Leitura funcional única e explícita do fluxo contextual no boundary já aprovado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-40-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-context%20status-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-content%20status%20funcional-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui a linha das PRs 38 e 39 ao consolidar a leitura funcional do fluxo contextual em um status único no retorno de `content`. Após calibrar quando buscar contexto e sinalizar a tentativa, o próximo passo mínimo é comunicar o estado final de forma direta no boundary já aprovado.
>
> - preserva `content` como boundary funcional centralizador
> - adiciona status contextual derivado e explícito
> - reduz ambiguidade na leitura combinada entre flags
> - mantém o recorte pequeno, interno e revisável
>
> **Este PR não expõe documentos, chunks, scores, tracing, dashboard operacional ou nova API paralela.**

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

A PR 38 reduziu consultas desnecessárias ao calibrar quando o contexto local deve ser buscado. A PR 39 passou a informar de forma explícita se houve tentativa contextual. O próximo passo mínimo natural é consolidar essa leitura em um único status funcional no retorno de `content`.

Esta PR adiciona um campo derivado no metadata do fluxo de `content`, permitindo leitura imediata do resultado contextual sem depender da interpretação combinada entre múltiplas flags. A arquitetura permanece a mesma; a evolução está apenas na comunicação funcional do estado final.

---

## 2. Objetivo do PR

- adicionar `knowledgeContextStatus` no retorno funcional de `content`
- representar de forma explícita ausência de tentativa, tentativa sem contexto e contexto incluído
- melhorar a legibilidade do metadata para quem consome o endpoint
- preservar compatibilidade com o fluxo atual
- evoluir o boundary existente sem expandir a arquitetura

---

## 3. Decisão Arquitetural

A decisão central desta PR é derivar o novo status no próprio `ContentService`, que já é o owner do boundary funcional de `content`. O `AiService` permanece responsável pelos fatos brutos do fluxo interno, enquanto `content` passa a devolver uma leitura consolidada e pública do resultado contextual.

Com isso, a PR evita deslocar responsabilidade para camadas inferiores e também evita criar um contrato paralelo apenas para representar um detalhe derivado. Não há novo endpoint, nova superfície pública independente ou ampliação de observabilidade. A mudança fica inteiramente dentro da estrutura já aprovada.

---

## 4. Escopo

- adicionar `knowledgeContextStatus` no metadata funcional de `content`
- derivar o status a partir das flags já existentes no retorno interno
- preservar os campos já expostos no contrato atual
- ajustar testes proporcionais ao slice
- manter inalterado o comportamento atual de execução

---

## 5. Fora de Escopo

- remoção imediata das flags atuais
- retorno de documentos ao cliente
- exposição de chunks ou scores de similaridade
- tracing distribuído
- dashboard administrativo
- múltiplas fontes documentais
- reranking
- estratégias híbridas de busca
- observabilidade expandida

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Requisição funcional] --> B[ContentController]
    B --> C[ContentService]
    C --> D[Resultado bruto do AiService]
    D --> E[Derivação do status funcional]
    E --> F[Resposta com metadata consolidado]
```

O fluxo principal permanece o mesmo. A mudança desta PR está apenas na consolidação do estado contextual antes da resposta final ser devolvida ao consumidor.

---

## 7. Contratos Mínimos

```ts
type KnowledgeContextStatus =
  | 'not_attempted'
  | 'attempted_without_context'
  | 'included';

type ContentExecuteOutput = {
  output: string;
  metadata: {
    knowledgeQuery: string;
    knowledgeLimit: number;
    attemptedKnowledgeContext: boolean;
    includedKnowledgeContext: boolean;
    knowledgeContextStatus: KnowledgeContextStatus;
  };
};
```

O contrato permanece enxuto e o novo campo é apenas derivado das informações já existentes. Não há introdução de payload documental, estrutura de debug ou metadata operacional expandido.

---

## 8. Regras de Implementação

A derivação do status deve permanecer simples, explícita e centralizada no `ContentService`.

- sem tentativa contextual: `not_attempted`
- houve tentativa sem contexto incluído: `attempted_without_context`
- contexto incluído no prompt final: `included`

A implementação não deve duplicar chamadas, redistribuir responsabilidades internas nem introduzir novas dependências. O objetivo aqui é apenas consolidar a leitura funcional do estado final no boundary já existente.

---

## 9. Critérios de Review

- `content` permanece como boundary funcional centralizador
- o novo status representa corretamente os cenários possíveis do fluxo atual
- as flags atuais continuam coerentes com o status derivado
- o ajuste permaneceu pequeno e proporcional ao slice
- não houve expansão arquitetural indevida
- o contrato externo ficou mais claro sem ficar mais pesado
- a documentação e os testes seguem proporcionais ao recorte

---

## 10. Critérios de Aceite

- [ ] o retorno funcional inclui `knowledgeContextStatus`
- [ ] o status reflete corretamente o fluxo executado
- [ ] o comportamento atual de execução foi preservado
- [ ] o contrato externo continua enxuto
- [ ] os testes afetados continuam passando
- [ ] nenhuma nova superfície pública foi criada

---

## 11. Conclusão

A PR 40 consolida a comunicação funcional do fluxo contextual construído nas PRs anteriores. Sem ampliar arquitetura nem expor detalhes internos, o sistema passa a oferecer uma leitura mais direta e previsível do estado final do contexto em `content`.

Trata-se de uma evolução pequena, coerente e revisável: mais clareza no boundary existente, preservando simplicidade, continuidade e baixo ruído para review.
