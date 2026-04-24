# 🤖 PR 77 — Fase 2: Consolidação Funcional da Alternativa Correta

## Fortalecimento do uso de `correctAnswer` na composição final do fluxo avançado

---

![PR](https://img.shields.io/badge/PR-77-2563eb?style=for-the-badge\&logo=gitpullrequest\&logoColor=white) ![Tipo](https://img.shields.io/badge/Tipo-nestjs%20slice-e11d48?style=for-the-badge\&logo=nestjs\&logoColor=white) ![Fase](https://img.shields.io/badge/Fase-2-f59e0b?style=for-the-badge\&logo=dependabot\&logoColor=white) ![Escopo](https://img.shields.io/badge/Escopo-serverless-059669?style=for-the-badge\&logo=serverless\&logoColor=white) ![Status](https://img.shields.io/badge/Status-ready-111827?style=for-the-badge\&logo=githubactions\&logoColor=white)

> [!IMPORTANT]
> Esta PR reposiciona a evolução da fase avançada para um eixo funcional: integridade da alternativa correta no resultado final. O foco sai de refinamentos contextuais anteriores e entra na previsibilidade de `answerKey.correctAlternative`, preservando o recorte incremental e a arquitetura vigente.

## 1. Síntese Executiva

O pipeline já recebe `correctAnswer` no input e o propaga entre etapas intermediárias. Entretanto, a consolidação desse valor no fechamento do `answerKey` ainda pode ser tornada mais explícita e resiliente.

Esta PR fortalece a prioridade funcional de `correctAnswer` durante a composição final do resultado processado, reduzindo dependência de fallback implícito e elevando consistência sem expansão arquitetural.

## 2. Objetivo do PR

Consolidar o uso de `correctAnswer` como fonte prioritária para preenchimento de `answerKey.correctAlternative`, mantendo compatibilidade com o fluxo atual e previsibilidade operacional.

## 3. Decisão Arquitetural

A responsabilidade permanece distribuída entre os componentes já existentes. Não há criação de novos agents, módulos, camadas ou contratos paralelos.

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#22d3ee','lineColor':'#64748b','secondaryColor':'#111827','tertiaryColor':'#1e293b','background':'#020617'}}}%%
flowchart LR
    A[Input.question.correctAnswer] --> B[InitialQuestionProcessingAgent]
    B --> C[AnswerKeyAgent]
    C --> D[Output.answerKey.correctAlternative]
```

Diretrizes aplicadas:

* recorte pequeno
* prioridade explícita da alternativa correta
* fallback controlado quando ausente
* preservação do contrato final
* zero sobreengenharia

## 4. Escopo da PR

### Incluído

* revisar uso de `correctAnswer` no fluxo avançado
* explicitar prioridade de preenchimento para `correctAlternative`
* garantir consistência entre valor intermediário e output final
* reforçar cenários com presença e ausência da alternativa correta
* reduzir inferência implícita no fechamento do answer key

### Fora de Escopo

* inferência automática por IA
* validação semântica entre alternativas e justificativa
* novos agents
* redesign do orchestrator
* score de confiança
* mudanças amplas de contrato

## 5. Fluxo Arquitetural

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#0f172a','primaryTextColor':'#e5e7eb','primaryBorderColor':'#a78bfa','lineColor':'#64748b','secondaryColor':'#111827','tertiaryColor':'#1e293b','background':'#020617'}}}%%
sequenceDiagram
    participant O as Orchestrator
    participant I as InitialProcessing
    participant A as AnswerKeyAgent

    O->>I: process(question)
    I-->>O: base.correctAlternative
    O->>A: compose(answerKey)
    A-->>O: correctAlternative consolidada
```

## 6. Contratos Mínimos

Sem alteração estrutural obrigatória no output final.

```ts
{
  answerKey: {
    correctAlternative,
    justification,
    source
  }
}
```

A evolução ocorre na regra de priorização e consistência de preenchimento do campo.

## 7. Estratégia de Implementação

* preferir ajustes locais nos services/agents existentes
* evitar abstrações prematuras
* manter compatibilidade com specs atuais
* tratar ausência de `correctAnswer` de forma explícita
* manter regras de fallback legíveis e testáveis

## 8. Critérios de Review

* a alternativa correta ficou mais previsível no fluxo?
* houve redução de fallback implícito?
* o output final permaneceu estável?
* o recorte permaneceu pequeno?
* a solução evitou expansão desnecessária?

## 9. Critérios de Aceite

* `correctAlternative` consolidada corretamente
* cenários com e sem `correctAnswer` cobertos
* testes verdes
* nenhuma regressão no orchestrator
* output final consistente

## 10. Impacto Esperado

| Vetor        | Resultado                            |
| ------------ | ------------------------------------ |
| Consistência | Maior previsibilidade do campo final |
| Manutenção   | Regras mais claras e localizadas     |
| Risco        | Baixo, sem mudança estrutural        |
| Evolução     | Base funcional mais sólida           |

## 11. Conclusão

A PR 77 fortalece um ponto funcional crítico do pipeline avançado: a integridade da resposta correta no resultado final. O ganho principal é previsibilidade prática com baixo risco e sem repetir ciclos anteriores de refinamento contextual.
