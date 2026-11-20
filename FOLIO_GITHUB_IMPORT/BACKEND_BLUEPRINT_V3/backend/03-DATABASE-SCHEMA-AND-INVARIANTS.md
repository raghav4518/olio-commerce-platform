# Backend 03 — Database Schema, Constraints and Invariants

## 1. General schema standards

The repository audit must determine whether an existing schema can be evolved safely. If no adequate DB exists, use a relational transactional database as the default. PostgreSQL is a strong candidate, but the final provider and version are an ADR after repository and hosting review.

### Naming and data types

- Use consistent `snake_case` names.
- Primary keys: one consistent UUID/ULID strategy; do not mix ID schemes casually.
- Money: `BIGINT`/decimal-safe integer minor units and a 3-character ISO currency code. Define currency exponent per ISO 4217; do not assume every currency has two decimals.
- Rates: fixed-precision decimal or integer basis points; never binary float.
- Timestamps: timezone-aware UTC timestamps.
- Hashes: fixed-length binary/hex according to chosen algorithm.
- JSON: only for bounded provider payload snapshots or flexible metadata; do not bury relational, query-critical fields in arbitrary JSON.
- Statuses: constrained enums/check constraints/reference tables with application transition validation.
- Every external provider ID must be stored separately from internal IDs and have an appropriate uniqueness constraint.

## 2. Core table definitions (logical columns)

This is a data dictionary baseline. The agent must produce exact DDL and migrations after stack audit. Column names may be adapted to established repository conventions only if semantic meaning and constraints remain equivalent.

### 2.1 Identity

**`users`**

- `id` PK
- `email_normalized` unique (if email is the account identifier)
- `email_verified_at` nullable
- `status` (`pending_verification`, `active`, `suspended`, `closed`)
- `display_name` optional
- `locale` optional
- `created_at`, `updated_at`, `closed_at` nullable

Email normalization rules must be explicit and safe; preserve original display form separately if needed. Do not apply provider-specific transformations such as removing dots from Gmail addresses globally.

**`password_credentials`**

- `user_id` PK/FK
- `password_hash`
- `hash_algorithm` / parameter metadata
- `password_changed_at`
- `created_at`, `updated_at`

Never store plaintext, reversible passwords or password hints.

**`user_sessions`**

- `id` PK (opaque, high-entropy session reference or keyed hash representation)
- `user_id` FK
- `session_secret_hash` if secrets are stored separately
- `created_at`, `last_seen_at`, `expires_at`, `revoked_at`
- `auth_context`, `mfa_satisfied_at`, `ip_prefix_hash`/minimal device metadata as approved

Do not log raw session cookies. Session secret material must be non-recoverable from DB backups.

**`roles`, `permissions`, `role_permissions`, `user_role_assignments`**

- Unique role slug and permission key.
- Unique role/permission pair.
- Role assignment has `assigned_by_user_id`, `assigned_at`, `revoked_at`, and optional scope.
- Do not allow users to grant permissions they do not possess (prevent privilege escalation).

**`email_verification_tokens`, `password_reset_tokens`**

Store token hashes only, expiry, single-use consumed timestamp, user FK, created time and minimal request metadata. Add indexes for lookup hash and expiry cleanup.

### 2.2 Catalog and asset metadata

**`ebooks`**

- `id` PK, unique public `slug`
- `title`, optional `subtitle`, `short_description`, `description`
- `author_id` FK nullable only if approved
- `language_code`, optional `isbn`
- `publication_date`, `publisher_label` optional
- `status` (`draft`, `scheduled`, `published`, `archived`)
- `published_at`, `created_by_user_id`, `updated_by_user_id`
- SEO fields and public metadata
- `created_at`, `updated_at`, `archived_at`

**`categories`**

- `id`, unique `slug`, `name`, description, `parent_id` self-FK
- `sort_order`, `status`, SEO fields, timestamps
- Prevent cycles in category hierarchy at service/database level.

**`digital_assets`**

- `id`, `asset_kind` (`cover`, `ebook_source`, `sample`, `attachment`, etc.)
- `storage_provider`, `bucket/container`, `object_key`, `object_version`
- `original_filename` (sanitized display value), `detected_mime_type`, `size_bytes`
- `sha256_checksum`, `scan_status`, `processing_status`
- `uploaded_by_user_id`, `created_at`, `quarantined_at`, `deleted_at`
- No public URL field for original files. Store object key and version only.

**`product_file_versions`**

- `id`, `ebook_id`, `asset_id`, `format` (`pdf`, `epub`, `other-approved`)
- `purpose` (`main`, `preview`, `cover`, `sample`)
- `version_number`, `is_current`, `access_policy_key`
- `created_at`, `created_by_user_id`, optional release notes
- Unique (`ebook_id`, `purpose`, `version_number`); enforce one current main version per ebook/purpose.

### 2.3 Prices, carts and orders

**`product_prices`**

- `id`, `ebook_id`, `currency_code`, `amount_minor`
- `valid_from`, `valid_until`, `status`, `created_by_user_id`
- Check amount is non-negative; one applicable active price per ebook/currency/time window.

**`carts`, `cart_items`**

- Cart has owner user ID or a securely generated guest key, never both without explicit merge semantics.
- Cart items reference ebook and quantity; digital single-user ebook quantity defaults to one and should be constrained unless gifting/licensing is later approved.
- Cart is mutable; checkout creates a price/discount/tax snapshot so later price changes do not mutate an active order.
- Expire stale carts by configured policy; do not delete order history.

**`checkout_sessions`**

- `id`, `cart_id`, customer/user or guest reference, `status`, `expires_at`
- Price/tax/discount snapshot or canonical quote reference
- `idempotency_key_hash`, created/updated timestamps
- A checkout session must be invalidated/requoted when cart contents, price, coupon eligibility or relevant tax rules change.

**`orders`**

- `id`, human-readable unique `order_number`
- `customer_user_id` nullable only if guest checkout is approved
- `checkout_session_id`, `currency_code`
- `subtotal_minor`, `discount_minor`, `tax_minor`, `total_minor`
- `status` (`awaiting_payment`, `paid`, `partially_refunded`, `refunded`, `cancelled`)
- `placed_at`, `paid_at`, `cancelled_at`, timestamps
- immutable billing/contact and tax snapshots as legally required

**`order_items`**

- `id`, `order_id`, `ebook_id` nullable only if catalog deletion policy requires retaining snapshots
- `product_title_snapshot`, `product_slug_snapshot`, `author_snapshot`
- `quantity`, `unit_amount_minor`, `discount_amount_minor`, `tax_amount_minor`, `line_total_minor`
- `currency_code`, `product/file version reference snapshot`
- Unique/validation rules to prevent unapproved duplicate lines for the same ebook.

### 2.4 Payment and refund records

**`payment_attempts`**

- `id`, `order_id`, provider key, provider payment/session reference
- `amount_minor`, `currency_code`, `status`, `created_at`, `expires_at`, `updated_at`
- Do not store PAN, CVV or full cardholder authentication data.

**`payment_transactions`**

- `id`, `payment_attempt_id`, provider transaction ID unique within provider
- transaction type (`authorization`, `capture`, `void`, `refund`, `chargeback`, `fee`, etc.)
- amount, currency, provider status, canonical status, occurred/recorded timestamps
- immutable provider reference and minimal normalized metadata

**`payment_webhook_events`**

- unique (`provider_key`, `provider_event_id`)
- event type, received timestamp, signature_verified_at, payload hash, minimally retained/redacted payload
- processing status, attempt count, processed timestamp, failure classification
- Never mark a webhook verified before cryptographic verification passes.

**`refunds` / `refund_items`**

- refund ID, order/payment references, requested amount, currency, reason code, free-text reason with access controls
- status and actor timestamps: requested, reviewed, approved/rejected, submitted, succeeded/failed
- provider refund reference unique within provider
- refund item amounts sum to refund total; refunded total cannot exceed refundable captured amount.

### 2.5 Entitlements and reading

**`entitlements`**

- `id`, `user_id`, `order_item_id` unique for purchase grant
- `ebook_id`, `access_status` (`pending`, `active`, `suspended`, `revoked`, `expired`)
- `granted_at`, optional `starts_at`, optional `expires_at`, `revoked_at`, reason/source
- `allowed_file_version_policy` reference/snapshot
- Entitlement activation occurs only after a verified paid state or approved comp grant.

**`download_events`**

- entitlement, asset version, request timestamp, outcome, size and coarse request context
- Avoid storing a complete raw IP/device fingerprint unless justified and disclosed.

**`reading_progress`, `bookmarks`**

- Unique progress identity per entitlement + reader/document version strategy.
- Store normalized location using a format-aware locator (EPUB CFI/progression or PDF page/position); do not assume page numbers map across formats.
- User must be able to reset/delete progress where appropriate.

### 2.6 Support, audit and infrastructure records

Support tables must separate customer-visible messages from staff-only notes. Internal notes and sensitive attachments must never appear in customer DTOs. Audit and outbox/inbox tables must be append-only or state-transitioned with tightly controlled update paths.

## 3. Mandatory database invariants

The agent must implement and test:

1. Unique normalized account identity under the approved identity policy.
2. Unique product slugs; prevent duplicate current product prices for overlapping validity windows.
3. No published ebook without an approved current main asset that passed file validation and malware scanning.
4. Order total equals the stored sum of item values, discounts, delivery (not applicable for digital) and tax under the stored quote.
5. All money amounts in an order use one currency unless explicit multi-currency order support is designed.
6. Captured payment amount and currency reconcile to the order's payable amount; partial capture support must be explicit if provider supports it.
7. Duplicate provider webhook event IDs do not duplicate state changes.
8. A refund cannot exceed the remaining refundable captured balance.
9. An order item cannot create duplicate purchase entitlements on webhook retry.
10. A revoked/suspended/expired entitlement cannot issue a new reader/download grant.
11. Every staff role assignment and sensitive action records actor and timestamp.
12. A support internal note cannot be returned by customer-facing queries.
13. A file in quarantine or with failed scanning cannot be delivered or attached to a published ebook.
14. Object keys and provider secrets must not be exposed in public DTOs.
15. Deleting a user/product does not cascade-delete legally or financially required order, payment, refund and audit facts.

## 4. Index strategy

At minimum evaluate indexes for:

- `users(email_normalized)` unique.
- sessions by user/status/expiry and token hash.
- catalog status + publication timestamp; slug unique; category hierarchy.
- product price lookup by ebook/currency/status/validity.
- carts by user/status/updated_at; cart items by cart.
- orders by customer/date, order number unique, status/date.
- order items by order and ebook.
- payment attempts by order/status; provider references unique.
- webhook provider/event ID unique and processing queue index.
- refunds by order/status/request date and provider refund reference unique.
- entitlements by user/status/grant time; unique order item.
- support ticket by status/priority/assignee/updated time.
- audit events by actor/object/time.
- outbox by published status/created time; idempotency by scope/key unique.

Validate indexes against query plans and realistic test data; avoid indexing every column.

## 5. Migration discipline

- Use versioned, repeatable migration tooling.
- Never edit an already applied production migration; create a forward migration.
- Use expand → migrate/backfill → contract for risky schema changes.
- Backfills must be resumable, observable and throttled.
- Test migrations from the previous production schema and from a clean database.
- Include rollback strategy; where rollback is unsafe, document a forward-fix plan.
- Keep migrations compatible with rolling deployment windows.
- Verify backup and restore before destructive migration.

## 6. Retention, deletion and anonymization

Implement a data-retention registry with owner/legal-approved periods by data class. Support account closure, export, correction and deletion/anonymization workflows as legally applicable. Do not set arbitrary universal retention periods. Preserve transactions, tax records, fraud evidence and litigation holds only for approved lawful purposes. Delete or anonymize other data after approved expiry. Apply deletion to replicas, search indexes, analytics projections and backups according to the documented retention and backup-erasure policy.

## 7. Required deliverable before schema coding

Produce:

- Mermaid ERD reviewed by engineering and product owner.
- Data dictionary with every table/column, type, nullability, default and purpose.
- Foreign keys, unique/check constraints and index list.
- Transaction boundary matrix.
- Data classification and retention mapping.
- Migration plan, seed strategy and rollback/backfill plan.
- Automated schema invariant tests.
