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
- Each microservice is a Domain-Driven Design bounded context and organizes business logic by domain module or aggregate.

## 3. Macro Decision

### Main Recommendation

Use `TypeScript + Node.js + NestJS` as the base stack for the microservices.

Reasons:

- strong alignment with the TypeScript portfolio goal
- good support for HTTP, Kafka, dependency injection, modules, and tests
- mature documentation for Kafka-based microservices
- reduces accidental variation between services and makes the architecture more visible

### Persistence Recommendation By Domain

| Service | Database | Access Library | PACELC Profile | Reason |
| --- | --- | --- | --- | --- |
| `auth-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | credentials, sessions, revocation, and uniqueness need consistency |
| `customer-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | profiles, addresses, privacy data, and ownership rules need consistency |
| `seller-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | KYC, seller status, team permissions, and payout references need consistency |
| `catalog-service` | `PostgreSQL` | `Prisma ORM + JSONB` | `PC/EC` | product source of truth needs integrity, while attributes need some flexibility |
| `search-service` | `OpenSearch` | official OpenSearch client | `PA/EL` | search is a derived read model optimized for relevance, facets, and low latency |
| `recommendation-service` | `Apache Cassandra` | Node.js driver for Cassandra | `PA/EL` | personalized feeds are high-volume, read-optimized, and eventually consistent |
| `cart-service` | `Redis` | official Redis client | `PA/EL` | active carts need low latency, TTL, and fast mutation before checkout |
| `promotion-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | coupon usage, campaign state, and eligibility limits need consistency |
| `orders-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | order lifecycle, idempotency, and audit trail need consistency |
| `payment-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | payments, refunds, reconciliation, and finance audit need consistency |
| `inventory-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | stock reservations and overselling prevention need consistency |
| `shipping-service` | `MongoDB` | MongoDB Node.js driver | `PA/EL` | carrier payloads and tracking timelines benefit from flexible documents |
| `return-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | return eligibility, disputes, and refund orchestration need consistency |
| `review-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | verified-purchase reviews, moderation state, and rating audit need consistency |
| `question-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | product Q&A ownership, answers, and moderation need consistency |
| `notification-service` | `Apache Cassandra` | Node.js driver for Cassandra | `PA/EL` | delivery history is append-heavy and tolerates eventual consistency |
| `settlement-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | seller ledger, commissions, holds, and payouts need consistency |
| `support-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | tickets, assignment, case history, and audit need consistency |
| `admin-service` | `PostgreSQL` | `Prisma ORM` | `PC/EC` | policies, overrides, and audit logs need consistency |
| `analytics-service` | `ClickHouse` | official ClickHouse client | `PA/EL` | analytics needs high-volume inserts, aggregates, and dashboard queries |

This combination best balances:

- architectural coherence
- clear trade-offs
- specialized databases only where their access pattern is clearly justified
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
| Customer | `PC/EC` | profile ownership, addresses, and privacy workflows require consistency |
| Seller | `PC/EC` | seller lifecycle, KYC, team roles, and payout references require consistency |
| Catalog | `PC/EC` | listing lifecycle and category integrity require a consistent source of truth |
| Search | `PA/EL` | search can be rebuilt from events and prioritizes low-latency reads |
| Recommendation | `PA/EL` | feeds are derived and tolerate eventual consistency |
| Cart | `PA/EL` | active cart state prioritizes low latency and can be revalidated at checkout |
| Promotion | `PC/EC` | coupon usage and campaign limits require consistency |
| Orders | `PC/EC` | orders are a critical business trail and cannot disappear or diverge |
| Payment | `PC/EC` | payment state, refunds, and reconciliation require consistency |
| Inventory | `PC/EC` | incorrect inventory causes overselling and breaks trust |
| Shipping | `PA/EL` | tracking data and external payloads tolerate eventual consistency |
| Return | `PC/EC` | return decisions, disputes, and refunds require consistency |
| Review | `PC/EC` | verified-purchase rules and moderation require consistency |
| Question | `PC/EC` | answer ownership and moderation state require consistency |
| Notification | `PA/EL` | notification delivery is high-volume, append-heavy, and reprocessable |
| Settlement | `PC/EC` | seller ledger and payouts require financial consistency |
| Support | `PC/EC` | case assignment, status, and audit trail require consistency |
| Admin | `PC/EC` | policy changes and operational overrides require consistency |
| Analytics | `PA/EL` | dashboards and aggregates are derived from event streams |

## 5. Recommended Cross-Cutting Stack

### Runtime And Language

- `Node.js 24 LTS`
- `TypeScript`

### Service Framework

- `NestJS`
- `@nestjs/microservices` for every microservice that uses `NestJS` and needs broker communication

### Domain-Driven Internal Structure

Every `NestJS` microservice is a DDD bounded context. Inside each service, organize business logic by domain module or aggregate instead of global technical folders.

Recommended layers inside each domain module:

- `domain`: entities, value objects, domain services, domain events, aggregate rules, and repository ports
- `application`: use cases, commands, queries, orchestration, transaction boundaries, and outbox coordination
- `infrastructure`: database adapters, external service clients, Kafka producers, and repository implementations
- `interfaces`: HTTP controllers, Kafka consumers, request DTOs, response DTOs, and transport-specific validation

Mandatory rules:

- `domain` must not depend on `NestJS`, `Prisma`, HTTP DTOs, Kafka clients, or external SDKs
- `interfaces` must not implement business rules
- `application` coordinates use cases but keeps transport details out
- `infrastructure` implements ports defined by `domain` or `application`
- repositories access only their own microservice database
- domain events are named in business language and are published through application/outbox workflows

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

---

## 6.6 `customer-service`

### Responsibilities

- customer profile management
- delivery addresses
- preferences, wishlists, and followed sellers
- privacy export workflows

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Customer data is a consistency-oriented domain. Address ownership, default address selection, profile updates, and privacy export workflows should not diverge.

This service uses a `PC/EC` profile because correctness matters more than unrestricted availability.

### Suggested Data Model

- `customers`
- `customer_addresses`
- `customer_preferences`
- `wishlists`
- `followed_sellers`
- `outbox_events`

### Published Events

- `customer.profile.created.v1`
- `customer.profile.updated.v1`
- `customer.address.updated.v1`

---

## 6.7 `seller-service`

### Responsibilities

- seller onboarding
- store profile management
- KYC and business document review state
- seller users, roles, and operational status

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Seller status determines whether listings, payouts, and fulfillment operations are allowed. KYC and team permissions are strong-integrity workflows.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `sellers`
- `seller_profiles`
- `seller_documents`
- `seller_team_members`
- `seller_status_history`
- `outbox_events`

### Published Events

- `seller.application.submitted.v1`
- `seller.approved.v1`
- `seller.suspended.v1`
- `seller.profile.updated.v1`

---

## 6.8 `catalog-service`

### Responsibilities

- category tree management
- product and listing source of truth
- listing lifecycle
- product variations, attributes, and media metadata

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`
- `JSONB` for flexible listing attributes

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Catalog is the source of truth for listings. Listing state, category integrity, seller ownership, moderation state, and product snapshots need consistency. Product attributes vary by category, so `JSONB` is useful for flexible attributes without giving up relational integrity.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `categories`
- `products`
- `listings`
- `listing_variations`
- `listing_media`
- `listing_moderation_events`
- `outbox_events`

### Published Events

- `catalog.listing.created.v1`
- `catalog.listing.published.v1`
- `catalog.listing.updated.v1`
- `catalog.listing.blocked.v1`

---

## 6.9 `search-service`

### Responsibilities

- keyword search
- category browse
- filters, facets, sorting, and autocomplete
- search index updates from domain events

### Recommended Stack

- `NestJS`
- `OpenSearch`
- official OpenSearch client

### Chosen Database

- `OpenSearch`

### PACELC Rationale

Search is a derived read model. If a partition happens, the system can serve slightly stale results while the source-of-truth services remain consistent.

This service uses a `PA/EL` profile because low-latency reads, relevance, and faceted queries matter more than strict consistency.

### Suggested Data Model

- `listing_search_documents`
- `autocomplete_terms`
- `category_facets`
- `search_synonyms`

### Consumed Events

- `catalog.listing.published.v1`
- `catalog.listing.updated.v1`
- `inventory.stock.adjusted.v1`
- `review.rating.updated.v1`
- `promotion.price.updated.v1`

---

## 6.10 `recommendation-service`

### Responsibilities

- personalized product feeds
- related products
- frequently bought together suggestions
- merchandising feed materialization

### Recommended Stack

- `NestJS`
- `Apache Cassandra`
- Node.js driver for Cassandra

### Chosen Database

- `Apache Cassandra`

### PACELC Rationale

Recommendations are derived data and can be rebuilt from behavior, catalog, search, and order events. Stale recommendations are acceptable; slow or unavailable recommendations reduce user experience.

This service uses a `PA/EL` profile.

### Suggested Data Model

- `recommendations_by_customer`
- `related_products_by_listing`
- `popular_products_by_category`
- `feed_items_by_segment`

### Consumed Events

- `orders.order.created.v1`
- `catalog.listing.published.v1`
- `search.product_viewed.v1`
- `analytics.behavior_aggregated.v1`

---

## 6.11 `cart-service`

### Responsibilities

- active customer carts
- saved-for-later items
- cart validation before checkout
- abandoned cart signals

### Recommended Stack

- `NestJS`
- `Redis`
- official Redis client

### Chosen Database

- `Redis`

### PACELC Rationale

Active cart state is latency-sensitive and temporary. Cart contents can be revalidated against catalog, inventory, promotion, shipping, and seller services before order creation.

This service uses a `PA/EL` profile. Redis is acceptable here because the cart is not the final system of record for orders, inventory, or payments.

### Suggested Data Model

- `cart:{customerId}`
- `guest_cart:{sessionId}`
- `saved_items:{customerId}`
- `cart_validation_snapshots`

### Published Events

- `cart.abandoned.v1`
- `cart.checkout_submitted.v1`

---

## 6.12 `promotion-service`

### Responsibilities

- coupons
- campaign rules
- discount eligibility
- usage limits and promotion status

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Promotion usage must not exceed configured limits. Coupon reservations, campaign status, eligibility, and final price calculation need transactional consistency.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `campaigns`
- `coupons`
- `coupon_redemptions`
- `promotion_rules`
- `promotion_eligible_listings`
- `outbox_events`

### Published Events

- `promotion.campaign.created.v1`
- `promotion.coupon.redeemed.v1`
- `promotion.price.updated.v1`

---

## 6.13 `payment-service`

### Responsibilities

- payment intents
- payment captures and provider webhooks
- refunds
- fraud signals
- reconciliation

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Payments are financial records. Authorization, capture, refund, reconciliation, and provider webhook deduplication must be consistent and auditable.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `payment_intents`
- `payment_transactions`
- `payment_methods`
- `refunds`
- `provider_webhooks`
- `fraud_reviews`
- `outbox_events`

### Published Events

- `payment.authorized.v1`
- `payment.captured.v1`
- `payment.failed.v1`
- `payment.refunded.v1`

---

## 6.14 `return-service`

### Responsibilities

- return eligibility
- returns and exchanges
- disputes and claims
- return inspection outcomes

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Returns affect orders, inventory, payments, and settlements. Eligibility decisions, inspections, disputes, and admin overrides must be consistent and auditable.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `return_cases`
- `return_items`
- `return_messages`
- `return_inspections`
- `dispute_cases`
- `outbox_events`

### Published Events

- `return.requested.v1`
- `return.approved.v1`
- `return.rejected.v1`
- `return.inspected.v1`

---

## 6.15 `review-service`

### Responsibilities

- product reviews
- product ratings
- seller ratings
- moderation state for review content

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Verified-purchase validation, moderation state, rating aggregates, and seller reputation affect marketplace trust.

This service uses a `PC/EC` profile. Aggregates can be cached or projected later, but the review source of truth should remain consistent.

### Suggested Data Model

- `product_reviews`
- `seller_reviews`
- `review_media`
- `review_reports`
- `rating_aggregates`
- `outbox_events`

### Published Events

- `review.product.created.v1`
- `review.product.moderated.v1`
- `review.rating.updated.v1`

---

## 6.16 `question-service`

### Responsibilities

- public product questions
- seller answers
- Q&A moderation
- customer notifications for answers

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Product Q&A needs ownership and moderation consistency. Incorrect answer ownership or moderation state can create trust and compliance issues.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `product_questions`
- `product_answers`
- `question_reports`
- `outbox_events`

### Published Events

- `question.created.v1`
- `question.answered.v1`
- `question.moderated.v1`

---

## 6.17 `settlement-service`

### Responsibilities

- seller ledger
- marketplace commissions and fees
- payout scheduling
- balance adjustments from refunds and disputes

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Settlement is a financial ledger. Payouts, holds, fees, chargebacks, and adjustments must be strongly consistent and auditable.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `seller_ledger_entries`
- `seller_balances`
- `payout_batches`
- `payout_items`
- `commission_rules`
- `outbox_events`

### Published Events

- `settlement.balance_updated.v1`
- `settlement.payout_scheduled.v1`
- `settlement.payout_paid.v1`

---

## 6.18 `support-service`

### Responsibilities

- customer and seller support tickets
- case assignment
- support messages
- escalations linked to orders, returns, listings, or accounts

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Support workflows need consistent ticket assignment, message history, status transitions, and audit trails.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `support_tickets`
- `support_ticket_messages`
- `support_assignments`
- `support_escalations`
- `outbox_events`

### Published Events

- `support.ticket.created.v1`
- `support.ticket.assigned.v1`
- `support.ticket.closed.v1`

---

## 6.19 `admin-service`

### Responsibilities

- backoffice policy configuration
- operational overrides
- moderation queues
- admin audit logs

### Recommended Stack

- `NestJS`
- `PostgreSQL`
- `Prisma ORM`

### Chosen Database

- `PostgreSQL`

### PACELC Rationale

Admin actions can change seller status, listing visibility, return outcomes, and financial workflows. These changes require strong consistency and an explicit audit trail.

This service uses a `PC/EC` profile.

### Suggested Data Model

- `admin_audit_logs`
- `moderation_tasks`
- `policy_configs`
- `operational_overrides`
- `outbox_events`

### Published Events

- `admin.policy.updated.v1`
- `admin.override.applied.v1`
- `admin.moderation.completed.v1`

---

## 6.20 `analytics-service`

### Responsibilities

- marketplace dashboards
- seller reporting
- operational metrics
- event-based analytics and exports

### Recommended Stack

- `NestJS`
- `ClickHouse`
- official ClickHouse client

### Chosen Database

- `ClickHouse`

### PACELC Rationale

Analytics is derived from domain events. It needs fast ingestion, columnar aggregation, and dashboard queries more than strict consistency for every read.

This service uses a `PA/EL` profile.

### Suggested Data Model

- `events_raw`
- `sales_daily_by_seller`
- `listing_performance_daily`
- `marketplace_metrics_hourly`
- `search_behavior_daily`

### Consumed Events

- all relevant domain events from Kafka

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

## 7.8 `OpenSearch`

### Pros

- strong fit for keyword search, relevance, filters, sorting, and facets
- good choice for denormalized listing search documents
- supports rebuilding indexes from Kafka events
- keeps search concerns out of the catalog source of truth

### Cons

- should not become the system of record
- index freshness is eventually consistent
- relevance tuning and index design add operational work
- requires careful event replay and reindexing strategy

## 7.9 `Redis`

### Pros

- very low latency
- TTL support fits temporary active cart state
- simple data structures work well for cart mutation
- useful for cache, rate limiting, locks, and ephemeral workflow state

### Cons

- not a good system of record for strong business domains
- persistence and failover require explicit operational care
- cart state must be revalidated before checkout
- should not own orders, payments, inventory, settlement, or identity records

## 7.10 `ClickHouse`

### Pros

- excellent fit for high-volume analytics inserts
- strong columnar aggregation performance
- useful for dashboards, seller reports, and operational metrics
- works well with event-driven analytics pipelines

### Cons

- not a transactional database
- not a good source of truth for business workflows
- schema and partition design require planning
- data is usually derived and eventually consistent

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
  <service-name>/
    src/
      <domain-module>/
        domain/
          entities/
          value-objects/
          events/
          repositories/
        application/
          commands/
          queries/
          use-cases/
        infrastructure/
          database/
          messaging/
          repositories/
        interfaces/
          http/
          kafka/
          dtos/
        <domain-module>.module.ts
      config/
      main.ts
    test/
    prisma/ # only for services that use Prisma
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

## 8.4 DDD Migration Rule

Existing services can migrate incrementally from the current technical-folder structure. Do not block feature work only to move files. However, every new domain capability should be created inside a domain module using the `domain`, `application`, `infrastructure`, and `interfaces` layers.

When touching existing code for a meaningful business change, prefer moving only the affected flow toward the DDD structure. Avoid large mechanical refactors that do not change behavior or improve a domain boundary.

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
- DDD bounded contexts per microservice
- domain modules organized by aggregate or business capability
- explicit separation between `domain`, `application`, `infrastructure`, and `interfaces`
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

### `Redis` As The System Of Record For Strong Domains

Not recommended for domains that require strong auditability, financial correctness, inventory correctness, or identity consistency.

Reason:

- Redis is better for ephemeral state, cache, rate limiting, and low-latency active cart data
- orders, payments, inventory, settlement, identity, and seller state need stronger transactional guarantees
- cart state must be revalidated before checkout, so Redis does not become the final system of record

## 12. Recommended Roadmap

## Phase 1

- aggregator structure
- local Kafka
- `auth-service`
- `customer-service`
- `seller-service`
- `catalog-service`
- `orders-service`
- `inventory-service`

## Phase 2

- `cart-service`
- `promotion-service`
- `payment-service`
- `shipping-service`
- `notification-service`
- observability
- DLQ and retries

## Phase 3

- `search-service`
- `review-service`
- `question-service`
- `return-service`
- `settlement-service`
- `support-service`
- `admin-service`

## Phase 4

- `recommendation-service`
- `analytics-service`
- reporting exports
- advanced personalization
- extraction of each service into its own repository
- aggregator becomes the repository for Compose, documentation, and local automation

## 13. Final Recommended Decision

If this were my portfolio project, I would follow exactly this combination:

- service base: `Node.js + TypeScript + NestJS`
- messaging integration in NestJS services: `@nestjs/microservices`
- broker: `Apache Kafka`
- `auth-service`: `PostgreSQL + Prisma`
- `customer-service`: `PostgreSQL + Prisma`
- `seller-service`: `PostgreSQL + Prisma`
- `catalog-service`: `PostgreSQL + Prisma + JSONB`
- `search-service`: `OpenSearch + official client`
- `recommendation-service`: `Cassandra + Node.js driver`
- `cart-service`: `Redis + official client`
- `promotion-service`: `PostgreSQL + Prisma`
- `orders-service`: `PostgreSQL + Prisma`
- `payment-service`: `PostgreSQL + Prisma`
- `inventory-service`: `PostgreSQL + Prisma`
- `shipping-service`: `MongoDB + official driver`
- `return-service`: `PostgreSQL + Prisma`
- `review-service`: `PostgreSQL + Prisma`
- `question-service`: `PostgreSQL + Prisma`
- `notification-service`: `Cassandra + Node.js driver`
- `settlement-service`: `PostgreSQL + Prisma`
- `support-service`: `PostgreSQL + Prisma`
- `admin-service`: `PostgreSQL + Prisma`
- `analytics-service`: `ClickHouse + official client`

This uses PostgreSQL as the default consistency-first database, then introduces specialized databases only where PACELC and access patterns justify them.

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
