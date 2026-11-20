# Backend 18 — Database Implementation and Migration Deliverables

**Version:** 3.0.0  
**Purpose:** Turn the logical V2 data model into a real, tested schema in the repository's actual database.

## 1. Current documented state

V2 contains a logical ERD, logical column definitions, constraints/invariants, and index guidance. It does not ship a repository-specific executable migration history or prove which database provider the current application uses. PostgreSQL is a reasonable relational candidate, but the agent must inspect the repository and current data before locking a database or generating destructive migrations.

Do not treat this blueprint's logical model as an excuse to replace an existing suitable database.

## 2. Required repository audit before migrations

Record:
- database engine and exact version;
- ORM/query builder/migration tool and version;
- current schema and migration history;
- current live/dev/staging data locations;
- existing foreign keys, indexes, triggers, extensions and constraints;
- backup status and restore test status;
- current connection pooling/transaction behavior;
- current environment configuration and secret management;
- current tests that depend on schema/seed data.

Produce a schema diff and data-risk report before altering an existing database.

## 3. Required logical domains/entities to reconcile

Use V2 `backend/02-DATA-MODEL-AND-ERD.md` and `backend/03-DATABASE-SCHEMA-AND-INVARIANTS.md` as the canonical domain starting point. Reconcile at least these concepts and avoid duplicate entities:

### Identity and access
- user/account identity;
- customer profile;
- staff profile;
- roles, permissions and role-permission assignments;
- user role assignments;
- sessions and MFA/challenges;
- email verification/password recovery tokens or their provider-safe equivalents.

### Catalog and content
- ebook/product;
- author/contributor attribution (profile/content only, no author portal);
- category and ebook-category association;
- collection and collection-item association;
- ebook file asset and immutable file version;
- cover and preview asset;
- price/price history or versioned price data;
- publication/content version;
- CMS page/section/placement and media asset.

### Commerce and finance
- cart and cart item;
- checkout quote or order draft where needed;
- order and immutable order item snapshot;
- payment attempt;
- provider event/webhook receipt;
- refund request and refund transaction;
- financial ledger/journal or reconciliation model;
- coupon and coupon eligibility/usage;
- invoice/receipt and invoice line/tax snapshot;
- provider settlement/reconciliation record.

### Entitlement and reading
- ebook entitlement;
- entitlement event/history;
- download/access event;
- reader session if needed;
- reading progress;
- bookmark;
- highlight/note if those controls are present and approved.

### Support and operations
- support ticket;
- ticket assignment/history;
- public ticket message;
- private internal note (separate visibility field/table and strict serializer policy);
- attachment metadata;
- knowledge base article/version;
- saved reply/macro;
- SLA configuration and event history, if configured;
- notification and user preferences;
- audit event.

Do not add marketplace seller, royalty, split settlement, subscription, gifting or affiliate entities unless scope is formally expanded.

## 4. Required executable artifacts

After selecting/confirming the repository database and migration tooling, generate:
1. canonical ERD in Mermaid or DBML and a rendered image if tooling supports it;
2. database dictionary (table, column, type, nullability, default, meaning, classification);
3. executable initial migrations for new development installations;
4. incremental migrations for existing installations;
5. indexes and unique constraints based on access paths and invariants;
6. referential constraints and safe delete behavior;
7. development-only seed scripts with obviously fake data;
8. rollback/down strategy where supported; for irreversible migrations, a forward-fix plan;
9. data migration/backfill scripts with dry-run, counts, checkpointing and recovery notes;
10. migration tests on a blank database and a copy/fixture of the pre-existing schema;
11. schema drift check in CI;
12. backup and restore test record.

Do not create only an ERD screenshot and call the database complete. Do not embed production credentials in migration files.

## 5. Database invariants

At minimum, enforce in the database wherever possible:
- primary and foreign keys for all owned relationships;
- unique normalized email/identity rules consistent with account policy;
- unique product slug and unique external/internal product identifiers where applicable;
- unique provider payment reference within provider/account scope;
- unique provider event ID within provider scope to stop webhook replay;
- unique idempotency key within operation/owner scope;
- one active entitlement for a given customer/order-line/product scope as policy defines;
- money fields as integer minor units + currency code;
- quantity constraints appropriate for digital goods;
- non-negative amounts except explicit ledger reversals;
- valid state enumerations/check constraints where practical;
- immutable order item/payment/refund/invoice snapshots after finalization;
- ticket owner and message visibility constraints;
- no cascade delete from users/catalogue into financial or audit history;
- timestamps stored consistently, normally UTC;
- indexes supporting owner-scoped reads and queue operations.

The application must still validate invariants and authorization; database constraints do not replace service-layer checks.

## 6. Transaction boundaries

Document and test transaction boundaries for:
- placing an order and reserving/locking a quote;
- recording a verified payment and scheduling fulfilment;
- creating entitlement from an outbox event;
- approving/executing a refund;
- coupon usage redemption;
- staff permission changes and their audit event;
- ticket assignment/status and activity history;
- file version publication.

Use the transactional outbox/inbox pattern or a demonstrably equivalent reliable mechanism for webhook-to-fulfilment processing. Avoid a design where a database commit succeeds but a required event is silently lost.

## 7. Data deletion, retention and privacy

- Classify personal, financial, authentication, content and operational data.
- Define retention periods after legal review.
- Preserve statutory financial records and audit evidence as required.
- Provide a privacy-aware account deletion/anonymization workflow.
- Keep ownership links necessary to prevent cross-account access.
- Separate hard delete from deactivate/archive.
- Ensure log and analytics systems do not become shadow databases of sensitive content.
- Do not delete paid order, refund, ledger or audit records to satisfy a generic cascade.

## 8. Migration run acceptance

A migration phase is done only when:
- blank DB install succeeds;
- upgrade from previous supported schema succeeds;
- data count/integrity checks pass;
- indexes and constraints are verified;
- rollback/forward-fix approach is documented;
- all migration tests pass;
- backups exist before any production migration;
- restore is demonstrated;
- no production data is destroyed or reset;
- generated ORM types/schema code are in sync.
