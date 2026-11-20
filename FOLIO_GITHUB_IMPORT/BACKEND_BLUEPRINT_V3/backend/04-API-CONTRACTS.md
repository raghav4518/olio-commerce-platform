# Backend 04 — API Contracts and Integration Rules

## 1. API design principles

- Use HTTPS only outside local development.
- Expose one versioned API used by all three panels; separate customer, admin and support policy namespaces without duplicating domain logic.
- REST/JSON is the default. Use multipart only for controlled file upload initiation/finalization or provider-supported upload flows.
- Publish and validate an OpenAPI document. `contracts/openapi.yaml` is the starter contract, not a complete endpoint specification until all operations below are added.
- Prefer resource-oriented nouns and explicit action endpoints only for state transitions/commands.
- Do not expose internal DB schema directly as the public API shape.
- Do not return secrets, raw storage keys, payment credentials, internal notes or provider payloads unnecessarily.
- Validate request bodies, path/query parameters and content types server-side.

## 2. API versioning and URLs

Recommended base path: `/api/v1`.

Logical groups:

- `/api/v1/public/*` — public storefront and catalog reads.
- `/api/v1/auth/*` — login, registration, verification, reset, session actions.
- `/api/v1/me/*` — authenticated customer's own profile, cart, orders, library and support.
- `/api/v1/admin/*` — owner/admin operations with permission checks.
- `/api/v1/support/*` — support workspace operations with restricted scope.
- `/api/v1/webhooks/{provider}/*` — provider callbacks; separate authentication and rate-limit policy.

These prefixes are organizational. The coding agent may align them to established repository conventions, but must preserve the permission boundaries and document mapping.

## 3. Common request and response conventions

### Request metadata

- `X-Request-ID`: optional caller correlation ID; server validates/creates a trusted internal ID.
- `Idempotency-Key`: required for checkout creation, payment/refund commands and other money/access-affecting retriable POST operations.
- `If-Match`/ETag or version field: recommended for concurrent edits to products, settings and support tickets.
- CSRF token/header: required for cookie-authenticated state-changing browser requests.

### Success envelope

Use one consistent convention across all endpoints. A suggested resource response is:

```json
{
  "data": { "id": "resource-id", "status": "active" },
  "meta": { "requestId": "server-generated-id" }
}
```

Collections should use cursor pagination:

```json
{
  "data": [],
  "meta": {
    "requestId": "server-generated-id",
    "page": { "nextCursor": null, "hasMore": false }
  }
}
```

Do not mix offset and cursor pagination for the same collection without a reason. Use stable ordering by an indexed field plus unique ID.

### Errors

Use RFC 9457 Problem Details (`application/problem+json`) with stable `type`, `title`, HTTP `status`, safe `detail`, `instance`, `code`, `requestId` and field-level validation errors where appropriate. Never return stack traces, SQL errors, provider secrets or internal exception data.

Typical codes: `VALIDATION_ERROR`, `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `IDEMPOTENCY_CONFLICT`, `RATE_LIMITED`, `PAYMENT_PENDING`, `PAYMENT_FAILED`, `ENTITLEMENT_INACTIVE`, `FILE_NOT_READY`, `PROVIDER_UNAVAILABLE`.

## 4. Endpoint inventory — public storefront

| Method | Path | Purpose | Authorization |
|---|---|---|---|
| GET | `/public/storefront` | Public store settings/navigation/homepage composition | Public |
| GET | `/public/ebooks` | Search/filter/paginate published ebooks | Public |
| GET | `/public/ebooks/{slug}` | Public product details and current display price | Public |
| GET | `/public/categories` | Active category tree | Public |
| GET | `/public/categories/{slug}` | Category details and products | Public |
| GET | `/public/authors/{slug}` | Public author profile and ebooks | Public |
| GET | `/public/collections` | Published collections | Public |
| GET | `/public/collections/{slug}` | Collection details and ebooks | Public |
| GET | `/public/pages/{slug}` | Published CMS page | Public |
| GET | `/public/search` | Search public metadata | Public; abuse controls |
| GET | `/public/ebooks/{id}/preview` | Preview authorization/preview metadata | Public only if preview published |

Public responses must exclude private file object keys, unpublished metadata, internal margins/costs, moderation notes and buyer-only file locations.

## 5. Endpoint inventory — authentication and customer

| Method | Path | Purpose | Authorization |
|---|---|---|---|
| POST | `/auth/register` | Create account if self-registration is approved | Public, rate-limited |
| POST | `/auth/sessions` | Authenticate and create secure session | Public, rate-limited |
| DELETE | `/auth/sessions/current` | End current session | Authenticated |
| POST | `/auth/sessions/revoke-all` | Revoke all sessions | Authenticated + step-up as configured |
| POST | `/auth/email-verification` | Request/resend verification | Public/authenticated, throttled |
| POST | `/auth/email-verification/confirm` | Consume single-use token | Public, token validated |
| POST | `/auth/password-reset/request` | Request generic password reset response | Public, rate-limited |
| POST | `/auth/password-reset/confirm` | Set new password with token | Public, token validated |
| POST | `/auth/mfa/challenge` | Complete required MFA challenge | Auth challenge context |
| GET | `/me` | Current account summary | Customer |
| GET/PATCH | `/me/profile` | View/update permitted profile fields | Self |
| GET/PATCH | `/me/preferences` | Communication and reading preferences | Self |
| GET | `/me/orders` | Own order history | Self |
| GET | `/me/orders/{orderId}` | Own order detail and invoice link | Self + ownership policy |
| GET | `/me/library` | Own active library entitlements | Self |
| GET | `/me/library/{entitlementId}` | Entitlement detail and allowed formats | Self + active entitlement |
| POST | `/me/library/{entitlementId}/download-grants` | Issue short-lived single-purpose download grant | Self + active entitlement |
| POST | `/me/library/{entitlementId}/reader-sessions` | Start authorized reader session | Self + reader-enabled entitlement |
| PATCH | `/me/reading-progress/{entitlementId}` | Update format-aware reading location | Self + active entitlement |
| GET/POST/DELETE | `/me/wishlist` | Manage wishlist | Self |
| POST | `/me/reviews` | Submit product review subject to policy | Self + purchase/eligibility rule |
| GET/POST | `/me/support-tickets` | List/create own support tickets | Self |
| GET | `/me/support-tickets/{ticketId}` | Read own ticket and customer-visible messages | Self + ownership policy |
| POST | `/me/support-tickets/{ticketId}/messages` | Reply to own ticket | Self + ticket state |
| POST | `/me/checkout-sessions` | Validate cart, lock quote and create checkout session/order | Customer/approved guest |
| GET | `/me/checkout-sessions/{id}` | Get checkout status and safe payment action metadata | Checkout owner |
| POST | `/me/checkout-sessions/{id}/payment-attempts` | Create provider payment attempt | Checkout owner + idempotency |

Guest endpoints, account-claim behaviour and whether guest carts/orders exist are blocked by decision D-005.

## 6. Endpoint inventory — cart and coupons

| Method | Path | Purpose | Authorization |
|---|---|---|---|
| GET | `/me/cart` | Retrieve current cart | Customer/approved guest |
| POST | `/me/cart/items` | Add ebook to cart | Customer/approved guest |
| PATCH | `/me/cart/items/{itemId}` | Change supported item attributes/quantity | Cart owner |
| DELETE | `/me/cart/items/{itemId}` | Remove item | Cart owner |
| DELETE | `/me/cart` | Empty cart | Cart owner |
| POST | `/me/cart/coupon-validation` | Validate coupon and return quote, not redeem | Cart owner |

Cart pricing is advisory until checkout. Checkout must revalidate publication status, price, coupon eligibility, tax configuration and entitlement/duplicate purchase policy.

## 7. Endpoint inventory — owner/admin

| Method | Path | Purpose | Required permission (example) |
|---|---|---|---|
| GET | `/admin/dashboard` | Authorized operational KPIs | `dashboard.read` |
| GET/POST | `/admin/ebooks` | List/create ebooks | `catalog.read` / `catalog.create` |
| GET/PATCH/DELETE | `/admin/ebooks/{id}` | Read/update/archive product | `catalog.read/edit/archive` |
| POST | `/admin/ebooks/{id}/publish` | Publish after validations | `catalog.publish` |
| POST | `/admin/ebooks/{id}/unpublish` | Unpublish product | `catalog.unpublish` |
| POST | `/admin/ebooks/{id}/file-upload-sessions` | Start secure file upload | `assets.upload` |
| POST | `/admin/assets/{assetId}/finalize` | Verify uploaded asset and start scan | `assets.upload` |
| GET | `/admin/assets/{assetId}` | View processing status | `assets.read` |
| GET/POST/PATCH | `/admin/categories` | Manage category hierarchy | `catalog.categories.manage` |
| GET/POST/PATCH | `/admin/authors` | Manage public author records | `catalog.authors.manage` |
| GET/POST/PATCH | `/admin/collections` | Manage collections | `content.collections.manage` |
| GET | `/admin/orders` | Search/filter orders | `orders.read` |
| GET | `/admin/orders/{id}` | Order detail and timeline | `orders.read` |
| GET | `/admin/payments` | Search payment attempts/transactions | `payments.read` |
| GET | `/admin/refunds` | Refund queue | `refunds.read` |
| POST | `/admin/refunds/{id}/approve` | Approve refund | `refunds.approve` + step-up |
| POST | `/admin/refunds/{id}/reject` | Reject with reason | `refunds.reject` |
| POST | `/admin/refunds/{id}/submit` | Submit to provider where separated from approval | `refunds.execute` + step-up |
| GET | `/admin/customers` | Search customers | `customers.read` |
| GET | `/admin/customers/{id}` | Customer summary | `customers.read` |
| POST | `/admin/customers/{id}/suspensions` | Suspend with reason | `customers.suspend` |
| DELETE | `/admin/customers/{id}/suspensions/current` | Reactivate under policy | `customers.reactivate` |
| GET | `/admin/entitlements` | Search entitlements | `entitlements.read` |
| POST | `/admin/entitlements/{id}/suspend` | Suspend access with reason | `entitlements.suspend` |
| POST | `/admin/entitlements/{id}/restore` | Restore access with reason | `entitlements.restore` |
| GET/POST/PATCH | `/admin/coupons` | Manage coupon definitions | `marketing.coupons.manage` |
| GET/POST/PATCH | `/admin/cms/pages` | Manage CMS pages/revisions | `content.pages.manage` |
| GET/POST/PATCH | `/admin/settings` | Manage allowed store settings | granular `settings.*` permissions |
| GET | `/admin/audit-logs` | Search audit events | `audit.read` |
| GET/POST/PATCH | `/admin/roles` | Manage roles/permissions | `security.roles.manage` + MFA |
| GET/POST/PATCH | `/admin/staff` | Manage staff accounts/assignments | `security.staff.manage` + MFA |

## 8. Endpoint inventory — support/operations

| Method | Path | Purpose | Required permission |
|---|---|---|---|
| GET | `/support/queue` | Tickets assigned/available to staff | `support.queue.read` |
| GET | `/support/tickets/{id}` | Ticket with allowed customer/order context | `support.tickets.read` |
| POST | `/support/tickets/{id}/assignments` | Assign/reassign ticket | `support.tickets.assign` |
| PATCH | `/support/tickets/{id}` | Update status/priority/category | `support.tickets.update` |
| POST | `/support/tickets/{id}/messages` | Send customer-visible response | `support.messages.send` |
| POST | `/support/tickets/{id}/internal-notes` | Add private staff-only note | `support.notes.create` |
| GET | `/support/customers/{id}/summary` | Limited customer context | `support.customers.lookup` + access reason |
| GET | `/support/orders/{id}/summary` | Limited order/payment context | `support.orders.lookup` + access reason |
| POST | `/support/refund-requests` | Create refund investigation/request | `support.refunds.request` |
| POST | `/support/download-issues` | Record/report delivery issue | `support.downloads.investigate` |

Support APIs must not expose full gateway credentials, card details, customer passwords, private staff notes beyond the role, or broad bulk customer exports.

## 9. Webhook endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/webhooks/payments/{provider}` | Verify and persist payment-provider events |
| POST | `/webhooks/email/{provider}` | Process delivery/bounce/suppression events |

Webhook handlers must verify raw-body signatures using provider-specific rules, timestamp/replay controls when available, and provider-specific key rotation. They must persist the verified event with a unique provider event ID before acknowledging success. Never use a browser redirect as a webhook substitute.

## 10. OpenAPI completion rules

`contracts/openapi.yaml` defines the contract conventions and initial paths. Before a feature is implementation-ready:

- Every endpoint in the inventory must have operation ID, tags, summary, security requirements, path/query/header parameters, request schema, success schema and problem responses.
- All DTOs must be explicit; do not use an unbounded `object` for domain payloads.
- Pagination, sorting and filters must be specified.
- Enum values and nullability must match the implementation/database.
- Admin/support operations must list required permission scopes in descriptions or extensions and have automated authorization tests.
- Webhook request bodies must reflect provider verification requirements; do not pretend one generic schema works for all providers.
- Run an OpenAPI parser/linter and breaking-change check in CI.

## 11. Rate limiting and abuse controls

Apply endpoint-specific controls to login, registration, password reset, email verification, search, coupon validation, checkout creation, payment-attempt creation, downloads, reader session creation, public review creation, support message posting and webhooks. Rate-limit by several signals where lawful/appropriate (account, IP prefix, session, device risk) without creating an easy denial-of-service vector against shared networks. Return `429` with `Retry-After` and a safe error response.

## 12. CORS, CSRF and browser security

- Allow only configured origins; do not use wildcard credentialed CORS.
- If using cookie sessions, use `Secure`, `HttpOnly`, appropriate `SameSite`, CSRF protection and strict Origin/Referer validation for state-changing requests.
- Set appropriate HSTS, Content-Security-Policy, frame/object restrictions, content-type sniffing protection and referrer policy at the edge/app layer.
- Avoid tokens in URL query parameters, local storage or analytics events.
- Use restrictive redirect allowlists after authentication and payment return.

## 13. Contract testing

- Generate client types from OpenAPI where practical; avoid separate hand-maintained incompatible DTOs.
- Validate request/response payloads in integration tests.
- Run compatibility checks for changes to required fields, enum removals, status semantics and authorization requirements.
- Webhook adapters have provider-specific fixture/signature tests.
