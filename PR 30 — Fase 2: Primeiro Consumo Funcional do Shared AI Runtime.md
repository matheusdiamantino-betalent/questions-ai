# 🧩 PR 30 — Fase 2: Primeiro Consumo Funcional do Shared AI Runtime
## Validação do runtime compartilhado de IA em um fluxo real de negócio dentro de `modules/content`

---

<div align="left">

![PR](https://img.shields.io/badge/PR-30-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-primeiro%20consumo%20funcional-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-content%20consumindo%20shared%20ai-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é a continuação direta da PR 29. Depois de materializar a foundation mínima em `shared/ai`, o próximo passo correto é validar essa base em um consumo funcional real, pequeno e controlado, dentro de `modules/content`.
>
> - mantém a foundation compartilhada introduzida na PR 29
> - faz `content` consumir a capability já consolidada em `shared/ai`
> - preserva a separação entre base compartilhada e boundary funcional consumidor
>
> **Este PR não expande a foundation, não introduz LangGraph, não abre múltiplos fluxos de negócio, não cria framework genérico de agents e não amplia o recorte além do primeiro consumo funcional necessário.**

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

A PR 29 consolidou a primeira foundation funcional mínima em `shared/ai`, reunindo o runtime compartilhado e as capacidades básicas já necessárias para execução com LLM e uso controlado de retrieval, sem transformar a fase em uma entrega estrutural maior do que o recorte pedia.

A PR 30 continua exatamente desse ponto. Em vez de ampliar a base compartilhada ou abrir novas frentes em paralelo, ela valida essa foundation em um único fluxo funcional real dentro de `modules/content`, comprovando que a capability criada na PR anterior já consegue ser consumida por um boundary de negócio de forma direta, útil e revisável.

O ganho desta entrega não está em criar nova infraestrutura, e sim em mostrar que a infraestrutura já criada é suficiente para suportar um caso real sem duplicação local, sem reabrir arquitetura e sem deslocar responsabilidades para o módulo consumidor.

---

## 2. Objetivo do PR

- validar a foundation da PR 29 em um fluxo funcional real
- fazer `modules/content` consumir `shared/ai` como capability compartilhada
- reaproveitar `AiService`, `EmbeddingsService`, `LangchainModel`, `LangchainLib` e o acesso vetorial já existentes
- evitar duplicação de lógica de IA dentro do boundary consumidor
- confirmar a separação entre capability compartilhada e fluxo funcional de negócio

---

## 3. Decisão Arquitetural

A decisão central desta PR é manter `shared/ai` como dono das capacidades reutilizáveis e fazer `modules/content` consumir essas capacidades apenas no ponto funcional em que elas são necessárias. A arquitetura aprovada permanece a mesma: `shared/ai` concentra runtime, serviços e integração técnica; `content` continua responsável apenas pelo seu caso de uso.

Nesta etapa, a evolução não acontece por nova reorganização estrutural nem por abstrações adicionais. Ela acontece por uma integração objetiva entre o boundary consumidor e os componentes já materializados na PR 29. O valor desta PR está justamente em provar esse consumo real usando a base já disponível, sem criar uma segunda fundação local e sem inflar o módulo de negócio.

---

## 4. Escopo

Entram nesta PR apenas os pontos necessários para o primeiro consumo funcional da foundation:

- integração de `content.controller.ts`, `content.service.ts` e `content.module.ts` com a capability já disponível em `shared/ai`
- consumo de `AiService` e `EmbeddingsService` a partir do boundary `content`
- reaproveitamento dos componentes já existentes em `shared/ai`, incluindo `langchain.lib.ts`, `langchain.model.ts` e `document-embeddings.dao.ts`, sem duplicação local
- composição mínima entre entrada HTTP, serviço funcional do módulo e execução compartilhada
- manutenção do shape atual do projeto, com fluxo principal explícito e poucas moving parts

---

## 5. Fora de Escopo

Ficam explicitamente fora desta PR:

- expansão estrutural adicional dentro de `shared/ai`
- novos módulos consumidores além de `content`
- múltiplos fluxos funcionais de consumo ao mesmo tempo
- abstração genérica para diversos tipos de prompt, agent ou tool execution
- LangGraph, planner, memória conversacional ou orquestração complexa
- redesign dos contratos públicos existentes sem pressão real do fluxo
- retries, cache, observabilidade expandida, fila dedicada, fallback de modelo ou preparação indireta da próxima fase
- mudanças de responsabilidade em `ingestion`, que permanece apenas como contexto anterior já consolidado

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A["request em modules/content"] --> B["ContentController"]
    B --> C["ContentService"]
    C --> D["AiService / EmbeddingsService"]
    D --> E["LangchainModel / LangchainLib"]
    D --> F["DocumentEmbeddingsDao"]
    E --> G["resultado útil"]
    F --> G
    G --> H["resposta do fluxo de content"]
```

---

## 7. Contratos Mínimos

Esta PR não deve inflar o contrato público do sistema nem introduzir uma interface genérica de IA. O contrato novo, quando existir, deve ser apenas o mínimo necessário para permitir que `content` acione a capability compartilhada e receba um resultado útil de forma explícita.

No boundary consumidor, o contrato deve refletir somente a necessidade real do caso atendido nesta entrega. Em `shared/ai`, os contratos internos permanecem pequenos, orientados ao fluxo já existente e sem generalização prematura para cenários ainda não abertos.

---

## 8. Regras de Implementação

O controller em `content` deve permanecer fino e focado em HTTP. O `ContentService` deve concentrar apenas a orquestração mínima do caso de uso. Em `shared/ai`, `AiService`, `EmbeddingsService`, `LangchainModel`, `LangchainLib` e `DocumentEmbeddingsDao` continuam responsáveis pelas capacidades reutilizáveis já introduzidas, sem deslocar regra de negócio para dentro da camada compartilhada.

Também é importante que o primeiro consumo não abra múltiplos caminhos paralelos. O reviewer deve conseguir entender a entrega lendo poucos arquivos centrais e enxergando rapidamente o fluxo principal: entrada em `content`, delegação para a capability compartilhada e retorno útil. Esta PR existe para comprovar o uso real da foundation já criada, não para ampliar abstrações aproveitando o primeiro ponto de consumo.

---

## 9. Critérios de Review

O review desta PR deve validar principalmente os seguintes pontos:

- a PR continua a PR 29 de forma natural, sem reabrir a fase foundation
- `modules/content` consome `shared/ai` como capability compartilhada real
- o fluxo funcional entregue é pequeno, concreto e fácil de revisar
- não houve duplicação de `AiService`, `EmbeddingsService`, `LangchainModel`, `LangchainLib` ou acesso vetorial dentro do módulo consumidor
- o boundary de negócio permaneceu consumidor, e não dono da infraestrutura compartilhada
- não foram criadas abstrações genéricas além da pressão real deste caso
- o fluxo principal continua explícito e com pouca cerimônia
- a entrega valida a utilidade da foundation sem tentar expandir o roadmap inteiro

---

## 10. Critérios de Aceite

- [ ] existe um fluxo funcional real em `modules/content` consumindo a foundation introduzida na PR 29
- [ ] `content.controller.ts`, `content.service.ts` e `content.module.ts` estão integrados ao uso de `shared/ai` sem duplicação local
- [ ] `AiService` e `EmbeddingsService` são reutilizados como capability compartilhada do projeto
- [ ] `LangchainModel`, `LangchainLib` e `DocumentEmbeddingsDao` permanecem concentrados em `shared/ai`
- [ ] o controller do boundary permanece fino e o fluxo principal pode ser entendido rapidamente
- [ ] não foram introduzidos LangGraph, múltiplos agents, tool registry genérico ou expansão estrutural indevida
- [ ] a implementação permanece proporcional ao slice e aderente ao shape atual do projeto

---

## 11. Conclusão

A PR 30 é o passo que transforma a foundation mínima da PR 29 em consumo funcional concreto. Depois de consolidar `shared/ai` como base compartilhada e materializar seus componentes iniciais de runtime e retrieval, a evolução correta agora é validá-los em um boundary real de negócio, pequeno e controlado.

O ganho desta entrega está em confirmar, com um único caso funcional revisável, que a separação arquitetural aprovada continua correta: capability compartilhada em `shared/ai`, boundary funcional em `modules/content`, integração explícita e baixo ruído. A PR mantém a fase coesa, prova reutilização real com a árvore atual do projeto e evita antecipar expansão estrutural antes de existir pressão concreta para isso.
