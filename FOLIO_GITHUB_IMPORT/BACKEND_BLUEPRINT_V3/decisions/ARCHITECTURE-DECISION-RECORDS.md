# Architecture Decision Records (ADRs)

**Status note:** These are proposed decisions for review, not automatically approved owner decisions. The repository audit may show that an existing choice should be retained. Record approval evidence and date before treating an ADR as accepted.

## ADR-0001 — Begin with a modular monolith

- **Status:** Proposed
- **Context:** Three panels share transactional business domains; initial scope is one owner-operated store.
- **Options:** Modular monolith; microservices; serverless functions per domain.
- **Recommendation:** Modular monolith with explicit domain modules and background workers. Separate deployment processes only where needed, from one coherent codebase initially.
- **Rationale:** Keeps transactions and operations manageable while preserving future extraction boundaries.
- **Consequences:** Modules must own their logic/data; no direct cross-module table mutation. Scaling bottlenecks are measured before extraction.

## ADR-0002 — Use one primary relational source of truth

- **Status:** Proposed
- **Context:** Orders, payments, refunds, entitlements and permissions require transactions, constraints and auditability.
- **Options:** Relational DB; document DB; polyglot multi-database design.
- **Recommendation:** One managed relational transactional database; PostgreSQL is the leading candidate after repository audit.
- **Consequences:** Search/cache/analytics stores, if later added, are derived projections and never the financial/entitlement source of truth.

## ADR-0003 — Use versioned REST APIs described by OpenAPI

- **Status:** Proposed
- **Context:** Three independently developed panel interfaces need stable contracts.
- **Recommendation:** REST/JSON under `/api/v1`, with OpenAPI source of truth, typed clients where practical, RFC 9457 errors and contract tests.
- **Version note:** OpenAPI 3.2.1 is the current referenced version as of this package date. Confirm codegen, linter and framework support; if unsupported, use the newest compatible OAS 3.1.x version and record the reason rather than producing invalid tooling output.

## ADR-0004 — Store purchased ebook originals in private object storage

- **Status:** Proposed
- **Context:** Digital files are purchased content; permanent public URLs are unacceptable.
- **Recommendation:** Private object storage, quarantine ingestion, checksums/scanning, immutable versions, short-lived scoped delivery grants. Public CDN only for reviewed covers/previews.
- **Consequences:** File delivery, revocation window, signed URL exposure and recovery need explicit tests/runbooks.

## ADR-0005 — Prefer secure server-managed browser sessions

- **Status:** Proposed
- **Context:** The initial panels are browser-based first-party interfaces.
- **Recommendation:** Opaque server-side sessions in Secure HttpOnly cookies, CSRF and Origin controls, session rotation/revocation, MFA for staff.
- **Alternative:** Existing sound OIDC/session architecture may be retained if audited and documented. Do not introduce browser-stored long-lived bearer tokens by default.

## ADR-0006 — Keep financial amounts integer/minor-unit and immutable snapshots

- **Status:** Proposed
- **Context:** Financial correctness and future reconciliation.
- **Recommendation:** Integer minor units with ISO currency code and explicit currency exponent. Snapshot prices/discounts/tax at order creation. Use immutable transactions and compensating records.

## ADR-0007 — Use transactional outbox and idempotent consumers

- **Status:** Proposed
- **Context:** Payment webhooks, notifications, refunds, asset scanning and entitlement grants span DB and external services.
- **Recommendation:** Persist outbox row in the same DB transaction as business state; use durable worker delivery, idempotent handlers and dead-letter operations.

## ADR-0008 — No multi-vendor architecture in initial release

- **Status:** Proposed; depends on owner confirmation of D-001
- **Context:** Current approved panels do not include seller/author portals.
- **Recommendation:** Treat authors as public metadata, not seller accounts. Do not add seller wallets, vendor payout logic or tenant isolation until marketplace scope is explicitly approved.

## ADR-0009 — Do not promise DRM-proof content protection

- **Status:** Proposed
- **Context:** Web readers and downloadable formats cannot prevent all copying, photography or screenshots.
- **Recommendation:** Use layered controls (private storage, entitlement checks, scoped grants, monitoring and optional proportionate watermarking) and accurate customer language.

## ADR-0010 — Defer final frontend visual system

- **Status:** Proposed; owner is currently designing frontend direction
- **Context:** Backend blueprint can define API/identity/data contracts without freezing visual treatment.
- **Recommendation:** Do not implement final palette, typography, component styling, page composition or interactions until the design direction and selected visual references are approved. Maintain responsive/accessibility needs as non-visual requirements.
