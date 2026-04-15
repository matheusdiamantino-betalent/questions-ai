# 🧩 PR 34 — Fase 2: Base Local Inicial de Conhecimento no Shared AI Runtime
## Primeira ingestão vetorial mínima do Código Civil em corpus jurídico local sobre a foundation compartilhada existente

---

<div align="left">

![PR](https://img.shields.io/badge/PR-33-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-base%20local%20de%20conhecimento-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ingestao%20vetorial%20juridica%20inicial-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é a continuação natural da PR 32. Após validar a entrada mínima de LangGraph dentro de `shared/ai`, o próximo passo funcional correto é dar utilidade real à foundation já consolidada por meio de uma base local inicial de conhecimento, usando uma fonte concreta, estável e juridicamente relevante como primeiro corpus vetorial.
>
> - preserva a foundation compartilhada consolidada nas PRs 29, 30, 31 e 32
> - materializa o primeiro corpus jurídico real para uso local dentro de `shared/ai`
> - usa o **Código Civil (Lei nº 10.406/2002)**, a partir da versão compilada publicada no Planalto, como fonte inicial fixa e controlada
> - valida um fluxo mínimo de leitura, particionamento, embedding e persistência vetorial sem expor `modules/content` aos detalhes da ingestão
>
> **Este PR não introduz crawler genérico, múltiplas fontes, sincronização automática, reindexação sofisticada, versionamento documental, pipeline universal de parsing ou expansão transversal de knowledge ingestion.**

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

A PR 29 consolidou a foundation inicial de `shared/ai` como boundary técnico reutilizável. A PR 30 validou o primeiro consumo funcional dessa base em `modules/content`. A PR 31 expandiu o uso real do runtime compartilhado no mesmo fluxo, preservando o boundary aprovado. A PR 32 introduziu LangGraph de forma mínima e controlada como capacidade interna do runtime, sem transformar a arquitetura em uma plataforma inflada de agents.

A PR 33 continua essa trilha com um passo mais útil do que expandir orchestration: introduzir a primeira base local real de conhecimento sobre a foundation já existente. O foco agora deixa de ser apenas provar capacidade interna e passa a validar um fluxo concreto de ingestão vetorial com uma fonte jurídica fixa, estável e revisável, permitindo que o projeto tenha um primeiro corpus local persistido para consultas sem depender apenas de execução textual isolada.

Nesta entrega, a fonte escolhida é o **Código Civil (Lei nº 10.406, de 10 de janeiro de 2002)**, utilizando a versão compilada disponibilizada no portal oficial do Planalto como corpus inicial. O ganho funcional está em fechar o primeiro ciclo mínimo de conhecimento local: ler uma base legal real, particionar de forma simples, gerar embeddings e persistir esse material no banco vetorial. Isso cria a primeira base jurídica local utilizável dentro de `shared/ai`, mantendo o escopo pequeno, explícito e alinhado ao feedback funcional já recebido nas PRs anteriores.

---

## 2. Objetivo do PR

- introduzir a primeira ingestão vetorial mínima de corpus jurídico real em `shared/ai`
- usar o Código Civil publicado no Planalto como fonte inicial fixa da base local de conhecimento
- validar o fluxo mínimo de leitura, particionamento, embedding e persistência vetorial
- manter o consumo dessa capacidade encapsulado no boundary compartilhado
- transformar a foundation existente em uma base local concretamente utilizável sem inflar o recorte

---

## 3. Decisão Arquitetural

A decisão central desta PR é tratar a base local de conhecimento como continuação prática do runtime compartilhado já aprovado, e não como abertura de uma nova frente de infraestrutura documental. Em vez de criar um sistema genérico de coleta e indexação, a entrega assume uma única fonte inicial fixa e controlada como corpus de validação.

Esse posicionamento preserva o shape do projeto. `shared/ai` continua sendo o owner das capacidades reutilizáveis, agora agregando o primeiro fluxo mínimo de knowledge ingestion. O módulo consumidor não passa a conhecer scraping, chunking, embeddings, documentos ou detalhes de armazenamento vetorial. Esses pontos permanecem internos ao boundary compartilhado.

A escolha pelo Código Civil como primeiro corpus reduz ambiguidade, facilita review e ancora a entrega em uma base legal estável, pública e imediatamente útil para o domínio pretendido. O objetivo da PR não é resolver o problema completo de aquisição de conhecimento, mas provar que a arquitetura atual já comporta uma primeira base local útil e persistida com o menor número possível de moving parts.

---

## 4. Escopo

- definir o Código Civil compilado no portal do Planalto como primeira fonte jurídica fixa do projeto
- obter o conteúdo dessa fonte dentro de um fluxo mínimo e controlado
- aplicar particionamento simples e legível sobre o conteúdo obtido
- gerar embeddings para os trechos produzidos
- persistir os registros no banco vetorial já previsto na foundation compartilhada
- manter o fluxo concentrado em `shared/ai`, sem vazamento estrutural para `modules/content`
- deixar validado o primeiro uso real de conhecimento local dentro da sequência das PRs 29–32

---

## 5. Fora de Escopo

- crawler genérico para múltiplos domínios
- ingestão de múltiplas leis, códigos ou bases documentais
- sincronização automática de fontes remotas
- atualização incremental sofisticada
- versionamento documental
- deduplicação avançada
- parser universal para HTML, PDF e outros formatos
- pipeline completo de reindexação
- painel operacional de ingestão
- expansão de LangGraph para orquestrar a ingestão
- rollout transversal imediato para outros consumidores
- qualquer camada estrutural criada apenas como preparação para futuras fontes

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'dark','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#39ff14','lineColor':'#00e5ff','secondaryColor':'#111827','tertiaryColor':'#111827'}}}%%
flowchart LR
    A["Código Civil no Planalto"] --> B["Leitura controlada do conteúdo"]
    B --> C["Particionamento mínimo"]
    C --> D["Geração de embeddings"]
    D --> E["Persistência vetorial em shared/ai"]
    E --> F["Base local inicial disponível"]
```

O fluxo desta PR permanece pequeno e linear. Não há orquestração rica, fan-out de fontes nem pipeline expandido. A entrega apenas fecha o primeiro ciclo mínimo necessário para transformar uma fonte legal concreta em base vetorial local utilizável dentro da foundation compartilhada.

---

## 7. Contratos Mínimos

Os contratos públicos devem permanecer simples neste slice. A PR não exige abertura de um contrato genérico de documentos, coleções, conectores ou registries de fonte. O que existir de contrato interno deve ser estritamente o suficiente para representar a origem fixa do conteúdo inicial, o texto particionado a ser vetorizado e os metadados mínimos úteis para persistência e leitura posterior.

O módulo consumidor não deve depender do shape interno de chunk, processo de ingestão ou estratégia de embedding. Toda a modelagem adicional deve permanecer encapsulada em `shared/ai` e limitada ao que é necessário para esta entrega.

Exemplo de metadado mínimo aceitável para este slice:

```ts
type KnowledgeChunkSource = {
  sourceKey: 'codigo-civil-planalto';
  sourceUrl: 'https://www.planalto.gov.br/ccivil_03/leis/2002/l10406compilada.htm';
  title: 'Código Civil - Lei nº 10.406/2002';
};
```

---

## 8. Regras de Implementação

A implementação deve favorecer um caminho único, direto e totalmente legível. A fonte inicial deve ser explícita, fixa e revisável. O processo de leitura do conteúdo deve evitar generalização prematura. O particionamento deve ser simples, estável e suficiente apenas para viabilizar embeddings úteis neste primeiro corpus. A persistência vetorial deve registrar o mínimo necessário para permitir reutilização futura sem inflar a modelagem agora.

Não devem ser introduzidos conectores genéricos, hierarquia de loaders, abstrações de source provider, factories para chunking, pipelines universais, jobs sofisticados, retries, versionamento ou mecanismos de sincronização. Também não faz parte deste slice transformar a ingestão em um subdomínio próprio. O foco é apenas validar a primeira base local concreta, com fluxo visível, pequeno e fácil de revisar.

Sempre que houver dúvida entre desenhar uma estrutura expansível ou manter a entrega coesa e proporcional ao problema atual, esta PR deve favorecer a segunda opção.

---

## 9. Critérios de Review

- a PR introduz uma primeira base local de conhecimento com fonte concreta, controlada e juridicamente útil
- o fluxo de ingestão vetorial permanece pequeno, explícito e revisável
- `shared/ai` continua sendo o owner das capacidades reutilizáveis
- `modules/content` não recebe complexidade indevida do processo de ingestão
- a entrega responde a uma necessidade real de corpus local, e não a uma abstração hipotética
- não houve abertura de pipeline genérico nem foundation paralela
- a modelagem e a implementação permanecem proporcionais ao slice
- a PR mantém continuidade arquitetural direta com as PRs 29, 30, 31 e 32

---

## 10. Critérios de Aceite

- [ ] existe uma fonte jurídica inicial fixa usada como primeiro corpus local
- [ ] o Código Civil compilado no Planalto é lido dentro de um fluxo mínimo e controlado
- [ ] o conteúdo é particionado no nível necessário para vetorização inicial
- [ ] embeddings são gerados para os trechos produzidos
- [ ] os registros são persistidos no banco vetorial previsto na foundation compartilhada
- [ ] a base local inicial fica disponível para uso posterior no runtime compartilhado
- [ ] não há expansão indevida para múltiplas fontes ou pipeline genérico
- [ ] testes e validações existentes continuam passando

---

## 11. Conclusão

A PR 33 desloca a evolução do projeto de capacidade técnica isolada para utilidade prática mínima. Depois de consolidar a foundation compartilhada e validar a entrada inicial de LangGraph, o próximo passo correto é transformar essa base em algo concretamente útil: uma primeira base local de conhecimento construída sobre o Código Civil, persistida vetorialmente e mantida sob o boundary já aprovado de `shared/ai`.

A entrega permanece pequena, coerente e revisável. Em vez de expandir arquitetura, ela fecha um ciclo funcional real com baixo ruído: fonte concreta, particionamento simples, embedding e persistência vetorial. Isso mantém a sequência das PRs saudável, incremental e orientada a valor técnico real.
