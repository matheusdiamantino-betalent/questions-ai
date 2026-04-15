# 🧩 PR 38 — Fase 2: Refinamento Mínimo do Metadata Funcional da Base Local em `content`
## Consolidação semântica do retorno funcional sobre uso de contexto local no boundary centralizador já aprovado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-38-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-metadata%20refinement-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-content%20refinando%20metadata-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR dá continuidade direta à PR 37. Após expor visibilidade funcional mínima sobre o uso da base local em `content`, o próximo passo natural é consolidar a semântica desse retorno, tornando o metadata mais claro, consistente e aderente ao que o fluxo realmente sabe.
>
> - preserva `content` como boundary funcional centralizador
> - refina o contrato externo sem expandir surface pública
> - mantém `shared/ai` e `shared/ingestion` como capacidades internas
> - melhora clareza funcional com recorte pequeno e revisável
>
> **Este PR não adiciona score, chunks, lista de documentos, dashboard, observabilidade expandida ou nova API pública.**

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

A PR 37 tornou explícito no retorno funcional de `content` quando o fluxo incluiu contexto local recuperado internamente. O próximo passo natural é refinar essa comunicação para que o contrato externo reflita com mais precisão o comportamento real do sistema.

Esta PR atua nesse ponto específico: ajustar nomes, consistência e semântica do metadata já existente, sem alterar o boundary aprovado e sem introduzir novos detalhes operacionais.

O foco permanece no menor passo útil: melhorar clareza funcional do retorno sem inflar arquitetura.

---

## 2. Objetivo do PR

- refinar a semântica do metadata retornado por `content`
- alinhar nomenclaturas ao comportamento real do fluxo
- preservar o contrato pequeno e funcional
- manter `content` como boundary centralizador
- evoluir com baixo ruído e alta revisabilidade

---

## 3. Decisão Arquitetural

A decisão central desta PR é tratar o metadata como parte do contrato funcional do boundary já existente, e não como porta para exposição de detalhes internos.

Por isso, a evolução acontece apenas sobre a semântica do retorno atual: nomes mais precisos, consistência entre camadas e clareza sobre o que de fato ocorreu no fluxo.

Não há novas rotas, novos módulos ou trilhas paralelas. O sistema continua concentrando a capability em `content`, enquanto recuperação vetorial e composição contextual permanecem internas.

---

## 4. Escopo

- revisar nomenclaturas do metadata funcional
- alinhar retorno de `AiService` e `ContentService`
- preservar shape enxuto do contrato externo
- ajustar testes afetados pela consolidação semântica
- manter comportamento funcional já aprovado

---

## 5. Fora de Escopo

- score de similaridade no retorno
- quantidade de chunks utilizados
- lista de documentos recuperados
- ids de fontes públicas
- dashboard operacional
- trilha administrativa
- observabilidade expandida
- múltiplas fontes documentais
- reranking adicional
- nova API pública de knowledge base
- reorganização estrutural para cenários futuros

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Requisição funcional] --> B[ContentController]
    B --> C[ContentService]
    C --> D[AiService]
    D --> E[Recuperação de contexto]
    E --> F[Resposta funcional com metadata refinado]
```

O fluxo público permanece inalterado. O ajuste desta PR acontece na qualidade semântica do retorno já existente.

---

## 7. Contratos Mínimos

O contrato continua pequeno. A mudança esperada é tornar o campo de sinalização semanticamente fiel ao que o sistema realmente sabe.

```ts
type ContentExecuteOutput = {
  output: string;
  metadata: {
    knowledgeQuery: string;
    knowledgeLimit: number;
    includedKnowledgeContext: boolean;
  };
};
```

A intenção é comunicar que o contexto foi incluído no fluxo, e não afirmar algo que o modelo internamente utilizou de forma verificável.

---

## 8. Regras de Implementação

A implementação deve seguir o menor caminho útil. O comportamento funcional permanece o mesmo; o foco está em nomenclatura, consistência e atualização proporcional dos testes.

`ContentController` permanece fino. `ContentService` continua como owner do caso de uso funcional. `AiService` segue responsável pela composição contextual e sinalização interna.

Se houver dúvida entre expandir o contrato ou apenas refiná-lo semanticamente, esta PR deve favorecer a segunda opção.

---

## 9. Critérios de Review

- `content` permanece como boundary funcional centralizador
- o metadata está semanticamente mais preciso
- `AiService` e `ContentService` estão consistentes entre si
- não houve exposição de detalhes internos de retrieval
- o ajuste ficou pequeno, claro e proporcional ao slice
- a PR se posiciona como continuação natural da PR 37

---

## 10. Critérios de Aceite

- [ ] o fluxo funcional permanece centralizado em `content`
- [ ] o metadata usa nomenclatura aderente ao comportamento real
- [ ] `AiService` e `ContentService` compartilham o mesmo contrato semântico
- [ ] nenhum detalhe interno adicional vazou no retorno externo
- [ ] testes afetados foram ajustados e continuam passando
- [ ] a implementação permaneceu pequena e revisável

---

## 11. Conclusão

A PR 38 representa o passo incremental esperado após a PR 37: consolidar semanticamente o metadata funcional da base local em `content`.

Sem expandir surface pública, sem abrir novas trilhas e sem inflar o contrato, a entrega melhora a clareza do boundary já aprovado e mantém a evolução do projeto simples, consistente e aderente ao momento atual.

