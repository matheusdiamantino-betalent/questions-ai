# 🔄 PR 19 — Foundation Mínima de Banco Vetorial para Uso Local
## Introdução do primeiro suporte operacional mínimo para persistência e consulta de embeddings no ambiente local

---

<div align="left">

![PR](https://img.shields.io/badge/PR-19-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-vector%20store%20foundation-7c3aed?style=for-the-badge&logo=postgresql&logoColor=white)
![Fase](https://img.shields.io/badge/fase-inicio%20do%20modulo%20de%20dados-1d4ed8?style=for-the-badge&logo=databricks&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-local%20vector%20storage-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR introduz a **foundation mínima de banco vetorial para uso local**, adicionando suporte real de persistência e consulta de embeddings utilizando PostgreSQL + pgvector.

---

## 1. Síntese Executiva

Esta PR inicia o módulo vetorial com persistência real e busca por similaridade, sem inflar arquitetura.

---

## 2. Objetivo do PR

- persistir embeddings
- consultar por similaridade
- manter implementação mínima

---

## 3. Decisão Arquitetural

- PostgreSQL + pgvector
- SQL explícito
- sem abstrações

---

## 4. Escopo

- migration com extensão vector
- tabela document_embeddings
- DAO + Service mínimos
- testes reais

---

## 5. Fora de Escopo

- retrieval
- agents
- ranking
- abstrações

---

## 6. Fluxo

Texto → Embedding → Banco → Similaridade → Resultado

---

## 7. Conclusão

Foundation mínima funcional pronta para evolução incremental.
