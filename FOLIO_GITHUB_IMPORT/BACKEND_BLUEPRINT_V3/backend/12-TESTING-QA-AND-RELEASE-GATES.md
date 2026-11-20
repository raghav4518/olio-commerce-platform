# Backend 12 — Testing, Quality Assurance and Release Gates

## 1. Test pyramid

- Unit tests for domain rules, price calculations, transitions and permission policies.
- Integration tests against the actual selected relational DB and realistic transaction semantics.
- Contract tests for API, webhooks, event schemas and provider adapters.
- End-to-end tests across all three interfaces once frontend implementation exists.
- Security tests for auth, object authorization, injection, file delivery and business-flow abuse.
- Resilience tests for provider timeouts, duplicate events, worker restarts, DB interruptions and queue backlog.
- Load tests for realistic traffic and file-delivery patterns.

Mocks are useful for deterministic unit tests but do not replace integration tests against real DB constraints, payment sandbox/webhook fixtures or object storage behaviour.

## 2. Domain test matrix

### Identity and authorization

- Registration verification token expiry, replay and single-use.
- Login throttling, generic errors and session rotation.
- Password reset token expiry/replay and session revocation.
- MFA enforcement for staff and step-up sensitive operations.
- Suspended user and revoked session denial.
- Cross-customer IDOR attempts across orders, library, downloads, progress and tickets.
- Cross-role denial for every admin/support operation.
- Permission grant escalation and revoked role propagation.

### Catalog/assets

- Draft/published/archived visibility.
- Cannot publish without validated main file, metadata and approved price.
- Category cycles and duplicate slug prevention.
- Concurrent edit/version conflicts.
- MIME mismatch, file signature mismatch, size limits, path traversal and ZIP bomb.
- Malware scan fail/pending/outage behaviour.
- Private storage object cannot be retrieved without approved grant.

### Commerce/pricing

- Price/tax/discount snapshot determinism.
- Coupon expiry, usage limits, per-customer limits, stacking and concurrent redemption.
- Cart change during checkout.
- Price change between browse and checkout.
- Duplicate purchase policy.
- Currency mismatch and rounding.
- Idempotent checkout creation under concurrent requests.

### Payments/refunds

- Valid/invalid/duplicate/delayed/out-of-order provider webhook.
- Browser redirect without webhook does not mark paid.
- Provider transaction amount/currency/order mismatch.
- Payment success followed by DB outage and worker recovery.
- Timeout after provider side effect followed by safe reconciliation.
- Partial/full refund and over-refund attempt.
- Refund timeout and duplicate submit.
- Dispute/chargeback event handling.
- Settlement mismatch/reconciliation.

### Entitlements/reader

- Only paid order item creates purchase entitlement.
- Duplicate event cannot duplicate entitlement.
- Comp grant requires explicit permission/reason.
- Suspended/revoked/expired entitlement cannot create reader/download grant.
- Short-lived URL/grant expires and cannot be reused outside scope.
- Version replacement follows approved prior-buyer policy.
- Progress and bookmarks are account-isolated and format-aware.

### Support

- Internal notes are never returned in customer API.
- Support agent sees only permitted customer/order context.
- Ticket assignment, escalation and status transitions are valid.
- Support cannot execute refund unless separately authorized.
- Customer can only read/reply to own tickets.

## 3. Non-functional testing

- Performance: baseline and peak load for search, checkout, webhook intake, library, reader-session creation, file delivery and reports.
- Concurrency: coupon limits, payment retries, refunds, entitlement grants, staff role changes.
- Resilience: dependency outage, queue retry, worker termination, DB connection loss, object storage throttle.
- Accessibility/frontend integration: API errors, validation and auth states must be consumable by accessible UI; final UI WCAG review belongs to frontend workstream.
- Compatibility: supported browsers/devices and file-reader engines are set through reader decision and tested.
- Data migration: clean install, prior version upgrade, rollback/forward-fix and backfill resumption.

## 4. Contract and schema verification

- OpenAPI parser/linter runs in CI.
- Breaking API change detection runs against baseline contract.
- DB migrations run on a clean database and a copy of the previous schema/data fixture.
- Event/JSON schemas validate producer and consumer samples.
- Authorization policy tests are mandatory for every protected endpoint.
- Provider webhooks have signature fixtures and negative tests.

## 5. Security verification

Map verification to selected OWASP ASVS 5.0.0 requirements and API Security Top 10. Include:

- SAST, SCA, secret scan and container/image scan where relevant.
- DAST against staging.
- Manual review of payment, auth, file upload, support and admin privilege boundaries.
- Rate-limit and abuse tests.
- SSRF and file-parser security review if remote import/parsing features are introduced.
- Log redaction review.
- Backup encryption and restore permissions review.

Any unresolved high/critical finding requires documented risk owner, compensating control, expiry date and approval. Do not silently waive findings.

## 6. Release gates

### Gate A — Contract ready

- Product scope and owner-blocking decisions resolved for the features in the release.
- ERD/data dictionary/constraints reviewed.
- OpenAPI and event contracts validated.
- Permission matrix and state machines accepted.

### Gate B — Feature ready

- Acceptance criteria pass.
- Unit/integration/contract tests pass.
- Negative authorization tests pass.
- Audit and operational telemetry exist.
- Failure/retry/recovery path tested.

### Gate C — Staging ready

- Production-like deployment and provider sandbox tested.
- Migrations and rollback/forward-fix rehearsed.
- E2E flows pass for all relevant panels.
- Security and load test findings triaged.
- Support/finance runbooks are usable.

### Gate D — Production ready

- Live provider account and webhooks configured/verified.
- Tax, invoice, refund, privacy and consumer policies professionally reviewed for launch markets.
- Backup, DB restore and object recovery tested.
- Alerts, on-call ownership and incident response ready.
- Security sign-off and owner approval recorded.
- No unresolved critical/high risk without explicit risk acceptance.

## 7. Required test evidence

For every phase, retain CI run IDs or test reports, exact command/results, test environment, fixtures, coverage where meaningful and known failure list. Do not report “all tests passed” unless all required suites ran against the intended commit/environment.
