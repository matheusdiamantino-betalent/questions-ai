# DLS IA

Sistema inteligente para geração automatizada de questões de concurso no formato verdadeiro/falso, com gabaritos comentados.

## Visão Geral

O DLS IA é uma **plataforma de IA baseada em agents** com uma base de conhecimento vetorial compartilhada. Diferentes agents especializados consomem essa base para executar tarefas específicas.

### Arquitetura Multi-Agent

```
┌─────────────────────────────────────────────────────────┐
│           BASE DE CONHECIMENTO VETORIAL                 │
│  (PostgreSQL + PGVector)                                │
│  - Legislação completa (leis, artigos, resoluções)      │
│  - Materiais didáticos (livros, apostilas, doutrina)    │
│  - Embeddings semânticos                                │
│  - Busca por similaridade                               │
└────────────────────┬────────────────────────────────────┘
                     │
          ┌──────────┴──────────┐
          │  Consumido por      │
          └──────────┬──────────┘
                     ↓
   ┌─────────────────────────────────────────────────┐
   │              AGENTS ESPECIALIZADOS              │
   ├─────────────────────────────────────────────────┤
   │                                                 │
   │ Agent Gerador de Questões V/F (v1)             │
   │    - Transforma múltipla escolha em V/F        │
   │    - Gera gabaritos comentados                 │
   │                                                 │
   │ Agent Chatbot (futuro)                         │
   │    - Responde dúvidas sobre legislação         │
   │    - Contextual com base vetorial              │
   │                                                 │
   │ Agent Gerador de Questões Inéditas (futuro)   │
   │    - Cria questões originais                   │
   │    - Baseado em temas da legislação            │
   │                                                 │
   │ Agent de Marketing (futuro)                    │
   │    - Gera conteúdo para redes sociais          │
   │    - Posts educativos sobre leis               │
   │                                                 │
   │ Agent de Material Didático (futuro)            │
   │    - Mapas mentais de leis                     │
   │    - PDFs com resumos estruturados             │
   │    - Flashcards e diagramas                    │
   │                                                 │
   │ Outros agents (futuro)                         │
   │    - Analisador de mudanças legislativas       │
   │    - Comparação de versões de leis             │
   │    - Alertas de atualizações                   │
   │                                                 │
   └─────────────────────────────────────────────────┘
```

### Componentes Principais

**1. Base de Conhecimento (Core)**

- Banco vetorial com toda legislação
- Sistema de embeddings e busca semântica
- APIs de consulta compartilhadas

**2. Agents (Plugáveis)**

- Cada agent é um módulo independente
- Todos consomem a mesma base de conhecimento
- Fácil adicionar novos agents sem afetar existentes

**3. Integração MySQL**

- Consulta IDs do serviço principal
- Grava questões processadas e rascunhos
- Cache local para performance

## Importante: Separação de Responsabilidades

**Este microserviço NÃO armazena questões finais.**

### O que é gerenciado pelo Serviço Principal (MySQL):

- **Questões aprovadas e publicadas**
- Cadastros base: Bancas, Leis, Artigos, Órgãos, Anos
- Usuários e permissões
- Validação e aprovação de questões
- Interface administrativa

### O que é gerenciado por este Microserviço:

**PostgreSQL + PGVector:**

- Base de conhecimento vetorial (legislação)
- Materiais didáticos uploadados (livros, apostilas, doutrina)
- Logs de processamento dos agents

**Redis:**

- Cache de IDs do serviço principal (para performance)
- Métricas e monitoramento de IA
- Filas de jobs (Bull/BullMQ) para processamento assíncrono

**Fluxo:** Microserviço processa e retorna questões → Serviço Principal valida e armazena no MySQL.

## Objetivo

Receber PDFs de provas e gabaritos como entrada e gerar automaticamente questões adaptadas ao formato DLS:

**Entrada:**

- PDF da prova (questões de múltipla escolha no formato original)
- PDF/documento do gabarito (que pode ou não estar comentado)

**Processamento:**

- Extração e parsing dos PDFs
- Identificação de questões e alternativas
- Transformação para formato verdadeiro/falso
- Geração de gabaritos comentados

**Saída:**

- Enunciados reformulados para formato verdadeiro/falso
- Gabaritos comentados explicativos
- Questões classificadas e catalogadas

### Formato das Questões

#### Estrutura do Enunciado

```
[Art. x, Código ou Lei] BANCA ANO - Cargo/Concurso (Órgão)
```

**Exemplo:**

```
[Art. 1º, Lei 9.492/97] IESES 2026 - Tabelião / Oficial de Registros - Provimento (TJ-PA)
```

#### Exemplo de Transformação

**Entrada (Questão Original):**

```
QUESTÃO 01. Conforme a resolução 583 do CNJ é correto afirmar:
a) Na hipótese de declaração de inexistência de pacto antenupcial...
b) Conforme a resolução 583 para fins de referida averbação...
c) A declaração complementar deverá ser registrada...
d) Faculta-se a averbação do regime de bens posteriormente...
e) É obrigatória a averbação do regime de bens posteriormente...

Comentários: [Gabarito detalhado com fundamentação legal]
```

**Saída (Questão Formato DLS):**

```
QUESTÃO 01

[Art. 1º, Lei 6.015/73] IESES 2026 - Tabelião / Oficial de Registros - Provimento (TJ-PA)

Os serviços concernentes aos Registros Públicos, estabelecidos pela legislação civil,
destinam-se a garantir a publicidade, autenticidade, segurança e eficácia dos atos jurídicos.

( ) Verdadeiro
( ) Falso

Gabarito: Verdadeiro

Art. 1º Os serviços concernentes aos Registros Públicos, estabelecidos pela legislação
civil para autenticidade, segurança e eficácia dos atos jurídicos, ficam sujeitos ao
regime estabelecido nesta Lei. (Lei 6.015/73)
```

## Arquitetura do Sistema

### 1. Base de Conhecimento Compartilhada

**Banco Vetorial (PostgreSQL + PGVector)**

Repositório central de conhecimento legal acessível por todos os agents:

- **Legislação completa**: Leis, decretos, resoluções, portarias
  - Estrutura granular: Artigos, parágrafos, incisos, alíneas
  - Metadados ricos: Tipo, número, data, órgão, status
- **Materiais Didáticos**: Livros, apostilas, doutrina, artigos
  - Upload via API de PDFs
  - Processamento automático e chunking inteligente
  - Busca semântica integrada
- **Embeddings semânticos**: Busca por similaridade de contexto em toda a base

**APIs de Acesso à Base:**

```typescript
// Busca semântica
buscarArtigo(query: string, k: number): Artigo[]

// Busca por similaridade
buscarSimilares(embedding: number[], threshold: number): Artigo[]

// Busca específica
buscarPorReferencia(lei: string, artigo: string): Artigo

// Contexto expandido
obterContextoArtigo(artigoId: string): ContextoLegal
```

### 2. Sistema de Agents Plugáveis

Arquitetura modular onde cada agent é independente e especializado:

#### Agent 1: Gerador de Questões V/F (v1.0)

**Objetivo**: Transformar questões de múltipla escolha em verdadeiro/falso

**Sub-agents**:

#### Agent de Busca de Informações Legais

- **Responsabilidade**: Consultar e validar informações de leis e normas
- **Fontes**:
  - Base local (banco vetorial)
  - Internet (apenas fontes oficiais, como site do Planalto)
- **Função**: Garantir precisão jurídica das questões

#### Agent de Adaptação de Enunciado

- **Responsabilidade**: Transformar enunciados originais para o formato DLS
- **Entrada**: Alternativa correta da questão de múltipla escolha
- **Saída**: Afirmação única em formato verdadeiro/falso
- **Função**:
  - Extrair conteúdo da alternativa correta
  - Reformular como afirmação objetiva
  - Manter rigor técnico e precisão jurídica
  - Garantir clareza e concisão

#### Agent de Geração de Gabarito Comentado

- **Responsabilidade**: Extrair e formatar a fundamentação legal
- **Entrada**: Comentários da questão original
- **Saída**: Gabarito estruturado com:
  - Resposta (Verdadeiro/Falso)
  - Texto completo do artigo/lei citado
  - Referência legal (Lei, Resolução, etc.)
- **Função**:
  - Identificar fundamentação legal nos comentários
  - Buscar texto completo dos artigos citados
  - Validar informações em fontes oficiais

#### Agent de Classificação

- **Responsabilidade**: Extrair e categorizar metadados da questão
- **Atribuições**:
  - Identificar artigo/parágrafo cobrado
  - Extrair lei/código/resolução
  - Identificar banca organizadora
  - Ano do concurso
  - Cargo/concurso
  - Órgão
- **Função**:
  - Parser do título/cabeçalho da questão
  - Classificação baseada no banco de dados
  - Normalização de metadados

#### Agent 2: Chatbot de Legislação (futuro)

**Objetivo**: Responder perguntas sobre legislação em linguagem natural

**Funcionalidades**:

- Interpretação de perguntas em português
- Busca semântica na base vetorial
- Respostas contextualizadas com citações legais
- Histórico de conversação

**Casos de uso**:

- "O que diz a lei sobre registros públicos?"
- "Qual a diferença entre o Art. 1º e o Art. 2º da Lei 6.015/73?"
- "Como funciona o regime de bens no casamento?"

---

#### Agent 3: Gerador de Questões Inéditas (futuro)

**Objetivo**: Criar questões originais baseadas na legislação

**Funcionalidades**:

- Analisar artigos da base vetorial
- Gerar questões didáticas inéditas
- Múltiplos formatos (V/F, múltipla escolha, dissertativa)
- Controle de dificuldade

**Casos de uso**:

- Gerar 10 questões fáceis sobre Lei de Registros Públicos
- Criar questões avançadas sobre regime de bens
- Gerar simulado completo sobre legislação notarial

---

#### Agent 4: Gerador de Conteúdo para Marketing (futuro)

**Objetivo**: Criar conteúdo educativo para redes sociais e marketing

**Funcionalidades**:

- Posts educativos sobre artigos de lei
- Infográficos com resumos legais
- Threads explicativas
- Casos práticos simplificados

**Casos de uso**:

- "Você sabia? Art. 1º da Lei 6.015/73 explica..."
- Thread: "Entenda o regime de bens em 5 tweets"
- Post: "3 mudanças importantes da Resolução 583/2024"

---

#### Agent 5: Gerador de Material Didático em PDF (futuro)

**Objetivo**: Criar materiais de estudo visuais e estruturados

**Funcionalidades**:

- **Mapas mentais de leis**: Visualização hierárquica de artigos
- **Resumos em PDF**: Síntese de legislação com formatação profissional
- **Flashcards**: Material para memorização
- **Esquemas e diagramas**: Fluxogramas de procedimentos legais
- **Comparativos**: Tabelas comparando leis ou versões

**Casos de uso**:

- Gerar mapa mental da Lei 6.015/73 (Registros Públicos)
- PDF resumido: "Lei de Registros Públicos - Principais Artigos"
- Flashcards: 50 questões sobre regime de bens
- Diagrama: Fluxo de registro de imóveis
- Comparativo: Lei 6.015/73 vs Resolução 583/2024

**Tecnologias**:

- PDFKit ou Puppeteer para geração de PDF
- D3.js ou Mermaid para mapas mentais
- Templates customizáveis

---

#### Agent 6: Analisador de Mudanças Legislativas (futuro)

**Objetivo**: Detectar e explicar alterações em leis

**Funcionalidades**:

- Comparar versões de leis
- Identificar artigos alterados/revogados
- Gerar resumo de impactos
- Alertas de mudanças relevantes

---

#### Orquestrador de Agents

- **Responsabilidade**: Coordenar múltiplos agents
- **Roteamento**: Direcionar requisições para agent apropriado
- **Composição**: Combinar resultados de múltiplos agents quando necessário

### 3. Integração com Serviço Principal

**Comunicação entre Serviços**

- **Acesso direto ao banco MySQL** do serviço principal
- Consultar IDs de bancas, leis, artigos via queries SQL
- Gravar questões processadas e rascunhos no MySQL
- Interface administrativa do serviço principal valida e aprova questões

**Responsabilidades do Serviço Principal:**

- Validação e aprovação de questões
- Armazenamento final das questões aprovadas
- Gerenciamento de usuários e permissões
- Interface administrativa
- Cadastro de bancas, leis, artigos

## Fluxo de Trabalho

### Pipeline de Geração (Assíncrono)

```
┌─────────────────────────────────────────────────────────┐
│              SERVIÇO PRINCIPAL DLS                      │
├─────────────────────────────────────────────────────────┤
│  [Usuário faz upload de PDFs]                           │
│   - PDF da prova (questões múltipla escolha)            │
│   - PDF do gabarito (pode estar comentado)             │
│         ↓                                               │
│  [API Request] ──────────────────────┐                  │
└──────────────────────────────────────│──────────────────┘
                                       │
                                       ↓ HTTP POST (multipart/form-data)
┌─────────────────────────────────────────────────────────┐
│           MICROSERVIÇO DLS IA (Este Projeto)            │
│              (PostgreSQL + MySQL + Redis)               │
├─────────────────────────────────────────────────────────┤
│  [Recebe PDFs]                                          │
│         ↓                                               │
│  [Armazena PDFs temporariamente]                        │
│         ↓                                               │
│  [Cria Job na Fila (Bull/BullMQ)]                       │
│         ↓                                               │
│  [Retorna jobId imediatamente] ──────┐                  │
│                                       │                 │
│  ┌────────────────────────────────┐   │                 │
│  │  WORKER (Background Process)   │   │                 │
│  ├────────────────────────────────┤   │                 │
│  │  Processa jobs da fila         │   │                 │
│  │         ↓                      │   │                 │
│  │  [Agent Orquestrador]          │   │                 │
│         ↓                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │ FASE 0: Extração e Parsing de PDFs              │   │
│  │ [Agent de Extração de PDF]                       │   │
│  │  - Extrai texto dos PDFs                        │   │
│  │  - Identifica questões e alternativas           │   │
│  │  - Identifica gabarito e comentários            │   │
│  │  - Estrutura dados em formato JSON              │   │
│  └──────────────────────────────────────────────────┘   │
│         ↓                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │ FASE 1: Análise e Classificação                 │   │
│  │ [Agent de Classificação]                         │   │
│  │  - Extrai metadados (banca, ano, etc.)          │   │
│  │  - Identifica artigo/lei cobrado                │   │
│  └──────────────────────────────────────────────────┘   │
│         ↓                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │ FASE 2: Resolução de IDs                        │   │
│  │ [Consulta MySQL do Serviço Principal]           │   │
│  │  - Busca ID da banca (readonly)                 │   │
│  │  - Busca ID da lei (readonly)                   │   │
│  │  - Busca ID do artigo (readonly)                │   │
│  └──────────────────────────────────────────────────┘   │
│         ↓                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │ FASE 3: Busca de Informações Legais             │   │
│  │ [Agent de Busca]                                 │   │
│  │  - Consulta banco vetorial local (PostgreSQL)   │   │
│  │  - Se necessário: busca em fontes oficiais      │   │
│  └──────────────────────────────────────────────────┘   │
│         ↓                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │ FASE 4: Adaptação do Enunciado                  │   │
│  │ [Agent de Adaptação]                             │   │
│  │  - Reformula alternativa correta como V/F       │   │
│  └──────────────────────────────────────────────────┘   │
│         ↓                                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │ FASE 5: Geração do Gabarito                     │   │
│  │ [Agent de Gabarito]                              │   │
│  │  - Extrai/busca fundamentação legal             │   │
│  └──────────────────────────────────────────────────┘   │
│  │         ↓                      │   │                 │
│  │  [Questões Estruturadas]       │   │                 │
│  │         ↓                      │   │                 │
│  │  [GRAVA no MySQL (batch)]      │   │                 │
│  │         ↓                      │   │                 │
│  │  [Atualiza status do job]      │   │                 │
│  │         ↓                      │   │                 │
│  │  [Webhook callback (opcional)] │   │                 │
│  └────────────────────────────────┘   │                 │
└───────────────────────────────────────┼─────────────────┘
                                        │
                                        ↓ HTTP Response (imediato)
┌─────────────────────────────────────────────────────────┐
│              SERVIÇO PRINCIPAL DLS                      │
│                    (MySQL)                              │
├─────────────────────────────────────────────────────────┤
│  [Recebe jobId]                                         │
│         ↓                                               │
│  [Polling: GET /jobs/{jobId}/status]                    │
│   ou                                                    │
│  [Aguarda webhook callback]                             │
│         ↓                                               │
│  [Job completo: busca questões no MySQL]                │
│         ↓                                               │
│  [Interface de validação]                               │
│         ↓                                               │
│  [Admin aprova/rejeita]                                 │
│         ↓                                               │
│  [Atualiza status da questão no MySQL]                  │
│  ÚNICO LUGAR onde questões são armazenadas              │
└─────────────────────────────────────────────────────────┘
```

### Exemplo de Processamento

**1. Entrada:**

- Questão de múltipla escolha (a, b, c, d, e)
- Comentários com fundamentação legal
- Gabarito indicando alternativa correta (ex: D)

**2. Processamento:**

- Agent extrai alternativa D (correta)
- Agent busca artigo citado nos comentários
- Agent reformula alternativa D como afirmação
- Agent formata gabarito com fundamentação

**3. Saída:**

- Questão V/F estruturada
- Gabarito: Verdadeiro
- Fundamentação: Texto completo do artigo

## Tecnologias Principais

- **Framework**: NestJS
- **Banco de Dados**:
  - PostgreSQL + PGVector (próprio - embeddings e legislação)
  - MySQL (serviço principal - leitura de IDs e gravação de questões)
  - Redis (cache temporário - IDs, métricas, filas)
- **Query Builder**: Kysely (para MySQL, sem tipagens)
- **ORM**: Prisma (para PostgreSQL próprio)
- **Job Queue**: Bull/BullMQ (filas de processamento assíncrono)
- **PDF Processing**:
  - pdf-parse ou pdf.js (extração de texto)
  - LLM (estruturação e parsing inteligente)
- **IA/ML**: Sistema de agents com LLM
- **TypeScript**: Linguagem principal

## Arquitetura de Microserviços

```
┌─────────────────────┐         ┌──────────────────────┐
│  SERVIÇO PRINCIPAL  │────────►│   MICROSERVIÇO IA    │
│       (DLS)         │  REST   │     (Este)           │
│   (AdonisJS)        │         │   (NestJS)           │
├─────────────────────┤         ├──────────────────────┤
│ - Validação         │         │ - Processamento IA   │
│ - Aprovação         │         │ - Banco Vetorial     │
│ - Armazenamento     │         │ - Geração Questões   │
│ - Interface Admin   │         │ - Classificação      │
│ - Cadastros Base    │         │ - Query MySQL ──┐    │
│   (Bancas, Leis)    │         │ - Grava questões│    │
└─────────────────────┘         └─────────────────┼────┘
        │                                  │      │
        ↓                                  ↓      │
  ┌──────────┐                      ┌──────────┐ │
  │  MySQL   │◄─────────────────────┤PostgreSQL│ │
  │          │ Kysely (read/write)  │+ PGVector│ │
  └──────────┘                      └──────────┘ │
        ▲                                         │
        └─────────────────────────────────────────┘
```

## Data Structures

### Input (from Main Service)

```typescript
interface ProcessExamRequest {
  // PDF files
  examPdf: File; // PDF with multiple choice questions
  answerKeyPdf: File; // PDF with answer key (may or may not be commented)

  // Metadata (optional - can be extracted from PDFs)
  metadata?: {
    bank?: string; // Ex: "IESES"
    year?: number; // Ex: 2026
    position?: string; // Ex: "Tabelião / Oficial de Registros"
    organization?: string; // Ex: "TJ-PA"
  };

  // Callback URL (optional)
  webhookUrl?: string; // URL to call when processing is complete
}

interface ProcessExamResponse {
  jobId: string; // Job ID to track progress
  status: 'queued' | 'processing' | 'completed' | 'failed';
  estimatedTime?: number; // Estimated time in seconds
  createdAt: string;
}
```

### Extracted Question (after PDF parsing - Phase 0)

```typescript
interface ExtractedQuestion {
  number: string; // Ex: "QUESTÃO 01"
  statement: string; // Question statement
  alternatives: {
    letter: string; // a, b, c, d, e
    text: string;
  }[];
  correctAnswer: string; // Correct alternative letter
  comments?: string; // Comments from answer key (if available)
}
```

### Job Status (for tracking)

```typescript
interface JobStatus {
  jobId: string;
  status: 'queued' | 'processing' | 'completed' | 'failed';
  progress: {
    current: number; // Current question being processed
    total: number; // Total questions found
    percentage: number; // Percentage complete
  };
  result?: {
    questionsCreated: number;
    questionIds: string[]; // IDs in main service MySQL
    errors: string[];
  };
  error?: string; // Error message if failed
  createdAt: string;
  startedAt?: string;
  completedAt?: string;
}
```

### Generated Question (Microservice Output)

```typescript
interface GeneratedQuestion {
  number: number;

  // Main service reference IDs
  articleId: string; // Article ID from main service DB
  lawId: string; // Law ID from main service DB
  bankId: string; // Bank ID from main service DB
  yearId?: string; // Year ID from main service DB (if applicable)

  // Textual values (for display)
  article: string; // Ex: "Art. 1º"
  law: string; // Ex: "Lei 6.015/73"
  bank: string; // Ex: "IESES"
  year: number; // Ex: 2026
  position: string; // Ex: "Tabelião / Oficial de Registros - Provimento"
  organization: string; // Ex: "TJ-PA"

  // Generated content
  statement: string; // True/false statement
  answer: boolean; // true = True, false = False
  justification: string; // Full article text
  comment?: string; // Additional comment (optional)
}
```

## Requisitos Funcionais

1. **Receber e processar PDFs** de provas e gabaritos
2. **Extrair questões** de múltipla escolha dos PDFs
3. **Identificar gabaritos** (com ou sem comentários)
4. **Transformar** para formato verdadeiro/falso
5. **Gerar gabaritos comentados** fundamentados
6. **Classificar questões** com metadados relevantes (artigo, lei, banca, ano, cargo, órgão)
7. **Validar informações** em fontes oficiais
8. **Permitir validação manual** via interface administrativa do serviço principal
9. **Extrair fundamentação legal** dos comentários ou buscar em fontes oficiais

## Regras de Transformação

### 1. Identificação do Artigo

- Extrair artigo citado do cabeçalho ou comentários
- Formato: `[Art. X, Lei Y]`
- Prioridade: cabeçalho > primeira citação nos comentários

### 2. Reformulação do Enunciado

- **Sempre usar a alternativa CORRETA** como base
- Transformar em afirmação objetiva
- Remover contextualizações desnecessárias
- Manter precisão técnica e termos legais
- Evitar duplas negativas

### 3. Casos Especiais

#### Questões com múltiplos artigos

- Criar uma questão para cada artigo relevante
- Priorizar artigo mais específico

#### Alternativa correta com múltiplas afirmações

- Separar em questões diferentes quando possível
- Ou manter conjunto de afirmações se indivisível

#### Fundamentação em Resoluções/Decretos

- Buscar texto oficial completo
- Indicar claramente origem (CNJ, etc.)
- Incluir data/número da resolução

### 4. Validação de Qualidade

- Enunciado deve ser claro e objetivo
- Gabarito deve citar fonte oficial
- Metadados completos e precisos
- Fundamentação legal deve ser literal (texto original)

## Próximos Passos

### Fase 1: Infraestrutura

- [ ] Configurar banco de dados PostgreSQL + PGVector
- [ ] Configurar Redis para caching e filas
- [ ] Setup de sistema de filas (Bull/BullMQ)
- [ ] Setup de embeddings para busca semântica
- [ ] Configurar conexão MySQL ao serviço principal (read/write)

### Fase 2: Base de Conhecimento

- [ ] Importar leis e códigos oficiais
- [ ] Sistema de classificação e indexação
- [ ] Pipeline de atualização de legislação
- [ ] Geração de embeddings vetoriais

### Fase 3: Desenvolvimento dos Agents

- [ ] Agent de Extração de PDF (parsing de provas e gabaritos)
- [ ] Agent de Classificação (parser de metadados)
- [ ] Agent de Resolução de IDs (consulta MySQL do serviço principal)
- [ ] Agent de Busca (RAG + web scraping oficial)
- [ ] Agent de Adaptação (reformulação LLM)
- [ ] Agent de Gabarito (extração + formatação)
- [ ] Agent Orquestrador (coordenação de todo o pipeline)

### Fase 4: API e Integração

- [ ] API REST para upload de PDFs (assíncrono)
- [ ] Endpoints de status de jobs
- [ ] Sistema de filas para processamento em background
- [ ] Workers para processar jobs
- [ ] Integração com serviço principal
- [ ] Sistema de webhooks/callbacks

### Fase 5: Gerenciamento de Materiais Didáticos

- [ ] API para upload de PDFs de materiais didáticos
- [ ] Processamento e chunking inteligente de PDFs
- [ ] Indexação de materiais no vector store
- [ ] Busca semântica em materiais didáticos
- [ ] Endpoints para listar e remover materiais
- [ ] Integração da busca com agents existentes

### Fase 6: Qualidade e Deploy

- [ ] Testes automatizados (unit + e2e)
- [ ] Validação de precisão legal
- [ ] Monitoramento e métricas
- [ ] Deploy e CI/CD
- [ ] Documentação de API

---

## Documentação Técnica

- **[Overview](docs/overview.md)** - Visão geral e roadmap do projeto
- **[Agents Architecture](docs/agents-architecture.md)** - Detalhes de implementação de cada agent
- **[Database Schema](docs/database-schema.md)** - Modelo de dados completo
- **[API Endpoints](docs/api-endpoints.md)** - Documentação da API REST
- **[Main Service Integration](docs/main-service-integration.md)** - Comunicação entre serviços

---

## Setup do Projeto

### Pré-requisitos

- Node.js 18+
- PostgreSQL 15+
- pnpm
- Chave de API do OpenAI/Anthropic

### Instalação

```bash
# Instalar dependências
pnpm install

# Configurar variáveis de ambiente
cp .env.example .env
# Editar .env com suas credenciais
```

### Configuração do Banco de Dados

```bash
# Criar banco e habilitar PGVector
psql -U postgres
CREATE DATABASE dls_ia;
\c dls_ia
CREATE EXTENSION vector;

# Executar migrations
pnpm prisma migrate dev
```

### Executar o Projeto

```bash
# desenvolvimento
pnpm run start

# modo watch
pnpm run start:dev

# produção
pnpm run start:prod
```

### Testes

```bash
# testes unitários
pnpm run test

# testes e2e
pnpm run test:e2e

# cobertura de testes
pnpm run test:cov
```

---

## Estrutura do Projeto

```
dls-ia/
├── docs/                          # Documentação técnica
│   ├── overview.md                # Visão geral do projeto
│   ├── agents-architecture.md     # Arquitetura dos agents
│   ├── database-schema.md         # Schema do banco
│   ├── api-endpoints.md           # Documentação da API
│   └── main-service-integration.md
│
├── src/
│   ├── agents/                    # Implementação dos agents
│   │   ├── orchestrator/
│   │   ├── pdf-extraction/        # Extração e parsing de PDFs
│   │   ├── classification/
│   │   ├── id-resolution/
│   │   ├── search/
│   │   ├── adaptation/
│   │   └── answer-key/
│   │
│   ├── modules/
│   │   ├── questions/             # Módulo de processamento
│   │   ├── legislation/           # Módulo de legislação
│   │   ├── integration/           # Cliente HTTP serviço principal
│   │   └── cache/                 # Gerenciamento de cache
│   │
│   ├── database/
│   │   ├── prisma/                # Schema Prisma
│   │   └── migrations/            # Migrations
│   │
│   └── common/
│       ├── interfaces/
│       ├── types/
│       └── utils/
│
├── prisma/
│   └── schema.prisma              # Schema do banco
│
├── test/
│   ├── unit/
│   └── e2e/
│
├── .env.example
├── package.json
└── README.md
```

---

## Variáveis de Ambiente

```bash
# PostgreSQL (Legislação e Logs)
DATABASE_URL="postgresql://user:password@localhost:5432/dls_ia"

# MySQL (Serviço Principal - Questões)
MYSQL_HOST="localhost"
MYSQL_PORT="3306"
MYSQL_USER="dls_ia_service"
MYSQL_PASSWORD="password"
MYSQL_DATABASE="dls_main"

# Redis (Cache)
REDIS_HOST="localhost"
REDIS_PORT="6379"
REDIS_PASSWORD=""
REDIS_DB="0"

# LLM / IA
OPENAI_API_KEY="sk-..."
OPENAI_MODEL="gpt-4"
OPENAI_EMBEDDING_MODEL="text-embedding-3-large"

# Aplicação
PORT=3000
NODE_ENV="development"
```
