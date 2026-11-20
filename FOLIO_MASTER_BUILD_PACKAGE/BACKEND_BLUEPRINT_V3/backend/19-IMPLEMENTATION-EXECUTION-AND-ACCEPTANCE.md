# Backend 19 — Implementation Execution, Cross-Panel Acceptance and Definition of Done

**Version:** 3.0.0  
**Purpose:** Execution contract for the AI coding agent. The agent must work against the real repository, not merely produce another plan.

## 1. Mandatory initial audit (no implementation edits yet)

Inspect and report:
- repository tree, branch, uncommitted changes and recent history;
- framework/language/runtime versions and package manifests;
- frontend app(s), route map and design system;
- backend services, API routes, middleware and auth;
- database provider, schema, migrations and seed data;
- object storage, file upload and ebook reader implementation;
- payment gateway code, webhook routes and configuration;
- notification/email services;
- tests, CI workflows and deployment files;
- current production/staging status if accessible.

Run baseline checks safely. Do not run destructive scripts, reset database, drop tables, overwrite secrets, deploy, or make live payments during the audit. Record exact commands, results and evidence.

## 2. Required artifact set in the repository

Create/update:
- `docs/architecture/` — C4/system context, containers/components, data flows and ADRs;
- `docs/product/` — approved decision register and requirement traceability;
- `docs/api/openapi.yaml` — complete validated API specification;
- `docs/database/` — ERD, schema dictionary, migration and data migration notes;
- `docs/security/` — threat model, permissions matrix, security test results;
- `docs/operations/` — deployment, rollback, backup/restore, incident and payment reconciliation runbooks;
- `docs/qa/` — screen test matrix, acceptance tests, release evidence.

Maintain `SCREEN-INVENTORY.csv` or an equivalent canonical tracker with every one of the 83 supplied screen IDs and status.

## 3. Execution phases

### Phase 0 — Audit and traceability
- Unzip and inventory both handoff archives.
- Inspect all 83 `code.html` files and the design system.
- Compare every screen against the V3 mapping and the original product specifications.
- Inspect existing repository and tests.
- Create gap/contradiction/risk report.
- Add route/component/API/data/permission/test mapping per screen.
**Gate:** Owner-visible audit and implementation plan with blockers; no hidden scope loss.

### Phase 1 — Decisions, architecture and contracts
- Confirm reusable stack.
- Resolve only decisions needed to unblock current work; present options and recommendations for owner decisions.
- Update ADRs.
- Complete schema and OpenAPI design.
- Define state machines and permission matrix.
**Gate:** No contradictory payment/order/entitlement semantics; schema and API are testable.

### Phase 2 — Engineering foundations
- Configure environments and secrets handling.
- Implement logging/correlation, error handling, validation, rate limiting, secure sessions and RBAC foundation.
- Add migration pipeline, test DB, CI checks and basic deployment baseline.
**Gate:** Clean install/migrations, auth tests and staff/customer access tests pass.

### Phase 3 — Catalog, content and digital assets
- Implement ebooks, contributor attribution, categories, collections, pricing, publication state, CMS and search.
- Implement secure upload, MIME/signature validation, malware scanning/quarantine, asset versioning and private storage.
- Implement preview assets separately from originals.
**Gate:** Admin creates/publishes a valid ebook; only published records appear publicly; private files are inaccessible without authorization.

### Phase 4 — Customer discovery and account
- Implement public homepage, explore, search, category, collections, author profile, product details, FAQ/legal/content.
- Implement sign-in, registration, verification, recovery and selected auth method.
- Implement profile, preferences, security, notifications, wishlist and reviews as approved.
**Gate:** All corresponding customer screens load persisted real data with correct access control.

### Phase 5 — Bag, checkout and payments
- Implement server-owned cart/bag and pricing quote.
- Implement guest/account rule and terms acceptance.
- Implement provider adapter and sandbox first.
- Implement payment attempt, webhook verification, idempotency, order transition, outbox and reconciliation.
- Implement approved production provider only after owner decision and merchant onboarding.
**Gate:** Sandbox end-to-end payment tests pass; live mode stays disabled until production payment gate evidence exists.

### Phase 6 — Entitlements, library, reader and downloads
- Implement entitlement fulfilment after verified payment.
- Implement library, reading progress, reader sessions, bookmarks/notes and authorized downloads.
- Select and verify PDF/EPUB renderer only after format decision and licensing/security evaluation.
**Gate:** Paid user can read/access owned book; non-owner and revoked/pending users are denied; reader progress is durable.

### Phase 7 — Customer orders, invoices and support
- Implement order history/detail, payment status, invoice/receipt under approved tax policy.
- Implement customer help, own support tickets, public messages and attachment access.
**Gate:** Customer can view only their own order/library/tickets; private staff notes cannot leak.

### Phase 8 — Support/Operations
- Implement queue/inbox, assignments, workspace, public replies, private internal notes, escalations, customer/order/payment/access investigations, knowledge base, saved replies, notifications, agent profile and manager reports.
- Implement granular staff permissions; refund execution/entitlement changes restricted.
**Gate:** Negative cross-role tests pass; agent actions are audited.

### Phase 9 — Owner/Admin
- Implement dashboard metrics, catalog, orders/refunds, customer management, finance, coupons/campaigns, CMS, analytics, support oversight, notifications, team, security/audit and settings.
- Ensure dashboard/report metrics are computed from persisted, defined data.
**Gate:** Every admin screen is connected and permissions are independently checked server-side.

### Phase 10 — Cross-panel integration
- Exercise full customer purchase and library access.
- Confirm admin order/transaction views reflect the same underlying records.
- Confirm support agents inspect the same records using limited safe data.
- Test refund request → review → provider execution → customer status/entitlement treatment according to policy.
- Verify all status labels and IDs are consistent.
**Gate:** Cross-panel E2E tests pass.

### Phase 11 — Security, performance and operational hardening
- Complete threat-model fixes, dependency/secret scans, security headers, privacy and retention.
- Run load tests against agreed targets.
- Verify backup restoration, deployment rollback, monitoring, alerts and incident runbooks.
**Gate:** Release evidence complete; unresolved critical/high issues block launch unless explicitly accepted by authorized owner.

### Phase 12 — Production deployment
- Deploy staging and complete smoke tests.
- Verify production configuration, domain/email, provider credentials and webhook.
- Require explicit go-live authorization.
- Deploy with rollback path and monitor production.
**Gate:** All `backend/17` launch gates have evidence.

## 4. Cross-panel critical acceptance scenarios

1. Admin creates a draft ebook; it is not public.
2. Admin uploads a valid ebook asset; scanning and validation succeed; file remains private.
3. Admin publishes it; public catalog/search/category/collection/detail screens show the same canonical record.
4. Customer previews only public sample content.
5. Customer adds ebook to Bag; server returns canonical quote.
6. Customer uses a valid coupon; server validates limits/eligibility.
7. Customer starts checkout; order and payment attempt are persisted with immutable amounts and currency.
8. Provider confirms payment with valid webhook; duplicate event does not duplicate capture/order/entitlement.
9. Customer library displays the purchased ebook only after entitlement creation.
10. Customer reader can open the content and save progress/bookmarks/notes.
11. Customer A cannot retrieve Customer B's order, ticket, library item, reader session or signed file.
12. Pending or failed payment cannot access paid file.
13. Admin sees the same order/payment/entitlement records and financial calculations.
14. Support agent sees permitted investigation context but no merchant secrets/full card credentials.
15. Support private note is visible to authorized staff only, never customer APIs.
16. Refund request is not automatically a completed refund; approval and provider execution are recorded.
17. Refund/access behavior follows approved policy only.
18. Unpublishing an ebook does not silently erase paid customer rights.
19. Replacing a file preserves the approved version entitlement and reader migration behavior.
20. Customer can request support from own order and see only public ticket replies.
21. Staff permission changes are audited and immediately invalidate/re-evaluate sessions/permissions according to policy.
22. Admin security screens show immutable audit events.
23. Email, notification, report, export and search features use actual persisted records.
24. All 83 pages have routes, responsive implementation, state coverage, and test evidence.

## 5. Coding and change-management rules

- Work in small, reviewable phases with one commit per completed phase or coherent feature.
- Do not commit secrets, credentials, customer data or downloaded commercial ebook content.
- Do not break existing behavior without tests and a migration.
- Add tests with every business rule.
- Avoid giant rewrites and speculative microservices.
- Do not fake success to make browser tests pass.
- Use actual server responses in customer-critical flows.
- For unfinished functions, remove/disable the production action or mark clearly as unavailable; do not leave a misleading active button.
- Never claim a test ran if it did not. Record command, environment, result, skipped tests and reason.
- Do not deploy to production or run real financial transactions without owner approval.

## 6. Progress reporting format

After each phase, report:
- Phase ID and scope;
- commit hash;
- files changed;
- migrations added/applied;
- API operations added/changed;
- screens implemented;
- tests executed and results;
- security checks and negative-path results;
- known defects;
- unresolved owner decisions;
- deployment impact and rollback plan;
- next phase prerequisites.

## 7. Definition of done per screen

A screen is complete only when:
- exact screen has an implemented route;
- relevant visual system is preserved;
- responsive behavior is verified at desktop/tablet/mobile as applicable;
- every actionable control has a real behavior or is explicitly disabled/unavailable;
- loading, empty, error, success and pending states are implemented where relevant;
- data is loaded/saved through the correct service;
- authentication and object-level permission are enforced on server;
- unit/integration/E2E coverage exists for critical behavior;
- browser console/network errors are addressed;
- no sample secrets, fabricated live metrics or fake transactions appear in production.

## 8. Definition of done for project

Project is complete only when:
- 83/83 screens are accounted for and tested or have an explicitly approved exclusion;
- all agreed workflows are implemented;
- database migrations install and upgrade safely;
- full OpenAPI contract validates and matches implementation;
- live payment provider and webhooks are configured and verified, or the product is explicitly not yet live;
- orders/refunds/ledger reconcile;
- entitlement and secure ebook delivery controls pass negative tests;
- staff/customer access boundaries pass;
- production build and staging E2E tests pass;
- backup restore and rollback are demonstrated;
- critical/high security issues are resolved or formally accepted;
- owner prerequisites and go-live authorization are recorded.

Never label FOLIO production-ready based only on successful compilation, visual similarity, mocked/sandbox payment or documentation generation.
