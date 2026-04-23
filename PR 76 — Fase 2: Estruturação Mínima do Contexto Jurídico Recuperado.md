# 🤖 PR 76 — Fase 2: Estruturação Mínima do Contexto Jurídico Recuperado

## Exposição controlada de sinais úteis da busca legal para o fluxo avançado

---

<div align="left">

![PR](https://img.shields.io/badge/PR-76-2563eb?style=for-the-badge\&logo=gitpullrequest\&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge\&logo=nestjs\&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge\&logo=dependabot\&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-contexto%20juridico%20estruturado-0891b2?style=for-the-badge\&logo=serverless\&logoColor=white)
![Status](https://img.shields.io/badge/status-proposed-16a34a?style=for-the-badge\&logo=githubactions\&logoColor=white)

</div>

> [!IMPORTANT]
> Esta PR dá continuidade direta à evolução funcional da fase 2 após a PR 75. O foco agora não é ampliar a busca jurídica, mas tornar o resultado recuperado mais explícito, consistente e útil para o restante do pipeline. O recorte permanece pequeno, incremental e sem reabertura arquitetural.

---

# 1. Síntese Executiva

Após fortalecer a robustez da referência jurídica, o próximo passo natural é melhorar a qualidade do artefato retornado por essa busca.

A PR 76 introduz uma estrutura mínima adicional no contexto jurídico recuperado, expondo sinais objetivos que facilitam o consumo interno do pipeline sem alterar a arquitetura existente.

---

# 2. Objetivo do PR

Tornar o resultado de `legalSearch` mais confiável e semanticamente explícito para os consumers internos.

Objetivos diretos:

* padronizar campos retornados pela busca jurídica
* explicitar quando existe conteúdo jurídico utilizável
* normalizar `fullText` de forma consistente
* tornar `source` mais previsível
* preservar compatibilidade com o fluxo atual

---

# 3. Decisão Arquitetural

A evolução ocorrerá dentro dos contratos e agents já existentes.

Não haverá:

* novo agent
* nova camada de orchestration
* pipeline paralelo
* scoring complexo
* múltiplos providers
* redesign estrutural

A decisão é enriquecer o resultado atual com sinais mínimos e úteis.

---

# 4. Escopo da PR

## Incluído

* refinamento de `LegalSearchResult`
* normalização consistente de `fullText`
* padronização de `source`
* inclusão de marcador mínimo de disponibilidade/uso do contexto
* adaptação proporcional do `InitialQuestionProcessingAgent`
* atualização de testes unitários relacionados

## Fora de Escopo

* ranking de resultados
* busca vetorial expandida
* múltiplas fontes externas
* heurísticas complexas
* mudanças no contrato público final da API
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
    A[Legal Reference] --> B[LegalSearchAgent]
    B --> C[LegalSearchResult Estruturado]
    C --> D[InitialQuestionProcessingAgent]
    D --> E[AnswerKey Final]
```

---

# 6. Contratos Mínimos

Evolução esperada no contrato interno de `legalSearch`, preservando compatibilidade do fluxo.

Exemplo conceitual:

```ts
{
  reference,
  fullText,
  source,
  hasUsableContent
}
```

A forma final deve permanecer mínima e proporcional ao slice.

---

# 7. Estratégia de Implementação

Ordem recomendada:

1. `agent.contracts.ts`
2. `legal-search.agent.ts`
3. `initial-question-processing.agent.ts`
4. specs do legal search
5. specs do processing
6. regressão do fluxo completo

Princípio central:

> melhorar a qualidade do resultado sem ampliar a complexidade do sistema.

---

# 8. Critérios de Review

Validar se:

* o recorte permaneceu pequeno
* o novo sinal agregado tem utilidade real
* não houve inflação de contrato
  n- o fluxo atual continuou simples
* compatibilidade foi preservada
* testes cobrem os novos cenários
* não houve overengineering

---

# 9. Critérios de Aceite

* `legalSearch` retorna estrutura mais consistente
* conteúdo utilizável fica explícito
* `source` permanece previsível
* consumers internos usam o novo shape sem regressão
* suíte de testes permanece verde

---

# 10. Impacto Esperado

Maior clareza semântica dentro do pipeline e menor necessidade de inferências espalhadas nos consumers.

O fluxo passa a consumir um artefato jurídico mais maduro e confiável.

---

# 11. Conclusão

A PR 76 representa um avanço natural após a robustez da PR 75: não buscar mais, mas retornar melhor.

É um incremento pequeno, funcional e coerente com a evolução progressiva dos agents avançados.
