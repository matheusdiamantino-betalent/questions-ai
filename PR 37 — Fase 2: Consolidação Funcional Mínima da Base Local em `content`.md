# 🧩 PR 37 — Fase 2: Consolidação Funcional Mínima da Base Local em `content`
## Exposição funcional mínima do uso da base jurídica local pelo boundary centralizador já aprovado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-37-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-functional%20consolidation-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-content%20consolidando%20base%20local-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR ajusta o próximo passo da fase para permanecer aderente ao boundary já aprovado. Depois de validar ingestão, consumo funcional e refinamento inicial da recuperação, o foco agora é consolidar em `content` uma leitura funcional mínima sobre o uso da base local, sem reabrir `ingestion` como porta pública.
>
> - preserva `content` como centralizador funcional do fluxo
> - mantém `shared/ai` e `shared/ingestion` como capacidades internas
> - adiciona visibilidade funcional mínima sem abrir novo boundary público
> - mantém o recorte pequeno, interno e revisável
>
> **Este PR não reintroduz controller de `ingestion`, não cria dashboard, não abre gestão administrativa e não expande retrieval além do necessário.**

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

As PRs anteriores fecharam o núcleo técnico da fase: a base jurídica local passou a ser ingerida, persistida, consumida por `content` e refinada no uso contextual do fluxo real.

O passo seguinte precisa continuar coerente com a direção já aprovada. Como o boundary funcional foi concentrado em `content`, a evolução natural não é reabrir `ingestion` como porta pública, e sim consolidar nesse mesmo ponto uma leitura funcional mínima sobre o uso da base local.

Esta PR segue exatamente essa linha. O objetivo não é criar uma interface operacional paralela, mas tornar o consumo já existente mais verificável e explícito dentro do fluxo funcional centralizado.

---

## 2. Objetivo do PR

- consolidar `content` como boundary funcional da base local
- expor metadados mínimos sobre o uso do contexto recuperado
- tornar o consumo da base jurídica mais verificável no retorno funcional
- manter `shared/ai` e `shared/ingestion` como detalhes internos
- evoluir sem abrir novo controller público ou nova trilha administrativa

---

## 3. Decisão Arquitetural

A decisão central desta PR é preservar a concentração funcional em `content`. O sistema continua expondo a capability pelo caso de uso real, enquanto ingestion, persistência vetorial e recuperação contextual permanecem internos à implementação.

Não há reintrodução de `IngestionController`, nem criação de rota operacional paralela. A nova visibilidade entra no retorno funcional já existente, de forma pequena e proporcional, apenas para tornar mais explícito o uso da base local no fluxo centralizado.

Com isso, a evolução segue por continuidade controlada: consolidar o boundary aprovado antes de expandir novas capacidades.

---

## 4. Escopo

- adicionar metadados mínimos sobre uso da base local no fluxo de `content`
- explicitar no retorno funcional quando há consumo de contexto recuperado
- manter integração via `ContentService` e `AiService`
- preservar contracts enxutos e compatíveis com a fase atual
- cobrir o ajuste com testes proporcionais ao slice

---

## 5. Fora de Escopo

- reintrodução de controller de `ingestion`
- dashboard operacional
- leitura administrativa de execuções
- histórico completo de ingestões
- observabilidade expandida
- múltiplas fontes documentais
- reranking adicional
- hybrid search
- nova API pública de knowledge base
- reorganização estrutural para futuros cenários

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A[Requisição funcional] --> B[ContentController]
    B --> C[ContentService]
    C --> D[AiService]
    D --> E[Recuperação de contexto]
    E --> F[Resposta funcional com metadados mínimos]
```

O fluxo público permanece o mesmo. A diferença desta PR está em tornar mais explícito, no próprio boundary funcional, o uso da base local já consumida internamente.

---

## 7. Contratos Mínimos

O retorno deve continuar pequeno. A adição de metadados deve comunicar apenas o necessário para indicar uso funcional da base local, sem expor detalhes internos de retrieval, chunks, scores ou estruturas operacionais.

```ts
type ContentExecuteOutput = {
  output: string;
  metadata: {
    knowledgeQuery: string;
    knowledgeLimit: number;
    usedKnowledgeContext: boolean;
  };
};
```

Campos adicionais só devem entrar se forem estritamente necessários ao recorte.

---

## 8. Regras de Implementação

A implementação deve seguir o menor caminho útil. O ajuste deve acontecer no fluxo já existente de `content`, reutilizando `AiService` e contratos atuais sempre que possível.

`ContentController` permanece fino. `ContentService` continua centralizando o caso de uso funcional. `shared/ai` permanece owner da recuperação e da composição contextual. Nenhuma estrutura operacional paralela deve ser reintroduzida no boundary público.

Se houver dúvida entre ampliar surface pública ou manter a consolidação dentro de `content`, esta PR deve favorecer a segunda opção.

---

## 9. Critérios de Review

- `content` permanece como boundary funcional centralizador
- o retorno funcional passa a comunicar uso mínimo da base local
- `shared/ingestion` e `shared/ai` seguem internos
- não houve reabertura de `ingestion` como porta pública
- o ajuste ficou pequeno, claro e proporcional ao slice
- a PR se posiciona como continuação natural das PRs 35 e 36

---

## 10. Critérios de Aceite

- [ ] o fluxo de `content` continua centralizando o caso de uso funcional
- [ ] o retorno funcional inclui metadado mínimo sobre uso da base local
- [ ] nenhum controller público de `ingestion` foi reintroduzido
- [ ] detalhes internos de retrieval não vazam no contrato externo
- [ ] a implementação reutiliza o fluxo e os serviços já existentes
- [ ] testes existentes continuam passando

---

## 11. Conclusão

A PR 37 consolida o boundary funcional já aprovado ao tornar mais explícito, em `content`, o uso da base jurídica local. Em vez de reabrir uma porta pública operacional, a evolução permanece no fluxo centralizado que já foi validado na fase.

A entrega segue pequena e revisável: reforçar a clareza funcional do consumo da base local, preservar ownership interno e manter o projeto avançando sem redundância nem expansão arquitetural indevida.
