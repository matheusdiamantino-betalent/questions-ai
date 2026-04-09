# 🔄 PR 15 — Fase 1: Foundation Mínima de Observabilidade Local da Ingestion
## Introdução de logging local com Docker, Loki, Promtail e Grafana para suporte operacional mínimo

---

<div align="left">

![PR](https://img.shields.io/badge/PR-15-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-minimal%20local%20observability%20foundation-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge&logo=target&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-docker%20%2B%20loki%20%2B%20grafana-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Esta PR é **complementar à Fase 1 da ingestion** e introduz apenas a foundation mínima necessária de **logging e observabilidade local** para apoiar o fluxo já consolidado nas PRs anteriores.
>
> - manter o fluxo funcional já existente da `ingestion`
> - manter o ciclo operacional até `completed` / `failed`
> - introduzir visibilidade mínima de execução local
> - permitir inspeção básica de logs via Docker
> - disponibilizar uma stack local pequena com **Loki + Promtail + Grafana**
>
> **Esta PR não altera o domínio da ingestion, não introduz novas regras de negócio e não transforma observabilidade em dependência do processamento.**

---

## 📚 Sumário

1. [Síntese Executiva](#1-síntese-executiva)
2. [Objetivo do PR](#2-objetivo-do-pr)
3. [Decisão Arquitetural](#3-decisão-arquitetural)
4. [Escopo](#4-escopo)
5. [Fora de Escopo](#5-fora-de-escopo)
6. [Fluxo Arquitetural](#6-fluxo-arquitetural)
7. [Contratos Mínimos](#7-contratos-mínimos)
8. [Regras de Implementação](#8-regras-de-implementação)
9. [Infraestrutura Local via Docker](#9-infraestrutura-local-via-docker)
10. [Configuração Esperada](#10-configuração-esperada)
11. [Uso Operacional Local](#11-uso-operacional-local)
12. [Critérios de Review](#12-critérios-de-review)
13. [Critérios de Aceite](#13-critérios-de-aceite)
14. [Conclusão](#14-conclusão)

---

## 1. Síntese Executiva

Até a PR 14, a Fase 1 da ingestion consolidou um fluxo funcional mínimo:

- abertura persistida da operação
- enqueue mínimo
- consumo mínimo
- transição para `processing`
- encerramento terminal em `completed`
- encerramento terminal em `failed`
- persistência mínima de `failureReason`

Com isso, o domínio já possui o primeiro ciclo operacional básico completo.

O próximo passo correto aqui não é expandir retry, resiliência ou pipeline.  
O próximo passo correto é adicionar **visibilidade operacional mínima** sobre o que já existe, para facilitar execução local, debugging e inspeção do comportamento da aplicação durante desenvolvimento.

Esta PR introduz exatamente isso:

- foundation mínima de logging local
- foundation mínima de observabilidade local
- stack simples via Docker
- visualização básica de logs com **Grafana**
- armazenamento e consulta local com **Loki**
- coleta local simples com **Promtail**

Sem reabrir o domínio da ingestion e sem inflar o recorte com plataforma completa de observabilidade.

---

## 2. Objetivo do PR

Introduzir uma foundation mínima de observabilidade local para apoiar o fluxo já existente da `ingestion`, mantendo o recorte pequeno e operacional.

### Em termos práticos

Esta PR deve permitir apenas:

- subir uma stack local simples de observabilidade via Docker
- visualizar logs da aplicação localmente
- centralizar logs locais em Loki
- consultar logs no Grafana
- manter a aplicação funcionalmente igual do ponto de vista de domínio

### Resultado esperado

Ao final desta PR, a aplicação deve ser capaz de:

- continuar processando `ingestion` como já faz hoje
- emitir logs mínimos úteis para acompanhamento local
- disponibilizar esses logs em um ambiente local simples
- permitir inspeção manual do fluxo em desenvolvimento sem alterar a lógica do processamento

> [!NOTE]
> O objetivo desta PR **não** é introduzir observabilidade completa.
>
> O objetivo é apenas materializar uma **base local mínima e utilizável** para logging e consulta operacional do fluxo já existente.

---

## 3. Decisão Arquitetural

A decisão central desta PR é:

> **introduzir apenas observabilidade local mínima, desacoplada do domínio da ingestion e suportada por uma stack simples via Docker com Loki, Promtail e Grafana.**

A arquitetura funcional da ingestion permanece a mesma.

Esta PR adiciona apenas uma infraestrutura auxiliar e leve de apoio operacional.

### Isso significa

- manter a lógica da `ingestion` inalterada em sua essência
- adicionar logging mínimo e explícito
- utilizar Docker para facilitar o setup local
- usar Loki como storage/query engine de logs
- usar Promtail como coleta local simples
- usar Grafana como interface mínima de visualização
- evitar instrumentação profunda, frameworks paralelos ou abstrações prematuras

### Boundary exato desta PR

A observabilidade introduzida aqui deve ser entendida apenas como:

- emissão mínima de logs úteis
- coleta local desses logs
- consulta local desses logs
- uso operacional em desenvolvimento

Nada além disso é objetivo desta entrega.

---

## 4. Escopo

Esta PR inclui:

- introdução de foundation mínima de logging local
- stack local via Docker Compose
- inclusão de **Loki**
- inclusão de **Promtail**
- inclusão de **Grafana**
- configuração local mínima para coleta de logs
- documentação de subida, acesso e uso operacional básico
- preservação integral do fluxo funcional já existente da ingestion

### Em termos de implementação

Espera-se que esta PR cubra:

- definição do `docker-compose` da stack local
- configuração mínima do Loki
- configuração mínima do Promtail
- configuração mínima do Grafana
- definição simples de origem dos logs da aplicação
- documentação objetiva de como subir e validar o ambiente
- logs mínimos e explícitos no fluxo da aplicação, quando aplicável

### Unidade mínima concluída nesta PR

A unidade operacional mínima desta entrega deve continuar sendo simples:

- aplicação continua executando como antes
- logs passam a ser visíveis localmente
- a stack pode ser iniciada via Docker
- o operador local consegue inspecionar o comportamento sem depender de infraestrutura externa

> [!IMPORTANT]
> O recorte desta PR termina na introdução controlada de uma **stack local mínima de observabilidade**.
>
> Qualquer expansão para tracing distribuído, métricas ricas, alertas, dashboards elaborados ou observabilidade de produção fica fora desta entrega.

---

## 5. Fora de Escopo

Esta PR **não** inclui:

- OpenTelemetry completo
- tracing distribuído
- spans
- métricas de negócio
- métricas técnicas ricas
- alertas
- regras de alarme
- dashboards sofisticados
- correlação distribuída entre serviços
- observabilidade de produção
- retenção avançada
- autenticação sofisticada do Grafana
- HA
- clusterização
- pipelines completos de logs
- SIEM
- APM
- indexação avançada
- taxonomia rica de eventos operacionais

> [!NOTE]
> A regra permanece a mesma:
>
> **não implementar a próxima fase dentro da fase atual.**

---

## 6. Fluxo Arquitetural

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#0b1020",
    "primaryColor": "#111827",
    "primaryTextColor": "#e5f0ff",
    "primaryBorderColor": "#22d3ee",
    "lineColor": "#38bdf8",
    "secondaryColor": "#0f172a",
    "tertiaryColor": "#111827",
    "fontFamily": "Inter, Segoe UI, Arial",
    "clusterBkg": "#0f172a",
    "clusterBorder": "#334155"
  }
}}%%
flowchart LR
    A[HTTP Request]:::http --> B[Ingestion Flow]:::service
    B --> C[Application Logs]:::logs
    C --> D[Local Log Source]:::logs
    D --> E[Promtail]:::collector
    E --> F[Loki]:::storage
    F --> G[Grafana]:::ui

    classDef http fill:#111827,stroke:#38bdf8,color:#e0f2fe,stroke-width:2px;
    classDef service fill:#111827,stroke:#22c55e,color:#ecfdf5,stroke-width:2px;
    classDef logs fill:#451a03,stroke:#f59e0b,color:#fff7ed,stroke-width:2px;
    classDef collector fill:#172554,stroke:#60a5fa,color:#eff6ff,stroke-width:2px;
    classDef storage fill:#052e16,stroke:#22c55e,color:#f0fdf4,stroke-width:2px;
    classDef ui fill:#1e1b4b,stroke:#c084fc,color:#f5f3ff,stroke-width:2px;
```

> [!IMPORTANT]
> Neste recorte, **observabilidade não interfere no fluxo funcional da ingestion**.
>
> A stack local apenas consome e expõe logs para inspeção operacional em desenvolvimento.

---

## 7. Contratos Mínimos

Os contratos de domínio devem continuar pequenos e aderentes ao recorte.

### Regra principal desta PR

Esta entrega **não deve inflar contratos da ingestion**.

Isso significa:

- não alterar o payload da fila por causa de observabilidade
- não alterar contratos de DTO por causa de Grafana/Loki
- não introduzir metadata rica de tracing no contrato principal
- não transformar logs em requisito de negócio
- não acoplar a execução da ingestion à disponibilidade da stack local

### Evolução mínima aceitável

Caso exista evolução de logging na aplicação, ela deve ser apenas:

- explícita
- pequena
- operacional
- não invasiva
- removível sem quebrar o domínio

### Regra importante

Fora a materialização explícita de logs mínimos locais, esta PR **não amplia** contratos de payload, processamento ou execução.

---

## 8. Regras de Implementação

### Aplicação

A aplicação deve:

- continuar simples
- continuar executando sem dependência lógica do Grafana/Loki
- manter o fluxo funcional da ingestion inalterado
- emitir logs mínimos úteis
- não absorver infraestrutura futura desnecessária

### Logging

O logging deve continuar sendo:

- mínimo
- explícito
- legível
- operacional
- aderente ao recorte

### Regras do logging esperado

O logging desta PR deve fazer apenas:

1. registrar pontos mínimos úteis do fluxo
2. permitir leitura local
3. não introduzir engine própria de observabilidade
4. não exigir framework paralelo sofisticado
5. não reestruturar o domínio para acomodar logs

### Docker

A infraestrutura local deve:

- ser simples de subir
- ser local-only
- ser fácil de entender
- ser fácil de revisar
- não exigir múltiplas dependências ocultas
- permitir execução por um comando previsível

### Grafana / Loki / Promtail

A stack deve:

- subir localmente
- ter configuração mínima e explícita
- usar Loki como destino de logs
- usar Promtail como agente simples de coleta
- usar Grafana como visualização mínima

### Configuração

A configuração deve:

- permanecer centralizada no `environment.ts`, quando aplicável à aplicação
- seguir o padrão já adotado com Zod, quando houver variáveis novas da app
- não espalhar leitura de `process.env`
- não misturar configuração de aplicação com comportamento de domínio

---

## 9. Infraestrutura Local via Docker

A proposta mínima desta PR é disponibilizar uma stack local composta por:

- **Grafana**
- **Loki**
- **Promtail**

### Papel de cada componente

#### Loki
Responsável por armazenar e consultar logs localmente.

#### Promtail
Responsável por ler a origem configurada dos logs locais e enviar esses logs para o Loki.

#### Grafana
Responsável por oferecer a interface mínima de exploração, busca e leitura dos logs.

### Estrutura mínima esperada

Exemplo de estrutura de arquivos:

```text
docker/
  observability/
    grafana/
      provisioning/
        datasources/
          datasource.yml
    loki/
      config.yml
    promtail/
      config.yml

docker-compose.observability.yml
```

### Exemplo mínimo de `docker-compose`

```yaml
version: '3.9'

services:
  loki:
    image: grafana/loki:2.9.8
    container_name: local-loki
    ports:
      - '3100:3100'
    command: -config.file=/etc/loki/config.yml
    volumes:
      - ./docker/observability/loki/config.yml:/etc/loki/config.yml:ro

  promtail:
    image: grafana/promtail:2.9.8
    container_name: local-promtail
    command: -config.file=/etc/promtail/config.yml
    volumes:
      - ./docker/observability/promtail/config.yml:/etc/promtail/config.yml:ro
      - ./logs:/var/log/app:ro
    depends_on:
      - loki

  grafana:
    image: grafana/grafana:10.4.3
    container_name: local-grafana
    ports:
      - '3001:3000'
    volumes:
      - ./docker/observability/grafana/provisioning:/etc/grafana/provisioning:ro
    depends_on:
      - loki
```

### Exemplo mínimo de configuração do Loki

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /tmp/loki
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

storage_config:
  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  allow_structured_metadata: false
```

### Exemplo mínimo de configuração do Promtail

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: app-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: ingestion-app
          env: local
          __path__: /var/log/app/*.log
```

### Exemplo mínimo de datasource provisionado no Grafana

```yaml
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true
```

> [!NOTE]
> Os exemplos acima representam a **forma mínima esperada** da stack.
>
> Ajustes pontuais de paths, nomes de arquivos e portas são aceitáveis desde que mantenham o recorte pequeno e o setup simples.

---

## 10. Configuração Esperada

A configuração introduzida nesta PR deve continuar pequena.

### Na aplicação

Se houver configuração nova para habilitar ou controlar logging local, ela deve:

- ficar centralizada em `environment.ts`
- seguir o padrão do projeto
- usar Zod quando aplicável
- ser opcional
- não bloquear a execução funcional da ingestion

### Variáveis aceitáveis

Exemplos de variáveis aceitáveis, caso necessárias:

- `LOG_LEVEL`
- `LOG_DIR`
- `APP_ENV`

### Regras importantes

- não espalhar `process.env`
- não introduzir config difusa
- não acoplar a aplicação à stack de observabilidade
- a aplicação deve continuar funcional mesmo sem a stack local ativa

---

## 11. Uso Operacional Local

### Subida da stack

Exemplo:

```bash
docker compose -f docker-compose.observability.yml up -d
```

### Acesso esperado

- **Grafana** → `http://localhost:3001`
- **Loki** → `http://localhost:3100`

### Fluxo operacional mínimo esperado

1. subir a stack local
2. iniciar a aplicação
3. executar o fluxo da ingestion
4. gerar logs locais
5. validar coleta via Promtail
6. consultar logs no Grafana usando o datasource Loki

### Consulta mínima esperada no Grafana

Exemplo de busca simples:

```logql
{job="ingestion-app", env="local"}
```

### Resultado esperado

O operador local deve conseguir:

- ver logs chegando no Loki
- consultar logs da aplicação no Grafana
- acompanhar o fluxo da ingestion sem alterar o domínio
- usar essa base para debugging local simples

---

## 12. Critérios de Review

O review desta PR deve validar se:

- a PR 15 está corretamente posicionada como **foundation operacional complementar**
- a stack local via Docker permanece pequena e revisável
- Grafana, Loki e Promtail foram introduzidos sem excesso estrutural
- a ingestão continua funcionalmente igual
- os logs introduzidos são mínimos e úteis
- não existe acoplamento indevido entre domínio e observabilidade
- o setup local é objetivo e fácil de reproduzir
- não há OpenTelemetry completo, tracing distribuído, métricas ricas, alertas ou plataforma expandida escondida nesta entrega

---

## 13. Critérios de Aceite

Esta PR pode ser considerada aceita se:

- [ ] o fluxo funcional atual da `ingestion` continuar operando sem alteração de domínio
- [ ] existir uma stack local mínima via Docker
- [ ] Loki subir corretamente
- [ ] Promtail conseguir coletar logs da origem configurada
- [ ] Grafana conseguir consultar Loki
- [ ] os logs da aplicação puderem ser visualizados localmente
- [ ] o setup for simples de executar e revisar
- [ ] a aplicação continuar funcionando mesmo sem depender logicamente da stack
- [ ] o recorte permanecer pequeno, funcional e revisável
- [ ] não houver antecipação indevida de observabilidade expandida, tracing completo, alertas ou plataforma de produção

---

## 14. Conclusão

A PR 15 introduz o próximo passo correto **de suporte operacional local** para a Fase 1:

> **dar visibilidade mínima ao fluxo já funcional da ingestion por meio de uma stack local simples com Docker, Loki, Promtail e Grafana, preservando o recorte pequeno e sem expandir o domínio.**

Em resumo:

- as PRs anteriores consolidaram o fluxo funcional mínimo da ingestion
- a PR 15 não expande esse domínio
- a PR 15 introduz apenas apoio operacional local
- Loki centraliza os logs locais
- Promtail coleta os logs
- Grafana permite visualização simples
- o ambiente permanece pequeno, explícito e revisável

Esta entrega existe para melhorar **execução local, inspeção e debugging**, sem inflar a arquitetura e sem abrir nova frente de complexidade indevida.
