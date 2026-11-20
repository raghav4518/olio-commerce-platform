# Backend 11 — Infrastructure, Deployment, Observability and Disaster Recovery

## 1. Infrastructure design principles

Choose managed infrastructure that fits team expertise, expected workload, budget, data residency and recovery requirements. Avoid premature complexity. The final vendor/region decisions require repository and business review.

## 2. Logical infrastructure

- Edge/CDN/WAF/TLS termination.
- Three frontend deployments/origins or equivalent route-separated applications.
- Shared backend API runtime.
- Background worker runtime.
- Managed relational primary database with automated backups and point-in-time recovery where available.
- Private object storage for source ebooks and restricted attachments.
- Public CDN for reviewed cover/public preview assets only.
- Durable queue/event transport (may be managed queue or DB-backed job system at initial scale).
- Cache/rate-limit store only where necessary.
- Secret manager/KMS.
- Centralized structured logs, metrics, traces and alerting.

## 3. Environment separation

- Local/dev: fake data, local or sandbox adapters, no production secrets.
- Test/integration: automated migrations, ephemeral data, deterministic test services.
- Staging: provider sandbox, production-like routing/configuration, sanitized data only.
- Production: separate accounts/projects/network/secrets, restricted access and deployment approval.

Do not share production payment keys, email secrets, object storage buckets or DB credentials with non-production.

## 4. Network and access controls

- Only required services are publicly reachable.
- Database, queue, cache and private object storage are not exposed to the public internet unless the managed provider requires private authenticated endpoints.
- Use workload identity/short-lived credentials for service-to-service access where supported.
- Restrict admin/operations access to production by MFA, least privilege and audited access.
- Use egress controls for workers where reasonable, especially if handling uploaded files.
- Separate management plane access from public application access.

## 5. CI/CD pipeline

Minimum pipeline:

1. Formatting, lint and type checks.
2. Unit tests.
3. Schema/OpenAPI/JSON Schema validation.
4. Secret scan and dependency/SCA scan.
5. SAST and license policy checks.
6. Build reproducibly; produce artifact/SBOM where tooling permits.
7. Integration tests against supported DB/queue/object-storage adapters.
8. Migration dry-run and compatibility validation.
9. Deploy to staging.
10. E2E and security smoke tests.
11. Approval gate for production.
12. Progressive production deployment/health checks.
13. Post-deploy verification and rollback/forward-fix readiness.

Do not run destructive DB migrations in a separate uncoordinated deploy that can break old application instances. Use expand/contract migrations.

## 6. Configuration and release management

- Configuration is typed, validated at startup and separated by environment.
- Non-secret business settings may use authorized admin configuration.
- Credentials/keys are managed outside the DB-backed admin settings model.
- Feature flags have owner, purpose, default, expiry/review date and kill switch.
- Use controlled rollouts for high-risk payment, reader and entitlement changes.
- Every release is tied to a commit/build identifier and migration set.

## 7. Observability

### Structured logging

Include timestamp, severity, service, environment, version, request ID, trace ID, actor ID where safe, route/operation, outcome and error class. Redact or omit passwords, tokens, cookies, provider secrets, full webhook payloads, signed URLs, file bytes and unnecessary PII.

### Metrics

Platform:
- Request volume, latency percentiles, error rate and saturation.
- DB connections, slow queries, lock waits, replication/backup health.
- Queue depth, oldest message age, retries and dead letters.
- Worker duration/failure by job type.

Commerce:
- Checkout creation and completion rates.
- Payment attempts/capture success and pending age.
- Webhook ingestion/processing lag and duplicate rate.
- Amount/currency mismatch count.
- Refund submission/success/failure and unresolved age.
- Entitlement grant delay/failure.

Digital delivery:
- Upload/scan duration and rejection reason.
- Reader/download success/failure and latency.
- Unauthorized access attempts and grant issuance volume.

Support:
- Queue size/age, first response time, resolution time, reopen rate and escalation count.

These are observability measures, not approved business KPI definitions; metric formulas must be documented with product/finance.

### Distributed tracing

Propagate request/correlation/causation IDs across API, DB operations, outbox, workers and provider adapters. Avoid using PII or tokens as trace attributes.

## 8. Alerts and runbooks

Define actionable alerts with severity, owner, threshold, deduplication and runbook link. Minimum alerts: API outage/error budget breach, DB capacity/failure, queue backlog, webhook lag, payment mismatch, refund failure, entitlement grant failure, scan queue stuck, notification provider outage, backup failure, restore test failure, high auth attack signals and disk/storage growth.

## 9. Backup and disaster recovery

- Back up DB and critical configuration; object storage versioning/replication/backup policy must be designed separately.
- Encrypt backups and restrict restore permissions.
- Enable point-in-time recovery where available and appropriate.
- Define RPO and RTO with the owner based on cost and business impact; do not invent values.
- Test restore to an isolated environment on a schedule.
- Verify DB backup and digital object backup consistency; a DB restore without corresponding ebook objects is incomplete.
- Include queues/outbox state, encryption key recovery and DNS/provider credential recovery in the disaster plan.
- Document who can declare disaster, restore sequence, customer communication and post-recovery reconciliation.

## 10. Deployment and rollback

- Use health/readiness checks, graceful shutdown and worker drain.
- Use rolling/blue-green/canary strategy based on hosting capabilities.
- Make migrations backward-compatible with the previous release during rollout.
- Keep rollback artifacts and procedures; for irreversible schema changes, use forward-fix plans.
- Ensure payment webhooks remain accepted during deployments or are durably retried.
- Confirm background jobs are safe across mixed application versions.

## 11. Capacity and scaling

Load test realistic catalog/search, peak checkout, webhook bursts, simultaneous reader sessions, large downloads and support/admin reporting. Define capacity based on test results and budgets. Scale stateless API/workers horizontally when needed; use DB indexes/connection pooling and CDN/private delivery architecture before introducing distributed services. Do not advertise support for an arbitrary number of concurrent users without test evidence.

## 12. Operational documents required

- Deployment runbook.
- Rollback/forward-fix runbook.
- Database backup/restore runbook.
- Object storage recovery runbook.
- Payment reconciliation/refund runbook.
- Webhook incident runbook.
- File scanning/reader outage runbook.
- Credential rotation and emergency access runbook.
- Security incident and privacy breach response plan.
- On-call escalation and vendor contact register.
