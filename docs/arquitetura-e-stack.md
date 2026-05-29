# Portfolio Project: Event-Driven E-commerce

## 1. Goal

Build an e-commerce portfolio project with an explicit focus on software architecture, microservices, database per service, and asynchronous integration through Kafka events.

The main goal is not to minimize implementation effort. The goal is to build a project that demonstrates:

- domain separation
- service autonomy
- event-driven communication
- eventual consistency with explicit compensations
- idempotency
- traceability
- persistence decisions justified by real trade-offs

## 2. Architecture Premises

- Each microservice owns its own database.
- Apache Kafka is the central event broker.
- The system avoids shared databases.
- There are no distributed transactions across services.
- Initial business flows use choreography-based sagas.
- Each service must be independently runnable.
- The local structure must make future extraction into separate repositories easy.

## 3. Macro Decision

### Main Recommendation

Use `TypeScript + Node.js + NestJS` as the base stack for the microservices.

Reasons:

- strong alignment with the TypeScript portfolio goal
- good support for HTTP, Kafka, dependency injection, modules, and tests
- mature documentation for Kafka-based microservices
- reduces accidental variation between services and makes the architecture more visible

### Persistence Recommendation By Domain

- `auth-service`: `PostgreSQL`
- `orders-service`: `PostgreSQL`
- `inventory-service`: `PostgreSQL`
- `shipping-service`: `MongoDB`
- `notification-service`: `Apache Cassandra`

This combination best balances:

- architectural coherence
- clear trade-offs
- enough technology variety for a portfolio
- manageable operational cost in a local development environment

## 4. PACELC Applied To The Project

PACELC summarizes database choices this way:

- if there is a partition (`P`), does the system prefer `Availability` or `Consistency`?
- else (`Else`), does it prefer `Latency` or `Consistency`?

Profiles used in this project:

- `PC/EC`: domains where invariants matter more than unrestricted availability
- `PA/EL`: domains where throughput, availability, and low latency matter more than strictly consistent reads

### Service Mapping

| Service | PACELC Profile | Rationale |
| --- | --- | --- |
| Auth | `PC/EC` | identity, credentials, sessions, and revocation require consistency |
| Orders | `PC/EC` | orders are a critical business trail and cannot disappear or diverge |
| Inventory | `PC/EC` | incorrect inventory causes overselling and breaks trust |
| Shipping | `PA/EL` | tracking data and external payloads tolerate eventual consistency |
| Notification | `PA/EL` | notification delivery is high-volume, append-heavy, and reprocessable |

## 5. Recommended Cross-Cutting Stack

### Runtime And Language

- `Node.js 24 LTS`
- `TypeScript`

### Service Framework

- `NestJS`
- `@nestjs/microservices` for every microservice that uses `NestJS` and needs broker communication

### Internal API Layers

Every `NestJS` microservice must separate the API into three main layers:

- `controller`: responsible for HTTP endpoints, NestJS decorators, API input and output, boundary validation, and delegation to the service layer
- `service`: responsible for business rules, use-case orchestration, flow decisions, and decisions to publish domain events
- `repository`: responsible for persistence, queries, local transactions, and access to the microservice's own database

Mandatory rules:

- `controller` must not access the database directly
- `controller` must not implement business rules
- `service` must not depend on HTTP transport details
- `repository` must not publish domain events directly
- each `repository` accesses only its own microservice database

### Transport And Events

- `Apache Kafka` in `KRaft` mode
- in local development, use a combined broker/controller
- in more serious environments, split controllers from brokers
- `NestJS` services must use the native `@nestjs/microservices` package with `Transport.KAFKA`
- use `MicroserviceOptions`, `ClientKafka`, `EventPattern`, and `KafkaContext` when applicable

### Contracts

- `OpenAPI` for HTTP interfaces
- events versioned by schema
- pragmatic initial recommendation: `JSON + schema validation`

### Validation

- DTOs validated at the application boundary
- event contracts with versioned schemas

### Observability

- structured JSON logs
- correlation id
- trace id
- OpenTelemetry
- Prometheus + Grafana

### Mandatory Strategies For All Services

- `transactional outbox`
- idempotent consumers
- `dead-letter topic`
- retry with backoff
- health checks
- contract tests

### Recommended Outbox Strategy

For the first portfolio version:

- implement `transactional outbox` inside each service
- publish events through an application worker/poller

For a more advanced second version:

- evolve to `Debezium Outbox Event Router`

This order is pragmatic. The first version demonstrates the pattern. The second version demonstrates operational sophistication.

## 6. Service By Service

---

## 6.1 `auth-service`

### Responsibilities

- user registration and authentication
- access token and refresh token issuing
- session revocation
- identity event publishing

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`
- `JWT`
- password hashing with `Argon2`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

`Auth` is a strong-integrity domain. You do not want:

- a revoked session to appear valid
- duplicate users
- refresh tokens in an ambiguous state

For that reason, the right choice here is a `PC/EC` profile that prioritizes consistency.

### Suggested Data Model

- `users`
- `credentials`
- `refresh_tokens`
- `password_reset_tokens`
- `outbox_events`

### Published Events

- `auth.user.registered.v1`
- `auth.user.logged_in.v1`
- `auth.user.password_reset_requested.v1`
- `auth.user.password_changed.v1`

### Why This Choice Is Good For A Portfolio

It demonstrates:

- solid baseline security
- transactional modeling
- audit trail design
- reliable event publishing from a relational database

---

## 6.2 `orders-service`

### Responsibilities

- order creation
- persistence of the order lifecycle
- creation idempotency
- reaction to inventory reservation success or failure

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Orders are the narrative center of the e-commerce system. The service must preserve:

- state consistency
- valid transitions
- audit trail
- command idempotency

A `PC/EC` profile also makes sense here.

### Suggested Data Model

- `orders`
- `order_items`
- `order_status_history`
- `idempotency_keys`
- `outbox_events`

### Published Events

- `orders.order.created.v1`
- `orders.order.confirmed.v1`
- `orders.order.cancelled.v1`
- `orders.order.awaiting_stock.v1`

### Consumed Events

- `inventory.stock.reserved.v1`
- `inventory.stock.rejected.v1`

### Why This Choice Is Good For A Portfolio

This service makes these topics visible:

- state machine modeling
- transactional consistency
- event-driven choreography
- serious idempotency usage

---

## 6.3 `inventory-service`

### Responsibilities

- available stock control
- stock reservation
- reservation confirmation or release
- overselling prevention

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

There is a common temptation to use a more availability-oriented database here. For a serious portfolio, that is not recommended for initial inventory.

Reasons:

- inventory is a strong-integrity domain
- concurrency errors can create sales without product availability
- compensation is possible later, but the business cost is high

Therefore, `PC/EC` also makes sense.

### Important Note

If you want a more advanced version in the future, you can evolve to:

- batch-based reservation
- partitioning by `sku`
- materialized reads for queries

But the inventory system of record should remain consistent.

### Suggested Data Model

- `stock_items`
- `stock_reservations`
- `stock_movements`
- `outbox_events`

### Published Events

- `inventory.stock.reserved.v1`
- `inventory.stock.rejected.v1`
- `inventory.stock.released.v1`
- `inventory.stock.adjusted.v1`

### Consumed Events

- `orders.order.created.v1`
- `orders.order.cancelled.v1`

### Why This Choice Is Good For A Portfolio

It shows architectural maturity. The technology was not chosen just to add variety. The persistence model matches the business invariant.

---

## 6.4 `shipping-service`

### Responsibilities

- shipment creation
- carrier payload storage
- tracking and shipping events
- logistics status updates

### Recommended Stack

- `NestJS`
- `MongoDB`
- `MongoDB Node.js Driver`

### Chosen Database

- `MongoDB`

### PACELC Rationale

Shipping usually deals with:

- carrier-specific variable structures
- many tracking events
- the need to attach external payloads without relational friction

A profile closer to `PA/EL` makes sense here: availability and flexibility matter more than strict consistency on every read.

### Suggested Data Model

- `shipments`
- `shipment_tracking_events`
- `carrier_callbacks`
- `outbox_events`

### Published Events

- `shipping.shipment.created.v1`
- `shipping.shipment.dispatched.v1`
- `shipping.shipment.delivered.v1`
- `shipping.shipment.failed.v1`

### Consumed Events

- `inventory.stock.reserved.v1`
- `orders.order.confirmed.v1`

### Why This Choice Is Good For A Portfolio

It demonstrates:

- correct domain reading
- document database usage where data shape changes
- integration with external payloads without artificial normalization

---

## 6.5 `notification-service`

### Responsibilities

- future email, push, webhook, or SMS sending
- attempt logging
- retry control
- delivery history by channel

### Recommended Stack

- `NestJS`
- `Apache Cassandra`
- Node.js driver for Cassandra

### Chosen Database

- `Apache Cassandra`

### PACELC Rationale

Notification is the best domain to explicitly assume a `PA/EL` profile.

Reasons:

- high write volume
- append-heavy pattern
- predictable queries by recipient, campaign, status, and time window
- eventual consistency is acceptable
- reprocessing is a natural part of the domain

### Suggested Data Model

- `notifications_by_recipient`
- `notifications_by_status`
- `delivery_attempts_by_notification`

### Published Events

- `notification.sent.v1`
- `notification.failed.v1`
- `notification.retry_scheduled.v1`

### Consumed Events

- `auth.user.registered.v1`
- `orders.order.created.v1`
- `shipping.shipment.dispatched.v1`
- `shipping.shipment.delivered.v1`

### Why This Choice Is Good For A Portfolio

This service makes the practical application of PACELC clear:

- availability over strict consistency
- reads modeled by access pattern
- throughput-oriented design

## 7. Pros And Cons Of The Selected Technologies

## 7.1 `NestJS`

### Pros

- excellent fit with TypeScript
- dependency injection and modularity are useful for microservices
- native Kafka support in the ecosystem
- official broker integration through `@nestjs/microservices`
- allows Kafka to be used with the NestJS lifecycle, dependency injection, decorators, pipes, interceptors, and filters
- simple Swagger/OpenAPI integration
- architecture is easy to demonstrate in a portfolio

### Cons

- abstraction can be too high for simple cases
- risk of excessive boilerplate
- steeper learning curve than plain Fastify
- Kafka integration still requires explicit care with offsets, consumer groups, idempotency, and event semantics

## 7.2 `PostgreSQL`

### Pros

- strong consistency
- mature transactions
- excellent fit for business rules and integrity
- good support for indexes, constraints, and relational modeling
- fits very well with the outbox pattern

### Cons

- horizontal scaling is more difficult than with natively partition-oriented databases
- schemas and joins can become evolution bottlenecks if the domain boundaries are poorly designed
- not the best choice for highly heterogeneous payloads

## 7.3 `Prisma ORM`

### Pros

- great TypeScript developer experience
- strong typing
- simple migrations
- good productivity for a portfolio project

### Cons

- abstracts part of SQL and can hide relevant details
- very specific scenarios may require manual SQL
- less suitable if you want extreme control over every query from the beginning

## 7.4 `MongoDB`

### Pros

- flexible document model
- good fit for external payloads and tracking
- less painful schema evolution
- change streams are useful in event-driven scenarios

### Cons

- consistency and modeling require more discipline from the application
- easy to fall into poorly designed duplication
- transactions exist, but should not become a crutch for bad modeling

## 7.5 `MongoDB Node.js Driver`

### Pros

- official driver
- less magic than an ODM
- more control over indexes, collections, and queries

### Cons

- more verbose
- lower initial productivity than an ODM like Mongoose

## 7.6 `Apache Cassandra`

### Pros

- very high write throughput
- excellent availability
- tunable consistency
- very good for append-only workloads and predictable queries
- TTL and access-pattern modeling fit delivery logs well

### Cons

- modeling is more rigid and query-oriented
- no joins
- more complex operations
- not a good choice for rich transactional domains

## 7.7 `Apache Kafka`

### Pros

- natural backbone for event-driven architecture
- retention, replay, and partitioning
- strong decoupling between producers and consumers
- excellent architectural signal for a portfolio

### Cons

- significantly increases operational complexity
- requires discipline with event versioning
- without idempotency and outbox, the architecture becomes fragile

## 8. Repository Structural Decisions

## 8.1 Main Recommendation

Start with an aggregator repository that contains independent services by folder, without sharing domain code between them.

This gives you:

- an easy path to extract each service into its own repository
- less accidental coupling
- a cleaner architectural narrative
- a simple path to migrate later to `git submodule` or `git subtree`

## 8.2 Suggested Structure

```text
ecommerce/
  docs/
    arquitetura-e-stack.md
    event-storming.md
    kafka-topics.md
    architecture-decision-records/
  auth-service/
    src/
      controllers/
      services/
      repositories/
      events/
      main.ts
    test/
    prisma/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  orders-service/
    src/
      controllers/
      services/
      repositories/
      events/
      main.ts
    test/
    prisma/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  inventory-service/
    src/
      controllers/
      services/
      repositories/
      events/
      main.ts
    test/
    prisma/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  shipping-service/
    src/
      controllers/
      services/
      repositories/
      events/
      main.ts
    test/
    Dockerfile
    README.md
    package.json
    tsconfig.json
  notification-service/
    src/
      controllers/
      services/
      repositories/
      events/
      main.ts
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

## 8.3 Important Rule

Avoid creating a `shared/` package with shared business rules.

Share at most:

- event contracts
- schemas
- very small, purely technical utilities

If domain logic is shared, future separation into multiple repositories gets worse and service autonomy becomes weaker.

## 9. Service Communication

## 9.1 Recommended Style

- use `HTTP` only for the external edge and a few unavoidable synchronous commands
- use `Kafka` for business integration

## 9.2 Suggested Initial Flow

### Registration

1. `auth-service` registers the user.
2. It publishes `auth.user.registered.v1`.
3. `notification-service` sends a welcome message.

### Order Creation

1. `orders-service` creates the order with its initial status.
2. It publishes `orders.order.created.v1`.
3. `inventory-service` tries to reserve stock.
4. It publishes `inventory.stock.reserved.v1` or `inventory.stock.rejected.v1`.
5. `orders-service` confirms or cancels the order.
6. `shipping-service` creates the shipment when the order is ready.
7. `notification-service` sends notifications throughout the flow.

## 9.3 Topic Naming Convention

Suggestion:

- `<domain>.<aggregate>.<event>.v1`

Examples:

- `auth.user.registered.v1`
- `orders.order.created.v1`
- `inventory.stock.reserved.v1`
- `shipping.shipment.dispatched.v1`

## 10. Architectural Patterns With Strong Portfolio Value

If you want the project to feel senior, these patterns should appear clearly:

- `transactional outbox`
- idempotent consumers
- `correlation-id`
- retries with DLQ
- event versioning
- explicit separation between `controller`, `service`, and `repository`
- one `README` per service explaining responsibility, published events, and consumed events
- `ADR` documents for larger decisions

## 11. What Is Not Recommended For The First Version

### `Keycloak` Instead Of `auth-service`

Not for the first portfolio version.

Reason:

- it solves authentication, but hides your architectural design
- the repository starts to look like product integration instead of system design

You can add it later as an architectural comparison.

### `MongoDB` For Orders Or Inventory

Not recommended initially.

Reason:

- it weakens the strong-integrity narrative
- it increases the risk of inconsistent modeling

### `Redis` As The Main Database For Any Service

Also not recommended as a system-of-record database in the first version.

Reason:

- Kafka already covers the streaming and eventing side
- for this portfolio, Redis adds more value as cache or rate limiter than as a primary database

## 12. Recommended Roadmap

## Phase 1

- aggregator structure
- local Kafka
- `auth-service`
- `orders-service`
- `inventory-service`

## Phase 2

- `shipping-service`
- `notification-service`
- observability
- DLQ and retries

## Phase 3

- extraction of each service into its own repository
- aggregator becomes the repository for Compose, documentation, and local automation

## 13. Final Recommended Decision

If this were my portfolio project, I would follow exactly this combination:

- service base: `Node.js + TypeScript + NestJS`
- messaging integration in NestJS services: `@nestjs/microservices`
- broker: `Apache Kafka`
- `auth-service`: `PostgreSQL + Prisma`
- `orders-service`: `PostgreSQL + Prisma`
- `inventory-service`: `PostgreSQL + Prisma`
- `shipping-service`: `MongoDB + official driver`
- `notification-service`: `Cassandra + Node.js driver`

It shows enough breadth without turning into a technology fair.

This point matters: a good portfolio is not impressive because of tool quantity. It is impressive because of coherent decisions.

## 14. References

- PACELC by Daniel Abadi: <https://dbmsmusings.blogspot.com/2010/04/problems-with-cap-and-yahoos-little.html>
- Later PACELC discussion: <https://dbmsmusings.blogspot.com/2017/>
- NestJS microservices with Kafka: <https://docs.nestjs.com/microservices/kafka>
- NestJS microservices basics: <https://docs.nestjs.com/microservices/basics>
- Node.js release policy and LTS versions: <https://nodejs.org/en/about/previous-releases>
- Apache Kafka downloads and supported releases: <https://kafka.apache.org/community/downloads/>
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
- Cassandra TTL and time-series guidance: <https://cassandra.apache.org/doc/stable/cassandra/managing/operating/compaction/twcs.html>
- Debezium outbox event router: <https://debezium.io/documentation/reference/2.6/transformations/outbox-event-router.html>
