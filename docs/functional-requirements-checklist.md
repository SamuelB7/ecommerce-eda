# Functional Requirements Checklist

This checklist tracks implementation progress for `docs/functional-requirements.md`.

## 5.1 Identity And Access

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [x] | `FR-AUTH-001` | Customers can register using email and password. | `auth-service` | Implemented with customer signup and outbox event. |
| [x] | `FR-AUTH-002` | Sellers can register a seller-access account before onboarding. | `auth-service` | Implemented with seller signup and seller role. |
| [x] | `FR-AUTH-003` | Users can sign in and receive access and refresh tokens. | `auth-service` | Implemented with JWT access/refresh tokens and MFA challenge support. |
| [x] | `FR-AUTH-004` | Users can refresh tokens and revoke sessions. | `auth-service` | Implemented with refresh rotation, signout, session list, and session revoke. |
| [x] | `FR-AUTH-005` | Users can request password reset and change passwords. | `auth-service` | Implemented with reset tokens, password change, session revocation, and outbox events. |
| [x] | `FR-AUTH-006` | Users can enable and verify multi-factor authentication. | `auth-service` | Implemented with email OTP enrollment and signin challenges. |
| [x] | `FR-AUTH-007` | Admins can assign roles such as customer, seller, support, and admin. | `auth-service` | Implemented with admin-protected role APIs. |
| [x] | `FR-AUTH-008` | Users can deactivate or request deletion of their account. | `auth-service` | Implemented with account status, session revocation, closure request, and outbox events. |

## 5.2 Customer Account

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [x] | `FR-CUSTOMER-001` | Customers can manage profile data. | `customer-service` | Implemented with authenticated profile read/update APIs. |
| [x] | `FR-CUSTOMER-002` | Customers can manage multiple delivery addresses. | `customer-service` | Implemented with address CRUD and default address selection. |
| [x] | `FR-CUSTOMER-003` | Customers can manage communication preferences. | `customer-service` | Implemented with preferences read/update APIs. |
| [x] | `FR-CUSTOMER-004` | Customers can save products to wishlists. | `customer-service` | Implemented with wishlist item add/remove/list and share token APIs. |
| [x] | `FR-CUSTOMER-005` | Customers can follow sellers and stores. | `customer-service` | Implemented with followed seller add/remove/list APIs. |
| [x] | `FR-CUSTOMER-006` | Customers can view their purchase history. | `customer-service` | Implemented with order reference read model and paged listing API. |
| [x] | `FR-CUSTOMER-007` | Customers can request privacy data export. | `customer-service` | Implemented with privacy export request and status APIs. |

## 5.3 Seller Onboarding And Store Management

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-SELLER-001` | Users can apply to become sellers. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-002` | Sellers can submit KYC and business documents. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-003` | Sellers can manage public store profile data. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-004` | Sellers can configure payout bank account data. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-005` | Sellers can manage team members and permissions. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-006` | Sellers can configure handling time and shipping origin. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-007` | Admins can approve, reject, suspend, or reactivate sellers. | `seller-service` | Pending. |
| [ ] | `FR-SELLER-008` | Sellers can view account health and policy warnings. | `seller-service` | Pending. |

## 5.4 Product Catalog And Listings

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-CATALOG-001` | Admins can manage category trees. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-002` | Sellers can create product listings. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-003` | Sellers can update listing content and attributes. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-004` | Sellers can manage product variations. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-005` | Sellers can upload and organize listing media. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-006` | Listings can be published, paused, archived, or blocked. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-007` | Admins can moderate product content. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-008` | Public APIs can expose product detail pages. | `catalog-service` | Pending. |
| [ ] | `FR-CATALOG-009` | Sellers can bulk import and update listings. | `catalog-service` | Pending. |

## 5.5 Search, Browse, And Discovery

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-SEARCH-001` | Customers can search products by keyword. | `search-service` | Pending. |
| [ ] | `FR-SEARCH-002` | Customers can browse by category. | `search-service` | Pending. |
| [ ] | `FR-SEARCH-003` | Customers can filter by attributes, price, seller, rating, shipping, and availability. | `search-service` | Pending. |
| [ ] | `FR-SEARCH-004` | Customers can sort search results. | `search-service` | Pending. |
| [ ] | `FR-SEARCH-005` | Customers can use autocomplete and search suggestions. | `search-service` | Pending. |
| [ ] | `FR-SEARCH-006` | Search index updates when listings, stock, price, or ratings change. | `search-service` | Pending. |
| [ ] | `FR-REC-001` | Customers can receive personalized recommendations. | `recommendation-service` | Pending. |
| [ ] | `FR-REC-002` | Product detail APIs can expose related products. | `recommendation-service` | Pending. |

## 5.6 Pricing, Promotions, And Coupons

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-PROMO-001` | Sellers can set listing prices. | `catalog-service` | Pending. |
| [ ] | `FR-PROMO-002` | Admins and sellers can create promotional campaigns. | `promotion-service` | Pending. |
| [ ] | `FR-PROMO-003` | Customers can apply coupons during checkout. | `promotion-service` | Pending. |
| [ ] | `FR-PROMO-004` | APIs can calculate final price totals. | `promotion-service` | Pending. |
| [ ] | `FR-PROMO-005` | Promotions can enforce usage limits. | `promotion-service` | Pending. |
| [ ] | `FR-PROMO-006` | Free shipping promotions can be applied. | `promotion-service` | Pending. |
| [ ] | `FR-PROMO-007` | Admins can pause or cancel unsafe promotions. | `promotion-service` | Pending. |

## 5.7 Cart And Checkout

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-CART-001` | Customers can create and retrieve a cart. | `cart-service` | Pending. |
| [ ] | `FR-CART-002` | Customers can add, remove, and update item quantities. | `cart-service` | Pending. |
| [ ] | `FR-CART-003` | Customers can save items for later. | `cart-service` | Pending. |
| [ ] | `FR-CART-004` | Checkout can validate price, stock, seller status, and shipping eligibility. | `cart-service` | Pending. |
| [ ] | `FR-CART-005` | Checkout can group items by seller and shipment. | `cart-service` | Pending. |
| [ ] | `FR-CART-006` | Checkout can produce a complete quote. | `cart-service` | Pending. |
| [ ] | `FR-CART-007` | Customers can submit checkout to create an order. | `cart-service` | Pending. |
| [ ] | `FR-CART-008` | System can detect abandoned carts. | `cart-service` | Pending. |

## 5.8 Orders And Order Lifecycle

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-ORDER-001` | Orders can be created from validated checkout. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-002` | Multi-seller checkout can generate seller-specific order groups. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-003` | Orders can move through lifecycle statuses. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-004` | Customers can view order history and order detail. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-005` | Sellers can manage seller order queues. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-006` | Customers can cancel eligible orders. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-007` | Orders expose immutable item snapshots. | `orders-service` | Pending. |
| [ ] | `FR-ORDER-008` | Orders publish domain events for downstream workflows. | `orders-service` | Pending. |

## 5.9 Payments, Refunds, And Fraud Signals

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-PAY-001` | Checkout can create payment intents. | `payment-service` | Pending. |
| [ ] | `FR-PAY-002` | Customers can pay with supported payment methods. | `payment-service` | Pending. |
| [ ] | `FR-PAY-003` | Payments can be authorized, captured, rejected, or expired. | `payment-service` | Pending. |
| [ ] | `FR-PAY-004` | Payment provider webhooks can update transaction state. | `payment-service` | Pending. |
| [ ] | `FR-PAY-005` | Customers and admins can request refunds. | `payment-service` | Pending. |
| [ ] | `FR-PAY-006` | Payments can be split between marketplace and sellers. | `payment-service` | Pending. |
| [ ] | `FR-PAY-007` | Payments can be screened for fraud risk. | `payment-service` | Pending. |
| [ ] | `FR-PAY-008` | Customers can manage stored payment methods. | `payment-service` | Pending. |
| [ ] | `FR-PAY-009` | Finance operators can reconcile payments. | `payment-service` | Pending. |

## 5.10 Inventory And Stock Reservation

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-INV-001` | Sellers can manage stock per SKU. | `inventory-service` | Pending. |
| [ ] | `FR-INV-002` | Public APIs can expose availability. | `inventory-service` | Pending. |
| [ ] | `FR-INV-003` | Checkout can reserve stock before payment completion. | `inventory-service` | Pending. |
| [ ] | `FR-INV-004` | Reservations can be confirmed when orders are paid. | `inventory-service` | Pending. |
| [ ] | `FR-INV-005` | Reservations can be released on cancellation, payment failure, or timeout. | `inventory-service` | Pending. |
| [ ] | `FR-INV-006` | Sellers can view stock movement history. | `inventory-service` | Pending. |
| [ ] | `FR-INV-007` | Sellers can receive low-stock alerts. | `inventory-service` | Pending. |
| [ ] | `FR-INV-008` | Admins can audit or adjust stock. | `inventory-service` | Pending. |

## 5.11 Shipping, Delivery, And Tracking

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-SHIP-001` | Customers can request shipping quotes. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-002` | Sellers can configure shipping origins and fulfillment options. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-003` | Orders can create shipments after payment approval. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-004` | Sellers can generate shipping labels. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-005` | Shipment tracking can be updated by carriers or sellers. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-006` | Customers can track deliveries. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-007` | Delivery confirmation can update order status. | `shipping-service` | Pending. |
| [ ] | `FR-SHIP-008` | Shipping addresses can be validated. | `shipping-service` | Pending. |

## 5.12 Returns, Cancellations, And Disputes

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-RETURN-001` | Customers can check return eligibility. | `return-service` | Pending. |
| [ ] | `FR-RETURN-002` | Customers can open return or exchange requests. | `return-service` | Pending. |
| [ ] | `FR-RETURN-003` | Sellers can approve, reject, or request more information. | `return-service` | Pending. |
| [ ] | `FR-RETURN-004` | Return labels can be generated. | `return-service` | Pending. |
| [ ] | `FR-RETURN-005` | Returned items can be inspected. | `return-service` | Pending. |
| [ ] | `FR-RETURN-006` | Approved returns can trigger refunds or exchanges. | `return-service` | Pending. |
| [ ] | `FR-RETURN-007` | Customers and sellers can open disputes. | `return-service` | Pending. |
| [ ] | `FR-RETURN-008` | Admins can override return or dispute outcomes. | `return-service` | Pending. |

## 5.13 Reviews, Ratings, And Product Questions

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-REVIEW-001` | Customers can review purchased products. | `review-service` | Pending. |
| [ ] | `FR-REVIEW-002` | Reviews can include rating, text, images, and moderation status. | `review-service` | Pending. |
| [ ] | `FR-REVIEW-003` | Product pages can display rating aggregates. | `review-service` | Pending. |
| [ ] | `FR-REVIEW-004` | Customers can report abusive reviews. | `review-service` | Pending. |
| [ ] | `FR-REVIEW-005` | Sellers can receive seller ratings. | `review-service` | Pending. |
| [ ] | `FR-QUESTION-001` | Customers can ask public product questions. | `question-service` | Pending. |
| [ ] | `FR-QUESTION-002` | Sellers can answer product questions. | `question-service` | Pending. |
| [ ] | `FR-QUESTION-003` | Admins can moderate questions and answers. | `question-service` | Pending. |

## 5.14 Notifications

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-NOTIF-001` | System can send transactional notifications. | `notification-service` | Pending. |
| [ ] | `FR-NOTIF-002` | Customers can receive order, payment, shipping, return, and promotion updates. | `notification-service` | Pending. |
| [ ] | `FR-NOTIF-003` | Sellers can receive order, stock, return, and account health alerts. | `notification-service` | Pending. |
| [ ] | `FR-NOTIF-004` | Admins can manage notification templates. | `notification-service` | Pending. |
| [ ] | `FR-NOTIF-005` | Notification delivery attempts are tracked. | `notification-service` | Pending. |
| [ ] | `FR-NOTIF-006` | Notification preferences are enforced. | `notification-service` | Pending. |

## 5.15 Support, Moderation, And Administration

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-SUPPORT-001` | Customers and sellers can open support tickets. | `support-service` | Pending. |
| [ ] | `FR-SUPPORT-002` | Support agents can manage ticket queues. | `support-service` | Pending. |
| [ ] | `FR-SUPPORT-003` | Support agents can message customers and sellers. | `support-service` | Pending. |
| [ ] | `FR-ADMIN-001` | Admins can search customers, sellers, listings, orders, and payments. | `admin-service` | Pending. |
| [ ] | `FR-ADMIN-002` | Admins can moderate listings, reviews, questions, and seller accounts. | `admin-service` | Pending. |
| [ ] | `FR-ADMIN-003` | Admins can execute operational overrides. | `admin-service` | Pending. |
| [ ] | `FR-ADMIN-004` | Admins can manage marketplace policies. | `admin-service` | Pending. |
| [ ] | `FR-ADMIN-005` | Admin actions are audited. | `admin-service` | Pending. |

## 5.16 Seller Settlement And Financial Reporting

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-SETTLE-001` | Seller balances can be calculated from orders, refunds, fees, and holds. | `settlement-service` | Pending. |
| [ ] | `FR-SETTLE-002` | Marketplace commissions and seller fees can be applied. | `settlement-service` | Pending. |
| [ ] | `FR-SETTLE-003` | Payouts can be scheduled and executed. | `settlement-service` | Pending. |
| [ ] | `FR-SETTLE-004` | Funds can be held until delivery or return window completion. | `settlement-service` | Pending. |
| [ ] | `FR-SETTLE-005` | Sellers can download payout reports. | `settlement-service` | Pending. |
| [ ] | `FR-SETTLE-006` | Chargebacks, refunds, and disputes can adjust seller balances. | `settlement-service` | Pending. |

## 5.17 Analytics And Operational Reports

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-ANALYTICS-001` | Sellers can view sales dashboards. | `analytics-service` | Pending. |
| [ ] | `FR-ANALYTICS-002` | Sellers can view inventory and demand reports. | `analytics-service` | Pending. |
| [ ] | `FR-ANALYTICS-003` | Sellers can view listing performance. | `analytics-service` | Pending. |
| [ ] | `FR-ANALYTICS-004` | Admins can view marketplace operational metrics. | `analytics-service` | Pending. |
| [ ] | `FR-ANALYTICS-005` | Product and search behavior can feed recommendations. | `analytics-service` | Pending. |
| [ ] | `FR-ANALYTICS-006` | Reports can be exported. | `analytics-service` | Pending. |

## 7. Cross-Domain API Requirements

| Done | ID | Requirement | Primary Service | Notes |
| --- | --- | --- | --- | --- |
| [ ] | `FR-CROSS-001` | APIs that create orders, payments, stock reservations, refunds, and payouts must support idempotent commands. | Owning service | Pending. |
| [ ] | `FR-CROSS-002` | List APIs must support pagination. | Owning service | Pending. |
| [ ] | `FR-CROSS-003` | Sensitive APIs must enforce actor permissions. | `auth-service` | Pending outside implemented auth role APIs. |
| [ ] | `FR-CROSS-004` | Important business actions must publish domain events. | Owning service | Pending outside implemented auth outbox events. |
| [ ] | `FR-CROSS-005` | Important state changes must be auditable. | Owning service | Pending. |
