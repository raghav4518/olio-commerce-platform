# Backend 16 — Production Commerce, Payments, Refunds and Customer Fulfilment

**Version:** 3.0.0  
**Status:** Required implementation contract. Payment provider and legal/merchant configuration remain owner decisions.

## 1. Frontend evidence and interpretation

The supplied checkout HTML explicitly displays:
- guest checkout or sign-in;
- an email address used to link digital editions and invoice;
- UPI options labelled GPay, PhonePe, Paytm and BHIM;
- credit/debit cards (RuPay, Visa, Mastercard);
- Net Banking;
- acceptance of Digital Edition Terms and Refund Policy;
- a Pay button showing a rupee-denominated sample total;
- an explicit simulated failure link;
- line-item amounts, discount and configurable GST.

The archive also contains distinct Payment Successful, Payment Pending Verification, Payment Failed and Order Confirmation screens. A support investigation example refers to a UPI payment, and an access issue example refers to an EPUB download returning 403.

These are design references and sample/demo data. They do **not** approve the payment provider, merchant account, legal seller, currency/market, tax treatment, refund policy, supported ebook formats or live payment activation. The checkout sample's GST value of zero must never be copied as a production tax rate.

## 2. Required checkout architecture

All authoritative price and payment calculations happen server-side.

1. Customer opens Bag. Server returns current item availability, current published prices and coupon eligibility.
2. Customer proceeds to checkout. Server creates a quote snapshot or equivalent with a unique expiry and version.
3. Server recalculates discounts, currency, tax treatment, fees and total using the approved pricing/tax configuration. The browser may display the quote but may not determine the payable amount.
4. Customer identity is resolved according to the approved guest/account policy. The current design supports guest or sign-in and asks for email; D-005/D-006 must be accepted before production.
5. Customer accepts the exact, versioned Digital Edition Terms and Refund Policy. Record policy version, timestamp, account/order context and applicable consent evidence.
6. Backend creates a pending order and payment attempt in a database transaction. Store immutable order-item snapshots (product ID, title/edition, quantity, unit amount, discount allocation, tax treatment and currency).
7. Backend creates a provider-side payment intent/order through the selected adapter. Never expose merchant secret credentials to browser code.
8. Browser is redirected/embedded according to provider's supported flow; browser completion is only a user-experience signal.
9. Provider webhook is received at a public, rate-controlled endpoint. Verify provider signature over the raw request body before trusting it. Persist the provider event id and payload hash safely; reject invalid signatures.
10. Process events idempotently. If provider status is ambiguous, query the provider server-to-server using a secret credential before changing financial state.
11. Mark payment successful only after provider verification. In one durable transaction, update payment/order state and enqueue fulfilment via an outbox. A retryable worker creates the entitlement once.
12. Customer success screen and library show access only when the authoritative API confirms the paid/fulfilled state. Otherwise show Pending Verification or a safe failure/retry state.
13. Reconcile provider transaction/settlement exports against internal payment attempts, refunds and ledger entries.

Never grant an entitlement from the frontend redirect, URL query parameter, client JavaScript callback, manually editable order ID or unverified webhook.

## 3. Payment provider adapter

Create a provider-neutral interface in the repository's existing backend language. It should support provider implementations without coupling domain services to one vendor.

Required operations:
- create provider payment/order/intent;
- retrieve provider payment details;
- verify webhook signature;
- normalize provider events into canonical internal events;
- initiate/refetch refund where provider supports it;
- retrieve settlement/reconciliation data where available;
- return provider reference and safe customer-facing failure category;
- expose provider capability/configuration and health without exposing secrets.

Provider selection:
- D-003 remains unresolved. The UI suggests India-oriented methods but is not enough to select a vendor.
- Audit the legal merchant entity, operating country, settlement bank, desired payment methods, refund APIs, webhook reliability, settlement exports, fees, support and account eligibility.
- If no provider is owner-approved, implement the adapter interface and a clearly isolated sandbox/test adapter; produce a current comparison using official provider documentation and recommend one for approval.
- Do not silently choose a provider for live transactions.
- Do not ship a production configuration that can accidentally use sandbox credentials or fake success responses.
- Use provider-hosted/approved payment UI where appropriate to reduce PCI scope. Never store full card PAN, CVV, UPI PIN, netbanking password or raw payment credentials.

The current screen labels support UPI, cards and Net Banking as intended methods. Availability and naming must reflect the provider's actual capabilities and merchant approval. Do not render a method as usable unless the selected provider supports it in the configured market.

## 4. Canonical financial data and invariants

- Monetary values: integer minor units + ISO 4217 currency code.
- Persist provider identifiers with uniqueness constraints scoped by provider/account.
- Store amount, currency, order reference, payment-attempt reference, provider reference, timestamps, state transitions and safe failure reason.
- A payment attempt must reference one order and have immutable amount/currency after provider intent creation.
- Order item prices, discounts and tax calculation are immutable snapshots at order placement.
- Multiple attempts may exist for one order only under explicit retry rules; only one successful capture may fulfil a given purchase.
- Never sum raw client-submitted amounts.
- Prevent replay/duplicate provider events with unique provider event identifiers and idempotency keys.
- Use database transactions and row locking/optimistic concurrency where appropriate for concurrent webhook/return/retry processing.
- Persist webhook receipt before asynchronous processing; have retry/dead-letter visibility and replay-safe processing.
- Store secrets in a managed secret store or environment injection, not database-readable UI fields or source control.
- Log references and correlation IDs; do not log payment credentials, secret payload fields or sensitive customer data.

## 5. Order, payment and fulfilment state boundaries

Treat order, payment, refund and entitlement as separate aggregates.

Minimum canonical payment states: `created`, `pending`, `authorized` (only if supported), `captured`/`successful`, `failed`, `cancelled`, `partially_refunded`, `refunded`, `disputed`/`chargeback` where supported.

Map provider-specific states through an explicit adapter mapping table. Unknown states must go to an observable safe state such as `needs_review`; do not map unknown events to success.

A pending or failed payment:
- does not become a completed paid order;
- does not create a paid entitlement;
- does not expose a private full ebook asset;
- can show a retry action only when the provider/order state permits it.

A successful verified payment:
- records provider evidence;
- updates order/payment state idempotently;
- creates an outbox event;
- creates one entitlement per eligible order line;
- creates invoice/receipt data only under the approved jurisdictional policy;
- makes the book visible in the customer's Library only after entitlement fulfilment.

For a webhook arriving before the browser return, the customer page must reflect the verified webhook result. For a browser return arriving before a webhook, the page must show Pending Verification and poll/refetch a safe order/payment status endpoint. Support manual reconciliation without granting access based on an agent's unsupported assertion.

## 6. Guest checkout and account claim

The design allows guest checkout or sign-in and uses email to associate digital editions/invoices. D-005 and D-006 remain open.

If guest checkout is approved:
- create an unclaimed guest purchase identity associated with verified email;
- send a time-limited, single-use claim link/code;
- require proof of email ownership before attaching paid entitlements to a registered account;
- prevent an email string alone from granting account access;
- define behavior if the email already belongs to an account;
- retain tax/order records according to policy;
- test account-claim replay, expiry, account takeover and duplicate-registration cases.

If guest checkout is not approved, require account registration/sign-in and define whether email verification is required before payment or before accessing the Library.

## 7. Coupons, discounts and pricing

- Coupon validity must be evaluated on the server at quote and again at order creation.
- Enforce start/expiry, usage cap, per-customer cap, eligibility and stacking rules atomically.
- Do not trust client discount amount.
- Prevent race conditions across concurrent coupon redemptions.
- Store allocated discount per order item for correct refunds/reporting.
- Avoid modifying paid order snapshots when catalogue prices later change.
- Define duplicate-purchase behavior for digital ebooks before launch.

## 8. Tax, invoices and legal terms

D-002/D-004 and relevant legal decisions remain unresolved. The checkout UI shows configurable GST but a displayed sample amount is not a tax ruling.

Before live payments:
- confirm launch jurisdiction(s), seller legal entity, currency, tax registration and invoicing obligations with qualified professionals;
- implement a versioned tax calculation/configuration service, not hard-coded rates;
- define whether prices include/exclude tax and how discounts affect taxable value;
- store the tax inputs, rule version and line-level result on the order;
- generate invoices/receipts with legally required seller/customer/transaction details;
- keep invoice numbering unique, immutable and jurisdiction-aware;
- version terms/refund policy and store evidence of customer acceptance;
- publish only legally approved policy content.

The application must not invent tax rates or generate an invoice that falsely implies legal compliance.

## 9. Refund and chargeback implementation

Refunds are a separate stateful record linked to the original order and payment.

Support:
- customer refund request or support ticket;
- eligibility/policy evaluation;
- support investigation;
- manager/finance/owner review;
- approval or rejection;
- provider refund execution;
- asynchronous refund result/webhook;
- reconciliation and ledger update;
- customer notification;
- approved entitlement treatment.

Requester/approver/executor permissions must be separated according to D-025/D-032. Prevent self-approval where maker-checker is enabled. Require reason, amount, currency, original capture reference, operator, timestamp, approval trail and provider refund reference.

Do not revoke ebook entitlement automatically at refund request. The final action must follow approved D-011/D-012 policy and completed financial refund state. For partial refunds, define exact line/quantity allocation and access consequence before activation.

Chargebacks/disputes, if enabled, must have a documented case workflow, evidence retention, provider state mapping and entitlement policy.

## 10. Financial ledger and reconciliation

Maintain auditable ledger/transaction records sufficient to explain:
- gross item sales;
- allocated discounts;
- taxes collected/payable;
- gateway fees;
- successful captures;
- completed refunds;
- chargebacks/disputes;
- settlement deposits;
- net settlement.

Do not calculate finance dashboard metrics from front-end sample figures. Define metric formulas and reporting timezone. Reconcile daily (or at an owner-approved cadence) provider reports to internal records. Flag unmatched, duplicate, amount/currency-mismatched, stale-pending and reversed transactions for review.

Do not edit captured payment rows in place to make totals reconcile. Use reversing/adjustment entries with audit evidence.

## 11. Customer screen behavior by payment result

- **Successful:** display only after server says payment is captured/verified. Show order reference and library link only after entitlement exists; otherwise explain fulfilment is still processing.
- **Pending:** explicitly say verification is in progress; display order reference, safe refresh/poll mechanism and support entry. No claim of purchase success.
- **Failed:** explain safe reason category, whether retry is available, and how to avoid duplicate payment. Do not display raw gateway diagnostics.
- **Order confirmation:** read from the canonical order service, not from query parameters or cart state.
- **Support investigation:** staff see normalized status and provider reference only; never display merchant secrets or full card data.

## 12. Payment acceptance tests

Required tests include:
1. Valid successful webhook creates one capture and one set of entitlements.
2. Duplicate webhook does not create duplicates.
3. Invalid webhook signature is rejected and audited.
4. Webhook for unknown provider payment goes to reconciliation/review without entitlement.
5. Browser success redirect without webhook/provider verification remains pending.
6. Payment amount/currency mismatch does not fulfil order.
7. Duplicate checkout submission creates no duplicate paid order/capture.
8. Concurrent retries cannot capture/fulfil twice.
9. A late failure event cannot downgrade an already verified success without provider-specific state validation.
10. Pending/failed payment does not expose full ebook or create paid library record.
11. Refund request alone does not revoke access.
12. Approved refund executes once and records provider reference.
13. Refund webhook retry is idempotent.
14. Failed refund remains visible and reconcilable.
15. Provider outage shows safe retry/pending behavior.
16. Sandbox credentials cannot be used by production profile.
17. Customer A cannot read Customer B's payment/order.
18. Support Agent cannot approve/execute refund without explicit permission.
19. Financial report totals match verified ledger and reconciliation fixtures.
20. UPI/card/netbanking methods are only shown when enabled for the approved provider/account.

## 13. Production payment gate

Live checkout is blocked until the owner has approved the payment provider and merchant account, the provider's live credentials are stored securely, the live webhook is registered and verified, the settlement account and supported methods are confirmed, tax/legal checkout terms are approved, and end-to-end live smoke tests have passed with a low-value controlled transaction and documented reconciliation/refund where feasible.

A successful sandbox test is required but is not proof that live payments are enabled.
