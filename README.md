# FIAP Bank – LedgerSync: Plataforma de Reconciliação Financeira

> **Disciplina:** IT Architecture Design-Styles – C4 Model & Engenharia de Software  
> **Professor:** Leonardo Pinho  
> **Tema:** Open Banking & Arquitetura Orientada a Eventos  

## 1. Story Telling – O Problema e o Tema

Todo mês, o departamento de tesouraria do **FIAP Bank** enfrenta um obstáculo recorrente: o fechamento do caixa. O banco opera com investimentos internacionais e movimenta milhares de transações entre Brasil e Estados Unidos — câmbio, transferências, taxas e estornos. Contudo, quando chega o momento de conciliar o saldo da **ledger** (livro-razão interno) com o saldo real das contas mantidas nos bancos parceiros americanos, o processo se revela frágil e excessivamente manual.

Atualmente, os analistas exportam planilhas de sistemas distintos e passam horas cruzando dados. Qualquer divergência, por menor que seja, paralisa o fechamento contábil. A raiz do problema está na natureza assimétrica das fontes: a ledger opera em lotes diários, enquanto as contas bancárias oscilam em tempo real com liquidações, estornos e tarifas. Conciliar o estático com o dinâmico sem uma camada de automação robusta é ineficiente e arriscado.

O tema escolhido para este trabalho é **Open Banking & Arquitetura Orientada a Eventos com Data Streaming**, um dos temas sugeridos na Aula 1 da disciplina, alinhado aos valores de Pioneirismo e Tecnologia do FIAP Bank. A solução proposta — **LedgerSync** — consiste em uma plataforma que captura eventos transacionais em tempo real oriundos dos bancos parceiros, concilia automaticamente com a ledger interna e notifica imediatamente qualquer divergência, eliminando a dependência do fechamento mensal para identificar problemas.

## 2. O Que Esperamos Aprender com Este Projeto

O trabalho nos permite aplicar, na prática, os conceitos estudados ao longo das quatro aulas da disciplina:

- **Aula 1 (Fundamentos):** aplicar Design Thinking e técnicas de antecipação de mudanças em uma estratégia de design, definindo requisitos e escolhendo a arquitetura ideal para o problema.
- **Aula 2 (Partes Interessadas e ASR):** praticar o mapeamento de stakeholders, identificar o que eles esperam ganhar e eleger os Requisitos Arquitetonicamente Significativos (ASR) que guiam as decisões de design.
- **Aula 3 (Notação C4 e Padrões):** exercitar a notação C4 nos três níveis (Contexto, Contêiner, Componente) e relacionar a arquitetura com padrões estudados: Camadas (Layered), Portas e Adaptadores, Pipe and Filter e Arquitetura Orientada a Serviços.
- **Aula 4 (Avaliação):** submeter a arquitetura a uma avaliação criteriosa — o "boletim escolar" — verificando se os diagramas respondem às perguntas certas para o público certo.

## 3. Perguntas Que Precisam Ser Respondidas

1. Como capturar eventos transacionais em tempo real de múltiplos bancos parceiros, cada qual com seu formato proprietário?
2. Como modelar o estado da ledger para que reflita o saldo real com a menor latência possível?
3. Qual estratégia de reconciliação adotar: processamento em lote, em streaming ou uma abordagem híbrida?
4. Como assegurar idempotência no reprocessamento de eventos financeiros, evitando duplicidades que comprometam os saldos?
5. Como tratar as divergências que o sistema não consegue resolver de forma automatizada?

## 4. Principais Riscos

| Risco | Impacto | Probabilidade |
|---|---|---|
| Inconsistência entre ledger e saldo real por eventos duplicados | Alto | Média |
| Latência excessiva no pipeline de eventos, atrasando alertas | Alto | Média |
| Heterogeneidade nos formatos de eventos enviados pelos bancos parceiros | Médio | Alta |
| Resistência da equipe de tesouraria em abandonar o processo manual | Médio | Alta |
| Vazamento de dados financeiros sensíveis | Crítico | Baixa |
| Sobrecarga da plataforma nos picos de fechamento mensal | Alto | Média |

## 5. Plano Para Aprender o Necessário

| O que aprender | Como | Prazo |
|---|---|---|
| Padrões de arquitetura orientada a eventos (Event Sourcing, CQRS, Saga) — complementando os padrões vistos na Aula 3 | Estudo dirigido e prova de conceito | Semana 1-2 |
| Integração com APIs de Open Banking (Bacen, parceiros EUA) | Documentação oficial e ambiente sandbox | Semana 2-3 |
| Apache Kafka e Kafka Streams para processamento de streams | Laboratório prático | Semana 2-3 |
| Modelagem de domínio contábil (partidas dobradas) | Consultoria com especialista contábil | Semana 1 |
| Comparação entre reconciliação batch e streaming | Spike técnico comparativo | Semana 3 |

## 6. Plano Para Reduzir os Riscos

| Risco | Mitigação |
|---|---|
| Eventos duplicados | Idempotência via `idempotency-key` e deduplicação no broker de mensageria |
| Latência no pipeline | Particionamento por conta no Kafka e consumidores paralelos escaláveis |
| Formatos heterogêneos | Camada de normalização (Anti-Corruption Layer) com Schema Registry centralizado |
| Resistência da tesouraria | Interface simplificada e período de operação assistida (shadow mode) |
| Vazamento de dados | Criptografia TLS 1.3 em trânsito e AES-256 em repouso; anonimização em ambientes de desenvolvimento |
| Pico de carga no fechamento | Auto-scaling dos consumidores no Kubernetes; dimensionamento elástico |

## 7. Partes Interessadas (Stakeholders)

Seguindo a técnica de mapeamento de partes interessadas apresentada na Aula 2 — identificar quem está pagando pelo software, quem vai usá-lo, quais hubs concentram mais setas de entrada ou saída e se existem conflitos de interesse — chegamos ao seguinte mapa:

**Quem paga pelo projeto:**
- **Diretoria Financeira / CFO** — patrocinadora e dona do processo de fechamento contábil. É a principal interessada na redução de custos operacionais e no encurtamento do ciclo de fechamento.

**Quem usa o sistema:**
- **Tesouraria** (Analistas e Gerentes) — executam e aprovam a conciliação. São os mais impactados pela mudança do processo manual para o automatizado.
- **Auditoria Interna** — consome os dados de rastreabilidade para fins de conformidade.

**Hubs de integração (muitas setas de entrada/saída):**
- **Bancos Parceiros (EUA)** — provedores dos dados transacionais. A plataforma depende da qualidade e da pontualidade dos eventos que eles enviam.
- **ERP Contábil** — destino final dos lançamentos conciliados.

**Potenciais conflitos de interesse:**
- **Reguladores (Bacen, SIPC)** — seus interesses (conformidade estrita, auditoria completa) podem conflitar com os da Tesouraria (agilidade, simplicidade). A arquitetura precisa equilibrar ambos.
- **Tesouraria × TI** — a tesouraria deseja uma ferramenta simples e familiar; a TI propõe uma stack moderna que exige adaptação cultural.

## 8. O Que Cada Stakeholder Espera Obter

| Stakeholder | Expectativa |
|---|---|
| CFO | Fechamento contábil em horas, redução de erros operacionais e corte de custos |
| Tesouraria (Analistas) | Fim do cruzamento manual de planilhas; detecção de divergências em tempo real |
| Tesouraria (Gerentes) | Aprovação do fechamento com segurança; visão consolidada da posição financeira |
| TI / Engenharia | Arquitetura moderna, escalável e com baixa sobrecarga operacional |
| Compliance / Auditoria | Trilha de auditoria imutável e rastreabilidade de ponta a ponta |
| Bancos Parceiros | Integração padronizada, com menos chamados por divergência |
| Reguladores (Bacen, SIPC) | Garantia de que a posição financeira reportada é fidedigna e auditável |

## 9. Quem São os Usuários

- **Analistas de Tesouraria** — executam a conciliação diária dos saldos.
- **Gerentes de Tesouraria** — aprovam o fechamento mensal e investigam divergências críticas.
- **Auditores Internos** — consultam o histórico de reconciliações para fins de auditoria.
- **Sistemas Externos** — APIs dos bancos parceiros e o ERP contábil da instituição (usuários não humanos, mas que precisam ser considerados como atores no C4).

## 10. O Que Cada Usuário Deseja Realizar

- **Analistas:** Verificar se o saldo da ledger coincide com o saldo bancário; classificar divergências; resolver pendências.
- **Gerentes:** Aprovar o fechamento com segurança; obter visão consolidada da posição financeira.
- **Auditores:** Rastrear cada lançamento desde a origem bancária até o registro contábil final, sem lacunas.
- **Sistemas Externos:** Enviar e receber eventos transacionais de forma confiável e auditável.

## 11. Pior Cenário Possível

Uma divergência não detectada entre a ledger e o saldo real levar o FIAP Bank a reportar incorretamente sua posição financeira aos reguladores (Bacen e SIPC). As consequências incluiriam multas de valor elevado, risco de perda da licença operacional e dano irreparável à reputação da instituição. Considerando que a confiança é um dos valores fundamentais do banco, as repercussões seriam catastróficas.

## 12. Diagrama de Arquitetura — Modelo Freeform (Versão Inicial)

O diagrama abaixo representa o primeiro esboço da arquitetura, elaborado durante as discussões iniciais da equipe:

```mermaid
flowchart TB
    subgraph Bancos_Parceiros["Bancos Parceiros (EUA)"]
        B1[Sistema Bancário EUA 1]
        B2[Sistema Bancário EUA 2]
        B3[Sistema Bancário EUA N]
    end

    subgraph LedgerSync["Plataforma LedgerSync"]
        direction TB
        ING["Ingestion\n(REST/Webhook/SFTP)"]
        NORM["Normalization Engine\n(Schema Registry)"]
        EVB["Event Bus\n(Apache Kafka)"]
        REC["Reconciliation Engine\n(Kafka Streams)"]
        LED["Ledger Service\n(Internal DB)"]
        ALERT["Alert & Exception Service"]
        DASH["Dashboard / API"]
    end

    subgraph Fora["Usuários e Sistemas"]
        ERP["ERP Contábil"]
        UI["Analistas de Tesouraria"]
        AUD["Auditoria"]
    end

    B1 & B2 & B3 -->|"transações"| ING
    ING --> NORM
    NORM --> EVB
    EVB --> REC
    EVB --> LED
    REC -->|"divergências"| ALERT
    REC -->|"saldo conciliado"| LED
    LED --> DASH
    ALERT --> DASH
    DASH --> UI
    DASH --> AUD
    LED -->|"export"| ERP
```

## 13. Descrição de Cada Componente

| Componente | Responsabilidade |
|---|---|
| **Sistemas Bancários (EUA)** | Fontes externas de verdade que detêm os saldos reais das contas. Enviam eventos transacionais (débito, crédito, estorno, taxa) em formatos proprietários. |
| **Ingestion** | Porta de entrada da plataforma. Recebe dados via REST, Webhook ou SFTP. Realiza a autenticação, validação e encaminhamento de cada evento. |
| **Normalization Engine** | Camada anti-corrupção que converte os formatos heterogêneos dos bancos parceiros para um schema canônico interno. Utiliza Schema Registry para versionamento. |
| **Event Bus (Apache Kafka)** | Espinha dorsal assíncrona da plataforma. Oferece durabilidade, ordenação por conta/partição e capacidade de replay de eventos. Permite múltiplos consumidores independentes. |
| **Reconciliation Engine** | Núcleo da plataforma. Consome eventos externos e eventos da ledger, comparando-os dentro de janelas temporais configuráveis. Identifica matches, breaks e exceções. |
| **Ledger Service** | Mantém o livro-razão interno do FIAP Bank no modelo de partidas dobradas. Cada lançamento é registrado de forma imutável, servindo como fonte de verdade para a posição financeira. |
| **Alert & Exception Service** | Notifica a equipe de tesouraria sobre divergências não resolvidas automaticamente. Cria tickets de investigação e escala conforme a criticidade. |
| **Dashboard / API** | Interface para analistas e gerentes visualizarem o status da conciliação, relatórios e divergências. Disponibiliza API para integração com o ERP contábil. |

## 14. Requisitos Arquitetonicamente Significativos (ASR)

Seguindo a classificação de ASR apresentada na Aula 2, organizamos os requisitos que mais influenciam as decisões arquiteturais em quatro frentes: restrições, atributos de qualidade, requisitos funcionais influentes e outros fatores.

### 14.1 Restrições Técnicas e de Negócio

| Tipo | Restrição | Impacto no Design |
|---|---|---|
| Negócio | A plataforma deve estar operacional antes do próximo ciclo de fechamento trimestral | Força a escolha de componentes maduros e equipe com familiaridade nas tecnologias |
| Técnica | Deve ser executada em ambiente cloud (AWS), por política corporativa do FIAP Bank | Determina a escolha de serviços gerenciados (EKS, MSK) e infraestrutura como código |
| Negócio | A equipe de engenharia disponível tem experiência em Kotlin e Python, não em Elixir ou Rust | Restringe as linguagens de implementação dos serviços |
| Técnica | Os bancos parceiros se comunicam via REST, Webhook ou SFTP; não é viável impor um protocolo único | Exige uma camada de ingestion flexível e adaptadores por protocolo |

### 14.2 Atributos de Qualidade (Propriedades de Runtime)

| Atributo | Cenário | Resposta Arquitetural |
|---|---|---|
| **Confiabilidade (Reliability)** | Um banco parceiro fica indisponível por 2 horas | Dead-letter queue no Kafka; retry com backoff exponencial; nenhum evento é perdido |
| **Desempenho (Performance)** | Pico de 50 mil transações na hora do fechamento mensal | Particionamento por conta; consumidores em paralelo com auto-scaling no Kubernetes |
| **Segurança (Security)** | Dados de saldo e transação trafegam entre componentes | TLS 1.3 em trânsito; AES-256 em repouso; anonimização em ambientes de desenvolvimento |
| **Disponibilidade (Availability)** | O Reconciliation Engine falha durante uma janela de conciliação | Kafka Streams com state store em RocksDB permite reprocessamento do ponto de falha |
| **Modificabilidade (Modifiability)** | Um novo banco parceiro é integrado com formato de eventos diferente | Anti-Corruption Layer com adapter por banco; Schema Registry versionado |

### 14.3 Requisitos Funcionais com Impacto Arquitetural

| Requisito | Por que influencia a arquitetura |
|---|---|
| Idempotência no processamento de eventos | Exige `idempotency-key` e deduplicação em todas as camadas de consumo |
| Rastreabilidade ponta a ponta | Impõe que cada evento carregue um `correlationId` preservado do Ingestion até o ERP |
| Janela de conciliação configurável por banco | O Reconciliation Engine precisa de state store com TTL variável por parceiro |
| Conciliação manual assistida para exceções | O Dashboard precisa de uma tela de "match sugerido" com capacidade de override pelo analista |

### 14.4 Outros Influenciadores

- **Tempo:** o prazo de entrega do MVP é de 3 meses, exigindo escopo enxuto e reuso de componentes.
- **Conhecimento da equipe:** a familiaridade com PostgreSQL e Kafka direcionou as escolhas de persistência e mensageria.
- **Preconceitos arquiteturais:** a preferência por Event Sourcing veio de experiências anteriores positivas da equipe, mas foi validada contra os requisitos reais do problema.

## 15. Sobre o Que o Diagrama Ajuda a Raciocinar

O diagrama Freeform orienta a reflexão sobre quatro pontos centrais:

- **Separação de responsabilidades:** cada componente possui uma função bem definida — ingestão, normalização, conciliação, persistência e notificação. Isto materializa o princípio de coesão funcional discutido no padrão de Camadas (Aula 3).
- **Desacoplamento via Event Bus:** o Kafka atua como ponto central de integração. Nenhum componente se comunica diretamente com outro, o que permite evolução independente de cada módulo.
- **Duas fontes de verdade:** a ledger interna e os sistemas bancários externos são duas realidades que precisam ser confrontadas. O sistema não presume uma verdade absoluta única.
- **Divergência como evento de negócio:** divergências não são tratadas como erros de sistema, mas como ocorrências esperadas que exigem um workflow próprio de tratamento.

## 16. Padrões Essenciais no Diagrama — Relação com os Padrões da Aula 3

A Aula 3 apresentou quatro padrões fundamentais de arquitetura. Abaixo, mostramos como cada um deles se manifesta na plataforma LedgerSync:

| Padrão da Aula 3 | Como aparece no LedgerSync |
|---|---|
| **Camadas (Layered)** | A arquitetura organiza-se em camadas lógicas: Ingestion (entrada), Normalization + Event Bus (transformação), Reconciliation + Ledger (domínio), Dashboard (apresentação). As camadas superiores dependem das inferiores, sem dependências cíclicas. |
| **Portas e Adaptadores (Ports & Adapters)** | O Normalization Engine atua como camada de domínio que expõe portas (schemas canônicos). Cada banco parceiro é tratado como um adaptador de entrada que implementa a interface da porta. O ERP é um adaptador de saída. |
| **Pipe and Filter** | O fluxo Ingestion → Normalization → Kafka → Reconciliation → Ledger forma um pipeline onde cada estágio é um filtro que transforma os dados. Os pipes (tópicos Kafka) transportam os eventos preservando ordem. |
| **Arquitetura Orientada a Serviços (SOA)** | Cada container (Ingestion Service, Reconciliation Engine, Ledger Service, Alert Service) é um serviço independente, implantável separadamente, que se comunica via mensageria assíncrona (pub/sub no Kafka), seguindo o princípio de "endpoints inteligentes, tubos burros". |

Além dos padrões da aula, identificamos dois padrões complementares que emergem naturalmente da solução:

| Padrão Complementar | Ocorrência |
|---|---|
| **Event Sourcing** | A ledger armazena a sequência imutável de eventos em vez de um estado de saldo corrente. Permite reconstruir a posição financeira em qualquer instante e atende diretamente ao requisito de auditoria. |
| **CQRS (implícito)** | O Reconciliation Engine escreve os resultados de conciliação; o Dashboard lê as projeções materializadas do Ledger Service. Caminhos de escrita e leitura são segregados, otimizados para suas respectivas cargas. |

## 17. Padrões Ocultos

Uma análise mais aprofundada revela padrões que não estão explicitamente representados no diagrama, mas que são necessários para o funcionamento correto da plataforma:

- **Outbox Pattern:** o Ledger Service precisa assegurar atomicidade entre a escrita no banco de dados e a publicação do evento no Kafka. Se uma das operações falhar sem a outra, o sistema torna-se inconsistente.
- **Saga (coreografada):** o fluxo de reconciliação pode ser interpretado como uma saga de múltiplos passos (receber evento, normalizar, comparar, decidir match/break, persistir, notificar). A falha em qualquer etapa exige compensação.
- **Strangler Fig:** a nova plataforma não substituirá o processo manual abruptamente. Haverá um período de shadow mode, durante o qual o sistema novo opera em paralelo ao legado, permitindo validação gradual.

## 18. Metamodelo

O metamodelo define os conceitos fundamentais do domínio e seus relacionamentos:

```
Cliente (1) ─── (N) Conta ─── (1) Banco Parceiro
                           │
                           └── (N) Transação (externa)
                                            │
Ledger (1) ─── (N) Lançamento Contábil ────┘ (match 0..1)
                                            │
Reconciliação (N) ─── (2) [Transação + Lançamento]
                                            │
Divergência (0..N) ─── (1) Reconciliação
```

**Entidades principais:**

- **Conta:** conta bancária real mantida em um banco parceiro.
- **Transação (externa):** evento proveniente do banco parceiro (débito, crédito, taxa, estorno).
- **Lançamento Contábil:** registro na ledger interna, obedecendo ao modelo de partidas dobradas.
- **Reconciliação:** pareamento entre uma ou mais transações externas e seus correspondentes lançamentos contábeis.
- **Divergência:** situação em que não é possível estabelecer um pareamento (lançamento órfão, transação sem correspondência, diferença de valor).

## 19. O Metamodelo Pode Ser Discernido no Diagrama Único?

Apenas parcialmente. O diagrama Freeform representa a estrutura de componentes e o fluxo de dados, mas o metamodelo — entidades, relacionamentos e cardinalidades — não está explícito. Seria necessário um diagrama de classes ou entidade-relacionamento complementar. O C4 Model endereça essa limitação ao separar os níveis de abstração: o Contexto mostra atores e sistemas; o Contêiner revela a estrutura técnica; o Componente expõe como as entidades do metamodelo circulam dentro de cada serviço.

## 20. O Diagrama Está Completo?

Não. O diagrama Freeform cobre o fluxo principal, mas omite diversos aspectos relevantes:

- Mecanismos de resiliência: dead-letter queues, políticas de retry, circuit breakers.
- Infraestrutura transversal: observabilidade (logs, métricas, tracing distribuído), autenticação e autorização.
- Processos batch complementares: o fechamento mensal ainda pode exigir um job de consolidação.
- Período de shadow mode: fase de validação em que o sistema opera em paralelo ao legado.

## 21. O Diagrama Poderia Ser Simplificado Mantendo Sua Eficácia?

Sim. Para um público executivo (CFO, Diretoria), uma representação reduzida seria suficiente:

> **Bancos Parceiros → LedgerSync → Tesouraria / ERP**

Esse é justamente o nível de Contexto do C4 Model. O detalhamento interno (Kafka, Schema Registry, Reconciliation Engine) só é relevante para o público técnico. A virtude do C4 Model está exatamente nisto: cada nível de zoom é adequado a um público distinto.

## 22. Discussões Relevantes da Equipe

Três debates foram particularmente produtivos durante a elaboração do projeto:

1. **Batch versus streaming para a reconciliação:** discutimos se o motor de conciliação deveria operar como um job noturno (mais simples e previsível) ou como um pipeline de streaming contínuo (mais rápido, porém mais complexo). Optamos por uma abordagem híbrida: streaming para alertas em tempo real e batch para o fechamento oficial mensal.

2. **Modelo de persistência da ledger:** Event Sourcing com consistência eventual versus CRUD tradicional com transações ACID. Escolhemos Event Sourcing porque o histórico imutável atende diretamente aos requisitos de auditoria. Adicionamos projeções materializadas para evitar a recomputação completa em cada consulta.

3. **Responsabilidade pelo domínio da reconciliação:** a reconciliação pertence ao bounded context da Ledger ou constitui um domínio separado? Decidimos segregá-los: a Ledger é a fonte de verdade; a Reconciliation é o processo que compara fontes distintas.

## 23. Decisões Que a Equipe Teve Dificuldade Para Tomar

1. **Escolha do broker de mensageria:** Apache Kafka (maior carga operacional) ou uma solução gerenciada como Amazon SQS/SNS (menor complexidade de infraestrutura). Kafka prevaleceu devido à capacidade de replay de eventos e à ordenação por partição — características críticas para eventos financeiros.

2. **Armazenamento da ledger:** PostgreSQL com tabela de eventos versus EventStoreDB (banco especializado em Event Sourcing). O PostgreSQL foi escolhido por familiaridade da equipe e pela maturidade do ecossistema de ferramentas disponíveis.

3. **Linguagem do Reconciliation Engine:** Python (prototipagem rápida, adequação ao perfil de equipes de dados) versus Kotlin (performance superior e integração nativa com Kafka Streams). Kotlin foi selecionado por oferecer maior robustez em ambientes de produção.

## 24. Decisões Tomadas Sob Incerteza

1. **Volume real de transações do FIAP Bank:** sem dados históricos precisos, superdimensionamos o particionamento do Kafka (100 partições) e projetamos auto-scaling desde a concepção inicial.

2. **Adoção de blockchain:** embora mencionada nas informações do case, optamos por prototipar sem blockchain. O Event Sourcing já provê rastreabilidade imutável. Caso surja futuramente a necessidade de compartilhar trilhas de auditoria com entidades externas de forma descentralizada, um ledger distribuído (DLT) poderá ser acoplado como camada complementar.

3. **Integração com bancos parceiros:** desconhecemos o número exato de bancos e seus protocolos. A camada de Anti-Corruption foi projetada como extensível (adapter por banco), mas o esforço real de integração permanece como incógnita.

## 25. Ponto de Decisão Sem Retorno

A adoção de **Event Sourcing como padrão de persistência da ledger** constitui um ponto de não retorno arquitetural. Uma vez que os eventos se tornam a fonte primária de verdade:

- Migrar para um modelo CRUD convencional exigiria reescrever integralmente a camada de domínio.
- Todas as consultas, relatórios e integrações passam a depender das projeções materializadas.
- O custo de reverter essa decisão após a entrada em produção seria proibitivo.

Em contrapartida, essa decisão desbloqueia capacidades valiosas: replay completo do histórico transacional, trilha de auditoria imutável e flexibilidade para responder a novas perguntas de negócio simplesmente criando novas projeções sobre o mesmo stream de eventos.

## 26 a 29. Arquitetura C4 Model — Três Camadas

### 26-27. Nível 1: Diagrama de Contexto

```mermaid
C4Context
    title FIAP Bank – LedgerSync (Diagrama de Contexto)

    Person(analista, "Analista de Tesouraria", "Concilia saldos e investiga divergências")
    Person(gerente, "Gerente de Tesouraria", "Aprova fechamento mensal")
    Person(auditor, "Auditor Interno", "Verifica rastreabilidade")
    
    System_Ext(banco_parceiro, "Bancos Parceiros (EUA)", "Fornecem transações e saldos reais")
    System_Ext(erp, "ERP Contábil", "Sistema contábil corporativo")
    
    System_Boundary(ledgersync, "Plataforma LedgerSync") {
        System(plataforma, "LedgerSync", "Ingere eventos, concilia ledger com bancos e emite alertas de divergência")
    }

    Rel(analista, plataforma, "Visualiza conciliações, resolve pendências", "HTTPS/Web")
    Rel(gerente, plataforma, "Aprova fechamento, consulta relatórios", "HTTPS/Web")
    Rel(auditor, plataforma, "Consulta trilha de auditoria", "HTTPS/Web")
    Rel(banco_parceiro, plataforma, "Envia eventos de transação", "REST/Webhook/SFTP")
    Rel(plataforma, erp, "Exporta lançamentos conciliados", "REST/CSV")
```

### 28. Nível 2: Diagrama de Contêiner

```mermaid
C4Container
    title FIAP Bank – LedgerSync (Diagrama de Contêiner)

    Person(analista, "Analista de Tesouraria")
    Person(gerente, "Gerente de Tesouraria")
    Person(auditor, "Auditor Interno")

    System_Ext(banco_parceiro, "Bancos Parceiros (EUA)")
    System_Ext(erp, "ERP Contábil")

    System_Boundary(ledgersync, "Plataforma LedgerSync") {
        Container(webapp, "Dashboard SPA", "React + TypeScript", "Interface para analistas, gerentes e auditores")
        Container(api, "API Gateway", "Kotlin + Spring Boot", "Autenticação, rate limiting e roteamento")
        Container(ingestion, "Ingestion Service", "Go", "Recebe eventos dos bancos parceiros (REST/Webhook/SFTP)")
        Container(normalizer, "Normalization Service", "Kotlin", "Converte formatos proprietários para o schema canônico")
        Container(kafka, "Event Bus", "Apache Kafka", "Backbone assíncrono com particionamento por conta")
        Container(recon_engine, "Reconciliation Engine", "Kotlin + Kafka Streams", "Compara eventos externos com a ledger, detecta divergências")
        Container(ledger_svc, "Ledger Service", "Kotlin + Spring Boot", "Mantém o livro-razão interno (Event Sourcing)")
        Container(alert_svc, "Alert & Exception Service", "Kotlin", "Notifica divergências e gerencia workflow de investigação")
        ContainerDb(ledger_db, "Ledger DB", "PostgreSQL", "Event store e projeções materializadas")
        ContainerDb(cache, "Cache", "Redis", "Saldos recentes, sessões e rate limiting")
        Container(schema_reg, "Schema Registry", "Confluent", "Versionamento de schemas de eventos")
    }

    Rel(analista, webapp, "Usa", "HTTPS")
    Rel(gerente, webapp, "Usa", "HTTPS")
    Rel(auditor, webapp, "Usa", "HTTPS")
    Rel(webapp, api, "Chama API", "REST/HTTPS")
    
    Rel(banco_parceiro, ingestion, "Envia transações", "REST/Webhook/SFTP")
    Rel(ingestion, kafka, "Publica evento bruto", "Kafka")
    Rel(kafka, normalizer, "Consome evento bruto", "Kafka")
    Rel(normalizer, kafka, "Publica evento normalizado", "Kafka")
    Rel(normalizer, schema_reg, "Valida schema", "REST")
    
    Rel(kafka, recon_engine, "Consome eventos normalizados", "Kafka Streams")
    Rel(kafka, ledger_svc, "Consome eventos normalizados", "Kafka")
    Rel(ledger_svc, ledger_db, "Lê e escreve eventos", "JDBC")
    Rel(recon_engine, ledger_db, "Consulta projeções da ledger", "JDBC")
    Rel(recon_engine, kafka, "Publica Match/Break", "Kafka")
    Rel(kafka, alert_svc, "Consome Break", "Kafka")
    Rel(alert_svc, kafka, "Publica alerta processado", "Kafka")
    
    Rel(api, kafka, "Consome status", "Kafka")
    Rel(ledger_svc, erp, "Exporta conciliado", "REST")
```

### 29. Nível 3: Diagrama de Componente — Reconciliation Engine

```mermaid
C4Component
    title Reconciliation Engine – Diagrama de Componente

    Container_Boundary(recon_engine, "Reconciliation Engine") {
        Component(matcher, "Transaction Matcher", "Kafka Streams Topology", "Pareia transações externas com lançamentos da ledger usando janela temporal e regras configuráveis")
        Component(break_detector, "Break Detector", "Kafka Streams Topology", "Detecta transações sem match, lançamentos órfãos e divergências de valor")
        Component(window_mgr, "Window Manager", "State Store", "Gerencia janelas de conciliação por banco (T+0, T+1, T+2)")
        Component(tolerance_engine, "Tolerance Engine", "Kafka Streams Processor", "Aplica tolerâncias (ex.: ignora diferenças inferiores a R$ 1,00)")
        Component(match_store, "Match Store", "RocksDB", "Estado dos matches na janela ativa")
        Component(recon_repo, "Reconciliation Repository", "Kotlin", "Persiste resultados finais no PostgreSQL")
        Component(status_emitter, "Status Emitter", "Kotlin", "Publica eventos MATCHED/BREAK/PENDING no Kafka")
        Component(config_api, "Configuration API", "REST Controller", "Permite ajustar regras e tolerâncias em runtime")
    }

    ContainerDb(ledger_db, "Ledger DB", "PostgreSQL")
    Container(kafka, "Event Bus", "Apache Kafka")
    Container(api_gateway, "API Gateway")

    Rel(kafka, matcher, "normalized.events + ledger.entries")
    Rel(matcher, window_mgr, "Usa")
    Rel(matcher, match_store, "Estado")
    Rel(matcher, break_detector, "Não pareados")
    Rel(break_detector, tolerance_engine, "Tolerância")
    Rel(tolerance_engine, recon_repo, "Persiste")
    Rel(recon_repo, ledger_db, "Escreve", "JDBC")
    Rel(recon_repo, status_emitter, "Notifica")
    Rel(status_emitter, kafka, "reconciliation.status", "Kafka")
    Rel(api_gateway, config_api, "Chama", "REST")
    Rel(config_api, tolerance_engine, "Atualiza regras")
```

### 30. Nível 4: Código (Opcional)

Estrutura de pacotes sugerida para o **Ledger Service** (Kotlin + Spring Boot):

```
com.fiapbank.ledgersync
├── ledger
│   ├── domain
│   │   ├── LedgerEntry.kt          // entidade: lançamento contábil
│   │   ├── AccountId.kt            // value object
│   │   ├── Money.kt                // value object: valor + moeda
│   │   ├── EntryType.kt            // enum: DEBIT | CREDIT
│   │   └── LedgerAggregate.kt      // aggregate root
│   ├── events
│   │   ├── EntryRecorded.kt
│   │   ├── ReconciliationMatched.kt
│   │   └── ReconciliationBroke.kt
│   ├── infrastructure
│   │   ├── EventStore.kt
│   │   ├── Projection.kt
│   │   └── KafkaPublisher.kt       // outbox pattern
│   └── application
│       ├── RecordEntryUseCase.kt
│       ├── GetBalanceQuery.kt
│       └── GetAuditTrailQuery.kt
```

## 31. Avaliação da Arquitetura

Seguindo a abordagem de "boletim escolar" apresentada na Aula 4, submetemos a arquitetura a uma avaliação baseada em perguntas específicas e acionáveis, conforme a pirâmide de avaliação (custo versus valor):

| Dimensão avaliada | Pergunta | Resultado |
|---|---|---|
| Atributo de qualidade: Confiabilidade | Se um banco parceiro falhar por 2 horas, quantos eventos são perdidos? | Nenhum. Dead-letter queue + retry garantem entrega. |
| Atributo de qualidade: Desempenho | O pipeline suporta 50 mil transações/hora com latência inferior a 30 segundos? | Sim, com particionamento por conta e consumidores paralelos. Teste de carga necessário para confirmar. |
| Atributo de qualidade: Segurança | Os dados em repouso e em trânsito estão criptografados? | Sim. TLS 1.3 + AES-256. Dados anonimizados em dev. |
| Escolha tecnológica | A stack escolhida (Kotlin, Kafka, PostgreSQL) é dominada pela equipe? | Parcialmente. Treinamento em Kafka Streams está no plano de aprendizado (Semana 2-3). |
| Completude da descrição | Os diagramas respondem às perguntas dos diferentes públicos? | Sim. O Contexto atende executivos; o Contêiner atende arquitetos; o Componente atende desenvolvedores. |
| Preparação para liberação | Existe um plano de shadow mode validado com a tesouraria? | Parcial. O conceito está definido, mas a operação assistida precisa de acordo com o stakeholder. |

### Checklist C4 Model

Adicionalmente, validamos os diagramas contra os critérios do [c4model.com/review](https://c4model.com/review/):

| Critério | Contexto | Contêiner | Componente |
|---|---|---|---|
| Título e legenda presentes | ✔ | ✔ | ✔ |
| Elementos possuem nome e tipo | ✔ | ✔ | ✔ |
| Setas indicam direção e descrição | ✔ | ✔ | ✔ |
| Foco em um único nível de abstração | ✔ | ✔ | ✔ |
| Tecnologias explicitadas nos rótulos | ✔ | ✔ | ✔ |
| Sem mistura de níveis de abstração | ✔ | ✔ | ✔ |
| Público-alvo identificável | ✔ | ✔ | ✔ |

## 32. Vídeo de Apresentação

Conforme solicitado, será produzido um vídeo abrangendo os seguintes pontos:

- Contextualização do problema: a conciliação de ledger do FIAP Bank.
- Apresentação da solução LedgerSync.
- Explicação dos três níveis do C4 Model (Contexto → Contêiner → Componente), relacionando cada nível ao seu público-alvo.
- Destaque para os Requisitos Arquitetonicamente Significativos (ASR) que guiaram as decisões.
- Principais decisões arquiteturais e seus trade-offs, com ênfase nos padrões estudados em aula.

Cada integrante do grupo apresentará uma parte do conteúdo. O vídeo será compartilhado com o professor nos endereços `profleonardo.pinho@fiap.com.br` e `leonardo.c.pinho@gmail.com`.

## Stack Tecnológica — Resumo

| Camada | Tecnologia |
|---|---|
| Frontend | React + TypeScript |
| API Gateway | Kotlin + Spring Boot |
| Ingestion | Go |
| Normalization | Kotlin |
| Event Bus | Apache Kafka + Confluent Schema Registry |
| Reconciliation Engine | Kotlin + Kafka Streams |
| Ledger Service | Kotlin + Spring Boot + PostgreSQL |
| Cache | Redis |
| Alert Service | Kotlin |
| Observabilidade | OpenTelemetry + Prometheus + Grafana |
| Infraestrutura | Kubernetes (EKS) + Terraform |

## Referências

- **Aulas da disciplina** — Slides das Aulas 1 a 4, Prof. Leonardo Pinho (FIAP)
- [C4 Model — Documentação Oficial](https://c4model.com/)
- [C4 Model — Checklist de Revisão](https://c4model.com/review/)
- [Apache Kafka Streams — Documentação](https://kafka.apache.org/documentation/streams/)
- [Event Sourcing — Martin Fowler](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Domain-Driven Design — Eric Evans](https://www.domainlanguage.com/ddd/)
- [Padrões de Design — Refactoring Guru](https://refactoring.guru/pt-br/design-patterns)
