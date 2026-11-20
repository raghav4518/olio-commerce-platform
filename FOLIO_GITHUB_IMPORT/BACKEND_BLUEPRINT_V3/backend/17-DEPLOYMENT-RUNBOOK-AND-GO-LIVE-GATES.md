# Backend 17 — Deployment Runbook and Production Go-Live Gates

**Version:** 3.0.0  
**Status:** Provider- and hosting-neutral until the source repository and owner deployment choices are audited.

## 1. Deployment principle

Deploy the existing stack after audit. Do not select infrastructure based on fashion or rewrite the system to fit a preferred vendor. Keep frontend, backend, relational database, private object storage and background processing separated by trust boundary even if they are initially deployed on one managed platform.

## 2. Required environments

Maintain isolated:
- local development;
- automated CI/test;
- staging/pre-production;
- production.

Each environment has separate database, storage bucket, payment credentials, webhook secret, email credentials, signing keys, domain configuration and analytics. Production data must never be copied into local or preview environments without approved anonymization.

## 3. Environment/configuration inventory

Document each variable with owner, environment, source, rotation method and whether it is secret. Expected categories:
- public app URL and API URL;
- database connection and pool configuration;
- cache/queue connection if used;
- object storage endpoint, bucket, region and credentials;
- CDN configuration and signed delivery settings;
- payment provider mode, merchant ID/public key, server secret, webhook secret and supported methods;
- email sender/provider credentials;
- session signing/encryption keys;
- token/key rotation identifiers;
- rate-limit backend;
- monitoring/error reporting DSN;
- logging/trace correlation;
- feature flags and maintenance mode.

All secrets must be injected using a secrets manager or platform secret facility. Never put secrets in `.env.example` with real values, browser bundles, HTML, commits, screenshots or logs.

## 4. Build and deployment pipeline

CI should:
1. install locked dependencies;
2. run formatting/lint/type checks appropriate to the stack;
3. run unit and integration tests;
4. validate OpenAPI and generated clients/types;
5. run database migration tests on an ephemeral database;
6. run authorization/payment/webhook/entitlement regression tests;
7. run frontend route/build tests;
8. run end-to-end browser tests for critical paths;
9. run dependency and secret scans;
10. build immutable artifacts and publish versioned images/assets.

CD should:
- deploy to staging first;
- apply only reviewed backward-compatible migrations;
- run smoke tests;
- require approval for production deployment;
- deploy progressively where platform supports it;
- monitor error rate, latency, payment failures and queue lag;
- support rollback of application artifact and a documented forward-fix/data recovery plan for schema changes.

## 5. Web/domain/security baseline

- Use approved domain and DNS.
- Enforce HTTPS and HSTS after validating subdomain/TLS setup.
- Set secure cookies, SameSite and CSRF controls appropriate to auth architecture.
- Set restrictive CORS and Content Security Policy based on required integrations.
- Validate security headers.
- Configure email domain SPF, DKIM and DMARC.
- Prevent public listing/indexing of private ebook storage and attachments.
- Verify CDN cache rules never cache personalized library/order/support responses.
- Restrict admin/support access according to approved operational policy.
- Apply rate limits at edge and application layers.
- Ensure admin/staff MFA before launch.
- Check dependency licenses and vulnerability policy.

## 6. Monitoring and operations

Provide structured, correlated logs and metrics for:
- request rates, latency and error rates;
- authentication failures and rate limiting;
- payment creation, verification and webhook processing;
- duplicate/replayed webhook attempts;
- pending payment age;
- refund execution/failure;
- fulfilment/outbox queue lag;
- entitlement grant/revocation;
- file upload scanning and failures;
- download authorization/abuse;
- email failures;
- database pool, slow queries and locks;
- support queue/SLA state;
- background job retries and dead letters.

Do not log passwords, tokens, card data, UPI PIN, session cookies, private ebook contents or unnecessary PII. Define alert severity, recipient and runbook for each critical signal.

## 7. Backup and disaster recovery

- Automated encrypted database backups.
- Point-in-time recovery if supported and budgeted.
- Object-storage versioning/replication/lifecycle policy appropriate to assets.
- Backup encryption and access control.
- Documented RPO/RTO approved by owner.
- Regular restore drills to isolated environment.
- Verify database and object assets restore consistently.
- Include secrets/configuration recovery and key rotation.
- Record restoration results and elapsed times; do not claim PITR or disaster readiness without performing a restore drill.

## 8. Pre-launch security and legal checks

- Production threat model reviewed.
- Authorization/IDOR regression suite passes.
- Staff MFA and privileged-session policy enabled.
- Upload validation/scanning configured.
- Private file access tested from unauthenticated and wrong-account contexts.
- Payment webhook signature/idempotency and reconciliation tests pass.
- No secrets in code/artifacts.
- Privacy policy, terms, refund policy, cookie/analytics choices approved.
- Seller identity, tax and invoice details professionally reviewed.
- Data retention/deletion process approved.
- Incident response and breach escalation contacts documented.

## 9. Production smoke tests

Run using production configuration with low-value/controlled test records and approved procedures:
- public homepage/catalog/search;
- customer registration/login/recovery;
- customer checkout using actual live provider and allowed test transaction policy;
- verify server-to-server payment and webhook;
- verify one correct order and one entitlement;
- verify library and reader access;
- verify unauthorized account cannot open the same entitlement;
- verify invoice/receipt output;
- verify support ticket create/reply/internal note separation;
- verify admin catalog/publishing permission;
- verify refund path and reconciliation only when allowed;
- verify email delivery;
- verify backups and monitoring alerts.

Never use live customer funds or issue a real refund as a casual test. Coordinate controlled test with provider and finance owner.

## 10. Go-live checklist — every item must have evidence

| Gate | Evidence required | Blocker if missing |
|---|---|---|
| Product scope | Accepted release scope and owner decisions | Yes |
| Repository audit | Stack, routes, data and gap report | Yes |
| Database | Versioned migrations, constraints, backup/restore proof | Yes |
| API | Complete OpenAPI, contract tests, versioning | Yes |
| Auth/RBAC | Server-side authorization tests, staff MFA | Yes |
| Customer flows | E2E test evidence for all in-scope customer paths | Yes |
| Admin/support | E2E and permission matrix tests | Yes |
| Payments | Approved provider, live merchant account, credentials and verified webhooks | Yes |
| Refunds | Approved policy, permission/approval workflow, provider path | Yes |
| Tax/invoices | Professional review and validated jurisdiction rules | Yes |
| Ebook delivery | Private storage, entitlement checks, reader/download tests | Yes |
| Legal/privacy | Approved legal texts, consents, retention/deletion controls | Yes |
| Email/domain | Verified sender domain and transactional email smoke test | Yes |
| Monitoring | Alerts and operational dashboards | Yes |
| Backups | Restore drill and accepted RPO/RTO | Yes |
| Security | Review findings resolved or accepted risk signed off | Yes |
| Performance | Load test results against owner-approved target | Yes |
| Rollback | Application rollback and database recovery plan tested | Yes |
| Operations | Support escalation/incident ownership and runbooks | Yes |

## 11. Honest deployment status

A deployment is:
- **Development-ready** when local setup and automated development tests pass.
- **Staging-ready** when integrated staging flows and operational checks pass.
- **Production-deployable** when all release gates are met and owner prerequisites are present.
- **Live** only after the owner approves go-live and production smoke evidence is recorded.

The agent must never label sandbox-only payment, mock email, local storage, fake analytics or unverified backup as production service capability.
