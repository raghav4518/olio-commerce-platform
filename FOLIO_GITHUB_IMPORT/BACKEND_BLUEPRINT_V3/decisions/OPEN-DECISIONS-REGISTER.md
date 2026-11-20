# Open Decisions and Approval Register

**As-of date:** 20 November 2026  
**Rule:** The coding agent must not silently turn a proposed default into an approved commercial, legal, provider or infrastructure decision. Update status, owner, date and impact when decisions are made.

| ID | Decision | Current status | Recommended direction (not approved) | Needed before |
|---|---|---|---|---|
| D-001 | Initial business model | Owner confirmation required | Single-vendor owner-operated store; no seller portal | Product freeze |
| D-002 | Launch countries and currency | Owner confirmation required | India-first only if confirmed; currency configured by market | Price/payment/tax implementation |
| D-003 | Payment gateway | Not selected | Compare current merchant eligibility, UPI/cards/net banking, webhooks, refunds, settlement exports, fees and support | Live checkout |
| D-004 | Tax and invoice rules | Professional review required | Versioned configurable tax engine; no hard-coded rates | Live transactions |
| D-005 | Guest checkout/account claim | Not selected | Evaluate customer account checkout vs guest order with verified claim flow | Checkout/auth |
| D-006 | Email verification policy | Not selected | Verify email before account recovery and library claim; decide whether required before payment | Auth/library |
| D-007 | Supported ebook formats | Not selected | PDF and EPUB are candidates; confirm upload, sale, sample and reader support per format | Asset/reader |
| D-008 | Reader vs download access | Not selected | Both may be offered if formats, rights, support and privacy are ready | Entitlement |
| D-009 | Watermarking | Not selected | Optional purchaser watermark as deterrent, not DRM guarantee | Delivery |
| D-010 | Download limit/expiry | Not selected | Avoid arbitrary limits; define reasonable customer recovery and anti-abuse policy | Delivery |
| D-011 | Refund eligibility and time window | Not selected; legal/business review | Publish explicit policy by jurisdiction and content-access context | Refunds |
| D-012 | Entitlement after refund | Not selected | Policy-driven; do not revoke until approved terms and process are set | Refund integration |
| D-013 | Unpublishing effect on previous buyers | Policy required | Preserve paid entitlements unless an approved legal/security exception applies | Catalog |
| D-014 | File version replacement policy | Not selected | Preserve immutable old version until buyer migration policy is approved | Asset versions |
| D-015 | Reader engine | Not selected | Evaluate maintained EPUB/PDF engines for accessibility, security, license and performance | Reader |
| D-016 | Frontend framework | Repository audit pending | Retain suitable existing stack; no premature rewrite | Frontend planning |
| D-017 | Backend language/framework | Repository audit pending | Retain suitable existing stack if maintainable and secure | Backend implementation |
| D-018 | Database/provider/region | Not selected | Relational DB; PostgreSQL candidate; decide residency/backup features | Schema/deploy |
| D-019 | Object storage/CDN provider | Not selected | Private managed object storage and selective CDN | Asset pipeline |
| D-020 | Email/SMS/WhatsApp providers | Not selected | Transactional email first; add channels with consent/cost review | Notifications |
| D-021 | Search provider | Not selected | Start with DB search if adequate; add dedicated search from measured need | Search scale |
| D-022 | Analytics provider | Not selected | Server-side financial metrics; privacy-conscious product analytics | Analytics |
| D-023 | Hosting/CI/CD/observability provider | Repository audit pending | Managed platform aligned to team and budget | Deployment |
| D-024 | Staff MFA and session policy | Owner approval required | Require MFA for all owner/staff; define idle/absolute session expiry | Admin/support launch |
| D-025 | Refund approval authority and thresholds | Owner approval required | Support requests/investigates; finance/owner approves/executes | Refund launch |
| D-026 | Data retention/deletion periods | Legal review required | Purpose-based retention schedule and legal hold process | Production launch |
| D-027 | Final brand palette/logo/fonts | Not selected | Select after choosing specific reference shots | Frontend design |
| D-028 | Exact Dribbble reference shots | Not selected | Choose 2–4 individual references from https://dribbble.com/tags/book-landing-page | Frontend design |
| D-029 | Admin/support dark mode | Not selected | Light-first; optional dark mode after QA scope approval | Frontend design |
| D-030 | Subscriptions/gifting/bundles | Out of release-one scope | Defer; each changes order, entitlement and refund rules | Scope change |
| D-031 | Order quantity/duplicate purchase policy | Not selected | Default digital personal-use item quantity 1; confirm duplicate purchase behaviour | Cart/checkout |
| D-032 | Refund maker-checker policy | Not selected | Separate requester and approver for defined high-risk thresholds | Finance controls |
| D-033 | Reader analytics consent | Not selected | Separate functional progress sync from optional behavioural analytics | Reader/privacy |
| D-034 | RPO/RTO and availability objectives | Not selected | Define with business owner from outage impact and budget | Production infra |
| D-035 | Load targets and expected traffic | Not selected | Define launch workload and growth assumptions; test before scale claims | Capacity planning |
| D-036 | Support SLA and business hours | Not selected | Define first-response/resolution targets and escalation policy | Support operations |
| D-037 | Account identity and login methods | Not selected | Decide email/password, passwordless/passkeys, social login, phone OTP scope | Identity design |
| D-038 | Guest purchase data retention | Not selected | Align with account claim, refund, tax and privacy law | Checkout/privacy |
| D-039 | Digital rights and takedown process | Not selected | Require ownership/licensing attestation and documented complaint/takedown flow | Catalog/legal |
| D-040 | Privacy/cookie analytics policy | Legal review required | Choose minimal analytics/consent approach for launch markets | Production |
| D-041 | OpenAPI/tooling version | Repository audit pending | OAS 3.2.1 if full toolchain supports it; otherwise documented compatible 3.1.x | API contract CI |
| D-042 | Data migration and deletion of prior test data | Repository audit pending | Audit existing records; never run destructive cleanup without backup/approval | Repository migration |

## Decision template

```markdown
## ADR-D-XXX — <Decision>
- Status: Proposed | Accepted | Rejected | Superseded
- Date:
- Owner/approver:
- Context:
- Options considered:
- Decision:
- Consequences:
- Security/privacy/legal impact:
- Data migration impact:
- Cost/operations impact:
- Requirements affected:
- Approval evidence:
```

## Decision handling rules

1. Abstractions may be built to isolate provider choice, but production behaviour must not be fabricated.
2. Tax, refund, retention, legal, consent, entitlement and account policies require explicit approval/review.
3. If an unresolved decision blocks a phase, stop that dependent work and provide options, trade-offs and a recommendation.
4. Sandbox adapters must be unmistakable and unavailable in production configuration.
5. Record each accepted decision in an ADR and update affected API, schema, tests and product documents.


## Frontend design evidence recorded in V3 (not owner approvals)

The supplied Stitch export shows guest-or-sign-in checkout, an email used to associate digital editions/invoices, UPI (GPay/PhonePe/Paytm/BHIM), credit/debit cards (RuPay/Visa/Mastercard), Net Banking, rupee-denominated sample values and configurable GST display. It includes separate success/pending/failed payment screens and reader/support examples mentioning EPUB.

These are requirements signals to implement or validate. They do **not** resolve D-002 through D-015, D-037, D-038 or the legal/provider decisions. In particular:
- Do not assume the displayed sample GST amount/rate is legally correct.
- Do not select a payment gateway from a visual label.
- Do not assume a rupee display alone establishes legal launch jurisdiction or merchant entity.
- Do not silently activate guest checkout until account claim, email verification, guest-record retention and support/refund behavior are approved.
- Do not implement PDF/EPUB delivery as production-supported until format scope, reader engine, licenses and file pipeline are verified.
- Do not revoke entitlement at refund request time; use the accepted refund/entitlement policy.
