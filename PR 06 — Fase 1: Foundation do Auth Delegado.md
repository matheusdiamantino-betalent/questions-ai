# 🔐 PR 06 — Fase 1: Foundation do Auth Delegado
## Slice mínimo para consumir a identidade administrativa da API principal

---

<div align="left">

![PR](https://img.shields.io/badge/PR-06-2563eb?style=for-the-badge&logo=gitpullrequest&logoColor=white)
![Tipo](https://img.shields.io/badge/tipo-auth%20delegado-7c3aed?style=for-the-badge&logo=nestjs&logoColor=white)
![Fase](https://img.shields.io/badge/fase-1-0f766e?style=for-the-badge&logo=dependabot&logoColor=white)
![Escopo](https://img.shields.io/badge/escopo-identidade%20remota%20e%20prote%C3%A7%C3%A3o%20http-0891b2?style=for-the-badge&logo=serverless&logoColor=white)
![Status](https://img.shields.io/badge/status-pronto%20para%20review-16a34a?style=for-the-badge&logo=githubactions&logoColor=white)

</div>

---

> [!IMPORTANT]
> Este PR implementa apenas o recorte mínimo do auth delegado da API de IA:
>
> - receber `Authorization: Bearer <token>`;
> - consultar a API principal;
> - preservar o contrato externo relevante do endpoint de perfil;
> - reduzir localmente o payload autenticado para `request.user.id`;
> - proteger rotas HTTP locais.
>
> **Este PR não implementa autorização local por role/scope, cache, retry avançado, circuit breaker, decorators customizados ou expansão prematura do slice.**

---

## 📚 Sumário

1. [Síntese Executiva](#1-síntese-executiva)
2. [Contexto e Objetivo](#2-contexto-e-objetivo)
3. [Decisão Arquitetural](#3-decisão-arquitetural)
4. [Escopo e Fora de Escopo](#4-escopo-e-fora-de-escopo)
5. [Contratos](#5-contratos)
6. [Estrutura Técnica](#6-estrutura-técnica)
7. [Fluxo do Auth Delegado](#7-fluxo-do-auth-delegado)
8. [Responsabilidades por Arquivo](#8-responsabilidades-por-arquivo)
9. [Environment e Configuração](#9-environment-e-configuração)
10. [Regras de Simplicidade Aplicadas](#10-regras-de-simplicidade-aplicadas)
11. [Segurança e Falha](#11-segurança-e-falha)
12. [Rotas Técnicas](#12-rotas-técnicas)
13. [Critérios de Aceite](#13-critérios-de-aceite)
14. [Conclusão](#14-conclusão)

---

## 1. Síntese Executiva

A API principal continua sendo a autoridade de autenticação e identidade administrativa do ecossistema.

Este PR faz a API de IA consumir remotamente essa identidade, sem duplicar login, emissão de token, guard do sistema principal ou acesso direto ao Redis do auth legado.

### Resultado

- a API principal continua autenticando;
- a API de IA consome o perfil autenticado remotamente;
- a API de IA preserva explicitamente o contrato externo relevante do endpoint de perfil;
- a API de IA reduz localmente apenas o payload anexado em `request.user`;
- a API de IA protege rotas locais com `AuthGuard`.

```text
API principal autentica.
API de IA consome o perfil autenticado.
API de IA preserva o contrato externo do endpoint de perfil.
API de IA reduz apenas request.user.id.
API de IA protege a borda HTTP local.
```

---

## 2. Contexto e Objetivo

A API principal já possui autenticação administrativa própria e permanece como fonte da verdade para identidade.

Com base no mapeamento realizado na API principal:

- o auth administrativo usa o guard `admin`;
- o guard `admin` usa driver `oat`;
- o token provider do guard `admin` usa Redis;
- o provider do guard `admin` usa o model `Admin`;
- o endpoint de perfil autenticado retorna o admin autenticado;
- o payload de perfil pode conter dados administrativos além do `id`, inclusive `roles`.

### Implicação para a API de IA

A API de IA:

- não replica o guard da API principal;
- não acessa o Redis do auth principal;
- não reimplementa login administrativo;
- não adiciona autorização rica local neste slice;
- preserva o contrato útil do endpoint remoto;
- reduz localmente apenas o payload anexado à request.

---

## 3. Decisão Arquitetural

### Decisão central

**Auth delegado mínimo com consumo remoto de identidade**

### Regra implementada

```text
Auth continua centralizado na API principal.
A API de IA consome a identidade autenticada via HTTP.
A API de IA preserva o contrato do endpoint de perfil.
A API de IA reduz apenas request.user ao mínimo necessário.
```

> [!NOTE]
> O objetivo deste PR não é sofisticar o módulo de auth.
>
> O objetivo é estabelecer a foundation mínima, segura e revisável para:
>
> - consumo remoto de identidade;
> - proteção de rotas locais;
> - boundary de integração explícito;
> - payload local reduzido ao que o recorte realmente precisa.

---

## 4. Escopo e Fora de Escopo

### Escopo deste PR

- receber `Authorization: Bearer <token>`;
- chamar `GET /api/v1/profile` da API principal;
- representar o contrato externo relevante;
- validar o `id` recebido;
- transformar o resultado em `AuthenticatedUser`;
- anexar `request.user`;
- proteger rotas HTTP locais com `AuthGuard`;
- expor endpoints técnicos de validação ponta a ponta.

### Fora de escopo neste PR

- login administrativo;
- emissão de token;
- acesso Redis do auth principal;
- cache de introspecção;
- retry avançado;
- circuit breaker;
- autorização local por roles/scopes;
- decorators customizados;
- enriquecimento adicional de `request.user`;
- expansões prematuras de observabilidade específicas do slice.

---

## 5. Contratos

Para manter o boundary explícito sem inflar o slice, este PR preserva dois níveis de contrato: o contrato externo relevante da API principal e o contrato interno mínimo local.

### 5.1 Contrato externo da API principal

Tudo que representa o contrato real da API principal usa o prefixo:

- `MainApiV1...`

### 5.2 Endpoint autenticado consumido

```http
GET /api/v1/profile
Authorization: Bearer <token>
Accept: application/json
```

### 5.3 Contrato externo relevante

```ts
export type MainApiV1AdminRoleResponse = {
  id: number;
  name?: string;
  slug?: string;
};

export type MainApiV1AdminProfileResponse = {
  id: number | string;
  name?: string;
  email?: string;
  roles?: MainApiV1AdminRoleResponse[];
};
```

### 5.4 Contrato interno mínimo anexado à request

```ts
export type AuthenticatedUser = {
  id: number;
};
```

> [!IMPORTANT]
> O contrato externo da API principal não deve ser empobrecido artificialmente.
>
> A simplificação acontece apenas no payload local anexado à request.

### 5.5 Regra de transformação

```text
MainApiV1AdminProfileResponse
→ validação local do id
→ AuthenticatedUser
```

---

## 6. Estrutura Técnica

### Estrutura final do slice

```text
src/
├── app.module.ts
├── main.ts
├── modules/
│   ├── auth/
│   │   ├── infra/
│   │   │   ├── clients/
│   │   │   │   └── auth-api.client.ts
│   │   │   ├── guards/
│   │   │   │   └── auth.guard.ts
│   │   │   └── services/
│   │   │       └── auth.service.ts
│   │   ├── model/
│   │   │   └── v1/
│   │   │       └── auth.contracts.ts
│   │   └── auth.module.ts
│   └── health/
│       ├── infra/
│       │   ├── controllers/
│       │   │   └── health.controller.ts
│       │   └── services/
│       │       └── health.service.ts
│       └── health.module.ts
└── shared/
    └── config/
        └── environment.ts
```

### Leitura da estrutura

- `infra` concentra client, guard e service;
- `model/v1` concentra contrato externo relevante e contrato interno mínimo;
- `auth.module.ts` compõe o módulo;
- `health` valida o fluxo ponta a ponta;
- `app.module.ts` e `main.ts` fecham o wiring mínimo da aplicação;
- `environment.ts` centraliza a configuração necessária para integração.

> [!TIP]
> A arquitetura do projeto foi preservada sem criar fundação paralela nem espalhar cerimônia desnecessária pelo slice.

---

## 7. Fluxo do Auth Delegado

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#050b16",
    "primaryColor": "#0b1220",
    "primaryTextColor": "#ffffff",
    "primaryBorderColor": "#22d3ee",
    "lineColor": "#94a3b8",
    "secondaryColor": "#0b1220",
    "tertiaryColor": "#0b1220",
    "fontFamily": "Inter, Arial, sans-serif"
  },
  "flowchart": {
    "htmlLabels": true,
    "curve": "linear",
    "nodeSpacing": 30,
    "rankSpacing": 42
  }
}}%%
flowchart TD
    A["Request com bearer"] --> B["AuthGuard"]
    B --> C["AuthService"]
    C --> D["AuthApiClient"]
    D --> E["GET /api/v1/profile"]
    E --> F["MainApiV1AdminProfileResponse"]
    F --> G["Validação local do id"]
    G --> H["AuthenticatedUser"]
    H --> I["request.user.id"]
    I --> J["Request protegida"]

    classDef step1 fill:#0b1325,stroke:#3b82f6,stroke-width:2px,color:#ffffff;
    classDef step2 fill:#0a1a22,stroke:#22d3ee,stroke-width:2px,color:#ffffff;
    classDef step3 fill:#201d10,stroke:#eab308,stroke-width:2px,color:#ffffff;
    classDef step4 fill:#181629,stroke:#a78bfa,stroke-width:2px,color:#ffffff;
    classDef step5 fill:#25170f,stroke:#f97316,stroke-width:2px,color:#ffffff;
    classDef step6 fill:#112015,stroke:#84cc16,stroke-width:2px,color:#ffffff;
    classDef step7 fill:#10243a,stroke:#38bdf8,stroke-width:2px,color:#ffffff;
    classDef step8 fill:#221b2f,stroke:#f472b6,stroke-width:2px,color:#ffffff;
    classDef step9 fill:#14281d,stroke:#34d399,stroke-width:2px,color:#ffffff;
    classDef step10 fill:#1e293b,stroke:#f8fafc,stroke-width:2px,color:#ffffff;

    class A step1;
    class B step2;
    class C step3;
    class D step4;
    class E step5;
    class F step6;
    class G step7;
    class H step8;
    class I step9;
    class J step10;

    linkStyle 0 stroke:#9ca3af,stroke-width:2px;
    linkStyle 1 stroke:#9ca3af,stroke-width:2px;
    linkStyle 2 stroke:#9ca3af,stroke-width:2px;
    linkStyle 3 stroke:#9ca3af,stroke-width:2px;
    linkStyle 4 stroke:#9ca3af,stroke-width:2px;
    linkStyle 5 stroke:#9ca3af,stroke-width:2px;
    linkStyle 6 stroke:#9ca3af,stroke-width:2px;
    linkStyle 7 stroke:#9ca3af,stroke-width:2px;
    linkStyle 8 stroke:#9ca3af,stroke-width:2px;
```

---

## 8. Responsabilidades por Arquivo

### `src/modules/auth/model/v1/auth.contracts.ts`

Define:

- `MainApiV1...`
- `AuthenticatedUser`

### `src/modules/auth/infra/clients/auth-api.client.ts`

Responsável por:

- chamar `GET /api/v1/profile`;
- enviar `Authorization: Bearer <token>`;
- retornar o contrato externo da API principal.

### `src/modules/auth/infra/services/auth.service.ts`

Responsável por:

- chamar o client;
- validar o `id` do payload remoto;
- transformar o resultado em `AuthenticatedUser`.

### `src/modules/auth/infra/guards/auth.guard.ts`

Responsável por:

- validar o header;
- extrair o token;
- autenticar;
- anexar `request.user`.

### `src/modules/auth/auth.module.ts`

Responsável por:

- registrar `HttpModule`;
- registrar client, service e guard;
- compor o módulo de auth.

### `src/modules/health/infra/services/health.service.ts`

Responsável por:

- responder o health público;
- responder o health protegido com `userId`.

### `src/modules/health/infra/controllers/health.controller.ts`

Responsável por:

- expor `GET /health/live`;
- expor `GET /health/protected`;
- aplicar `AuthGuard` apenas na rota protegida.

### `src/modules/health/health.module.ts`

Responsável por:

- registrar controller e service de health;
- importar `AuthModule`.

### `src/shared/config/environment.ts`

Responsável por:

- centralizar schema e leitura de env;
- declarar a URL da API principal consumida pela integração.

### `src/app.module.ts`

Responsável por:

- compor `AuthModule` e `HealthModule`.

### `src/main.ts`

Responsável por:

- subir a aplicação com o bootstrap mínimo.

---

## 9. Environment e Configuração

O projeto mantém configuração centralizada em `environment.ts`.

### Regras aplicadas

- não ler `process.env` espalhado no módulo;
- não criar sistema paralelo de config;
- declarar novas variáveis no schema central;
- consumir config pronta no client/bootstrap;
- usar `HttpModule` e `HttpService` para integração HTTP externa.

### Variável adicionada neste slice

```env
MAIN_API_V1_URL=http://localhost:3333
```

> [!NOTE]
> O nome da env foi mantido explícito para identificar claramente a integração com a API principal e seu boundary versionado.

---

## 10. Regras de Simplicidade Aplicadas

Este PR foi ajustado para refletir o feedback técnico recebido e o padrão aprovado do projeto.

### Regras aplicadas

- usar `HttpService` / NestAxios;
- usar `firstValueFrom` para a chamada HTTP;
- não usar `Object.freeze`;
- não criar helpers sem reuso real;
- não criar funções soltas sem ganho concreto;
- não inflar o slice com preparação futura;
- manter guard, service e client simples;
- preservar fidelidade ao contrato externo;
- reduzir apenas o payload local de `request.user`;
- preferir código fácil de explicar e revisar.

> [!IMPORTANT]
> Simplificar não significa empobrecer o contrato da integração.
>
> Simplificar significa reduzir a implementação ao necessário, mantendo fidelidade ao que a API principal realmente entrega e ao que o recorte atual realmente precisa.

---

## 11. Segurança e Falha

### Regras de segurança

- aceitar apenas `Authorization: Bearer <token>`;
- negar ausência de token;
- negar scheme inválido;
- negar token vazio;
- negar payload remoto sem `id` válido;
- nunca trafegar token por query string;
- nunca considerar falha remota como sucesso.

### Matriz mínima de falha

| Situação | Resultado esperado |
|---|---|
| Header ausente | `401 Unauthorized` |
| Header vazio | `401 Unauthorized` |
| Scheme inválido | `401 Unauthorized` |
| Token vazio | `401 Unauthorized` |
| Token inválido na API principal | `401 Unauthorized` |
| Falha remota de auth | bloqueio controlado |
| Payload sem `id` válido | `401 Unauthorized` |

---

## 12. Rotas Técnicas

```http
GET /health/live
GET /health/protected
```

| Endpoint | Proteção | Objetivo |
|---|---|---|
| `GET /health/live` | pública | validar subida da aplicação |
| `GET /health/protected` | protegida | validar auth delegado ponta a ponta |

---

## 13. Critérios de Aceite

### Funcionais

- requests sem token devem ser rejeitadas;
- requests com bearer inválido devem ser rejeitadas;
- requests com token inválido na API principal devem ser rejeitadas;
- requests com payload remoto sem `id` confiável devem ser rejeitadas;
- requests válidas devem resolver `request.user.id`;
- `GET /health/protected` deve responder apenas com autenticação válida.

### Arquiteturais

- a API de IA não cria login próprio;
- a API de IA não emite token;
- a API de IA não acessa Redis do auth principal;
- a API de IA não replica autorização por role;
- a API de IA mantém boundary de integração explícito;
- apenas o payload da request local é reduzido ao mínimo;
- o código permanece simples, direto e aderente ao padrão do projeto.

---

## 14. Conclusão

Este PR implementa a foundation correta do auth da Fase 1:

**consumo remoto da identidade administrativa da API principal, preservação explícita do contrato do endpoint de perfil e resolução segura de `request.user.id` na API de IA.**

### Síntese final

A API principal autentica.  
A API de IA consome a identidade autenticada.  
A API de IA preserva o contrato externo do endpoint de perfil.  
A API de IA reduz apenas o payload anexado à request.  
A API de IA protege a borda HTTP local.
