### Fase 1: Arquitetura Base (O Ecossistema Pix Mock)

O objetivo é levantar a infraestrutura transacional baseada em microsserviços. Recomenda-se a utilização de um *monorepo* (projeto Maven multi-módulo).

* [ ] **1.1 Arranque da Infraestrutura:** Criar o projeto base e um `docker-compose.yml` para iniciar as dependências: MySQL (ou PostgreSQL), Kafka e Zookeeper/Kraft.
* [ ] **1.2 Microsserviço API Gateway (`gateway-api`):**
* Criar módulo Spring Boot (Java 17+).
* Desenvolver o endpoint `POST /pix/transfer` simulando a receção de um Pix (origem, destino, valor).
* Configurar o `KafkaTemplate` para publicar o evento de pagamento no tópico `pix-events`.


* [ ] **1.3 Microsserviço Processador (`payment-processor`):**
* Criar módulo Spring Boot.
* Desenvolver um `@KafkaListener` para consumir os eventos do tópico `pix-events`.
* Persistir a transação na base de dados utilizando Spring Data JPA.



### Fase 2: Telemetria, Datadog e Engenharia do Caos

Preparação do ambiente de falhas e injeção de dados de monitorização de nível empresarial.

* [ ] **2.1 Setup do Datadog Agent:** Adicionar o contentor oficial do *Datadog Agent* ao `docker-compose.yml`. Configurá-lo com a variável de ambiente `DD_API_KEY` (disponível no escalão gratuito) para escutar logs e métricas dos restantes contentores.
* [ ] **2.2 Métricas de Base de Dados no Datadog:** Adicionar a dependência `micrometer-registry-datadog` nos dois microsserviços. Configurar a exportação para garantir que as métricas do pool de ligações (*HikariCP*) são enviadas para a *dashboard* do Datadog.
* [ ] **2.3 Logs em JSON e APM (Tracing):** Configurar o Logback para saída estruturada em JSON e reencaminhá-la para o agente do Datadog. Adicionar as bibliotecas de *Datadog APM* para garantir que o `traceId` correlaciona perfeitamente o pedido do Gateway com o processamento no backend.
* [ ] **2.4 Botão de Pânico 1 (Database Lock):** Criar um endpoint oculto `POST /chaos/db-lock` no `payment-processor` que force o esgotamento do pool de ligações à base de dados (ex: abrindo uma transação e suspendendo a *thread* com um *sleep*).
* [ ] **2.5 Botão de Pânico 2 (CPU/Kafka Lag):** Criar o endpoint `POST /chaos/cpu-spike` para gerar lentidão artificial, forçando a acumulação de eventos não processados no tópico do Kafka.

### Fase 3: O Cérebro do SRE Copilot (Gemini + Spring AI + Datadog API)

A construção do engenheiro de suporte autónomo, agora integrado com a plataforma de monitorização.

* [ ] **3.1 Setup do Agente:** Criar o módulo `sre-copilot-agent` com a dependência `spring-ai-googleai-spring-boot-starter`.
* [ ] **3.2 Configuração do Gemini:** Gerar uma chave no Google AI Studio. Configurar no `application.yml` a `api-key` e definir o modelo como `gemini-1.5-flash`.
* [ ] **3.3 Skill 1 (Log Fetcher via Datadog):** Desenvolver uma função Java com `@Bean` e `@Description("Busca os logs de erro recentes de um microsserviço específico no Datadog")`. A função `fetchRecentErrorLogs(String serviceName)` fará um pedido HTTP GET à **Datadog Logs API**, filtrando por `status:error` e pelo nome do serviço alvo.
* [ ] **3.4 Skill 2 (Database Diagnostics via Datadog):** Desenvolver a *tool* `@Description("Verifica o status do pool de ligações com a base de dados consultando as métricas no Datadog")`. A função `checkDatabaseConnectionPool(String serviceName)` fará um pedido à **Datadog Metrics API** para extrair o valor atual da métrica `hikaricp.connections.active`.
* [ ] **3.5 Skill 3 (Kafka Diagnostics):** Desenvolver a *tool* `@Description("Verifica o atraso de mensagens num tópico do Kafka")`. Nome: `checkKafkaLag(String topicName)`.
* [ ] **3.6 Engenharia de Prompt (System Prompt):** Configurar o comportamento do Gemini: *"És um Engenheiro de Confiabilidade de Sistemas (SRE) Sénior numa War Room. Ao investigar anomalias, utiliza as tuas ferramentas de integração com o Datadog para cruzar logs e métricas. Emite um diagnóstico técnico formal com a causa raiz e o plano de mitigação."*
* [ ] **3.7 Gatilho de Investigação:** Criar o endpoint `POST /investigate` no agente, recebendo a string de alerta (ex: "Alerta P1: O sistema Pix apresenta timeout").

### Fase 4: Simulação da "War Room" (Validação e Portefólio)

O teste prático e a criação do material de vitrina para os recrutadores.

* [ ] **4.1 Script de Carga:** Utilizar K6 ou um *script* em *bash* para efetuar pedidos contínuos ao `gateway-api`.
* [ ] **4.2 Injeção de Falha:** Acionar o `/chaos/db-lock`. Observar no Datadog os gráficos de latência a subir e os erros a aparecerem.
* [ ] **4.3 Teste Autónomo:** Disparar o `/investigate`. Acompanhar no terminal os registos do Spring AI, visualizando o Gemini a decidir autonomamente chamar as APIs do Datadog para cruzar as métricas de *HikariCP* com os logs de erro.
* [ ] **4.4 Empacotamento do Portefólio:** Submeter para o GitHub com um `README.md` exaustivo, incluindo diagramas de arquitetura, capturas de ecrã dos painéis do Datadog e a transcrição da investigação resolvida pela Inteligência Artificial.
