# Functional Requirements

## 1. Overview

This document maps the functional API requirements for a marketplace e-commerce platform similar to Amazon and Mercado Livre. The platform supports customers, third-party sellers, administrators, support agents, and system automation.

The scope is API behavior only. Frontend applications will be planned later for the e-commerce site, seller panel, and admin panel. Requirements are mapped to existing services when available and to proposed services when the current repository does not yet contain the required domain owner.

## 2. Service Ownership Legend

### Existing Services

| Service | Ownership |
| --- | --- |
| `auth-service` | Identity, authentication, sessions, roles, access control, identity events. |
| `orders-service` | Order creation, order lifecycle, order history, seller order views, order events. |
| `inventory-service` | Stock records, stock availability, stock reservations, stock release, inventory events. |
| `shipping-service` | Shipping quotes, shipment creation, labels, tracking, delivery events. |
| `notification-service` | Email, SMS, push, in-app notification dispatch, templates, delivery history. |

### Proposed Services

| Service | Ownership |
| --- | --- |
| `customer-service` | Customer profile, addresses, preferences, wishlists, followed sellers. |
| `seller-service` | Seller onboarding, store profile, KYC, seller users, seller status. |
| `catalog-service` | Products, categories, listings, variations, attributes, listing media. |
| `search-service` | Search index, keyword search, filters, facets, sorting, autocomplete. |
| `recommendation-service` | Personalized recommendations, related products, merchandising feeds. |
| `cart-service` | Customer cart, saved items, checkout preparation, cart validation. |
| `promotion-service` | Coupons, campaigns, discounts, price calculation, promotion eligibility. |
| `payment-service` | Payment intents, captures, refunds, stored methods, webhooks, reconciliation. |
| `return-service` | Returns, exchanges, claims, disputes, return labels, return inspections. |
| `review-service` | Product reviews, ratings, seller ratings, moderation signals. |
| `question-service` | Product questions, seller answers, Q&A moderation. |
| `settlement-service` | Seller balances, commissions, payout schedules, payout reports. |
| `support-service` | Support tickets, case assignment, customer-seller-admin escalations. |
| `admin-service` | Backoffice workflows, moderation queues, operational controls, audit logs. |
| `analytics-service` | Business reports, seller metrics, operational metrics, event-based analytics. |

## 3. Actors And API Consumers

| Actor | API Consumer |
| --- | --- |
| `Customer` | E-commerce site, mobile app, support flows. |
| `Seller` | Seller panel, seller integrations. |
| `Admin` | Admin panel, moderation tools, operational tools. |
| `Support Agent` | Support panel, ticket workflows. |
| `System` | Scheduled jobs, event consumers, payment and shipping webhooks. |

## 4. Requirement Mapping Rules

- `Primary Service` owns the public API capability and domain state.
- `Supporting Services` provide validation, orchestration data, events, or side effects.
- External API consumers should use HTTP APIs. Cross-service business integration should prefer Kafka events.
- Requirement IDs are stable documentation identifiers and should be reused in future API specs, tickets, and tests.

## 5. Functional Requirements Matrix

### 5.1 Identity And Access

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-AUTH-001` | Customers can register using email and password. | `auth-service` | `customer-service`, `notification-service` | Create customer identity and send confirmation. |
| `FR-AUTH-002` | Sellers can register a seller-access account before onboarding. | `auth-service` | `seller-service`, `notification-service` | Create seller identity and start onboarding. |
| `FR-AUTH-003` | Users can sign in and receive access and refresh tokens. | `auth-service` | - | Authenticate credentials and issue tokens. |
| `FR-AUTH-004` | Users can refresh tokens and revoke sessions. | `auth-service` | - | Refresh, revoke, and list active sessions. |
| `FR-AUTH-005` | Users can request password reset and change passwords. | `auth-service` | `notification-service` | Create reset token, verify token, update password. |
| `FR-AUTH-006` | Users can enable and verify multi-factor authentication. | `auth-service` | `notification-service` | Enroll, challenge, verify, disable MFA. |
| `FR-AUTH-007` | Admins can assign roles such as customer, seller, support, and admin. | `auth-service` | `admin-service` | Manage roles and permission grants. |
| `FR-AUTH-008` | Users can deactivate or request deletion of their account. | `auth-service` | `customer-service`, `seller-service`, `orders-service` | Start account closure workflow with eligibility checks. |

### 5.2 Customer Account

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-CUSTOMER-001` | Customers can manage profile data. | `customer-service` | `auth-service` | Read and update profile fields. |
| `FR-CUSTOMER-002` | Customers can manage multiple delivery addresses. | `customer-service` | `shipping-service` | Create, update, delete, validate, and choose default addresses. |
| `FR-CUSTOMER-003` | Customers can manage communication preferences. | `customer-service` | `notification-service` | Update opt-in and opt-out settings. |
| `FR-CUSTOMER-004` | Customers can save products to wishlists. | `customer-service` | `catalog-service` | Add, remove, list, and share wishlist items. |
| `FR-CUSTOMER-005` | Customers can follow sellers and stores. | `customer-service` | `seller-service`, `notification-service` | Follow, unfollow, and receive seller updates. |
| `FR-CUSTOMER-006` | Customers can view their purchase history. | `customer-service` | `orders-service` | Retrieve customer order references and summaries. |
| `FR-CUSTOMER-007` | Customers can request privacy data export. | `customer-service` | `auth-service`, `orders-service`, `payment-service` | Start export workflow and expose export status. |

### 5.3 Seller Onboarding And Store Management

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-SELLER-001` | Users can apply to become sellers. | `seller-service` | `auth-service`, `notification-service` | Create seller application and onboarding status. |
| `FR-SELLER-002` | Sellers can submit KYC and business documents. | `seller-service` | `admin-service` | Upload document metadata and track review status. |
| `FR-SELLER-003` | Sellers can manage public store profile data. | `seller-service` | `catalog-service` | Update store name, description, logo, and policies. |
| `FR-SELLER-004` | Sellers can configure payout bank account data. | `seller-service` | `settlement-service` | Create and update payout account references. |
| `FR-SELLER-005` | Sellers can manage team members and permissions. | `seller-service` | `auth-service` | Invite, remove, and assign seller team roles. |
| `FR-SELLER-006` | Sellers can configure handling time and shipping origin. | `seller-service` | `shipping-service` | Manage fulfillment settings used by quotes and orders. |
| `FR-SELLER-007` | Admins can approve, reject, suspend, or reactivate sellers. | `seller-service` | `admin-service`, `notification-service` | Change seller operational status. |
| `FR-SELLER-008` | Sellers can view account health and policy warnings. | `seller-service` | `analytics-service`, `admin-service` | Retrieve seller performance and compliance status. |

### 5.4 Product Catalog And Listings

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-CATALOG-001` | Admins can manage category trees. | `catalog-service` | `admin-service`, `search-service` | Create, update, reorder, and deactivate categories. |
| `FR-CATALOG-002` | Sellers can create product listings. | `catalog-service` | `seller-service`, `inventory-service` | Create listing drafts with title, description, images, attributes, and SKU data. |
| `FR-CATALOG-003` | Sellers can update listing content and attributes. | `catalog-service` | `search-service` | Update listing fields and reindex searchable data. |
| `FR-CATALOG-004` | Sellers can manage product variations. | `catalog-service` | `inventory-service` | Create variation groups such as size, color, model, or package quantity. |
| `FR-CATALOG-005` | Sellers can upload and organize listing media. | `catalog-service` | `admin-service` | Attach images and videos, set primary media, remove media. |
| `FR-CATALOG-006` | Listings can be published, paused, archived, or blocked. | `catalog-service` | `seller-service`, `admin-service`, `search-service` | Change listing lifecycle state. |
| `FR-CATALOG-007` | Admins can moderate product content. | `catalog-service` | `admin-service`, `notification-service` | Approve, reject, block, and request listing changes. |
| `FR-CATALOG-008` | Public APIs can expose product detail pages. | `catalog-service` | `inventory-service`, `promotion-service`, `review-service`, `shipping-service` | Return listing detail, price, stock, rating, and shipping summary. |
| `FR-CATALOG-009` | Sellers can bulk import and update listings. | `catalog-service` | `inventory-service`, `seller-service` | Submit bulk jobs and retrieve import validation results. |

### 5.5 Search, Browse, And Discovery

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-SEARCH-001` | Customers can search products by keyword. | `search-service` | `catalog-service`, `inventory-service`, `promotion-service` | Search listings and return ranked results. |
| `FR-SEARCH-002` | Customers can browse by category. | `search-service` | `catalog-service` | List category results and subcategory navigation. |
| `FR-SEARCH-003` | Customers can filter by attributes, price, seller, rating, shipping, and availability. | `search-service` | `catalog-service`, `inventory-service`, `review-service`, `shipping-service` | Apply filters and return facets. |
| `FR-SEARCH-004` | Customers can sort search results. | `search-service` | `promotion-service`, `review-service` | Sort by relevance, price, rating, newest, and best seller. |
| `FR-SEARCH-005` | Customers can use autocomplete and search suggestions. | `search-service` | `analytics-service` | Return suggested terms and popular searches. |
| `FR-SEARCH-006` | Search index updates when listings, stock, price, or ratings change. | `search-service` | `catalog-service`, `inventory-service`, `promotion-service`, `review-service` | Consume events and update indexed documents. |
| `FR-REC-001` | Customers can receive personalized recommendations. | `recommendation-service` | `analytics-service`, `catalog-service`, `orders-service` | Return personalized product feeds. |
| `FR-REC-002` | Product detail APIs can expose related products. | `recommendation-service` | `catalog-service`, `search-service` | Return similar, complementary, and frequently bought together products. |

### 5.6 Pricing, Promotions, And Coupons

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-PROMO-001` | Sellers can set listing prices. | `catalog-service` | `promotion-service`, `seller-service` | Update base price and publish price change events. |
| `FR-PROMO-002` | Admins and sellers can create promotional campaigns. | `promotion-service` | `seller-service`, `admin-service`, `catalog-service` | Create campaign rules, schedules, and eligible listings. |
| `FR-PROMO-003` | Customers can apply coupons during checkout. | `promotion-service` | `cart-service`, `orders-service` | Validate coupon eligibility and reserve coupon usage. |
| `FR-PROMO-004` | APIs can calculate final price totals. | `promotion-service` | `cart-service`, `catalog-service`, `shipping-service` | Calculate item discounts, cart discounts, shipping discounts, and total savings. |
| `FR-PROMO-005` | Promotions can enforce usage limits. | `promotion-service` | `customer-service`, `orders-service` | Track per-customer and global usage. |
| `FR-PROMO-006` | Free shipping promotions can be applied. | `promotion-service` | `shipping-service`, `cart-service` | Evaluate shipping discount rules. |
| `FR-PROMO-007` | Admins can pause or cancel unsafe promotions. | `promotion-service` | `admin-service`, `notification-service` | Change campaign status and notify affected sellers. |

### 5.7 Cart And Checkout

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-CART-001` | Customers can create and retrieve a cart. | `cart-service` | `auth-service`, `customer-service` | Create, read, merge guest cart into customer cart. |
| `FR-CART-002` | Customers can add, remove, and update item quantities. | `cart-service` | `catalog-service`, `inventory-service` | Mutate cart items and validate listing status. |
| `FR-CART-003` | Customers can save items for later. | `cart-service` | `catalog-service` | Move items between active cart and saved list. |
| `FR-CART-004` | Checkout can validate price, stock, seller status, and shipping eligibility. | `cart-service` | `catalog-service`, `inventory-service`, `seller-service`, `shipping-service`, `promotion-service` | Run pre-checkout validation. |
| `FR-CART-005` | Checkout can group items by seller and shipment. | `cart-service` | `shipping-service`, `seller-service` | Build checkout groups for multi-seller orders. |
| `FR-CART-006` | Checkout can produce a complete quote. | `cart-service` | `promotion-service`, `shipping-service`, `payment-service` | Return item totals, discounts, shipping, taxes or fees, and payable total. |
| `FR-CART-007` | Customers can submit checkout to create an order. | `cart-service` | `orders-service`, `inventory-service`, `payment-service` | Submit idempotent checkout command and receive order reference. |
| `FR-CART-008` | System can detect abandoned carts. | `cart-service` | `notification-service`, `analytics-service` | Publish abandoned cart events and trigger recovery notifications. |

### 5.8 Orders And Order Lifecycle

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-ORDER-001` | Orders can be created from validated checkout. | `orders-service` | `cart-service`, `inventory-service`, `payment-service` | Create order and initial order items idempotently. |
| `FR-ORDER-002` | Multi-seller checkout can generate seller-specific order groups. | `orders-service` | `seller-service`, `shipping-service` | Split order into seller fulfillment groups. |
| `FR-ORDER-003` | Orders can move through lifecycle statuses. | `orders-service` | `payment-service`, `shipping-service`, `return-service` | Track pending, paid, preparing, shipped, delivered, canceled, returned, and refunded states. |
| `FR-ORDER-004` | Customers can view order history and order detail. | `orders-service` | `shipping-service`, `payment-service`, `catalog-service` | Return order list, detail, payment summary, shipment summary, and item snapshots. |
| `FR-ORDER-005` | Sellers can manage seller order queues. | `orders-service` | `seller-service`, `shipping-service` | List seller orders, acknowledge preparation, and mark ready to ship. |
| `FR-ORDER-006` | Customers can cancel eligible orders. | `orders-service` | `inventory-service`, `payment-service`, `shipping-service`, `notification-service` | Validate cancellation, update status, release stock, and trigger refund when needed. |
| `FR-ORDER-007` | Orders expose immutable item snapshots. | `orders-service` | `catalog-service` | Preserve title, SKU, seller, price, quantity, and listing metadata at purchase time. |
| `FR-ORDER-008` | Orders publish domain events for downstream workflows. | `orders-service` | `inventory-service`, `shipping-service`, `notification-service`, `settlement-service` | Publish created, paid, canceled, shipped, delivered, returned, and refunded events. |

### 5.9 Payments, Refunds, And Fraud Signals

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-PAY-001` | Checkout can create payment intents. | `payment-service` | `cart-service`, `orders-service` | Create payable transaction reference. |
| `FR-PAY-002` | Customers can pay with supported payment methods. | `payment-service` | `customer-service` | Process card, wallet, bank slip, instant payment, or other configured methods. |
| `FR-PAY-003` | Payments can be authorized, captured, rejected, or expired. | `payment-service` | `orders-service`, `notification-service` | Update payment status and notify order workflow. |
| `FR-PAY-004` | Payment provider webhooks can update transaction state. | `payment-service` | `orders-service`, `settlement-service` | Validate webhook, deduplicate event, update transaction. |
| `FR-PAY-005` | Customers and admins can request refunds. | `payment-service` | `orders-service`, `return-service`, `admin-service` | Create refund request, process refund, expose refund status. |
| `FR-PAY-006` | Payments can be split between marketplace and sellers. | `payment-service` | `settlement-service`, `seller-service`, `orders-service` | Calculate gross amount, fees, commissions, and seller net amount. |
| `FR-PAY-007` | Payments can be screened for fraud risk. | `payment-service` | `orders-service`, `customer-service`, `admin-service` | Score transaction and hold or release order flow. |
| `FR-PAY-008` | Customers can manage stored payment methods. | `payment-service` | `customer-service`, `auth-service` | Tokenize, list, set default, and remove payment methods. |
| `FR-PAY-009` | Finance operators can reconcile payments. | `payment-service` | `settlement-service`, `analytics-service` | Compare provider transactions, orders, refunds, and payouts. |

### 5.10 Inventory And Stock Reservation

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-INV-001` | Sellers can manage stock per SKU. | `inventory-service` | `seller-service`, `catalog-service` | Create, update, import, and list stock records. |
| `FR-INV-002` | Public APIs can expose availability. | `inventory-service` | `catalog-service`, `search-service` | Return sellable quantity or availability state. |
| `FR-INV-003` | Checkout can reserve stock before payment completion. | `inventory-service` | `cart-service`, `orders-service` | Create reservation with expiration. |
| `FR-INV-004` | Reservations can be confirmed when orders are paid. | `inventory-service` | `orders-service`, `payment-service` | Convert reserved stock into committed stock movement. |
| `FR-INV-005` | Reservations can be released on cancellation, payment failure, or timeout. | `inventory-service` | `orders-service`, `payment-service` | Release reserved quantity and publish availability changes. |
| `FR-INV-006` | Sellers can view stock movement history. | `inventory-service` | `orders-service`, `return-service` | List adjustments, reservations, releases, sales, and returns. |
| `FR-INV-007` | Sellers can receive low-stock alerts. | `inventory-service` | `notification-service`, `seller-service` | Configure thresholds and publish alert events. |
| `FR-INV-008` | Admins can audit or adjust stock. | `inventory-service` | `admin-service` | Record manual adjustments with reason and actor. |

### 5.11 Shipping, Delivery, And Tracking

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-SHIP-001` | Customers can request shipping quotes. | `shipping-service` | `cart-service`, `catalog-service`, `seller-service`, `promotion-service` | Quote delivery methods, price, and estimated dates. |
| `FR-SHIP-002` | Sellers can configure shipping origins and fulfillment options. | `shipping-service` | `seller-service` | Manage shipping profile, handling time, pickup, and drop-off options. |
| `FR-SHIP-003` | Orders can create shipments after payment approval. | `shipping-service` | `orders-service`, `seller-service` | Create shipment records for order groups. |
| `FR-SHIP-004` | Sellers can generate shipping labels. | `shipping-service` | `orders-service`, `payment-service` | Buy or reserve label and expose printable label URL. |
| `FR-SHIP-005` | Shipment tracking can be updated by carriers or sellers. | `shipping-service` | `orders-service`, `notification-service` | Receive tracking events and update shipment status. |
| `FR-SHIP-006` | Customers can track deliveries. | `shipping-service` | `orders-service` | Return shipment timeline and current delivery estimate. |
| `FR-SHIP-007` | Delivery confirmation can update order status. | `shipping-service` | `orders-service`, `settlement-service`, `notification-service` | Publish delivered event and trigger downstream workflows. |
| `FR-SHIP-008` | Shipping addresses can be validated. | `shipping-service` | `customer-service` | Validate address format, serviceability, and delivery restrictions. |

### 5.12 Returns, Cancellations, And Disputes

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-RETURN-001` | Customers can check return eligibility. | `return-service` | `orders-service`, `seller-service`, `catalog-service` | Evaluate return window, item status, policy, and reason. |
| `FR-RETURN-002` | Customers can open return or exchange requests. | `return-service` | `orders-service`, `shipping-service`, `notification-service` | Create return case and return item records. |
| `FR-RETURN-003` | Sellers can approve, reject, or request more information. | `return-service` | `seller-service`, `support-service`, `notification-service` | Manage seller-side return decision. |
| `FR-RETURN-004` | Return labels can be generated. | `return-service` | `shipping-service` | Create reverse-shipping label and tracking reference. |
| `FR-RETURN-005` | Returned items can be inspected. | `return-service` | `inventory-service`, `seller-service` | Record condition and decide restock, refund, or dispute. |
| `FR-RETURN-006` | Approved returns can trigger refunds or exchanges. | `return-service` | `payment-service`, `orders-service`, `inventory-service` | Orchestrate refund, exchange order, and stock movement. |
| `FR-RETURN-007` | Customers and sellers can open disputes. | `return-service` | `support-service`, `admin-service`, `orders-service` | Create claim case and track resolution. |
| `FR-RETURN-008` | Admins can override return or dispute outcomes. | `return-service` | `admin-service`, `payment-service`, `notification-service` | Apply final decision with audit reason. |

### 5.13 Reviews, Ratings, And Product Questions

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-REVIEW-001` | Customers can review purchased products. | `review-service` | `orders-service`, `catalog-service` | Validate purchase and create review. |
| `FR-REVIEW-002` | Reviews can include rating, text, images, and moderation status. | `review-service` | `catalog-service`, `admin-service` | Create, update, hide, and moderate review content. |
| `FR-REVIEW-003` | Product pages can display rating aggregates. | `review-service` | `catalog-service`, `search-service` | Return average rating, count, and distribution. |
| `FR-REVIEW-004` | Customers can report abusive reviews. | `review-service` | `admin-service` | Create moderation report and update review state. |
| `FR-REVIEW-005` | Sellers can receive seller ratings. | `review-service` | `orders-service`, `seller-service` | Create seller rating after purchase and expose score. |
| `FR-QUESTION-001` | Customers can ask public product questions. | `question-service` | `catalog-service`, `seller-service` | Create question for listing. |
| `FR-QUESTION-002` | Sellers can answer product questions. | `question-service` | `seller-service`, `notification-service` | Create seller answer and notify interested customers. |
| `FR-QUESTION-003` | Admins can moderate questions and answers. | `question-service` | `admin-service` | Hide, restore, or remove Q&A content. |

### 5.14 Notifications

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-NOTIF-001` | System can send transactional notifications. | `notification-service` | All business services | Send email, SMS, push, or in-app messages from domain events. |
| `FR-NOTIF-002` | Customers can receive order, payment, shipping, return, and promotion updates. | `notification-service` | `orders-service`, `payment-service`, `shipping-service`, `return-service`, `promotion-service` | Deliver customer lifecycle messages. |
| `FR-NOTIF-003` | Sellers can receive order, stock, return, and account health alerts. | `notification-service` | `orders-service`, `inventory-service`, `return-service`, `seller-service` | Deliver seller operational messages. |
| `FR-NOTIF-004` | Admins can manage notification templates. | `notification-service` | `admin-service` | Create, update, test, and publish templates. |
| `FR-NOTIF-005` | Notification delivery attempts are tracked. | `notification-service` | `analytics-service` | Store delivery status, retries, failures, and provider references. |
| `FR-NOTIF-006` | Notification preferences are enforced. | `notification-service` | `customer-service`, `seller-service` | Check opt-in rules before optional messages. |

### 5.15 Support, Moderation, And Administration

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-SUPPORT-001` | Customers and sellers can open support tickets. | `support-service` | `orders-service`, `seller-service`, `customer-service` | Create ticket linked to order, return, listing, or account. |
| `FR-SUPPORT-002` | Support agents can manage ticket queues. | `support-service` | `admin-service` | Assign, prioritize, comment, close, and reopen tickets. |
| `FR-SUPPORT-003` | Support agents can message customers and sellers. | `support-service` | `notification-service` | Send and store case messages. |
| `FR-ADMIN-001` | Admins can search customers, sellers, listings, orders, and payments. | `admin-service` | `auth-service`, `customer-service`, `seller-service`, `catalog-service`, `orders-service`, `payment-service` | Provide backoffice lookup APIs. |
| `FR-ADMIN-002` | Admins can moderate listings, reviews, questions, and seller accounts. | `admin-service` | `catalog-service`, `review-service`, `question-service`, `seller-service` | Approve, reject, hide, block, or restore entities. |
| `FR-ADMIN-003` | Admins can execute operational overrides. | `admin-service` | `orders-service`, `payment-service`, `inventory-service`, `return-service` | Apply audited corrections with reason codes. |
| `FR-ADMIN-004` | Admins can manage marketplace policies. | `admin-service` | `seller-service`, `return-service`, `promotion-service` | Configure seller rules, return rules, and moderation policies. |
| `FR-ADMIN-005` | Admin actions are audited. | `admin-service` | All business services | Record actor, target, action, timestamp, and reason. |

### 5.16 Seller Settlement And Financial Reporting

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-SETTLE-001` | Seller balances can be calculated from orders, refunds, fees, and holds. | `settlement-service` | `orders-service`, `payment-service`, `return-service`, `seller-service` | Maintain seller ledger and balance views. |
| `FR-SETTLE-002` | Marketplace commissions and seller fees can be applied. | `settlement-service` | `payment-service`, `admin-service` | Calculate commission, fixed fees, promotion fees, and adjustments. |
| `FR-SETTLE-003` | Payouts can be scheduled and executed. | `settlement-service` | `seller-service`, `payment-service` | Create payout batches and track payout status. |
| `FR-SETTLE-004` | Funds can be held until delivery or return window completion. | `settlement-service` | `orders-service`, `shipping-service`, `return-service` | Apply hold rules and release eligibility. |
| `FR-SETTLE-005` | Sellers can download payout reports. | `settlement-service` | `analytics-service` | Export settlement summaries and transaction-level details. |
| `FR-SETTLE-006` | Chargebacks, refunds, and disputes can adjust seller balances. | `settlement-service` | `payment-service`, `return-service`, `admin-service` | Apply ledger adjustments with audit trail. |

### 5.17 Analytics And Operational Reports

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-ANALYTICS-001` | Sellers can view sales dashboards. | `analytics-service` | `orders-service`, `payment-service`, `catalog-service` | Return revenue, units sold, average ticket, and conversion metrics. |
| `FR-ANALYTICS-002` | Sellers can view inventory and demand reports. | `analytics-service` | `inventory-service`, `orders-service`, `search-service` | Return low stock, sell-through, and demand signals. |
| `FR-ANALYTICS-003` | Sellers can view listing performance. | `analytics-service` | `catalog-service`, `search-service`, `review-service` | Return impressions, clicks, conversion, rating, and search performance. |
| `FR-ANALYTICS-004` | Admins can view marketplace operational metrics. | `analytics-service` | All business services | Return GMV, order volume, refund rate, dispute rate, delivery SLA, and seller health. |
| `FR-ANALYTICS-005` | Product and search behavior can feed recommendations. | `analytics-service` | `recommendation-service`, `search-service`, `catalog-service` | Collect and expose behavioral aggregates. |
| `FR-ANALYTICS-006` | Reports can be exported. | `analytics-service` | `seller-service`, `admin-service` | Generate CSV or spreadsheet exports for authorized users. |

## 6. Core API Workflows

### 6.1 Customer Registration

1. `auth-service` creates identity and credentials.
2. `customer-service` creates customer profile.
3. `notification-service` sends confirmation message.

### 6.2 Seller Onboarding

1. `auth-service` creates seller-access identity.
2. `seller-service` creates seller application and store draft.
3. `seller-service` receives KYC and payout information.
4. `admin-service` reviews application.
5. `notification-service` informs approval, rejection, or requested changes.

### 6.3 Listing Publication

1. `seller-service` validates seller is active.
2. `catalog-service` creates or updates listing.
3. `inventory-service` creates initial stock records.
4. `admin-service` may moderate listing.
5. `search-service` indexes listing when publishable.

### 6.4 Checkout And Order Creation

1. `cart-service` validates cart, stock, price, promotions, shipping, and seller eligibility.
2. `inventory-service` reserves stock.
3. `orders-service` creates order and seller groups.
4. `payment-service` creates and processes payment.
5. `orders-service`, `shipping-service`, `notification-service`, and `settlement-service` react to payment approval events.

### 6.5 Delivery And Settlement

1. `shipping-service` creates shipment and tracking timeline.
2. `orders-service` updates order state from shipment events.
3. `settlement-service` holds funds until settlement eligibility.
4. `settlement-service` pays sellers according to payout schedule.

### 6.6 Return And Refund

1. `return-service` validates return eligibility.
2. `shipping-service` creates return shipment when needed.
3. `return-service` records inspection result.
4. `payment-service` processes refund.
5. `inventory-service` restocks eligible items.
6. `settlement-service` adjusts seller balance.

## 7. Cross-Domain API Requirements

| ID | Functional Requirement | Primary Service | Supporting Services | API Capability |
| --- | --- | --- | --- | --- |
| `FR-CROSS-001` | APIs that create orders, payments, stock reservations, refunds, and payouts must support idempotent commands. | Owning service | API caller | Accept idempotency key and return prior result for repeated commands. |
| `FR-CROSS-002` | List APIs must support pagination. | Owning service | - | Return paged results and stable cursors or page metadata. |
| `FR-CROSS-003` | Sensitive APIs must enforce actor permissions. | `auth-service` | Owning service | Validate roles, ownership, and service-specific access rules. |
| `FR-CROSS-004` | Important business actions must publish domain events. | Owning service | Kafka consumers | Publish versioned events using `<domain>.<aggregate>.<event>.v1`. |
| `FR-CROSS-005` | Important state changes must be auditable. | Owning service | `admin-service`, `analytics-service` | Record actor, action, target, reason, and timestamp. |

