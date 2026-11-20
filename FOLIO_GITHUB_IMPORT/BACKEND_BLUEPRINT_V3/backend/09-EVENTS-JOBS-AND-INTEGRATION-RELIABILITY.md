# Backend 09 — Events, Background Jobs and Integration Reliability

## 1. Design objective

External calls and slow work must not make commerce state unreliable. The backend uses durable state transitions and retryable jobs. A job may execute more than once; every handler must be idempotent or guarded by a unique business-effect constraint.

## 2. Transactional outbox pattern

When a database transaction changes business state and requires a follow-up action:

1. Commit domain state and an `outbox_events` row in the same DB transaction.
2. A worker claims unpublished events using a concurrency-safe method.
3. Worker publishes/handles the event and records delivery/attempt status.
4. Consumer stores an inbox/deduplication record before applying the effect.
5. Failed operations retry with exponential backoff and jitter.
6. Exhausted events move to a dead-letter/review state with alerting and replay tooling.

Do not write to the DB and publish to a queue as two unrelated actions without a recovery mechanism.

## 3. Canonical domain event catalogue

Events are versioned contracts. Include `event_id`, `event_type`, `schema_version`, `occurred_at`, `producer`, `correlation_id`, `causation_id` and a minimized payload reference.

| Event | Produced after | Typical consumers |
|---|---|---|
| `catalog.ebook.created.v1` | Draft created | Audit, search projection |
| `catalog.ebook.published.v1` | Valid publish committed | Storefront cache/search, notifications |
| `catalog.ebook.unpublished.v1` | Unpublish committed | Cache/search invalidation |
| `asset.upload.finalized.v1` | Upload finalized | Malware/format scan worker |
| `asset.scan.completed.v1` | Scan completed | Catalog readiness workflow |
| `commerce.checkout.created.v1` | Checkout/order snapshot saved | Payment attempt flow/analytics |
| `payment.webhook.received.v1` | Verified event durably persisted | Payment handler |
| `payment.captured.v1` | Canonical capture persisted | Order paid/entitlement transaction, notification |
| `payment.failed.v1` | Failure persisted | Customer status notification, analytics |
| `refund.requested.v1` | Refund requested | Support/finance queue |
| `refund.approved.v1` | Approval committed | Provider refund command |
| `refund.succeeded.v1` | Provider outcome confirmed | Order aggregate, entitlement policy, notification |
| `entitlement.granted.v1` | Entitlement persisted | Library projection, access email |
| `entitlement.revoked.v1` | Revocation committed | Access cache invalidation, audit |
| `support.ticket.created.v1` | Ticket created | Assignment/notification |
| `support.ticket.updated.v1` | Ticket transition committed | Notification, SLA reporting |
| `notification.requested.v1` | Business event requires message | Notification worker |
| `customer.account.closed.v1` | Approved closure committed | Data lifecycle workflows |

Events must not include secrets, full payment payloads, unnecessary customer PII or signed file URLs.

## 4. Job catalogue

| Job | Trigger | Retry/handling |
|---|---|---|
| File malware/format scan | Asset finalize | Retry transient scanner outage; quarantine on unresolved result |
| Cover/preview processing | Asset scan pass | Idempotent by asset/version |
| Payment status reconciliation | Schedule/manual case | Provider rate limits; compare canonical state before update |
| Refund submission/status check | Approved refund | Provider idempotency; query before retry after timeout |
| Settlement import/reconciliation | Scheduled/manual | Cursor checkpoint and dedupe provider rows |
| Email/message delivery | Notification event | Exponential retry, bounce/suppression handling, dead-letter |
| Scheduled publication/campaign | Due time | Recheck status and schedule version before transition |
| Coupon expiry/usage summaries | Schedule/event | Derived, rebuildable projection |
| Analytics aggregation | Event/schedule | Idempotent time-window partitions |
| Abandoned cart reminders | Schedule | Consent and preference checks; do not email without legal basis |
| Data retention/deletion | Schedule/request | Legal hold checks and auditable completion |
| Backup verification | Schedule | Alert on missing/failed backup/restore check |

## 5. Retry policy

Each integration defines:

- retryable vs non-retryable error classification;
- max attempt count or deadline;
- exponential backoff with jitter;
- rate-limit handling using provider hints;
- idempotency requirements;
- poison-message/dead-letter path;
- operator replay procedure;
- alert threshold and owner.

Never retry permanent validation errors indefinitely. Never blindly retry a payment/refund command after a timeout without provider query/idempotency evidence.

## 6. Job execution model

- Jobs carry stable IDs and correlation IDs.
- Job payloads contain identifiers, not full customer/payment records where avoidable.
- Workers reload current authoritative state and verify it is still eligible before executing.
- Use a lease/visibility timeout and heartbeat for long jobs.
- Use bounded concurrency and provider-specific rate limits.
- Record start/end, attempts, result class and failure trace reference; sanitize logs.
- Expose a restricted operational view to authorized staff/admins without allowing arbitrary payload editing.

## 7. Notifications

Notification service owns templates, localization, preferences, provider routing, retry and delivery log. Domain services emit notification intent; they do not call email providers inline during payment/order transactions.

Transactional notifications include verification, password reset, order receipt, payment status, ebook access, refund status and support response. Marketing notifications require separate consent/preference and legal policy checks. Bounces/complaints update suppression state.

## 8. Cache and search projections

Cache/search are derived and disposable. Define invalidation on publish/unpublish/price/collection updates. Do not cache private customer/entitlement responses without strict identity scoping. Cache keys must include authorization-relevant dimensions or avoid caching protected data. A search index must never expose unpublished products or internal fields.

## 9. Event schema compatibility

- Version event names or schema versions.
- Add optional fields compatibly; do not repurpose fields.
- Consumers must tolerate unknown optional fields.
- Maintain schema registry or checked-in JSON Schemas.
- Contract-test producers/consumers.
- Retain replay capability within the approved event retention window.

## 10. Observability and operations

Expose metrics for queue age, backlog, job success/failure, retries, dead letters, outbox lag, webhook processing lag, scan duration and notification failure. Provide secure replay tooling with permission, reason, confirmation and audit. A replay must not bypass idempotency or state validation.
