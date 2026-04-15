# 🧪 PR 33 — Fase 2: Centralização da Estrutura de Testes em `__tests__`
## Reorganização não funcional dos arquivos de teste para reduzir ruído na árvore principal

---

<div align="left">

![PR](https://img.shields.io/badge/PR-33-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-refactor%20slice-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-centralizacao%20de%20testes-1d4ed8?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-estrutura%20de%20specs-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR entrega apenas uma reorganização estrutural dos testes automatizados, centralizando os arquivos `.spec.ts` em `src/__tests__` e preservando a mesma leitura lógica da aplicação.

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

## 1. Síntese Executiva
Após a evolução recente da Fase 2 e o aumento natural da cobertura de testes, surgiu a necessidade de separar com mais clareza arquivos de produção e arquivos de teste. Esta PR resolve esse ponto com um ajuste pequeno e não funcional: mover os `.spec.ts` para `src/__tests__`, mantendo a mesma organização lógica já conhecida no projeto.

## 2. Objetivo do PR
- centralizar os arquivos de teste em `src/__tests__`
- replicar a estrutura lógica atual de `modules` e `shared`
- reduzir ruído visual na árvore principal
- preservar execução da suíte existente

## 3. Decisão Arquitetural
A árvore de produção permanece intacta. Apenas os arquivos de teste são reorganizados para um local centralizado, sem acoplar essa mudança a refactors funcionais.

## 4. Escopo
- mover arquivos `.spec.ts`
- ajustar imports necessários
- validar execução da suíte

## 5. Fora de Escopo
- mudanças de produção
- refactors funcionais
- novos testes
- alteração de contratos

## 6. Fluxo Arquitetural
```mermaid
flowchart LR
A[specs distribuídos] --> B[src/__tests__]
B --> C[mesma estrutura lógica]
C --> D[suíte preservada]
```

## 7. Contratos Mínimos
Não há alteração de contratos funcionais.

## 8. Regras de Implementação
Aplicar somente o menor ajuste necessário para reorganizar os testes e manter tudo funcionando.

## 9. Critérios de Review
- testes centralizados
- estrutura preservada
- sem impacto funcional
- imports coerentes
- PR pequena e objetiva

## 10. Critérios de Aceite
- [ ] arquivos `.spec.ts` movidos
- [ ] suíte executando com sucesso
- [ ] sem alteração de produção
- [ ] recorte apenas estrutural

## 11. Conclusão
Esta PR melhora organização e legibilidade do repositório sem alterar comportamento da aplicação, mantendo o padrão incremental e revisável do projeto.
