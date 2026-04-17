# 🧩 PR 57 — Fase 2: Validação Mínima de Conectividade com Gemini API
## Teste controlado de autenticação e execução básica do provider externo sem ampliar o runtime atual

---

<div align="left">

![PR](https://img.shields.io/badge/PR-57-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-validacao%20minima%20gemini-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR adiciona uma validação mínima e isolada de conectividade com a Gemini API por meio de chave de ambiente e uma execução simples de geração de conteúdo. O objetivo é comprovar autenticação, comunicação com o provider externo e retorno básico de resposta sem reabrir o runtime atual.
>
> - adiciona configuração mínima para a chave Gemini
> - introduz um cliente ou serviço enxuto para chamada simples ao provider
> - valida autenticação e execução básica em fluxo controlado
>
> **Este PR não substitui o provider principal, não introduz fallback multi-provider, não altera ingestion, não modifica o orchestrator e não amplia a arquitetura já aprovada do runtime de IA.**

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

A PR anterior consolidou a primeira extração mínima da coordenação sequencial para o orchestrator, preservando o fluxo principal e mantendo a responsabilidade centralizada em uma composição simples entre agents. Com essa base estabilizada, o próximo passo mínimo não é ampliar a orquestração nem introduzir nova camada de runtime, mas apenas comprovar que a aplicação consegue se comunicar com um provider externo adicional em condições reais.

Esta PR segue exatamente essa lógica incremental. Em vez de antecipar arquitetura multi-provider ou acoplamentos desnecessários, ela limita a mudança à validação objetiva de conectividade com a Gemini API: chave configurada, chamada válida e resposta básica recebida. O ganho aqui é estritamente operacional e serve como prova controlada de integração externa, sem deslocar o foco da fase nem inflar a solução.

---

## 2. Objetivo do PR

- adicionar configuração mínima para consumo da Gemini API
- validar autenticação por API key em ambiente controlado
- executar uma chamada simples de geração de conteúdo
- confirmar retorno básico do provider externo
- manter a validação isolada do fluxo principal atual

---

## 3. Decisão Arquitetural

A decisão central desta PR é tratar Gemini como integração pontual de conectividade, e não como novo eixo estrutural do runtime. A arquitetura já aprovada é mantida integralmente. Não há introdução de abstrações genéricas de provider, roteamento dinâmico, estratégia compartilhada entre vendors nem reconfiguração do caminho principal já em uso.

O provider entra apenas por um ponto controlado, pequeno e explícito, com responsabilidade restrita à autenticação e execução básica. Isso protege o recorte, mantém a leitura simples para review e evita que um teste operacional de conexão seja transformado em fundação paralela antes de existir necessidade real.

---

## 4. Escopo

- adicionar variável de ambiente para `GEMINI_API_KEY`
- adicionar configuração mínima de modelo para teste controlado
- criar cliente ou serviço simples para chamada ao provider
- executar operação básica de geração de texto
- tratar erro mínimo de autenticação ou comunicação
- adicionar teste do wrapper de integração sem tocar o fluxo principal

---

## 5. Fora de Escopo

- substituição do provider principal atual
- fallback automático entre providers
- abstração multi-provider
- roteamento dinâmico por vendor
- integração com ingestion
- integração com orchestrator
- streaming
- multimodal
- histórico de conversa
- observabilidade expandida
- retries sofisticados
- expansão do runtime para uso funcional em produção

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
    A[Env GEMINI_API_KEY] --> B[Gemini Connectivity Service]
    B --> C[Simple Generate Request]
    C --> D[Gemini API]
    D --> E[Basic Text Response]
```

O fluxo permanece deliberadamente curto. Esta PR existe apenas para provar que a aplicação autentica, envia uma chamada simples ao provider e recebe uma resposta funcional em um caminho mínimo e verificável.

---

## 7. Contratos Mínimos

```ts
export type GeminiExecuteInput = {
  prompt: string;
};

export type GeminiExecuteOutput = {
  output: string;
};
```

Se já houver contrato interno reaproveitável com o mesmo formato, ele deve ser mantido. Nesta etapa, o contrato só precisa sustentar o teste de conectividade, sem ampliar modelagem, sem metadados artificiais e sem antecipar forma final de integração futura.

---

## 8. Regras de Implementação

A implementação deve manter controller fino, serviço explícito e fluxo visível. O wrapper de Gemini não deve absorver responsabilidades além da chamada mínima ao provider, nem introduzir camada genérica apenas porque agora existe um segundo vendor em teste. Qualquer acesso a configuração deve permanecer centralizado no padrão já adotado pelo projeto, evitando leitura dispersa de ambiente.

Também é importante que a integração fique claramente isolada do runtime principal: sem tocar orchestrator, sem infiltrar condicionais em flows existentes e sem preparar próximos passos dentro desta PR. O objetivo é simples e fechado: chave válida, request válido e resposta válida, com tratamento mínimo de erro quando autenticação ou comunicação falharem.

---

## 9. Critérios de Review

Validar se a PR permaneceu proporcional ao recorte e se a integração com Gemini ficou isolada do restante da arquitetura. Confirmar que a chave foi externalizada no padrão do projeto, que o serviço ou cliente criado é enxuto, que a chamada básica está correta e que existe tratamento mínimo para falhas sem inflar a solução.

Também deve ser verificado se não houve abertura indevida para arquitetura multi-provider, se o fluxo principal continua intacto e se a documentação reflete com precisão o limite real da entrega, sem sugerir evolução que ainda não entrou.

---

## 10. Critérios de Aceite

- [ ] existe configuração mínima para `GEMINI_API_KEY`
- [ ] existe configuração mínima de modelo para teste controlado
- [ ] existe serviço ou cliente mínimo para chamada Gemini
- [ ] a aplicação consegue autenticar com a chave configurada
- [ ] a chamada retorna resposta básica válida
- [ ] falhas de autenticação ou comunicação recebem tratamento mínimo
- [ ] nenhuma alteração foi feita em ingestion
- [ ] nenhuma alteração foi feita no orchestrator
- [ ] nenhuma abstração multi-provider indevida foi adicionada

---

## 11. Conclusão

A PR 57 mantém a linha incremental da fase ao adicionar apenas uma validação técnica pequena e controlada de conectividade com a Gemini API. O ganho entregue aqui é objetivo: comprovar que a aplicação consegue autenticar, executar uma chamada simples e obter resposta funcional de um provider externo adicional sem desviar a arquitetura aprovada.

Com isso, o projeto preserva o mesmo princípio aplicado nas PRs anteriores: primeiro validar o próximo passo mínimo real, depois decidir se existe razão concreta para qualquer evolução adicional. O recorte permanece pequeno, revisável e sem overengineering.
