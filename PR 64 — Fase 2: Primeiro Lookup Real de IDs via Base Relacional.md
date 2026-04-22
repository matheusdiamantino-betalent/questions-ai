# 🔎 PR 64 — Fase 2: Primeiro Lookup Real de IDs via Base Principal
## Ativação inicial da resolução real de referências externas sem catálogo local neste serviço

---

<div align="left">

![PR](https://img.shields.io/badge/PR-64-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-primeiro%20lookup%20real-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR inaugura a primeira resolução real de IDs após o realinhamento arquitetural da PR 63.
>
> - consulta referências na base da API principal
> - retorna IDs persistidos quando houver match
> - preserva fallback previsível quando não houver correspondência
> - mantém este serviço sem catálogo local duplicado
>
> **Este PR não introduz fuzzy matching, cache ou sincronização local.**

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

## 1. Síntese Executiva

Após a correção de boundary na PR 63, o próximo passo natural é conectar a resolução de IDs à fonte de verdade correta: a base da API principal. Esta entrega habilita o primeiro lookup funcional de referências externas, retornando IDs reais em correspondências diretas e preservando comportamento previsível quando não houver match.

## 2. Objetivo do PR

- implementar acesso real à base da API principal para resolução de IDs
- ativar primeiro lookup funcional no fluxo principal
- preservar contrato atual de saída
- manter fallback controlado para no-match
- validar evolução com testes objetivos

## 3. Decisão Arquitetural

A arquitetura segue o boundary revisado. Este serviço continua sem persistir `Law` e `Bank` localmente. O `IdResolutionAgent` permanece como orquestrador do fluxo mínimo, enquanto um boundary dedicado passa a consultar a base principal para obter referências reais.

## 4. Escopo

- criação de acesso dedicado à base principal
- primeiro lookup real para entidades suportadas nesta etapa
- retorno de IDs reais quando encontrados
- fallback atual quando não encontrados
- testes cobrindo match e ausência de match

## 5. Fora de Escopo

- catálogo local espelhado
- sincronização de dados
- fuzzy matching
- score de confiança
- cache Redis
- múltiplas estratégias paralelas
- heurísticas avançadas
- observabilidade expandida

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Metadata] --> B[IdResolutionAgent]
    B --> C[Reference Boundary]
    C --> D[(Base API Principal)]
    D --> E[IDs reais ou fallback]
    E --> F[Output]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F step6;
```

## 7. Contratos Mínimos

Os contratos externos permanecem inalterados.

```ts
ResolvedIds {
  lawId?: string | null;
  articleId?: string | null;
  bankId?: string | null;
  yearId?: string | null;
}
```

## 8. Regras de Implementação

- queries simples e legíveis
- priorizar match exato
- agent sem acesso direto a infra externa
- boundary dedicado concentrando integração
- fallback preservado
- sem antecipar próximas fases

## 9. Critérios de Review

- integração respeita boundary correto
- ausência de catálogo local duplicado
- agent permanece coeso
- fallback previsível
- testes cobrem match e no-match
- ausência de overengineering

## 10. Critérios de Aceite

- [ ] resolução consulta base principal
- [ ] IDs reais retornam quando houver match
- [ ] no-match mantém fallback esperado
- [ ] contrato externo permanece compatível
- [ ] suíte de testes verde

## 11. Conclusão

Esta PR conclui a transição entre fallback sintético e resolução real aderente ao boundary correto. O fluxo passa a consumir dados da fonte de verdade sem duplicação local, de forma incremental, controlada e revisável.
