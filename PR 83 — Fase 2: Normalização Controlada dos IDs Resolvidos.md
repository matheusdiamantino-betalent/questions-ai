# 🤖 PR 83 — Fase 2: Normalização Controlada dos IDs Resolvidos

## Fortalecimento da consistência estrutural dos identificadores no output avançado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-83-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/Tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/Fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/Escopo-ids%20normalizados-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/Status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR evolui o bloco `ids` do resultado avançado com normalização controlada dos identificadores já resolvidos, sem alterar arquitetura ou contrato público.
>
> - remove espaços residuais
> - converte valores vazios em `null`
> - preserva compatibilidade estrutural existente
>
> **Este PR não introduz novos IDs, novo lookup ou redesign do pipeline.**

## Sumário

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

# 1. Síntese Executiva

As PRs anteriores consolidaram partes centrais do output avançado, como `answerKey`, `adaptedStatement` e `metadata`. O próximo passo mínimo consiste em elevar a consistência do bloco `ids`, já presente no contrato final.

A mudança trata ruídos residuais de normalização após resolução e fallback, aumentando previsibilidade para consumidores downstream sem expandir escopo funcional.

# 2. Objetivo do PR

- normalizar strings dos identificadores resolvidos
- remover espaços residuais
- converter valores vazios em `null`
- preservar IDs sintéticos existentes
- manter o contrato público inalterado

# 3. Decisão Arquitetural

A responsabilidade permanece no `IdResolutionAgent`, componente já encarregado da resolução e consolidação dos identificadores.

A evolução ocorre no ponto correto do fluxo já aprovado, sem novos agentes, novas camadas, mudanças de DAO ou reorquestração.

# 4. Escopo

- normalização de `bankId`
- normalização de `lawId`
- normalização de `articleId`
- normalização de `yearId`
- normalização de demais campos já existentes em `ids`
- testes unitários proporcionais ao slice

# 5. Fora de Escopo

- novos identificadores
- nova estratégia de lookup
- cache adicional
- alteração de contratos públicos
- redesign do pipeline
- expansão estrutural da fase 2

# 6. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050b16",
    "primaryColor": "#0b1220",
    "primaryTextColor": "#ffffff",
    "primaryBorderColor": "#22d3ee",
    "lineColor": "#94a3b8"
  },
  "flowchart": { "curve": "linear" }
}}%%
flowchart LR
    A["Metadata"] --> B["IdResolutionAgent"]
    B --> C["IDs Resolvidos"]
    C --> D["Normalização"]
    D --> E["Output.ids"]

classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
classDef step5 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

class A step1;
class B step2;
class C step3;
class D step4;
class E step5;

linkStyle 0 stroke:#9ca3af,stroke-width:2px;
linkStyle 1 stroke:#9ca3af,stroke-width:2px;
linkStyle 2 stroke:#9ca3af,stroke-width:2px;
linkStyle 3 stroke:#9ca3af,stroke-width:2px;
```

# 7. Contratos Mínimos

Sem alteração estrutural no output final.

```ts
{
  adaptedStatement,
  answerKey,
  metadata,
  ids
}
```

A evolução incide apenas na consistência dos valores retornados em `ids`.

# 8. Regras de Implementação

- controller e orchestrator permanecem inalterados
- ajuste concentrado no `IdResolutionAgent`
- sem abstrações adicionais
- sem preparar próximas fases dentro desta PR
- priorizar simplicidade e legibilidade

# 9. Critérios de Review

- IDs retornam de forma mais limpa e previsível
- valores vazios são convertidos para `null`
- compatibilidade foi preservada
- recorte continua pequeno e objetivo
- ausência de overengineering

# 10. Critérios de Aceite

- [ ] campos de `ids` retornam normalizados
- [ ] valores vazios não poluem o output
- [ ] IDs existentes seguem compatíveis
- [ ] testes permanecem verdes
- [ ] nenhuma regressão no fluxo avançado

# 11. Conclusão

A PR 83 mantém a progressão incremental da fase 2 ao fortalecer um eixo relevante do resultado final: a consistência dos identificadores resolvidos.

Sem ampliar arquitetura ou contratos, o pipeline passa a entregar `ids` mais estáveis, limpos e confiáveis.
