# 🗄️ PR 63 — Fase 2: Realinhamento de Boundary na Resolução de IDs
## Remoção da referência relacional local e preservação do fallback mínimo aderente à base da API principal

---

<div align="left">

![PR](https://img.shields.io/badge/PR-63-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-feature%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-2-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-boundary%20realignment-9333ea?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-ready%20for%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR recalibra a direção iniciada na PR 62 após alinhamento de arquitetura em review.
>
> - remove catálogo relacional local indevido para `Law` e `Bank`
> - elimina a premissa de lookup via banco deste serviço
> - preserva resolução mínima com IDs sintéticos no fluxo atual
>
> **A fonte de verdade dessas referências permanece na base da API principal.**

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

Após revisão arquitetural, ficou definido que `Law` e `Bank` não pertencem ao banco deste serviço. Essas referências já existem na base da API principal e não devem ser replicadas localmente apenas para lookup. Esta PR corrige esse boundary, remove a modelagem indevida e mantém o fluxo atual com fallback sintético mínimo.

## 2. Objetivo do PR

- remover referências locais de `Law` e `Bank`
- eliminar lookup local dessas entidades
- simplificar a resolução atual para IDs sintéticos
- alinhar a implementação ao boundary definido em review
- preservar recorte pequeno e revisável

## 3. Decisão Arquitetural

Este serviço não é dono do catálogo de leis e bancas. A resolução real dessas referências deverá ocorrer em etapa futura por integração dedicada com a base da API principal. Nesta fase, o comportamento permanece mínimo, sem duplicação de persistência.

## 4. Escopo

- remoção de `Law` e `Bank` do schema Prisma
- remoção de tabelas e índices correspondentes da migration
- remoção da premissa de lookup local no fluxo de resolução
- simplificação do `IdResolutionAgent`
- ajuste dos testes para o novo comportamento

## 5. Fora de Escopo

- conexão dedicada com a base principal
- lookup real de IDs externos
- sincronização de catálogos
- cache distribuído
- fuzzy matching
- aliases ricos
- enriquecimento de metadata
- refactor estrutural amplo

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A[Entrada Metadata] --> B[Normalização]
    B --> C[Geração de IDs Sintéticos]
    C --> D[Fluxo Atual Preservado]
    D --> E[Lookup Real Futuro via Base Principal]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#102018,stroke:#22c55e,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
```

## 7. Contratos Mínimos

Nenhum contrato externo é expandido nesta etapa. O formato de saída permanece o mesmo, com IDs resolvidos no shape atual, agora sem dependência de persistência local indevida.

## 8. Regras de Implementação

- remover apenas o que conflita com o boundary correto
- preservar simplicidade do fluxo atual
- evitar nova fundação paralela
- manter código fácil de revisar
- não antecipar integração futura nesta PR
- reduzir ruído estrutural

## 9. Critérios de Review

- aderência ao comentário de arquitetura
- ausência de catálogo local indevido
- fluxo atual preservado
- recorte pequeno
- sem overengineering
- testes coerentes com o novo comportamento

## 10. Critérios de Aceite

- [ ] `Law` removido do schema local
- [ ] `Bank` removido do schema local
- [ ] migration ajustada
- [ ] agent sem dependência de lookup local
- [ ] testes atualizados
- [ ] comportamento atual preservado

## 11. Conclusão

Esta PR corrige o boundary antes de expandir a solução. Em vez de consolidar uma fundação incorreta no banco local, o projeto preserva um fluxo mínimo e prepara o terreno para uma futura integração real com a base da API principal.
