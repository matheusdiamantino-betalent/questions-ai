# 🔄 PR 19 — Foundation Mínima de Banco Vetorial para Uso Local
## Introdução do primeiro suporte operacional mínimo para armazenamento e consulta básica de embeddings no ambiente local

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
> Esta PR introduz a **foundation mínima de banco vetorial para uso local**, adicionando a primeira capacidade operacional real de persistir embeddings e executar consulta básica por similaridade utilizando PostgreSQL + pgvector.

---

## 📚 Sumário

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

Após a estabilização dos fluxos de ingestion e observabilidade, esta PR introduz o próximo passo mínimo necessário: a base de armazenamento vetorial.

A entrega foca exclusivamente em:

- persistência de embeddings
- consulta por similaridade
- validação real via testes

Sem expandir para retrieval completo ou agentes.

---

## 2. Objetivo do PR

- habilitar armazenamento vetorial local
- permitir inserção de embeddings
- permitir busca por similaridade
- manter o menor recorte funcional possível

---

## 3. Decisão Arquitetural

A implementação segue princípios claros:

- uso direto de PostgreSQL com pgvector
- SQL explícito para operações vetoriais
- ausência de abstrações genéricas
- DAO como camada de persistência
- service mínimo como orquestração

---

## 4. Escopo

Inclui:

- extensão `vector` via migration
- tabela `document_embeddings`
- DAO com operações de insert e search
- service mínimo
- testes de integração reais

---

## 5. Fora de Escopo

- geração de embeddings
- pipeline de retrieval
- chunking
- ranking
- abstrações de vector store
- múltiplos backends

---

## 6. Fluxo Arquitetural

```mermaid
flowchart LR
    A["Texto"] --> B["Embedding"]
    B --> C["Persistência (pgvector)"]
    C --> D["Consulta por similaridade"]
    D --> E["Resultado ordenado"]
```

---

## 7. Contratos Mínimos

Tabela:

- id (UUID)
- content (TEXT)
- embedding (VECTOR 1536)
- created_at (TIMESTAMP)

---

## 8. Regras de Implementação

- DAO isolado para persistência
- SQL explícito para similaridade
- sem abstração prematura
- sem camadas desnecessárias

---

## 9. Critérios de Review

- escopo pequeno
- implementação clara
- sem overengineering
- funcionamento validado

---

## 10. Critérios de Aceite

- persistência funcional
- busca vetorial funcional
- testes passando
- sem abstrações desnecessárias

---

## 11. Conclusão

Esta PR estabelece a base vetorial funcional com o menor esforço possível, permitindo evolução incremental controlada nas próximas fases.
