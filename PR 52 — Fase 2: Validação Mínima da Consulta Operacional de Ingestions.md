# ✅ PR 52 — Fase 2: Validação Mínima da Consulta Operacional de Ingestions
## Saneamento explícito de query params para listagem operacional com contratos previsíveis e seguros

---

<div align="left">

![PR](https://img.shields.io/badge/PR-52-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-agents%20basicos-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-validacao%20consulta%20operacional-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR continua diretamente as PRs 50 e 51. Após expor listagem operacional e paginação mínima, o próximo passo correto é validar e normalizar a entrada da API, reforçando previsibilidade sem alterar o fluxo interno já aprovado.
>
> - protege a borda da aplicação
> - remove ambiguidades de entrada
> - define defaults explícitos
> - melhora previsibilidade operacional
>
> **Esta PR não adiciona filtros avançados, busca textual, ordenação customizável ou nova arquitetura.**

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

As PRs anteriores abriram a consulta operacional de ingestions e adicionaram paginação mínima. Com isso, a API passou a receber parâmetros externos relevantes para navegação, volume de retorno e filtragem.

O próximo passo incremental é validar esses parâmetros de forma explícita. A PR 52 fortalece a borda da aplicação sem alterar o fluxo interno de domínio, garantindo entradas consistentes, defaults claros e respostas previsíveis.

---

## 2. Objetivo do PR

- validar `limit`
- validar `offset`
- validar `status`
- normalizar parâmetros textuais
- aplicar defaults seguros
- retornar erro claro para entradas inválidas
- cobrir o comportamento com testes

---

## 3. Decisão Arquitetural

A validação deve acontecer na camada de entrada da API (controller, DTO ou parsing equivalente), mantendo service e DAO focados em comportamento operacional e acesso a dados.

Não há criação de novos módulos, middlewares paralelos ou abstrações extras. O recorte permanece pequeno, localizado e aderente ao fluxo já existente.

---

## 4. Escopo

- aceitar `limit` apenas em faixa permitida
- aceitar `offset >= 0`
- validar `status` dentro dos valores suportados
- aplicar trim em parâmetros textuais quando existir
- definir defaults para ausência de parâmetros
- mensagens claras de erro
- testes de controller e service

---

## 5. Fora de Escopo

- busca textual
- filtros compostos avançados
- ordenação dinâmica
- cache de consulta
- métricas agregadas
- exportação CSV
- rate limiting
- autenticação granular
- mudanças no pipeline de ingestion

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
    A[HTTP Query Params] --> B[Validação / Normalização]
    B --> C[IngestionService.findMany]
    C --> D[IngestionDao.findMany]
    D --> E[Resultado paginado]
```

---

## 7. Contratos Mínimos

O contrato de entrada permanece pequeno e explícito, agora acompanhado por regras formais de saneamento.

```ts
export type ListIngestionsQuery = {
  limit?: number;
  offset?: number;
  status?: IngestionStatus;
};
```

Regras mínimas:

```ts
limit: 1..50
offset: >= 0
status: enum válido
```

---

## 8. Regras de Implementação

A mudança deve ficar concentrada na entrada da API. Evitar espalhar validação manual em múltiplos pontos ou replicar regras entre controller, service e DAO.

Se já existir DTO, parser ou pipe atual, evoluir o artefato existente no menor recorte possível. O objetivo é clareza e previsibilidade, não reestruturação.

---

## 9. Critérios de Review

O review deve validar se as regras ficaram claras, se entradas inválidas retornam erro corretamente e se os defaults permanecem consistentes.

Também deve confirmar que o service não recebeu responsabilidade indevida e que o escopo permaneceu pequeno e proporcional.

---

## 10. Critérios de Aceite

- [ ] `limit` inválido retorna erro
- [ ] `offset` inválido retorna erro
- [ ] `status` inválido retorna erro
- [ ] parâmetros válidos seguem fluxo normal
- [ ] defaults funcionam sem query params
- [ ] testes cobrindo cenários principais passam
- [ ] nenhuma nova estrutura desnecessária foi criada

---

## 11. Conclusão

A PR 52 fortalece a API operacional no ponto correto: a entrada. Após abrir listagem e paginação, validar parâmetros é o próximo avanço natural, pequeno e de alto valor funcional.

O sistema fica mais previsível e seguro sem aumento indevido de complexidade, mantendo a trilha incremental e coerente da fase atual.
