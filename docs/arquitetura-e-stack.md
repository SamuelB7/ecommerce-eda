# Projeto de Portfólio: E-commerce Orientado a Eventos

## 1. Objetivo

Construir um e-commerce para portfólio com foco explícito em arquitetura de software, microsserviços, banco por serviço e integração assíncrona por eventos via Kafka.

O objetivo principal aqui não é otimizar o menor esforço possível de implementação. O objetivo é montar um projeto que mostre:

- separação de domínios
- autonomia por serviço
- comunicação orientada a eventos
- consistência eventual com compensações explícitas
- idempotência
- rastreabilidade
- decisões de persistência justificadas por trade-offs reais

## 2. Premissas de Arquitetura

- Cada microsserviço terá seu próprio banco de dados.
- O broker central de eventos será Apache Kafka.
- O sistema evitará banco compartilhado.
- Não haverá transação distribuída entre serviços.
- Fluxos de negócio assíncronos serão resolvidos com saga por coreografia no início.
- Cada serviço deve ser executável de forma independente.
- A estrutura local deve facilitar a futura extração para repositórios separados.

## 3. Decisão Macro

### Recomendação principal

Use `TypeScript + Node.js + NestJS` como stack base dos microsserviços.

Motivo:

- forte aderência ao seu objetivo de portfólio em TypeScript
- bom suporte a HTTP, Kafka, DI, módulos e testes
- documentação madura para microservices com Kafka
- reduz variação acidental entre serviços e deixa a arquitetura mais visível

### Recomendação de persistência por domínio

- `auth-service`: `PostgreSQL`
- `orders-service`: `PostgreSQL`
- `inventory-service`: `PostgreSQL`
- `shipping-service`: `MongoDB`
- `notification-service`: `Apache Cassandra`

Essa combinação é a que melhor equilibra:

- coerência arquitetural
- clareza de trade-offs
- variedade tecnológica suficiente para portfólio
- custo operacional ainda controlável em ambiente local

## 4. PACELC Aplicado ao Projeto

PACELC resume a escolha assim:

- se houver partição (`P`), o sistema privilegia `Availability` ou `Consistency`?
- senão (`Else`), privilegia `Latency` ou `Consistency`?

Perfis usados neste projeto:

- `PC/EC`: domínios onde invariantes importam mais que disponibilidade irrestrita
- `PA/EL`: domínios onde throughput, disponibilidade e baixa latência valem mais que leitura estritamente consistente

### Mapeamento por serviço

| Serviço | Perfil PACELC | Justificativa |
| --- | --- | --- |
| Auth | `PC/EC` | identidade, credenciais, sessão e revogação exigem consistência |
| Orders | `PC/EC` | pedido é trilha de negócio crítica e não pode “sumir” ou divergir |
| Inventory | `PC/EC` | estoque incorreto gera overselling e quebra de confiança |
| Shipping | `PA/EL` | tracking e payloads externos toleram consistência eventual |
| Notification | `PA/EL` | entrega de eventos de notificação é massiva, append-heavy e reprocessável |

## 5. Stack Transversal Recomendada

### Runtime e linguagem

- `Node.js 24 LTS`
- `TypeScript`

### Framework de serviço

- `NestJS`
- `@nestjs/microservices` para todo microsserviço que usar `NestJS` e precisar se comunicar por broker

### Transporte e eventos

- `Apache Kafka` em modo `KRaft`
- em desenvolvimento local, use broker/controller combinado
- em ambiente mais sério, separe controllers de brokers
- serviços `NestJS` devem usar o pacote nativo `@nestjs/microservices` com `Transport.KAFKA`
- use `MicroserviceOptions`, `ClientKafka`, `EventPattern` e `KafkaContext` quando aplicável

### Contratos

- `OpenAPI` para interfaces HTTP
- eventos versionados por schema
- recomendação pragmática inicial: `JSON + validação de schema`

### Validação

- DTOs validados na borda da aplicação
- contratos de evento com schemas versionados

### Observabilidade

- logs estruturados em JSON
- correlation id
- trace id
- OpenTelemetry
- Prometheus + Grafana

### Estratégias obrigatórias para todos os serviços

- `transactional outbox`
- consumidores idempotentes
- `dead-letter topic`
- retry com backoff
- health checks
- testes de contrato

### Estratégia de outbox recomendada

Para a primeira versão do portfólio:

- implemente `transactional outbox` no próprio serviço
- publique os eventos com um worker/poller da aplicação

Para uma segunda versão mais avançada:

- evolua para `Debezium Outbox Event Router`

Essa ordem é pragmática. A primeira versão mostra o padrão. A segunda mostra sofisticação operacional.

## 6. Serviço por Serviço

---

## 6.1 `auth-service`

### Responsabilidades

- cadastro e autenticação
- emissão de access token e refresh token
- revogação de sessão
- publicação de eventos de identidade

### Stack recomendada

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`
- `JWT`
- hash de senha com `Argon2`

### Banco escolhido

- `PostgreSQL`

### Justificativa PACELC

`Auth` é domínio de integridade forte. Você não quer:

- sessão revogada aparecendo válida
- usuário duplicado
- refresh token em estado ambíguo

Por isso, a escolha correta aqui é um perfil `PC/EC`, priorizando consistência.

### Modelo de dados sugerido

- `users`
- `credentials`
- `refresh_tokens`
- `password_reset_tokens`
- `outbox_events`

### Eventos publicados

- `auth.user.registered.v1`
- `auth.user.logged_in.v1`
- `auth.user.password_reset_requested.v1`
- `auth.user.password_changed.v1`

### Por que essa escolha é boa para portfólio

Mostra:

- segurança básica bem feita
- modelagem transacional
- trilha de auditoria
- publicação confiável de eventos a partir de um banco relacional

---

## 6.2 `orders-service`

### Responsabilidades

- criação do pedido
- persistência do ciclo de vida do pedido
- idempotência de criação
- reação a sucesso ou falha de reserva de estoque

### Stack recomendada

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Banco escolhido

- `PostgreSQL`

### Justificativa PACELC

Pedido é o centro narrativo do e-commerce. O serviço precisa manter:

- consistência de estado
- transições válidas
- trilha de auditoria
- idempotência em comandos

Aqui também faz sentido um perfil `PC/EC`.

### Modelo de dados sugerido

- `orders`
- `order_items`
- `order_status_history`
- `idempotency_keys`
- `outbox_events`

### Eventos publicados

- `orders.order.created.v1`
- `orders.order.confirmed.v1`
- `orders.order.cancelled.v1`
- `orders.order.awaiting_stock.v1`

### Eventos consumidos

- `inventory.stock.reserved.v1`
- `inventory.stock.rejected.v1`

### Por que essa escolha é boa para portfólio

Esse serviço deixa visível:

- modelagem de máquina de estados
- consistência transacional
- coreografia orientada a eventos
- uso sério de idempotência

---

## 6.3 `inventory-service`

### Responsabilidades

- controle de saldo disponível
- reserva de estoque
- confirmação ou liberação de reserva
- prevenção de overselling

### Stack recomendada

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Banco escolhido

- `PostgreSQL`

### Justificativa PACELC

Aqui existe uma tentação comum de usar um banco mais orientado a disponibilidade. Para um portfólio sério, eu não recomendo isso para o estoque inicial.

Motivo:

- estoque é domínio de integridade forte
- erro de concorrência gera venda sem produto
- compensação depois existe, mas o custo de negócio é alto

Logo, também faz sentido `PC/EC`.

### Observação importante

Se no futuro você quiser uma versão mais avançada, pode evoluir para:

- reserva por lote
- particionamento por `sku`
- leitura materializada para consulta

Mas o sistema de registro do estoque deve continuar consistente.

### Modelo de dados sugerido

- `stock_items`
- `stock_reservations`
- `stock_movements`
- `outbox_events`

### Eventos publicados

- `inventory.stock.reserved.v1`
- `inventory.stock.rejected.v1`
- `inventory.stock.released.v1`
- `inventory.stock.adjusted.v1`

### Eventos consumidos

- `orders.order.created.v1`
- `orders.order.cancelled.v1`

### Por que essa escolha é boa para portfólio

Mostra maturidade arquitetural. Você não escolheu tecnologia “exótica” só para variar stack. Você escolheu a persistência mais coerente com a invariável de negócio.

---

## 6.4 `shipping-service`

### Responsabilidades

- criação de remessa
- armazenamento de payloads de transportadora
- tracking e eventos de expedição
- atualização de status logístico

### Stack recomendada

- `NestJS`
- `MongoDB`
- `MongoDB Node.js Driver`

### Banco escolhido

- `MongoDB`

### Justificativa PACELC

Expedição tende a lidar com:

- estruturas variáveis por transportadora
- muitos eventos de tracking
- necessidade de anexar payloads externos sem fricção relacional

Aqui faz sentido um perfil mais próximo de `PA/EL`: disponibilidade e flexibilidade importam mais que consistência estrita a cada leitura.

### Modelo de dados sugerido

- `shipments`
- `shipment_tracking_events`
- `carrier_callbacks`
- `outbox_events`

### Eventos publicados

- `shipping.shipment.created.v1`
- `shipping.shipment.dispatched.v1`
- `shipping.shipment.delivered.v1`
- `shipping.shipment.failed.v1`

### Eventos consumidos

- `inventory.stock.reserved.v1`
- `orders.order.confirmed.v1`

### Por que essa escolha é boa para portfólio

Mostra:

- leitura correta do domínio
- uso de banco documental onde o shape dos dados muda
- integração com payloads externos sem normalização artificial

---

## 6.5 `notification-service`

### Responsabilidades

- envio de e-mail, push, webhook ou SMS no futuro
- registro de tentativas
- controle de retries
- trilha de entregas por canal

### Stack recomendada

- `NestJS`
- `Apache Cassandra`
- driver Node.js para Cassandra

### Banco escolhido

- `Apache Cassandra`

### Justificativa PACELC

Notificação é o melhor domínio para assumir explicitamente um perfil `PA/EL`.

Motivo:

- alto volume de gravação
- padrão append-heavy
- consultas previsíveis por destinatário, campanha, status e janela de tempo
- consistência eventual é aceitável
- reprocessamento é parte natural do domínio

### Modelo de dados sugerido

- `notifications_by_recipient`
- `notifications_by_status`
- `delivery_attempts_by_notification`

### Eventos publicados

- `notification.sent.v1`
- `notification.failed.v1`
- `notification.retry_scheduled.v1`

### Eventos consumidos

- `auth.user.registered.v1`
- `orders.order.created.v1`
- `shipping.shipment.dispatched.v1`
- `shipping.shipment.delivered.v1`

### Por que essa escolha é boa para portfólio

Esse serviço deixa clara a aplicação prática de PACELC:

- disponibilidade acima de consistência estrita
- leitura modelada por padrão de acesso
- design orientado a throughput

## 7. Prós e Contras das Tecnologias Selecionadas

## 7.1 `NestJS`

### Prós

- excelente encaixe com TypeScript
- DI e modularidade úteis para microsserviços
- suporte nativo a Kafka no ecossistema
- integração oficial com brokers via `@nestjs/microservices`
- permite usar Kafka junto do ciclo de vida do NestJS, DI, decorators, pipes, interceptors e filtros
- Swagger/OpenAPI simples
- arquitetura fácil de demonstrar em portfólio

### Contras

- abstração alta demais para alguns casos simples
- risco de excesso de boilerplate
- curva de aprendizado maior que um Fastify puro
- a integração com Kafka ainda exige cuidado explícito com offsets, grupos de consumidor, idempotência e semântica de eventos

## 7.2 `PostgreSQL`

### Prós

- consistência forte
- transações maduras
- excelente para regras de negócio e integridade
- bom suporte a índices, constraints e modelagem relacional
- encaixa muito bem com outbox

### Contras

- horizontal scaling mais trabalhoso que bancos orientados a partição nativa
- schemas e joins podem virar gargalo de evolução se o domínio estiver mal recortado
- não é a melhor escolha para payloads muito heterogêneos

## 7.3 `Prisma ORM`

### Prós

- ótima DX em TypeScript
- tipagem forte
- migrations simples
- bom para produtividade de portfólio

### Contras

- abstrai parte do SQL e pode esconder detalhes relevantes
- cenários muito específicos às vezes exigem SQL manual
- menos adequado se você quiser controle extremo sobre cada query desde o início

## 7.4 `MongoDB`

### Prós

- modelo documental flexível
- bom encaixe para payloads externos e tracking
- evolução de schema menos dolorosa
- change streams são úteis em cenários orientados a eventos

### Contras

- consistência e modelagem exigem mais disciplina da aplicação
- fácil cair em duplicação mal pensada
- transações existem, mas não devem virar muleta para modelagem ruim

## 7.5 `MongoDB Node.js Driver`

### Prós

- driver oficial
- menos magia que um ODM
- mais controle sobre índices, collections e queries

### Contras

- mais verboso
- menos produtividade inicial que um ODM como Mongoose

## 7.6 `Apache Cassandra`

### Prós

- altíssimo throughput de escrita
- ótima disponibilidade
- consistência ajustável
- muito bom para workloads append-only e consultas previsíveis
- TTL e modelagem por acesso encaixam bem em logs de entrega

### Contras

- modelagem é mais rígida e orientada a query
- joins inexistentes
- operação mais complexa
- não é uma boa escolha para domínios transacionais ricos

## 7.7 `Apache Kafka`

### Prós

- backbone natural para arquitetura orientada a eventos
- retenção, replay e particionamento
- desacoplamento forte entre produtores e consumidores
- excelente sinal arquitetural para portfólio

### Contras

- aumenta bastante a complexidade operacional
- exige disciplina com versionamento de eventos
- sem idempotência e outbox, a arquitetura fica frágil

## 8. Decisões Estruturais do Repositório

## 8.1 Recomendação principal

Monte desde já um repositório agregador com serviços independentes por pasta, mas sem compartilhar código de domínio entre eles.

Isso te dá:

- facilidade para extrair cada serviço para um repositório próprio
- menor acoplamento acidental
- narrativa arquitetural mais limpa
- caminho simples para migrar depois para `git submodule` ou `git subtree`

## 8.2 Estrutura sugerida

```text
ecommerce/
  docs/
    arquitetura-e-stack.md
    event-storming.md
    topicos-kafka.md
    decisoes-arquiteturais/
  auth-service/
    src/
    test/
    prisma/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  orders-service/
    src/
    test/
    prisma/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  inventory-service/
    src/
    test/
    prisma/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  shipping-service/
    src/
    test/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  notification-service/
    src/
    test/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  contracts/
    events/
    http/
  platform/
    docker/
    kafka/
    observability/
    scripts/
  docker-compose.yml
  README.md
```

## 8.3 Regra importante

Evite criar um `shared/` com regra de negócio compartilhada.

Compartilhe no máximo:

- contratos de evento
- schemas
- utilitários puramente técnicos e muito pequenos

Se você compartilhar lógica de domínio, a separação futura em múltiplos repositórios fica pior e a autonomia dos serviços enfraquece.

## 9. Comunicação Entre Serviços

## 9.1 Estilo recomendado

- `HTTP` apenas para borda externa e poucos comandos síncronos inevitáveis
- `Kafka` para integração de negócio

## 9.2 Fluxo inicial sugerido

### Cadastro

1. `auth-service` registra usuário
2. publica `auth.user.registered.v1`
3. `notification-service` envia mensagem de boas-vindas

### Criação de pedido

1. `orders-service` cria pedido com status inicial
2. publica `orders.order.created.v1`
3. `inventory-service` tenta reservar estoque
4. publica `inventory.stock.reserved.v1` ou `inventory.stock.rejected.v1`
5. `orders-service` confirma ou cancela o pedido
6. `shipping-service` cria remessa quando o pedido estiver pronto
7. `notification-service` dispara notificações ao longo do fluxo

## 9.3 Convenção de tópicos

Sugestão:

- `<dominio>.<agregado>.<evento>.v1`

Exemplos:

- `auth.user.registered.v1`
- `orders.order.created.v1`
- `inventory.stock.reserved.v1`
- `shipping.shipment.dispatched.v1`

## 10. Padrões Arquiteturais que Valem Muito no Portfólio

Se você quer que o projeto “pareça sênior”, estes padrões devem aparecer claramente:

- `transactional outbox`
- consumidores idempotentes
- `correlation-id`
- retries com DLQ
- versionamento de eventos
- `README` por serviço explicando responsabilidade, eventos publicados e eventos consumidos
- `ADR` para decisões maiores

## 11. O que eu não recomendo para a primeira versão

### `Keycloak` no lugar do `auth-service`

Não para a primeira versão do portfólio.

Motivo:

- resolve autenticação, mas esconde seu desenho arquitetural
- o repositório fica parecendo integração de produto, não design de sistema

Você pode adicionar isso depois como comparação arquitetural.

### `MongoDB` para pedidos ou estoque

Não recomendo no início.

Motivo:

- enfraquece a narrativa de integridade forte
- aumenta risco de modelagem inconsistente

### `Redis` como banco principal de algum serviço

Também não recomendo como store principal na primeira versão.

Motivo:

- Kafka já cobre a parte de streaming/eventos
- para esse portfólio, Redis agrega mais como cache ou rate limit do que como banco de sistema de registro

## 12. Roadmap Recomendado

## Fase 1

- estrutura do agregador
- Kafka local
- `auth-service`
- `orders-service`
- `inventory-service`

## Fase 2

- `shipping-service`
- `notification-service`
- observabilidade
- DLQ e retries

## Fase 3

- extração de cada serviço para seu próprio repositório
- agregador vira repositório de compose, documentação e automação local

## 13. Decisão Final Recomendada

Se fosse meu projeto de portfólio, eu seguiria exatamente esta combinação:

- base de serviços: `Node.js + TypeScript + NestJS`
- integração de mensageria nos serviços NestJS: `@nestjs/microservices`
- broker: `Apache Kafka`
- `auth-service`: `PostgreSQL + Prisma`
- `orders-service`: `PostgreSQL + Prisma`
- `inventory-service`: `PostgreSQL + Prisma`
- `shipping-service`: `MongoDB + driver oficial`
- `notification-service`: `Cassandra + driver Node.js`

Ela mostra amplitude suficiente sem virar uma feira de tecnologia.

Esse ponto é importante: um bom portfólio não impressiona por quantidade de ferramentas. Ele impressiona por coerência de decisão.

## 14. Referências

- PACELC por Daniel Abadi: <https://dbmsmusings.blogspot.com/2010/04/problems-with-cap-and-yahoos-little.html>
- Discussão posterior de PACELC: <https://dbmsmusings.blogspot.com/2017/>
- NestJS microservices com Kafka: <https://docs.nestjs.com/microservices/kafka>
- NestJS microservices basics: <https://docs.nestjs.com/microservices/basics>
- Node.js release policy e versões LTS: <https://nodejs.org/en/about/previous-releases>
- Apache Kafka downloads e releases suportadas: <https://kafka.apache.org/community/downloads/>
- Kafka KRaft: <https://kafka.apache.org/40/operations/kraft/>
- PostgreSQL logical replication: <https://www.postgresql.org/docs/current/logical-replication.html>
- PostgreSQL JSON types: <https://www.postgresql.org/docs/current/static/datatype-json.html>
- Prisma ORM: <https://docs.prisma.io/docs/orm>
- MongoDB transactions: <https://www.mongodb.com/docs/current/core/transactions/>
- MongoDB change streams: <https://www.mongodb.com/docs/manual/changestreams/>
- MongoDB data modeling: <https://www.mongodb.com/docs/manual/data-modeling/>
- MongoDB Node.js driver: <https://www.mongodb.com/docs/drivers/node/current/>
- Cassandra overview: <https://cassandra.apache.org/doc/latest/cassandra/architecture/overview.html>
- Cassandra tunable consistency: <https://cassandra.apache.org/doc/latest/cassandra/architecture/dynamo.html>
- Cassandra data modeling: <https://cassandra.apache.org/doc/latest/cassandra/developing/data-modeling/index.html>
- Cassandra TTL/time-series guidance: <https://cassandra.apache.org/doc/stable/cassandra/managing/operating/compaction/twcs.html>
- Debezium outbox event router: <https://debezium.io/documentation/reference/2.6/transformations/outbox-event-router.html>
