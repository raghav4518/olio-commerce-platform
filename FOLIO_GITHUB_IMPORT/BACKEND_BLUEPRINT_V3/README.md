# FOLIO — Backend & Full-Stack Integration Blueprint

**Package:** FOLIO Backend Blueprint  
**Version:** 3.0.0  
**Prepared:** 20 November 2026  
**Status:** Frontend-aligned engineering specification and implementation handoff; not an implemented application  
**Product scope:** Exactly three panels — Customer, Owner/Admin, Support/Operations

## 1. Purpose

This package updates the existing FOLIO backend engineering blueprint V2.0 against the actual Google Stitch frontend export supplied on 20 November 2026.

The companion frontend archive contains 83 standalone screen exports and a FOLIO design-system document:

- 51 Customer screens (including public storefront, account/library/reader, payment states, customer help and customer ticket views);
- 18 Owner/Admin screens;
- 14 Support/Operations screens.

The screen inventory and recommended route/API/data/permission mappings are in `frontend-reference/` and `backend/15-FRONTEND-SCREEN-TO-BACKEND-MAPPING.md`.

The package is intended to be used together with:
1. the original frontend ZIP supplied by the product owner;
2. this backend blueprint ZIP;
3. the actual source repository (if one exists).

The AI/coding agent must inspect all three inputs before implementation.

## 2. Critical interpretation: the frontend ZIP is a design export

The provided frontend ZIP is **not** a runnable production frontend project. It contains 83 separate `code.html` screen exports and `folio_design_system.md`; it does not contain an application package manifest, app router, reusable component source tree, environment setup, backend client, migrations, or an integrated test suite.

Use those HTML files as the visual and interaction reference. Reconstruct the screens using the framework and architecture already present in the actual repository. Do not simply copy static HTML files into production and call the result complete.

The export currently references external Tailwind CDN, Google Fonts/Material Symbols, and 15 remote Google-hosted image URLs. The implementation agent must remove runtime dependence on Tailwind CDN, verify font licensing/availability, and replace remote design-reference images with controlled local or licensed production assets. Never assume the remote image URLs are permanent or licensed for commercial use.

## 3. Three approved panels

1. **Customer Panel** — public bookstore, discovery, bag/checkout, authentication, customer account, library, reader, orders, wishlist, reviews and self-service support.
2. **Owner/Admin Panel** — business operations, catalog, publishing, orders, customers, finance, marketing, content, analytics, support oversight, team, permissions, security and configuration.
3. **Support/Operations Panel** — role-limited internal ticket handling, conversations, customer/order investigation, ebook-access diagnostics, knowledge base, saved replies, agent settings and reports.

Do not add author, seller, publisher, affiliate, vendor or partner portals without a separately approved scope change. The public author profile page is a customer-facing editorial page, not an author portal.

## 4. Source-of-truth hierarchy

Resolve conflicts using this precedence:

1. Owner-approved decisions and signed-off ADRs in `decisions/`.
2. This V3 backend/technical contract for invariants, security, data ownership and system behavior.
3. The actual screen export and `frontend-reference/folio_design_system.md` for screen coverage, visual terminology and UI states.
4. The panel product specifications in `references/` for business requirements.
5. Existing repository implementation only where it is consistent and passes audit.
6. Agent assumptions are never authoritative.

When two inputs conflict, record the requirement IDs, evidence, impact, and proposed resolution. Do not silently discard a screen or business rule.

## 5. Existing V2 documentation retained

The backend architecture, domain model, data invariants, RBAC, commerce state machines, payment/refund/reconciliation model, digital assets/entitlements, event reliability, threat model, infrastructure, QA, implementation plan, standards, ADRs and original product specs from V2 have been retained as the foundation.

V3 adds the actual frontend export inventory, design-system reference, page-to-backend mapping, payment/production integration gates, database implementation deliverables, deployment runbook and a new master agent handoff.

## 6. Critical engineering baseline

- Audit the actual repository before changing code. Preserve a suitable existing stack; do not rewrite by default.
- Use one central backend and one authoritative set of domain services for all three panels.
- Prefer a modular monolith initially unless measured evidence justifies service extraction.
- Use a relational transactional database for orders, payments, entitlements, identity and support records. PostgreSQL is a candidate, not a silently approved provider.
- No browser code may connect directly to the database or hold permanent object-storage credentials.
- Money is integer minor units plus ISO currency code; do not use floating point for money.
- Store instants in UTC and apply explicit business/reporting timezone at presentation.
- Server-side authorization is mandatory on every endpoint and object access.
- Never trust a redirect, client event or browser success screen as proof of payment.
- Create paid access only after verified provider evidence and durable idempotent processing.
- Keep private ebook originals and customer attachments private by default.
- Use signed short-lived downloads or authenticated streaming where appropriate; do not claim perfect DRM.
- Preserve financial records, audit logs, orders and entitlement history. Prefer archive/anonymization over destructive deletion.
- Use outbox/inbox or an equivalent reliable event pattern for payment webhook processing and entitlement fulfilment.
- Keep secret values out of source control, browser bundles, logs and documentation.
- Test migrations, rollback/recovery, negative authorization paths and payment race conditions.
- No production launch claim until every production gate is supported by evidence.

## 7. Known gaps that the implementation agent must close

This package is a specification, not a finished codebase. V2's `contracts/openapi.yaml` is explicitly a starter contract (12 operations), not a complete contract for all screens. The database schema document defines logical schema/invariants; it is not a repository-specific executable migration set. No payment gateway, merchant account, launch jurisdiction/currency, tax policy, refund policy, storage provider, hosting provider, or production secrets are approved by this package.

The implementation agent must:
- inspect the source repository and identify its real stack/database;
- generate and validate complete migration files for that stack;
- complete and validate OpenAPI contracts against implemented controllers/services;
- implement a real gateway adapter in sandbox/test mode first;
- activate live payments only after an owner-selected provider, merchant onboarding and secrets are configured;
- implement all 83 exported screens as integrated routes/responsive views;
- provide test evidence and deployment/rollback instructions.

## 8. Required deliverables from the coding agent

The final implementation repository must contain, at minimum:

- architecture and ADR updates;
- completed database ERD and executable, versioned migrations;
- seed data that is safe and clearly development-only;
- complete OpenAPI contract;
- implemented customer/admin/support routes;
- real persistence and backend service integrations;
- payment gateway adapter and verified webhooks;
- order/refund lifecycle and reconciliation;
- ebook storage, validation, delivery and entitlements;
- server-enforced authentication/authorization;
- email/notification integration;
- logging, metrics, backups and restore verification;
- unit, integration, contract, end-to-end, accessibility and security tests;
- CI/CD, environment setup, deployment, rollback and operations runbooks;
- completed screen/requirement traceability matrix;
- release report distinguishing complete, blocked, deferred and unverified work.

## 9. Recommended reading order

1. `agent/MASTER-IMPLEMENTATION-PROMPT.md`
2. `PACKAGE-MANIFEST.md`
3. `frontend-reference/FRONTEND-EXPORT-TECHNICAL-OBSERVATIONS.md`
4. `frontend-reference/folio_design_system.md`
5. `frontend-reference/SCREEN-INVENTORY.csv`
6. `backend/15-FRONTEND-SCREEN-TO-BACKEND-MAPPING.md`
7. `decisions/OPEN-DECISIONS-REGISTER.md`
8. `references/PRODUCT-SCOPE-BASELINE.md`
9. `references/ADMIN-PANEL-PRODUCT-SPECIFICATION.md`
10. `references/CUSTOMER-AND-SUPPORT-PANEL-PRODUCT-SPECIFICATION.md`
11. `backend/00-TECHNICAL-OVERVIEW.md` through `backend/14-STANDARDS-AND-COMPLIANCE-REFERENCES.md`
12. `backend/16-PRODUCTION-COMMERCE-AND-PAYMENT-INTEGRATION.md`
13. `backend/17-DEPLOYMENT-RUNBOOK-AND-GO-LIVE-GATES.md`
14. `backend/18-DATABASE-IMPLEMENTATION-DELIVERABLES.md`
15. `backend/19-IMPLEMENTATION-EXECUTION-AND-ACCEPTANCE.md`

## 10. Definition of production readiness

“Production-ready” means the real deployed application has completed the agreed acceptance matrix, passed security and operational checks, has real provider credentials and verified webhooks, has working customer purchases and ebook fulfilment, has demonstrated backup restoration and rollback, and has no unresolved launch-blocking defects.

A compile success, static design preview, mocked checkout, sandbox transaction, generated documentation or untested migration is not production readiness.

Production activation may be blocked by owner-controlled prerequisites such as gateway merchant account/keys, legal seller details, approved tax/refund/privacy policies, verified domain/email sender, storage/hosting credentials and go-live approval. The agent should complete all non-blocked work and present these blockers precisely instead of fabricating them.
