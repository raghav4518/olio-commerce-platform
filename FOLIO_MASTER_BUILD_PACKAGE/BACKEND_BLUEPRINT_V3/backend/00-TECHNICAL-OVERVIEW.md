# Backend 00 — Technical Overview and Engineering Guardrails

**Status:** Proposed baseline pending repository audit and owner approval of open decisions.  
**System:** Single-vendor ebook commerce platform, three interfaces, one shared business backend.

## 1. System objective

Build a reliable digital-commerce system in which:

- Customers discover ebooks, create or access an account, purchase books, receive entitlements, read or download authorized files, and manage their orders.
- Owners control catalog, commerce, customers, payments, refunds, website content, marketing, reporting and system configuration.
- Support staff investigate customer and order problems using least-privilege permissions.

All three interfaces use the same canonical business records and business rules. No panel may implement its own payment, order, refund or entitlement logic.

## 2. Initial scope boundary

### In scope

- One owner-operated storefront for the initial release.
- Digital products only; no physical stock, shipping labels or warehouse system.
- Customer, Owner/Admin and Support/Operations interfaces.
- Catalog, category, author, collection, price, cart, checkout, order, payment, refund and entitlement domains.
- Private ebook asset ingestion and delivery.
- Customer library and supported reader/download workflows.
- Support tickets, customer lookup and operational investigation.
- Admin RBAC, audit, reports, content management and store configuration.

### Out of scope unless separately approved

- Seller/author/publisher onboarding or vendor settlements.
- Affiliate networks and referral payouts.
- Multi-tenant SaaS or multiple independent stores.
- Physical inventory, fulfillment and shipping.
- Subscription memberships, lending, rentals or gifting.
- DRM claims that promise copying or screenshots are impossible.
- Native mobile applications; responsive web/mobile readiness may be required.

## 3. Recommended technical shape

**Default recommendation:** modular monolith with separate deployable frontend applications (or clearly separated route/layout applications) and a shared backend API. Run asynchronous workers from the same codebase or a closely coupled worker process. Keep domain modules isolated in code and data ownership even when deployed together.

Why: the initial product has one business owner and shared transactional workflows. A modular monolith reduces distributed-system complexity while preserving boundaries that can later be extracted if evidence justifies it.

Do not adopt microservices, multiple databases, event sourcing, Kubernetes or a service mesh solely because they sound enterprise-grade. Each introduces operational burden and requires a concrete use case.

## 4. Core invariants

1. **Single source of truth:** the backend owns all prices, order status, payment status, entitlements, access rules and permissions.
2. **Payment proof:** only verified gateway evidence or an approved reconciliation process can mark a transaction captured/settled.
3. **Exactly-once business effect, at-least-once delivery:** assume webhooks and jobs can be delivered more than once; use idempotency and unique constraints to prevent duplicate financial effects or duplicate entitlements.
4. **No access by obscurity:** random IDs and unguessable URLs do not replace authorization.
5. **Private assets:** original ebook objects are never publicly readable by default.
6. **Snapshot commerce data:** order items preserve title, SKU, unit amount, currency, discount and tax facts at purchase time.
7. **Immutable financial history:** do not edit or delete settled payment/refund facts; represent corrections with compensating entries.
8. **Explicit policy:** refund, tax, download, entitlement and data-retention policies are configurable only after owner approval.
9. **Fail closed:** if authorization, payment verification, file scanning or entitlement state cannot be established, deny the operation and record an actionable error.
10. **Traceability:** every important state change can be traced to an actor, source event, request/correlation ID and timestamp.

## 5. Technical baseline recommendations

These are candidate defaults, not permission to overwrite an existing suitable stack:

| Concern | Baseline recommendation | Selection rule |
|---|---|---|
| Application shape | Modular monolith | Keep existing architecture if it already enforces equivalent boundaries |
| API | Versioned REST + OpenAPI | Use the version supported by project tooling; latest verified reference is OAS 3.2.1 |
| Transactional data | PostgreSQL or equivalent relational DB | Prefer one primary transactional database; verify managed service and team expertise |
| Identifiers | UUIDv7/ULID or secure random IDs | Use consistent strategy; do not expose sequential identifiers as security controls |
| Money | Integer minor units + currency code | Never float; preserve order snapshots |
| File storage | Private object storage | Provider adapter; no public bucket for originals |
| Cache/rate limiting | Redis or managed equivalent where justified | Cache is not a source of truth |
| Background jobs | Durable queue + worker | Must support retries, dead-letter review and idempotent handlers |
| Event reliability | Transactional outbox | Do not publish an event before its DB transaction commits |
| Secrets | Managed secret store | Never commit secrets or place them in frontend env vars |
| Observability | Structured logs, metrics, traces and alerts | Redact sensitive data and add correlation IDs |

## 6. Quality attributes

The system must be designed and tested for:

- **Correctness:** financial and access state transitions are deterministic and auditable.
- **Security:** authentication, authorization, input validation, abuse protection and file confidentiality.
- **Availability:** graceful handling of gateway, email, storage and queue outages.
- **Recoverability:** tested database restore and file/object recovery procedures.
- **Performance:** defined measurable targets after workload assumptions are agreed; no unsupported scale claims.
- **Accessibility:** backend supports semantic, accessible frontend behaviours; UI conformance is verified in the frontend workstream against WCAG 2.2.
- **Maintainability:** domain boundaries, contracts, tests, migration discipline and runbooks.
- **Privacy:** purpose limitation, data minimization, retention and deletion workflows based on applicable law.

## 7. Environments

Minimum environments:

- Local development with isolated test services.
- Shared integration/test environment.
- Staging/pre-production with provider sandbox credentials and production-like configuration.
- Production, isolated from non-production data and credentials.

Production data must never be copied into developer environments without approved minimization, anonymization and access controls.

## 8. What “complete backend design” means

Do not call the backend design complete until:

- The repository audit identifies the actual stack and current state.
- Owner decisions that block implementation are accepted or have approved defaults.
- ERD, data dictionary, constraints and migration sequence are reviewed.
- All in-scope HTTP operations are defined and OpenAPI validates.
- Authorization matrix and object-level access tests exist.
- Financial and entitlement state machines are implemented and tested.
- Provider adapters and webhook/reconciliation contracts are specified.
- File ingestion and delivery security controls are testable.
- Operational dashboards, backups, restore exercises, alerts and runbooks are defined.
- Requirements trace to implementation and automated tests.
