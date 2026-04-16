# ⚙️ PR 45 — Fase 2: Primeiro Consumo Operacional do PDF Extraction em Ingestion
## Integração mínima do PDF extraction ao fluxo de ingestion, reutilizando o `AiService` sem ampliar a arquitetura da fase

---

<div align="left">

![PR](https://img.shields.io/badge/PR-45-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-primeiro%20consumo%20operacional%20pdf-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente a PR 44. Depois de expor o `PdfExtractionAgent` via `AiService`, o próximo passo mínimo correto é aplicar esse capability em um fluxo real já existente, sem abrir uma trilha paralela nem expandir a fase além do necessário.
>
> - conecta `extractPdf()` ao fluxo operacional de `ingestion`
> - reutiliza o boundary compartilhado já aprovado em `shared/ai`
> - mantém o processamento explícito, pequeno e revisável
>
> **Este PR não implementa upload binário real, OCR, múltiplos formatos, roteamento por tipo de documento ou encadeamento avançado entre etapas.**

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

A PR 43 introduziu o primeiro agent concreto da fase por meio do `PdfExtractionAgent`. A PR 44 consolidou esse capability no boundary compartilhado ao disponibilizar seu consumo via `AiService`, sem espalhar conhecimento do agent pelos módulos consumidores. Com essa base já aprovada, o próximo passo mínimo natural é aplicar a extração em um fluxo operacional existente da aplicação.

Esta PR faz exatamente esse movimento dentro de `ingestion`. Em vez de abrir um pipeline novo ou antecipar uma orquestração maior, ela apenas conecta `AiService.extractPdf()` ao ponto necessário do fluxo e passa a tratar o `extractedText` como entrada útil para a etapa seguinte. O ganho aqui é funcional e direto: o capability deixa de ser apenas compartilhado e passa a ser efetivamente consumido em produção de código, preservando a arquitetura já definida.

---

## 2. Objetivo do PR

- integrar `AiService.extractPdf()` ao fluxo de `ingestion`
- utilizar o `extractedText` como conteúdo de entrada do processamento subsequente
- manter o caminho principal com alteração mínima e explícita
- validar o novo comportamento com testes objetivos
- continuar a fase sem introduzir nova arquitetura

---

## 3. Decisão Arquitetural

A arquitetura aprovada é mantida. `shared/ai` continua como boundary compartilhado para execução do capability de extração, enquanto `ingestion` permanece responsável pelo fluxo operacional que consome esse resultado. Não há redefinição de módulos, contratos amplos nem criação de camadas intermediárias para esta integração.

A decisão central desta PR é inserir uma delegação explícita ao `AiService.extractPdf()` no ponto correto do fluxo já existente. Isso preserva a separação entre consumo operacional e implementação do agent, evita acoplamento direto de `ingestion` com `PdfExtractionAgent` e mantém a progressão da fase no menor recorte funcional possível.

---

## 4. Escopo

- chamar `AiService.extractPdf()` dentro do fluxo de `ingestion`
- consumir `extractedText` como conteúdo processável na sequência do caminho atual
- ajustar o caminho principal apenas no necessário para acomodar a integração
- adicionar testes do comportamento introduzido nesta PR
- preservar contratos já existentes sempre que possível

---

## 5. Fora de Escopo

- upload real de arquivo PDF
- OCR ou leitura de conteúdo escaneado
- suporte a múltiplos tipos de documento
- roteamento por tipo de arquivo ou strategy de agents
- pipeline multi-step ou coordenação entre múltiplos agents
- observabilidade expandida da etapa
- persistência específica para binário ou metadados de arquivo
- redesenho do fluxo de `ingestion`

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
    A["Ingestion Input"] --> B["Ingestion Service / Flow"]
    B --> C["AiService.extractPdf()"]
    C --> D["extractedText"]
    D --> E["Existing Processing Path"]
```

---

## 7. Contratos Mínimos

Esta PR reutiliza os contratos mínimos já introduzidos para PDF extraction e não cria uma nova superfície compartilhada para o slice atual.

```ts
export type PdfExtractionInput = {
  extractedText: string;
};

export type PdfExtractionOutput = {
  extractedText: string;
};
```

Se houver ajuste local no payload de `ingestion`, ele deve permanecer restrito ao ponto de integração e não deve introduzir um contrato global novo sem necessidade real.

---

## 8. Regras de Implementação

A integração deve permanecer concentrada no fluxo principal de `ingestion`, com controller fino e service/processamento explícito. O consumo de extração deve ocorrer por meio do `AiService`, nunca por acoplamento direto ao agent concreto. O texto extraído deve seguir imediatamente para o caminho já existente, sem bifurcações artificiais e sem preparar suporte implícito para formatos ou etapas futuras.

Também não entra nesta PR qualquer foundation paralela para roteamento, pipeline, observabilidade ou coordenação entre capabilities. O objetivo é somente tornar o PDF extraction operacional dentro de um fluxo real, com a menor alteração necessária e com testes proporcionais ao recorte.

---

## 9. Critérios de Review

O review deve validar se a integração ficou explícita, simples e aderente à progressão entre PR 44 e PR 45. É importante confirmar que `ingestion` passou a consumir `AiService.extractPdf()` sem conhecer detalhes internos do agent, que o fluxo principal permaneceu legível e que não houve introdução de componentes estruturais adicionais para um problema ainda pequeno.

Também deve ser observado se o comportamento novo está coberto por testes objetivos, se o recorte continua pequeno e revisável e se a PR não embute antecipações como múltiplos formatos, strategy de agents ou pipeline maior do que o slice realmente pede.

---

## 10. Critérios de Aceite

- [ ] `ingestion` utiliza `AiService.extractPdf()` no ponto definido do fluxo
- [ ] o `extractedText` segue para o processamento subsequente já existente
- [ ] o consumo ocorre via `AiService`, sem acoplamento direto ao `PdfExtractionAgent`
- [ ] testes do novo caminho estão passando
- [ ] nenhum componente estrutural extra foi introduzido
- [ ] a suíte relacionada permanece verde

---

## 11. Conclusão

A PR 45 realiza o primeiro consumo operacional do PDF extraction em um fluxo real da aplicação. Ela continua diretamente a PR 44, transforma um capability já compartilhado em uso efetivo dentro de `ingestion` e preserva o desenho aprovado da fase. O avanço é pequeno, funcional e controlado, exatamente no nível de recorte esperado para a sequência atual.
