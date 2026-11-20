# 01 — Product Scope and Requirements

**Version:** 1.0  
**Status:** Baseline product definition; open decisions are tracked separately  
**Owner:** Product owner  
**Applies to:** Customer, Owner/Admin and Support/Operations panels

## 1. Product vision

Build an owner-operated digital bookstore where customers can discover ebooks, evaluate them with accurate product information and previews, purchase securely, and access purchased content through a personal library and supported reading/download experiences. The owner runs the business through the Admin Panel. Staff resolve customer and fulfilment problems through the Support/Operations Panel.

The product must be straightforward for a first-time buyer while giving the owner reliable operational control. The public store should feel like a distinctive book brand, not a generic template. Administrative workflows should prioritize clarity, data integrity and speed over decoration.

## 2. Product boundary

Exactly three panels are in scope.

| Panel | Who uses it | Responsibilities | Does not own |
|---|---|---|---|
| Customer Panel | Visitors and registered customers | Discover, search, evaluate, buy, read/download, manage library, orders, preferences and support requests | Global catalogue administration, payment credentials, staff permissions |
| Owner/Admin Panel | Owner and authorized administrators | Catalogue, pricing, orders, refunds, finance, customers, content, promotions, analytics, system configuration, permissions and audit | Customer-owned reading preferences or support conversation identity |
| Support/Operations Panel | Support agents and operations staff | Tickets, customer context, order/payment/access investigation, escalation and approved support actions | Global financial configuration, payment secrets, tax settings, pricing and staff-role administration unless separately granted |

The Customer Panel includes both the public bookstore and the authenticated customer workspace. They are not separate applications by product definition, even if route groups or deployments differ technically.

No author, publisher, affiliate, vendor, seller, partner or additional portal is approved in the current product scope.

## 3. Business model baseline

The owner has not formally approved single-vendor versus marketplace operation. For planning, use **single-vendor owner-operated commerce**: the platform owner publishes and sells the ebooks. This is a safe initial implementation assumption, not a final legal/commercial decision.

Do not implement third-party seller onboarding, seller accounts, commission splits, seller wallet balances, royalty calculations, vendor settlements or marketplace order splitting unless the owner approves the change in writing.

## 4. Product principles

1. **Customer clarity:** buyers must understand format, language, edition, price, access method and any restrictions before purchase.
2. **Low-friction commerce:** keep product discovery and checkout simple; never hide total price, recurring charges or material terms.
3. **Immediate, reliable fulfilment:** after verified payment, the order and access entitlement must reconcile even if a downstream service temporarily fails.
4. **Secure content delivery:** source ebooks are private; every read/download request is authorized and auditable.
5. **One source of truth:** all panels share canonical records and lifecycle rules.
6. **Least privilege:** staff access only the data and actions needed for their role.
7. **Accessible by default:** target WCAG 2.2 AA across all panels and the reader.
8. **Responsive by design:** the Customer Panel is mobile-first; Admin and Support are responsive but optimized for desktop workflows.
9. **Real operations:** no controls that only look functional; all displayed metrics must derive from real data.
10. **Honest protection claims:** watermarking and access controls can discourage casual redistribution but cannot guarantee prevention of screenshots, copying or external capture.

## 5. Primary user types

### 5.1 Customer roles

- Visitor: browses public content; may use cart and checkout according to the approved guest-checkout policy.
- Registered customer: has an account and may access owned library, orders, reading progress, downloads, support history and preferences.

### 5.2 Admin roles

- Owner/Super Admin.
- Operations Manager.
- Catalog Manager.
- Finance Manager.
- Marketing Manager.
- Analyst (read-only).

Role names are starting templates. Authorization must be permission-based, server-enforced and configurable. A person may hold multiple assigned roles.

### 5.3 Support roles

- Support Agent.
- Senior Support Agent / Escalation Lead.
- Support Supervisor.

Support roles must be distinct from Owner/Admin roles. Support agents may inspect scoped customer and order information but must not inherit admin privileges.

## 6. Product capability map

### 6.1 Customer Panel — P0 baseline

- Public homepage, category/collection browsing, search and filtering.
- Ebook detail pages with cover, accurate description, author, format, language, edition, price, sample/preview where available, and access terms.
- Cart, coupon validation, checkout and payment-state feedback.
- Authentication, registration, email verification and account recovery according to approved policy.
- My Library with entitlement-aware access.
- Online reader and/or download delivery based on approved format and access policy.
- Order history, order detail and invoice/receipt where supported.
- Support request creation and ticket conversation.
- Account security, active sessions and privacy controls.

### 6.2 Customer Panel — P1/P2 candidates

- Wishlist.
- Verified-purchase reviews and ratings.
- Reading progress sync, bookmarks and notes where supported by the reader and format.
- Recommendations and recently viewed books.
- Gift purchases, bundles and subscriptions only after business rules are approved.
- PWA/offline reading only after licensing, device security and offline-entitlement requirements are designed.

### 6.3 Owner/Admin Panel — P0 baseline

- Dashboard using defined metrics.
- Ebook, file/version, author, category, tag and collection management.
- Product draft, review/approval if selected, scheduling and publication.
- Orders, payment attempts, refunds, invoices and reconciliation visibility.
- Customer account and entitlement management.
- Digital access policy controls and file security administration.
- Core finance and sales reports.
- Admin users, permissions, audit logs and store configuration.

### 6.4 Owner/Admin Panel — P1/P2 candidates

- Homepage section builder, banners, landing pages and blog.
- Advanced promotions, customer segmentation and campaign reporting.
- Advanced conversion, reading and retention analytics.
- Affiliate program only if separately approved; no separate affiliate panel is part of current scope.

### 6.5 Support/Operations Panel — P0 baseline

- Support overview, own queue, unassigned queue and authorized ticket search.
- Ticket detail with customer conversation, internal notes, assignment, priority and escalation.
- Customer lookup with masked and purpose-limited information.
- Order, payment and entitlement investigation.
- Download/read access troubleshooting.
- Refund request creation or escalation; actual refund execution only for explicitly authorized roles.
- Customer-facing replies, macros and communication history.
- Staff audit trail for sensitive actions.

## 7. Core cross-panel workflows

### 7.1 Product publication

1. Authorized Admin creates a product draft.
2. Admin enters metadata, price, classification, format and access policy.
3. Files upload into a private quarantine area.
4. The backend verifies type, size, checksum, malware scan result and required metadata.
5. Admin previews the product page and available sample.
6. An authorized user publishes or schedules it according to the approved workflow.
7. Customer Panel reads published catalogue data; it must not expose draft or private file metadata.

### 7.2 Purchase and fulfilment

1. Customer adds an available ebook to cart.
2. Server recalculates price, discounts, tax and total; browser values are never authoritative.
3. Server creates a pending order/payment attempt using a unique idempotency key.
4. Customer completes payment using the approved gateway flow.
5. Backend verifies the gateway signature/status and records the provider event idempotently.
6. Order transitions to paid only according to the payment state machine.
7. Entitlement is created in the same transactional boundary as the order update where possible, or by a durable outbox/retry worker.
8. Customer Library reflects the entitlement after committed fulfilment.
9. Confirmation is sent asynchronously; notification failure must not undo a successful purchase.
10. Admin and Support views display the same canonical state and event history.

### 7.3 Refund and access handling

1. Customer requests a refund or support records a request.
2. System loads current order, payment, prior refunds and entitlement state.
3. Eligibility is evaluated using the owner-approved refund policy; do not invent a policy.
4. Support investigates and records evidence; approval rights are separate.
5. Authorized finance/admin action submits the refund to the payment provider with idempotency.
6. Provider result is reconciled asynchronously and recorded.
7. Entitlement handling follows the approved refund/access policy; do not automatically revoke access based on an unverified client request.
8. All actions and decisions are audited.

### 7.4 Customer support investigation

1. Customer creates a ticket from the relevant order/book when possible.
2. Support sees only authorized customer/order context.
3. Support may diagnose payment, order or entitlement state through read-only service views.
4. Any privileged correction uses an approved backend action with reason, permission check and audit event.
5. Support replies to customer and records resolution.
6. Admin analytics and customer account history read the same ticket record.

## 8. Canonical data ownership

| Domain | System of record | Customer access | Support access | Admin access |
|---|---|---|---|---|
| Product/catalogue | Catalog domain | Published data only | Read-only context | Manage |
| Price/discount | Pricing/commerce domain | Current calculated offer | Read-only | Manage |
| Cart/checkout | Commerce domain | Own cart | Usually not needed | Operational read where needed |
| Order | Order domain | Own orders | Scoped read/investigation | Full operational access |
| Payment/refund | Payment domain | Own payment status/receipt | Masked/scoped investigation; no secrets | Finance permissions |
| Entitlement | Entitlement domain | Own access | Scoped investigation; limited approved repair | Manage/revoke/restore with reason |
| Ebook asset | Digital asset service | Authorized delivery only | No source access by default | Upload/version/manage |
| Reading progress | Reader domain | Own progress | No access unless necessary and authorized | Aggregated analytics only by default |
| Support ticket | Support domain | Own tickets | Assigned/authorized tickets | Oversight and reporting |
| Staff roles/audit | Identity/governance domain | Not available | Limited own/supervisor view | Owner/admin permissions |

## 9. Product data definitions and invariants

- Money is stored as integer minor units with an explicit ISO currency code; never use binary floating point for financial arithmetic.
- An order item snapshots product title, SKU/identifier, selected format/edition, unit price, discount, tax and currency at purchase time. Later catalogue edits must not rewrite history.
- Orders, payment attempts, provider events, refunds and entitlements have independent statuses and identifiers.
- Payment status is provider-verified; browser redirects are not proof of payment.
- Entitlements are granted only after verified successful payment or an explicitly authorized complimentary/manual grant.
- Every manual grant, revoke, restore or refund correction requires actor, timestamp and reason.
- A product being unpublished does not automatically revoke valid existing purchases.
- A file version replacement must not silently invalidate previous buyers' access; migration policy is explicit.
- Customer ownership is enforced server-side on every query and mutation.
- Staff access is scoped by role, permission, assignment and purpose where appropriate.
- All timestamps are stored in UTC; rendered in user/store timezone explicitly.
- Financial and audit history is append-oriented; corrections create compensating records rather than mutating immutable history.

## 10. Non-functional targets

These are initial engineering targets, not claims about current system performance. The team must validate them through load testing and production telemetry.

| Area | Initial target / rule |
|---|---|
| Accessibility | WCAG 2.2 AA target across three panels and reader controls |
| Core Web Vitals | Aim for good thresholds at the 75th percentile: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 on eligible real-user measurements |
| API latency | Establish service-specific SLOs after baseline load tests; do not invent a single global threshold |
| Availability | Agree launch SLO and maintenance policy before production |
| Security | OWASP ASVS-based verification; target assurance level decided by threat model |
| Payments | Idempotent provider event processing, reconciliation and auditable state transitions |
| Files | Private storage, malware scanning, authorization, limited-lived delivery links |
| Backups | Defined RPO/RTO, tested restore procedures and encrypted backups before go-live |
| Scalability | Paginated, indexed and bounded list queries; asynchronous handling for non-immediate work |
| Observability | Structured logs, metrics, traces/correlation ids, alerts and actionable runbooks |

Performance target reference: [Google web.dev — Core Web Vitals](https://web.dev/articles/vitals). Measure the 75th percentile separately for mobile and desktop.

## 11. Explicit non-goals until approval

- Marketplace seller onboarding or multi-vendor settlement.
- Author/publisher or affiliate panel.
- Physical inventory, shipping or warehouse workflows.
- Accounting system replacement.
- Native iOS/Android apps.
- Subscription access or recurring billing.
- Gift purchases and family/shared libraries.
- AI-generated content, automated pricing or autonomous refund approval.
- Offline reader downloads without a designed device/security model.
- Guaranteed DRM or unbreakable anti-copy protection.

## 12. Product acceptance gates

The product cannot be declared launch-ready until:

1. The owner has approved all decisions marked `P0 / Decision required` in `04-OPEN-DECISIONS.md`.
2. Each detailed page requirement maps to a route/component, permission, API/service, data entity and test.
3. Purchase, payment, order, entitlement and library fulfilment works end-to-end under retries and failure injection.
4. Customer A cannot access Customer B's orders, downloads, tickets or reading data.
5. Support cannot invoke owner-only finance, configuration or permission actions.
6. Payment and refund states reconcile with provider records.
7. Files are private by default and cannot be accessed using guessed or expired identifiers/links.
8. Accessibility, performance, backups, monitoring, incident response and security checks have evidence.
9. No fake production data, dead UI controls, unresolved critical vulnerabilities or undocumented policy assumptions remain.

## 13. Detailed specifications

- Owner/Admin: `reference/02-admin-panel-specification.md`
- Customer and Support/Operations: `reference/03-customer-and-support-panel-specification.md`

These documents are part of this specification set and must be read in full.
