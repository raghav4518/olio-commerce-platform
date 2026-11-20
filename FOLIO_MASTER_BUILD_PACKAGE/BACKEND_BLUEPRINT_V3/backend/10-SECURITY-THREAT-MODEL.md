# Backend 10 — Security Architecture and Threat Model

## 1. Security objectives

Protect customer accounts, staff privileges, payment integrity, customer/order/reading data, private ebook assets, business configuration and service availability. Security must be enforced in backend controls and tested; a polished frontend or hidden menu item is not a security boundary.

Use OWASP ASVS 5.0.0 as the primary application-security verification baseline and OWASP API Security Top 10 for API threat review. The required ASVS level and exact controls must be risk-assessed and approved; do not claim formal compliance from a checklist alone.

## 2. Assets and trust boundaries

### Assets

- Customer credentials, sessions and MFA factors.
- Staff credentials, roles and privileged sessions.
- Customer PII, purchase history, support conversations and reading activity.
- Orders, prices, discounts, payment attempts, transactions, refunds and settlement evidence.
- Ebook source files, previews, storage keys and access grants.
- Admin configuration, gateway secrets, webhook secrets, email credentials and encryption keys.
- Audit records, backup data and deployment credentials.

### Trust boundaries

1. Browser/mobile client → edge/API.
2. Customer session → staff/admin authorization boundary.
3. API → primary DB.
4. API/worker → object storage.
5. API/webhook → payment provider.
6. Worker → email/message/search/analytics provider.
7. Production → staging/development.
8. Support role → finance/owner-only operations.

## 3. Threat model (STRIDE-oriented)

| Threat | Example | Required mitigations |
|---|---|---|
| Spoofing | Stolen customer/staff session | MFA for staff, secure sessions, rotation, revoke, anomaly alerts |
| Tampering | Browser changes order amount or entitlement ID | Server-side price calculation, object authorization, signed state, DB constraints |
| Repudiation | Staff denies issuing refund or revoking access | Append-only audit with actor, reason, time and correlation ID |
| Information disclosure | Predictable file URL leaks purchased ebook | Private object storage, scoped short-lived grants, access checks |
| Denial of service | Credential stuffing, download/search abuse | Rate limits, WAF, provider quotas, bounded queues, capacity controls |
| Elevation of privilege | Support agent calls admin endpoint | Deny-by-default RBAC, object-level policy, automated negative tests |

## 4. API-specific risks

Address at minimum:

- Broken object-level authorization (IDOR/BOLA) on orders, library, downloads, tickets, refunds and customers.
- Broken function-level authorization on admin/support routes.
- Broken authentication, credential stuffing and session fixation.
- Unrestricted resource consumption (search, upload, downloads, report exports).
- Business-flow abuse (coupon brute force, checkout replay, refund abuse, fake review, account creation abuse).
- Server-side request forgery if URL import or external resource fetching is added; do not add URL-based asset import without strict allowlists and egress controls.
- Security misconfiguration, verbose error responses, unsafe CORS and weak headers.
- Unsafe consumption of provider webhooks and external payloads.
- Inventory/resource authorization drift between DB, cache, object storage and search index.

## 5. Secure coding requirements

- Validate all untrusted input at the boundary using explicit schemas.
- Encode output for context; protect against XSS and stored HTML/script in ebook metadata, CMS, reviews, support messages and file metadata.
- Use parameterized queries/ORM-safe APIs; no raw string-concatenated SQL.
- Use CSRF controls for cookie-authenticated state changes.
- Restrict CORS, redirects, Content Security Policy, frame embedding and MIME sniffing.
- Apply content-type and file signature checks; never trust extension alone.
- Set body, upload, export and pagination limits.
- Use secure random values for tokens and grants.
- Use maintained cryptographic libraries; do not implement custom crypto.
- Redact PII, credentials, tokens, file URLs and payment payloads from logs.
- Run dependency/SCA scanning, secret scanning, SAST and targeted DAST in CI/release pipeline.

## 6. Secrets and key management

- Store secrets in a managed secret store or encrypted deployment secret mechanism.
- Separate secrets by environment and provider account.
- Rotate provider/API/webhook keys with documented dual-key transition where supported.
- Restrict who can read or rotate secrets; access is audit logged.
- Encrypt sensitive data in transit and at rest using managed KMS/key management where available.
- Separate encryption keys from encrypted data; define rotation and backup recovery.
- Never put secrets in `.env` committed to source control, frontend build variables, tickets, screenshots or logs.

## 7. Payment security boundary

Prefer hosted payment flow so the application never handles raw card entry. Verify payment signatures and reconcile provider-side state. Keep provider references minimal in logs. Determine PCI DSS obligations with the actual provider/acquirer and integration model; do not claim reduced scope or compliance without confirmation.

## 8. Ebook/file security boundary

Private object storage with block-public-access policy. Validate and scan uploads, quarantine unknown/unsafe assets, use short-lived grants, check current entitlement before issuing a grant, and avoid public origin exposure. CDN caching must distinguish public cover/preview from private purchased content. Do not claim browser DRM prevents screenshots or copying.

## 9. Audit and detection

Record sensitive operations: staff login/MFA changes, role changes, payment/refund actions, gateway configuration, bulk export, customer suspension, entitlement grant/revoke, file version replacement, security setting changes and data deletion. Audit logs should be append-only to application roles, with restricted reader roles and tamper-resistant retention where proportionate.

Alert on unusual failed logins, privilege changes, multiple refund failures, payment amount mismatches, invalid webhook bursts, unusual download patterns, repeated object authorization failures, secrets access anomalies, backup failures and dead-letter growth.

## 10. Privacy and Indian market review

The owner must identify launch jurisdictions and engage qualified counsel to assess applicable privacy/data-protection, consumer, ecommerce, tax and record-retention requirements. If India is a launch market, specifically review the Digital Personal Data Protection Act and applicable rules/commencement/notifications in force at launch, alongside other applicable Indian requirements. The application must support purpose limitation, notices, consent/preferences where required, access/correction/deletion handling, processor/vendor records and breach-response workflows as applicable.

Do not assume that one checkbox or a generic privacy policy satisfies legal requirements. Do not store more personal data than needed. Reading activity and support content may be sensitive in context and require restricted access.

## 11. Security release gates

- Threat model reviewed and risks assigned.
- No known critical/high exploitable findings left unresolved without a documented, approved exception.
- Authentication and session tests pass.
- Complete authorization matrix tests pass, including negative cross-customer and cross-role tests.
- Payment/webhook replay, signature and mismatch tests pass.
- File upload and private delivery tests pass.
- Secret scan and dependency security checks pass.
- Audit events verified for sensitive actions.
- Backup restore and incident runbooks exercised.
- Security headers, TLS, CORS, CSRF and rate limits verified in deployed environment.
