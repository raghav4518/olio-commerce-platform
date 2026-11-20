# FOLIO — GEMINI AI STUDIO MASTER IMPLEMENTATION PROMPT
## Build the complete three-panel ebook commerce platform using the extracted frontend design files, backend package, and actual application workspace

**Target environment:** Google AI Studio — Build Mode, using Gemini 4 Argon when the model is available in the user's model selector. If Argon is not selectable, use the strongest available coding/reasoning model and do not claim Argon was used. This prompt is designed to work with the current workspace capabilities; adapt commands to the tools actually available and report limitations honestly.

**Important input arrangement:** This repository contains the extracted Google Stitch frontend screens as ordinary files in `FRONTEND_DESIGN_EXPORT/`, plus the complete backend/product/engineering documentation in `BACKEND_BLUEPRINT_V3/`. No ZIP extraction is required.

You are the autonomous lead engineering agent responsible for taking FOLIO from its approved UI design and backend specification to a fully implemented, tested, deployable ebook-commerce application.

Use these coordinated source inputs in this repository:

1. **Frontend reference files:** `FRONTEND_DESIGN_EXPORT/` contains all 83 FOLIO screen HTML exports and `folio_design_system.md`, already extracted as normal files.
2. **Backend documentation:** `BACKEND_BLUEPRINT_V3/` contains the authoritative product/backend rules, database/domain design, payment/security/workflow requirements, screen-to-backend mapping, deployment gates and implementation prompt.
3. **Actual application workspace:** inspect and implement in the current repository. These design and documentation folders are references, not a runnable application by themselves.

---

# A. MISSION AND NON-NEGOTIABLE OUTCOME

Build one cohesive, working FOLIO application with exactly three panels:

1. **Customer Panel** — public bookstore, discovery, Bag, checkout, customer account, library, reader, orders and self-service support.
2. **Owner/Admin Panel** — owner business operations, catalog, orders, customers, finance, refunds, marketing, content, analytics, support oversight, staff, permissions, security and settings.
3. **Support/Operations Panel** — restricted staff workspace for ticket queues, conversations, customer/order/payment/access investigations, knowledge base, saved replies and operational reporting.

The application must use one authoritative backend and database for all three panels. Do not create separate databases or duplicate business logic per panel unless a documented architecture decision proves this necessary.

The goal is an application that can serve real customers, not a static design, clickable prototype, fake dashboard, frontend-only demo or collection of disconnected HTML pages.

**Do not claim the platform is production-ready until real persistent data, real configured payment integration, verified webhooks, secure ebook fulfilment, authorization tests, operational monitoring, backup/restore and release gates have been completed.**

# B. INPUTS — READ ALL OF THEM FIRST

The owner will supply three distinct inputs in the AI Studio workspace/repository:

1. **`BACKEND_BLUEPRINT_V3/`** — authoritative backend/product/engineering documents.
2. **`FRONTEND_DESIGN_EXPORT/`** — extracted Google Stitch frontend design references.
3. **The actual application repository/workspace** — the code that must be implemented and run.

## B1. Frontend design export — already extracted

The complete Stitch export is available as ordinary files under `FRONTEND_DESIGN_EXPORT/`. Do not look for or require a ZIP archive.

Verify it contains:
- 83 `code.html` screen exports;
- 51 Customer screens;
- 18 Owner/Admin screens;
- 14 Support/Operations screens;
- `folio_design_system.md`.

Inventory every file and preserve the original screen-folder names. Do not rename or overwrite the source design exports. Create an implementation tracker that records the original screen folder, intended route, responsive reference, frontend component, API contract, database entity, role/permission, and test case. Do not skip any page because another page looks similar. Desktop/mobile exports may be separate visual references for one responsive route.

The HTML exports are static design references, not a complete application source project. They do not provide an application router, package manifest, reusable local component tree, API client, real persistence or complete frontend test infrastructure. They use design-time CDN and remote assets. Rebuild them in the actual application architecture rather than merely serving the exports as static HTML.

Read the complete `FRONTEND_DESIGN_EXPORT/folio_design_system.md`. Preserve its editorial visual identity and accessibility/responsive guidance. Do not treat remote image URLs as stable licensed production assets. Replace them with owned/licensed local assets or approved sources. Do not depend on Tailwind CDN in production.

## B2. Backend documentation — supplied in this master package

Read the complete `BACKEND_BLUEPRINT_V3/` directory, including:
- README, package manifest and changelog;
- backend technical documents 00–19;
- product specifications and product scope;
- open-decision register and architecture decision records;
- frontend screen inventory and export observations;
- screen-to-backend mapping;
- payment/refund integration requirements;
- database implementation deliverables;
- deployment/go-live runbook;
- implementation phases, templates and acceptance criteria;
- OpenAPI starter contract and its completion requirements.

This package is the technical/business requirements baseline. Respect its source-of-truth hierarchy. Owner-approved decisions/ADRs override assumptions. Do not silently change payment, tax, legal, data-retention, refund, access or marketplace rules. The logical data model is not itself an executable migration set and the included OpenAPI file is not a complete implementation contract; complete, validate and test both for the actual repository.

## B3. Actual repository/workspace

Inspect the current working directory and Git repository. If it exists, preserve the established stack, useful code, routes, database, migrations, deployment and tests whenever they are suitable. Do not rewrite by default.

Google AI Studio Build Mode currently provides a full-stack web-app workspace (React frontend by default and Node.js server runtime) and supports importing an existing repository from GitHub. Use the capabilities exposed by the current session; do not assume tools, shell commands, database providers, secrets, or integrations that are not actually available. If this is a new AI Studio app with no repository, scaffold a maintainable project compatible with the supported environment after documenting the architecture choice. If an existing repository uses a different stack that cannot run in Build Mode, report the constraint and propose a safe migration/integration plan rather than silently discarding working code.

If no implementation repository exists, report that clearly and propose the smallest maintainable stack before scaffolding. Do not pretend the documentation and frontend reference folders alone constitute a runnable application.

# C0. GOOGLE AI STUDIO BUILD MODE WORKING RULES

- Work in the active Build Mode project, not an ordinary chat-only session.
- Use the Code view and live preview to inspect actual files and application behavior.
- If GitHub is connected, work on a dedicated feature branch and keep the repository as the durable source of truth. Do not overwrite `main` directly.
- Before large changes, inspect the current working tree and save/commit a checkpoint.
- Do not assume a long conversation preserves every implementation detail. Maintain durable progress files in the repository: `docs/IMPLEMENTATION-STATUS.md`, `docs/SCREEN-TRACEABILITY.csv`, `docs/DECISION-LOG.md`, and `docs/PHASE-REPORTS/`.
- After each coherent phase, run actual available tests/build checks, fix failures, update status, and commit if Git is connected.
- If context, execution time, output limits, or tool availability prevent finishing a phase, stop at a clean checkpoint, write exact remaining tasks and commands into `docs/IMPLEMENTATION-STATUS.md`, and report the blocker. Never pretend background work will continue after the session ends.
- Use the model actually selected in the interface. If Gemini 4 Argon is available, it may be selected for complex engineering; if not, continue with the strongest available model and state which model is in use when asked.
- AI Studio preview success is not equivalent to production deployment. Do not deploy to Cloud Run until the explicit staging and production gates in V3 pass and the owner authorizes deployment.

# C. INITIAL AUDIT — BEFORE EDITING SOURCE CODE

Perform a full read-only audit and record:

1. Git branch, working tree status and recent commits.
2. Folder structure and existing app entry points.
3. Frontend framework, routing, state, styling, package manager and runtime versions.
4. Backend language/framework, modules, middleware, API routes and current service boundaries.
5. Database vendor/version, ORM, schema, migrations, constraints and existing data.
6. Authentication, sessions, identity types and authorization.
7. File storage, CDN and ebook delivery.
8. Current payment provider/adapters, webhook endpoint and refund logic.
9. Email/SMS/notification integrations.
10. CI workflows, test commands, build, hosting and deployment configuration.
11. Current bugs, TODOs, incomplete modules and failed tests.
12. Existing security concerns and any production data risk.

Run safe baseline checks. Never run destructive database resets, drops, seed-overwrites, production deployments or live financial transactions during the audit.

Create:
- repository inventory;
- feature/gap matrix;
- requirement traceability matrix for all 83 screens;
- risk register;
- open decision list with evidence;
- phased implementation plan.

Then proceed into implementation. Do not stop after producing the plan unless an external owner decision is truly blocking the next safe action.

# D. FRONTEND IMPLEMENTATION — ALL 83 SCREENS

Implement the supplied screen inventory. Preserve screen coverage, user intent, visual identity and state behavior. Screens can share reusable components and responsive routes, but no user-visible functionality may disappear.

## D1. Customer Panel — 51 screens

Implement:
- Desktop homepage and mobile homepage;
- Explore Books desktop/mobile;
- Search Results desktop/mobile;
- Category page;
- Curated Collection page;
- Public Author Profile page (public content only, not an author portal);
- Ebook Product Details;
- Sample Reader;
- Shopping Bag;
- Checkout;
- Payment Successful;
- Payment Pending Verification;
- Payment Failed;
- Order Confirmation;
- Login desktop/mobile;
- Registration desktop/mobile;
- Email Verification;
- Forgot Password;
- Reset Password;
- OTP Sign-In only if the selected auth policy supports it;
- Customer Dashboard;
- My Library;
- Ebook Reader desktop/mobile;
- Reading History;
- Downloads;
- Bookmarks & Notes;
- My Orders;
- Order Details;
- Wishlist;
- My Reviews;
- Customer Notifications;
- Profile;
- Security;
- Preferences;
- Help Center;
- Customer Support Tickets;
- Create Support Request;
- Customer Ticket Conversation;
- FAQ;
- About Us;
- Contact Us;
- Privacy Policy;
- Terms & Conditions;
- Refund Policy;
- 404 page.

Use `frontend-reference/SCREEN-INVENTORY.csv` and `backend/15-FRONTEND-SCREEN-TO-BACKEND-MAPPING.md` as the implementation tracker. Verify count and do not claim 51/51 until every Customer entry is routed, integrated and tested.

## D2. Owner/Admin Panel — 18 screens

Implement:
- Overview Dashboard;
- All Ebooks;
- Add/Edit Ebook;
- Ebook Details;
- Categories & Collections;
- All Orders;
- Order & Refund Details;
- Customers;
- Customer Profile;
- Payments & Finance;
- Marketing & Coupons;
- Website & Content;
- Analytics & Reports;
- Support Oversight;
- Admin Notifications;
- Team & Permissions;
- Security & Audit Logs;
- Settings.

Use role/permission-gated API data. Never rely on hiding a button as the only protection.

## D3. Support/Operations Panel — 14 screens

Implement:
- Support Dashboard;
- Ticket Inbox;
- Ticket Workspace;
- Create Internal Support Ticket;
- Customer Lookup;
- Support Customer Profile;
- Order & Payment Investigation;
- Ebook Access & Delivery Diagnostics;
- Knowledge Base;
- Saved Replies;
- Support Notifications;
- Team Performance;
- Support Reports;
- Agent Profile & Settings.

Clearly separate public customer replies from private internal notes in UI, API serializer, database representation and tests.

## D4. Frontend engineering requirements

- Use current repository framework and coding conventions where suitable.
- Implement real routing, layouts, reusable controls and responsive behavior.
- Connect controls to actual service functions; no dead buttons, fake actions or fake success.
- Implement loading, empty, error, unauthorized, pending, success, disabled and retry states where relevant.
- Use schema-backed form validation and accessible labels.
- Keep search/filter/sort/pagination query state in a durable URL/query model where appropriate.
- Do not use sample customers, sample revenue, fake book ownership, fake reviews or fake orders in production paths.
- Build development seed data separately and label it as demo data.
- Prevent frontend leakage of server secrets and private record data.
- Use safe rich-text/HTML rendering; never inject arbitrary untrusted HTML.
- Verify breakpoints at 1440px, 768px, 390px and intermediate responsive widths.
- Use keyboard, focus, screen-reader and reduced-motion support.
- Track each screen with route, component, API operation, permissions, tests and evidence.

# E. BACKEND AND DATABASE — REAL PERSISTENCE

Build one authoritative backend with modular domain boundaries. Prefer a modular monolith initially unless repository evidence justifies services.

Implement the actual database using the database provider and migration tool chosen after repository audit. PostgreSQL is a candidate only if consistent with existing project and owner decisions.

At minimum, reconcile and implement the following domains:

## E1. Identity and access
- customer identity and profile;
- staff identity/profile;
- roles and permissions;
- role assignments;
- sessions;
- email verification and recovery;
- MFA for privileged staff;
- invitation lifecycle;
- account status and session revocation.

## E2. Catalog and digital content
- ebooks/products;
- contributor/author attribution;
- categories;
- collections;
- tags and search metadata;
- ebook cover and alternate assets;
- private ebook files;
- sample/preview files;
- immutable file versions;
- pricing and publication state;
- CMS pages, sections and storefront placements.

Author/contributor attribution is content data only. No author portal is authorized.

## E3. Commerce
- Bag/cart and items;
- price/coupon quote;
- checkout session/quote;
- orders and immutable order-line snapshots;
- payment attempts;
- webhook receipts/events;
- refund requests and provider refunds;
- invoices/receipts subject to approved law/policy;
- coupon uses;
- financial ledger/reconciliation.

## E4. Entitlement and reader
- entitlement;
- grant/revoke history;
- file access/download event;
- reader session if needed;
- reading progress;
- bookmarks;
- highlights/notes only as supported by approved reader policy.

## E5. Customer support and operations
- support tickets;
- assignments/history;
- public messages;
- private internal notes;
- safe attachments;
- knowledge base articles and versions;
- saved replies/macros;
- SLA settings/events if approved;
- notifications/preferences;
- audit events.

## E6. Database requirements
- Complete ERD and data dictionary.
- Versioned executable migrations for blank install and safe upgrade.
- Correct FK/unique/check constraints and indexes.
- Integer minor units + ISO currency for money.
- UTC timestamps.
- immutable financial/order snapshots.
- safe soft-delete/archive/anonymization behavior.
- no destructive cascades into financial/audit records.
- transaction boundaries and concurrency control.
- transactional outbox/inbox or equivalent reliable async processing.
- migration tests, data backfill scripts, rollback/forward-fix plan.
- development-only seed data.
- blank-database install, upgrade, backup and restore evidence.

Do not produce only logical Markdown tables and claim the database is implemented. Create actual migrations and test them in the repository.

# F. AUTHENTICATION, AUTHORIZATION AND TENANT/RECORD ISOLATION

Use deny-by-default authorization and least privilege.

Separate customer and staff contexts. A customer cannot become an admin/support user by changing local storage, URL, role claim or request body.

Every endpoint and object access must enforce:
- authenticated identity;
- role/permission;
- ownership/record scope;
- allowed state transition;
- input validation;
- audit requirements for sensitive actions.

Test cross-customer and cross-role IDOR for orders, invoices, tickets, messages, reader sessions, downloads, bookmarks, files, staff, settings and exports.

Support agents can investigate with limited data but cannot automatically refund, edit financial records, publish catalog items, change provider secrets or grant/revoke entitlement.

Admin roles must be granular: view/create/edit/delete/publish/approve/refund/export/manage-permissions as separately controlled. Enforce MFA and step-up authentication for privileged/high-risk actions.

Implement rate limits, brute-force controls, secure password hashing, secure session cookies where appropriate, CSRF protection, generic auth/recovery errors, secure password-reset tokens, audit logs and secrets rotation.

# G. PRODUCTION PAYMENT INTEGRATION — HIGH PRIORITY

The frontend checkout demonstrates:
- guest or sign-in flow;
- email association;
- UPI (GPay, PhonePe, Paytm, BHIM);
- credit/debit cards (RuPay, Visa, Mastercard);
- Net Banking;
- rupee-denominated sample prices;
- configurable GST display;
- separate success, pending and failed screens.

Treat these as design signals, not approval of market/currency/tax or a payment provider. `D-003` gateway selection, `D-002` jurisdiction/currency and `D-004` tax/invoice policy remain decision items.

## G1. Select and integrate provider safely

- Inspect existing provider integration first.
- If one is already implemented and merchant-approved, audit and complete it.
- If none is selected, research current official provider documentation and merchant eligibility for the confirmed operating country and required UPI/cards/netbanking/refunds/webhooks/settlements.
- Present a concise provider comparison and recommendation for owner approval.
- Build a provider-neutral payment adapter and a separate test/sandbox adapter.
- Do not silently enable an unapproved live gateway.
- Do not fabricate provider success.
- Never put server secrets into frontend bundles or commit them.

## G2. Required payment lifecycle

1. Server recalculates all prices, coupons, tax and payable total.
2. Server creates immutable order snapshot and pending payment attempt.
3. Server creates payment intent/order with provider.
4. Customer completes provider UI.
5. Browser return is informational only.
6. Backend validates signed webhook over raw body.
7. Persist event receipt and idempotently process.
8. Confirm via provider API if ambiguous.
9. Reject amount/currency/order reference mismatch.
10. Mark captured only after verified provider evidence.
11. Transactionally persist financial state and outbox event.
12. Fulfil one entitlement per eligible order line exactly once.
13. Customer success screen and Library query canonical backend state.
14. Reconcile payments, refunds and settlements.
15. Alert and support pending/failed/unmatched payments.

Handle duplicate webhook delivery, out-of-order events, timeout, retries, browser closure, double click, provider outage, payment success after timeout, partial refund and settlement mismatch.

## G3. Refund
Implement customer request/support investigation, approval policy, provider execution, async result, ledger/reconciliation and notification. Separate requester/approver/executor where required. A request is not a completed refund. Entitlement revocation follows explicitly approved refund-access policy; do not revoke at request time.

## G4. Tax, invoice and terms
Do not copy sample GST zero. Configure a versioned tax engine after jurisdiction/professional review. Record rule version and calculation snapshot. Store accepted version of Digital Edition Terms and Refund Policy. Legal texts must be owner-approved before checkout launch.

## G5. Production credential gate
Live payment activation requires an approved merchant/provider account, live credentials, registered and verified webhook, settlement account, supported method confirmation and controlled production smoke/reconciliation. Without these external prerequisites, finish the code and sandbox suite but mark live payment as BLOCKED—not complete.

# H. EBOOK FILES, READER AND CUSTOMER ACCESS

- Store original ebook files in private object storage.
- Validate extension, MIME, file signature, size and safe upload content.
- Quarantine/scan uploaded files before publication.
- Version files immutably.
- Publish explicit preview assets separately from full paid assets.
- Enforce server-side entitlement before every reader/download/stream.
- Use short-lived signed URLs or authenticated streaming.
- Log access/download events without storing file content in logs.
- Avoid cache leakage of personalized/private resources.
- Do not claim browser DRM can prevent screenshots/copying.
- Define file version replacement and access after refund according to owner-approved rules.
- EPUB/PDF are indicated in product specs and UI examples but must be verified against approved format/reader decisions and actual renderer/license/security.

Reader must use genuine supported document rendering, not an image/static mock. Persist reading progress, bookmarks and supported notes securely, with ownership and book-version checks.

# I. EMAIL, NOTIFICATIONS, SEARCH, CMS AND ANALYTICS

Implement real service integrations or clearly mark unavailable:
- transactional email: registration/verification, recovery, order, payment status, refund, support response, security notices;
- customer/admin/support notification center and preferences;
- search with safe query handling and server pagination;
- CMS content with safe sanitization and draft/publish/version workflow;
- coupons with server-side eligibility and concurrency protection;
- metrics from persisted business facts, with documented formulas;
- export jobs with authorization, audit, expiry and privacy controls.

Do not claim email is sent if provider call fails. Use retries, idempotency and delivery status. Do not use fake demo values for production analytics.

# J. API AND CONTRACTS

Complete `contracts/openapi.yaml` to cover all in-scope public/customer/admin/support operations and webhooks. The current V2 OpenAPI is explicitly a starter, not complete.

- Reconcile endpoints with existing routes and domain services.
- Use consistent versioning and error envelopes (Problem Details if compatible).
- Document auth/roles, request/response, validation, pagination, filters, state transitions and idempotency.
- Generate/check client types where the stack supports it.
- Add contract tests.
- Keep customer DTOs separate from staff DTOs.
- Do not return private internal ticket notes to customer APIs.
- Do not expose provider secrets, raw card data, private file keys or hidden internal error detail.

# K. QUALITY ASSURANCE AND SECURITY

Build and run:
- unit tests;
- database and migration tests;
- API integration tests;
- OpenAPI validation and contract tests;
- payment adapter/webhook signature and idempotency tests;
- refund tests;
- entitlement and file-access tests;
- RBAC/IDOR negative tests;
- customer/admin/support E2E tests;
- browser console/network checks;
- responsive visual checks;
- accessibility tests;
- load/performance tests against agreed traffic assumptions;
- dependency, secret and static security scans.

At minimum test:
- pending/failed payment never grants paid access;
- success redirect without server/provider verification never grants access;
- duplicate webhook creates one result;
- mismatched amount/currency is rejected;
- Customer A cannot read Customer B's order/ticket/reader/file;
- support agent cannot execute unauthorized refund;
- internal note never appears in customer ticket API;
- admin UI hidden action is still denied by API for unauthorized role;
- private ebook URL cannot be fetched after entitlement revocation/expiry;
- refund request alone does not revoke access;
- reports reconcile to ledger/settlement records;
- coupons cannot be oversubscribed in concurrency tests.

Use a risk-based security review aligned to current OWASP ASVS/API Security guidance and privacy law review for the launch jurisdiction. Do not claim certification without assessment.

# L. IMPLEMENTATION EXECUTION METHOD

Do not implement everything as one risky giant rewrite. Work through the V3 phases in dependency order:

0. repository and archive audit;
1. decisions, architecture and contracts;
2. engineering foundation, DB migrations, identity and RBAC;
3. catalog, CMS, upload and search;
4. customer discovery and account;
5. Bag, checkout, payment adapter, webhooks and reconciliation;
6. entitlements, Library, reader and downloads;
7. customer orders, invoices and support;
8. Support/Operations Panel;
9. Owner/Admin Panel;
10. cross-panel integration;
11. security, performance and operational hardening;
12. staging and production deployment.

After every coherent phase:
- run relevant tests;
- fix discovered defects;
- update traceability and docs;
- commit changes with a meaningful message;
- produce a phase report with commit hash, files changed, migrations, APIs, screens, tests, blockers and rollback notes.

Continue autonomously through all phases that are not blocked by owner-controlled legal/provider decisions. Do not pause for confirmation on routine implementation choices already constrained by the docs. For a genuinely blocking owner choice, present the exact decision, alternatives, recommendation, consequences and affected phase, then continue unrelated safe work.

# M. DEPLOYMENT AND OPERATIONS

Prepare:
- development, CI, staging and production environments;
- secure environment variables/secrets;
- CI build/test/migration validation;
- reviewed migration deployment;
- HTTPS/domain/DNS/security headers;
- transactional email sender authentication;
- private storage/CDN rules;
- monitoring, logging, alerts and correlation;
- database/object storage backups;
- proven restore procedure;
- RPO/RTO decisions;
- release and rollback runbook;
- incident/payment reconciliation runbooks.

Never expose secrets, production customer data or private ebook files. Never deploy destructive migrations without backups and approval.

# N. OWNER-ONLY DECISION REGISTER — DO NOT FABRICATE

Use `decisions/OPEN-DECISIONS-REGISTER.md`. Key unresolved decisions include:
- launch country/currency and seller legal entity;
- payment provider/merchant account;
- tax/GST/invoice rules;
- guest checkout and email-verification policy;
- supported ebook file formats and reader engine;
- download/watermark rules;
- refund eligibility and entitlement after refund;
- hosting/database/storage/email providers if not dictated by repository;
- staff MFA/session policy;
- privacy/data retention/legal terms;
- expected load, availability, RPO/RTO and support SLA.

For the frontend checkout, guest option, UPI/cards/netbanking and ₹ formatting are observed, but not sufficient to approve the relevant legal or provider decisions.

Implement configurable boundaries and sandbox behavior while decisions remain open. Do not turn a proposed default into live business policy.

# O. FINAL REQUIRED OUTPUT

The final response and repository documents must state:

1. What has actually been implemented.
2. Which of the 83 screens have completed routes and backend connections.
3. Which APIs and DB entities/migrations exist.
4. Which workflows are fully tested.
5. Payment provider, mode and test evidence.
6. Whether real live payments are active or blocked.
7. Refund and entitlement behavior and its approved policy reference.
8. Security and negative-test results.
9. Browser/responsive/accessibility results.
10. Deployment URL/environment only if actually deployed.
11. Backup/restore and rollback evidence.
12. Unresolved owner decisions and external credentials.
13. Known defects and deferred items.
14. Exact local setup, test, migration, deploy and operations commands.

**Never make unsupported claims.** If a provider credential, legal approval, domain, repository access, live deployment permission or other external item is missing, say so precisely. Finish all independent work, but do not label the application production-ready until the go-live gates pass.

Begin by listing the contents of `BACKEND_BLUEPRINT_V3/`, inspecting the extracted `FRONTEND_DESIGN_EXPORT/` directory, and auditing the current repository. Build the implementation tracker for all 83 screens and continue through the phases above. If either required input directory is genuinely unavailable, identify exactly which input is missing and stop only the dependent work.
