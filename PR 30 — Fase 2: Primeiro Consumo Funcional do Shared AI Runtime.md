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
> Esta PR é a continuação direta da PR 29. Depois de materializar a foundation mínima em `shared/ai`, o próximo passo correto é validar essa base em um único consumo funcional real, pequeno e controlado, dentro de `modules/content`.
>
> - mantém a foundation compartilhada introduzida na PR 29
> - valida essa foundation em um fluxo real de negócio dentro de `content`
> - preserva `shared/ai` como capability compartilhada e `content` como boundary consumidor
>
> **Este PR não expande a foundation, não introduz LangGraph, não abre múltiplos produtos, não cria framework genérico de agents e não amplia o recorte além do primeiro consumo funcional necessário.**

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

A PR 29 consolidou a primeira materialização funcional mínima de `shared/ai`, deixando o projeto com uma capability compartilhada real para runtime e consumo controlado de conhecimento, sem inflar a discussão estratégica em uma entrega estrutural maior do que o necessário.

A PR 30 continua exatamente desse ponto. Em vez de ampliar a foundation ou abrir novas frentes paralelas, ela valida essa base em um único fluxo funcional real dentro de `modules/content`, comprovando que a separação entre capability compartilhada e boundary de negócio funciona também em uso concreto.

O passo correto aqui não é reorganizar mais a estrutura, mas demonstrar que a estrutura já criada é suficiente para suportar um caso real com baixo ruído, sem duplicação de lógica e sem transformar o módulo consumidor em novo dono da infraestrutura de IA.

---

## 2. Objetivo do PR

- validar a foundation da PR 29 em um fluxo funcional real
- fazer `modules/content` consumir `shared/ai` como capability compartilhada
- evitar duplicação de lógica de IA no boundary consumidor
- manter a separação clara entre runtime compartilhado e regra de negócio
- estabelecer o primeiro uso concreto da base comum antes de qualquer expansão posterior

---

## 3. Decisão Arquitetural

A decisão central desta PR é manter `shared/ai` como dono das capacidades reutilizáveis e fazer `modules/content` consumir essa capability apenas no ponto funcional em que ela é necessária. A arquitetura aprovada permanece a mesma: a base compartilhada concentra runtime, integração e composição técnica, enquanto o módulo de negócio continua responsável apenas pelo seu caso de uso.

Nesta etapa, a evolução não acontece por nova fundação nem por abstrações adicionais. Ela acontece por uma integração objetiva entre o boundary consumidor e o runtime já disponível. O valor desta PR está justamente em provar o consumo real da base comum sem redesenhar o projeto, sem reabrir decisões anteriores e sem preparar cenários que ainda não entraram no recorte.

---

## 4. Escopo

Entram nesta PR apenas os pontos necessários para o primeiro consumo funcional da foundation:

- integração de `modules/content` com o runtime já disponível em `shared/ai`
- uso de um único fluxo funcional real para validar a capability compartilhada
- composição mínima entre entrada do boundary consumidor, execução compartilhada e retorno útil
- reaproveitamento da base existente sem duplicação local de lógica de IA
- manutenção do shape atual do projeto, com controller fino e fluxo principal explícito

---

## 5. Fora de Escopo

Ficam explicitamente fora desta PR:

- expansão estrutural adicional dentro de `shared/ai`
- múltiplos fluxos consumidores ao mesmo tempo
- abstração genérica para diversos tipos de prompt, agent ou tool execution
- LangGraph, planner, memória conversacional ou orquestração complexa
- rollout transversal para outros boundaries além do primeiro caso em `content`
- redesign de contratos públicos já existentes sem pressão real do fluxo
- retries, cache, observabilidade expandida, fila dedicada, fallback de modelo ou preparo indireto de próximas fases

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A["request em modules/content"] --> B["serviço funcional do boundary"]
    B --> C["shared/ai runtime"]
    C --> D["retrieval mínimo opcional"]
    C --> E["execução compartilhada"]
    E --> F["resultado útil"]
    F --> G["resposta do fluxo de content"]
```

---

## 7. Contratos Mínimos

Esta PR não deve inflar o contrato público do sistema nem introduzir uma API genérica de IA. O contrato novo, quando existir, deve ser apenas o mínimo necessário para permitir que o fluxo funcional de `content` acione a capability compartilhada e receba um resultado útil de forma explícita.

No boundary consumidor, o contrato deve refletir somente a necessidade real do caso atendido nesta entrega. Em `shared/ai`, os contratos internos devem permanecer pequenos, orientados ao fluxo e sem generalização prematura para cenários ainda inexistentes.

---

## 8. Regras de Implementação

O controller em `content` deve permanecer fino e focado em HTTP. O serviço funcional do módulo consumidor deve concentrar apenas a orquestração mínima do caso de uso. O runtime compartilhado em `shared/ai` continua responsável pelas capacidades reutilizáveis de execução e retrieval. DAO, quando envolvido, permanece restrito à persistência e ao acesso vetorial, sem absorver decisão de negócio do boundary consumidor.

Também é importante que o primeiro consumo não abra múltiplos caminhos paralelos. O reviewer precisa entender a entrega lendo poucos arquivos centrais e enxergando o fluxo principal com facilidade. Esta PR existe para comprovar o uso real da foundation já criada, não para expandir abstrações “aproveitando a oportunidade”.

---

## 9. Critérios de Review

O review desta PR deve validar principalmente os seguintes pontos:

- a PR continua a PR 29 de forma natural, sem reabrir a fase foundation
- `modules/content` consome `shared/ai` como capability compartilhada real
- o fluxo funcional entregue é pequeno, concreto e fácil de revisar
- não houve duplicação de lógica de IA dentro do módulo consumidor
- o boundary de negócio permaneceu consumidor, e não dono da infraestrutura compartilhada
- não foram criadas abstrações genéricas além da pressão real deste caso
- o fluxo principal continua explícito e com pouca cerimônia
- a entrega prova a utilidade da foundation sem tentar expandir o roadmap inteiro

---

## 10. Critérios de Aceite

- [ ] existe um fluxo funcional real em `modules/content` consumindo a foundation introduzida na PR 29
- [ ] `shared/ai` permanece como capability compartilhada reutilizável, sem duplicação local no consumidor
- [ ] o controller do boundary permanece fino e o fluxo principal pode ser entendido rapidamente
- [ ] o consumo do runtime compartilhado acontece por contratos mínimos e explícitos
- [ ] não foram introduzidos LangGraph, múltiplos agents, tool registry genérico ou expansão estrutural indevida
- [ ] a implementação permanece proporcional ao slice e aderente ao shape atual do projeto
- [ ] a PR valida uso real da foundation sem reabrir arquitetura aprovada

---

## 11. Conclusão

A PR 30 é o passo que transforma a foundation mínima da PR 29 em uso funcional concreto. Depois de consolidar `shared/ai` como base compartilhada e materializar seu runtime inicial, a evolução correta agora é validá-lo em um consumidor real, pequeno e controlado.

O ganho desta entrega está em confirmar, com um caso funcional único e revisável, que a separação arquitetural aprovada continua correta: capability compartilhada em `shared/ai`, boundary funcional em `modules/content`, integração explícita e baixo ruído. A PR mantém a fase coesa, prova reutilização real e evita antecipar expansão estrutural antes de existir pressão concreta para isso.
