# 🤖 PR 83 — Fase 2: Normalização Controlada dos IDs Resolvidos

## Fortalecimento da consistência estrutural dos identificadores no output avançado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-83-2563eb?style=for-the-badge\&logo=gitpullrequest\&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge\&logo=nestjs\&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge\&logo=dependabot\&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-ids%20normalizados-0891b2?style=for-the-badge\&logo=serverless\&logoColor=white)
![Status](https://img.shields.io/badge/status-ready-16a34a?style=for-the-badge\&logo=githubactions\&logoColor=white)

</div>

> [!IMPORTANT]
> Esta PR dá continuidade à fase avançada em um novo eixo funcional. Após consolidar `answerKey`, `adaptedStatement` e `metadata`, o foco evolutivo passa a ser a consistência estrutural dos identificadores resolvidos, preservando o recorte incremental e a arquitetura vigente.

---

# 1. Síntese Executiva

O pipeline avançado já produz `ids` como parte do output final. Entretanto, identificadores resolvidos podem carregar espaços residuais, valores vazios ou pequenas inconsistências estruturais após lookup e fallback.

A PR 83 fortalece a previsibilidade do bloco `ids` ao consolidar uma normalização controlada dos identificadores retornados.

---

# 2. Objetivo do PR

Garantir que os identificadores resolvidos do fluxo avançado sejam retornados de forma estável, limpa e consistente, sem alterar o contrato público já consumido pelo pipeline.

Objetivos diretos:

* normalizar strings de `ids`
* remover espaços residuais
* converter valores vazios em `null`
* preservar IDs sintéticos existentes
* reforçar previsibilidade estrutural do output final
* evoluir outro eixo funcional sem repetir ciclos anteriores

---

# 3. Decisão Arquitetural

A responsabilidade permanece no `IdResolutionAgent`, componente já responsável por resolver e consolidar identificadores do fluxo.

Não haverá:

* novo agent
* nova camada de orchestration
* redesign do orchestrator
* alteração de DAO
* cache adicional
* expansão estrutural da fase 2

A decisão é fortalecer a normalização no mesmo componente já responsável pela resolução de IDs.

---

# 4. Escopo da PR

## Incluído

* normalização de `bankId`
* normalização de `lawId`
* normalização de `articleId`
* normalização de `yearId`
* normalização de demais ids existentes no contrato
* transformação de valores vazios em `null`
* cobertura proporcional de testes unitários
* preservação integral do contrato público

## Fora de Escopo

* nova estratégia de lookup
* novos identificadores
* alteração de DAO
* cache distribuído
* redesign do pipeline
* expansão indevida da fase 2

---

# 5. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#0f172a",
    "primaryTextColor": "#e5e7eb",
    "primaryBorderColor": "#38bdf8",
    "lineColor": "#64748b",
    "secondaryColor": "#111827",
    "tertiaryColor": "#1e293b",
    "background": "#020617"
  }
}}%%
flowchart LR
    A[Metadata] --> B[IdResolutionAgent]
    B --> C[Resolved IDs]
    C --> D[Normalize IDs]
    D --> E[Output.ids]
```

---

# 6. Contratos Mínimos

Sem alteração estrutural no output final.

```ts
{
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

A evolução ocorre na consistência dos valores retornados em `ids`, não na expansão do contrato público.

---

# 7. Estratégia de Implementação

Ordem recomendada:

1. `id-resolution.agent.ts`
2. `id-resolution.agent.spec.ts`
3. regressão da suíte completa

Princípio central:

> elevar a consistência estrutural dos identificadores sem ampliar complexidade sistêmica.

---

# 8. Critérios de Review

Validar se:

* os IDs ficaram mais limpos e previsíveis
* valores vazios passam a ser `null`
* IDs sintéticos foram preservados
* o contrato final permaneceu inalterado
* o recorte permaneceu pequeno
* não houve overengineering

---

# 9. Critérios de Aceite

* campos de ids retornam normalizados
* campos vazios não poluem o output
* IDs existentes permanecem compatíveis
* testes permanecem verdes
* nenhuma regressão no fluxo avançado

---

# 10. Impacto Esperado

Maior consistência prática no bloco `ids` do output avançado, reduzindo ruído estrutural para consumidores downstream.

O sistema evolui além de `answerKey`, `adaptedStatement` e `metadata`, fortalecendo outro eixo central do resultado processado.

---

# 11. Conclusão

A PR 83 mantém a progressão madura da fase avançada ao atacar outro componente relevante do output final: os identificadores resolvidos.

Sem alterar arquitetura ou contrato público, o pipeline passa a entregar `ids` mais limpos, consistentes e confiáveis.
