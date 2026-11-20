# Backend 07 — Payments, Refunds and Financial Reconciliation

## 1. Payment integration principles

- Select the payment gateway only after owner approval and current review of supported countries, merchant eligibility, currencies, payment methods, fees, refund behaviour, settlement reporting, webhook semantics, disputes and support.
- Use the gateway's hosted checkout or hosted payment fields where practical to reduce exposure to card data. Final PCI scope must be confirmed with the acquirer/provider and a qualified assessor.
- The application must not store CVV, full PAN or raw card authentication secrets.
- Keep provider-specific logic inside an adapter. Domain services use canonical payment contracts.
- Use separate test/sandbox and production credentials, accounts, webhook secrets and callback URLs.
- Provider credentials belong in a secret manager, not source control, database plaintext, browser code, logs or ordinary configuration screens.

## 2. Payment adapter contract

Define and test an interface equivalent to:

- `createPaymentAttempt(order, customerContext, returnUrls, idempotencyKey)`
- `getPaymentStatus(providerReference)`
- `verifyWebhook(rawBody, headers)`
- `normalizeWebhook(verifiedEvent)`
- `capture(providerReference, amount)` if capture is a separate provider operation
- `voidOrCancel(providerReference)`
- `createRefund(paymentReference, amount, reason, idempotencyKey)`
- `getRefundStatus(providerRefundReference)`
- `fetchSettlementReport(dateRange/cursor)`
- `reconcileTransaction(providerReference)`

The adapter must specify provider idempotency support and retry semantics. If a provider lacks a safe operation, implement reconciliation before retrying rather than blindly resubmitting.

## 3. Checkout and payment lifecycle

1. Backend revalidates every cart item, publication state, current price, coupon, customer eligibility, currency and tax rules.
2. Backend calculates totals using integer minor units and persists a checkout quote/snapshot.
3. Backend creates an order and payment attempt in a pending state under an idempotent command.
4. Backend requests a provider checkout session or payment action.
5. Customer completes payment with provider.
6. Provider sends a signed webhook.
7. Backend verifies signature over the correct raw body, provider event identity and replay controls.
8. Backend inserts the webhook event under a unique provider/event constraint.
9. Backend updates the payment attempt/transaction and writes an outbox event in a DB transaction.
10. API acknowledges only after verified event persistence. Event processing can be asynchronous after durable storage.
11. Worker applies the canonical transition idempotently; when payment is confirmed, it marks the order paid and grants the appropriate entitlement(s) in one DB transaction.
12. Worker sends receipt/access notification and updates projections.
13. Customer frontend fetches authoritative order/library state.

Never mark paid based only on a browser redirect, query parameter, client-side SDK callback or screenshot/receipt upload.

## 4. Webhook security and reliability

- Read and preserve raw request bytes before JSON transformation if the provider's signature method requires them.
- Verify signature using active/rotating secret, constant-time comparison where applicable, timestamp tolerance and provider-defined replay controls.
- Enforce request body size, content type and rate limits.
- Persist provider, event ID, signature verification timestamp, payload hash and normalized state. Retain raw payload only when necessary, encrypted/access restricted and for an approved period.
- Unique constraint on `(provider_key, provider_event_id)` prevents duplicate processing.
- Acknowledge verified duplicate events successfully without repeating effects.
- Invalid signatures are rejected and security logged without echoing secret details.
- Processing failure after persistence is retried through queue/worker; the provider should not be relied on as the only retry mechanism.
- Support manual reprocessing with authorization, reason and audit event.

## 5. Payment amount validation

Before settling an order:

- Provider transaction belongs to the expected merchant/account and expected order/payment attempt.
- Amount and currency match the outstanding order amount under the selected capture model.
- Provider status is a valid captured/paid state, not merely created or authorized unless approved policy states otherwise.
- Transaction has not already been applied.
- Order is in a state where late payment can be safely handled.
- Any mismatch is quarantined for review; do not grant entitlement automatically.

Never accept amount, currency, order ID or provider reference solely from browser-submitted values.

## 6. Idempotency

Use idempotency keys for checkout creation, payment attempt creation, provider refund creation and admin commands that can create duplicate financial/access outcomes. Persist key scope, principal, operation, request hash, processing state, canonical response reference and expiry/retention policy.

Rules:

- Same key + same request returns the original result or current operation status.
- Same key + different request returns a conflict.
- Concurrent requests with the same key serialize to one logical operation.
- Keys are not global across all users; scope by user/guest, route/operation and business object as appropriate.
- Provider idempotency keys should be stable across retry attempts for the same logical provider command.

## 7. Refund process

1. Customer/support/admin creates a refund request with order, amount or line allocation and reason.
2. Backend computes refundable balance from captured transactions minus successful prior refunds/chargebacks.
3. Policy engine validates refund eligibility, time window, item/format/access facts and current policy version.
4. Request enters review/approval state.
5. Authorized finance/owner approves or rejects with reason; use maker-checker where required.
6. Backend submits provider refund with stable idempotency key.
7. Provider result is recorded as pending/submitted, not prematurely succeeded.
8. Webhook or reconciliation confirms success/failure.
9. Financial ledger and order/refund state are updated transactionally.
10. Entitlement treatment follows explicit owner-approved refund/access policy. Do not automatically revoke access until that policy is accepted.
11. Customer receives status notification and invoice/credit-note handling is triggered where required.

Partial refunds must allocate amounts to order items and tax/discount components consistently. Do not refund more than the remaining refundable captured balance.

## 8. Financial ledger and reconciliation

Maintain an immutable internal financial event journal for captured payments, fees, refunds, disputes, chargebacks and settlement differences. A full accounting/general-ledger system is not assumed in initial scope; identify clearly that the internal ledger is an operational reconciliation record, not statutory books.

Reconciliation process:

- Import/query provider settlements and transaction reports.
- Match by provider transaction/reference, amount, currency and date.
- Identify unmatched app transactions, unmatched provider transactions, fee/settlement differences, duplicate captures, delayed events and refunds without success.
- Create reconciliation case/status records with owner and resolution notes.
- Do not auto-correct a financial fact without preserving the original and adjustment evidence.
- Run scheduled reconciliation even if webhooks appear healthy.

## 9. Refund and chargeback operations

Record disputes/chargebacks as distinct financial states/events. Preserve evidence and relevant order, entitlement and delivery records subject to privacy/retention policy. Support can investigate; only authorized finance/owner roles can submit financial responses or make final settlement decisions.

## 10. Taxes, invoices and jurisdiction

Tax rules, tax point, classification, invoice fields, numbering, credit notes, rounding and retention are legal/accounting decisions. Keep the calculation engine configurable and versioned. Persist the tax quote/rule version used at checkout. Have a qualified tax professional validate India and any other launch jurisdiction before live transactions. Do not hard-code GST rates or assume a tax treatment in code.

## 11. Failure matrix

| Failure | Required behaviour |
|---|---|
| Customer closes payment window | Keep pending until expiry/reconciliation; show retry/status check |
| Browser return says success but webhook absent | Query provider server-to-server; do not grant entitlement yet |
| Webhook signature invalid | Reject; no state change |
| Duplicate webhook | Acknowledge duplicate after verifying; no duplicate effect |
| DB unavailable after webhook received | Do not falsely acknowledge durable processing; provider retry or durable ingress queue required |
| Worker fails after event persisted | Retry from durable event/outbox |
| Payment captured but order cancelled | Quarantine/late-payment compensation workflow; no silent entitlement grant |
| Amount/currency mismatch | Hold for finance review; no entitlement |
| Refund provider timeout | Query/reconcile before retry; do not duplicate refund |
| Refund succeeded but app update failed | Reconcile provider event and converge state idempotently |
| Settlement mismatch | Open reconciliation case and alert finance |

## 12. Acceptance tests

- Valid, invalid, duplicate, delayed and out-of-order webhooks.
- Correct amount/currency and mismatch cases.
- Same idempotency key parallel requests.
- Provider timeout before/after side effect.
- Payment succeeds but DB/worker fails.
- Refund partial/full, duplicate command, delayed webhook and provider failure.
- Chargeback/dispute state.
- Reconciliation of transactions, fees, settlements and refunds.
- No card sensitive authentication data in logs, DB, analytics or support APIs.
