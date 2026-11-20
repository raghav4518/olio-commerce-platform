# Ebook Commerce Platform — Customer & Support/Operations Panel Specification

**Document type:** Product Requirements Document (PRD) + UX/Sitemap Specification + Workflow Specification + Engineering Acceptance Criteria  
**Version:** 1.0  
**Status:** Draft for product-owner review and implementation planning  
**Scope:** Customer-facing storefront/account application and the internal Support/Operations application  
**Companion document:** `ebook-platform-admin-panel-specification.md`  
**Primary audience:** Product owner, UX/UI designer, frontend agent, backend agent, QA agent, security reviewer  
**Implementation rule:** This document defines the intended product and its required behaviour. It does not authorize an agent to invent unresolved business rules, omit requirements silently, or implement unrelated panels.

---

## 0. Instructions to the implementation agent

Read this document and the companion Admin Panel specification in full before changing code. Treat them as coordinated specifications for one product. The Admin Panel specification remains authoritative for owner-level catalogue, finance, content, configuration, and administrative controls. This document is authoritative for the Customer Panel and Support/Operations Panel.

### 0.1 Required working method

1. **Inspect before implementing.** Inspect the existing repository, framework, route structure, design system, authentication, database schema, API conventions, tests, deployment configuration, and the Admin Panel implementation. Do not assume the repository is greenfield.
2. **Create an architecture inventory.** Identify what already exists, what can be reused, what is incomplete, and what conflicts with these specifications. Include route inventory, domain models, auth boundaries, existing UI components, and integrations.
3. **Produce an implementation plan before coding.** Break work into milestones and small deliverable tickets. Identify dependencies, migrations, external services, test coverage, data migrations, and rollback risks.
4. **Respect the three-panel product boundary.** The product has exactly three panels: (1) Customer Panel, (2) Owner/Admin Panel, and (3) Support/Operations Panel. This document covers panels 1 and 3. Do not create Author, Publisher, Affiliate, Vendor, Seller, or other additional portals unless the owner changes the product scope in writing.
5. **Treat the Customer Panel as two experiences inside one panel.** The public bookstore and the signed-in customer account/library experience are two states/areas of the Customer Panel, not separate panels or separate products.
6. **Do not confuse Support with Admin.** Support staff must use a separate role-limited workspace. They must not receive owner permissions merely because they can inspect a customer or order.
7. **Do not guess unresolved commercial policies.** Use the decision register in Section 23. Record assumptions and seek owner approval at the appropriate product-design checkpoint. Do not silently decide refund windows, guest checkout policy, payment gateway, tax rules, or account deletion policy.
8. **Build real interactions, not decorative UI.** Every visible control must do something meaningful and permission-checked, or be omitted/clearly marked unavailable until implemented. No dead buttons, fake success messages, mock order states, fake analytics, or hardcoded production data.
9. **Enforce all security on the server.** Hiding navigation, disabling a button, or checking a role in frontend code is not authorization. Validate authentication, role, permission, object ownership, and current state on every protected backend operation.
10. **Use shared domain logic.** The customer, support, and admin applications must not implement conflicting copies of order, payment, refund, entitlement, or account state machines. Shared backend services/domain modules are the source of truth.
11. **Use real data definitions.** Every total, status, count, report, and timeline must come from persisted records and documented definitions. Clearly distinguish live, pending, delayed, and unavailable data.
12. **Protect existing purchases.** Do not hard-delete paid orders, payment attempts, refunds, invoices, audit records, or valid customer entitlements. Use approved retention, anonymization, archival, or legal deletion workflows.
13. **Test negative paths.** Test unauthorized access, cross-customer record access, expired links, duplicate webhooks, abandoned payment attempts, partial refunds, unavailable files, invalid EPUB/PDF content, and support actions outside permissions.
14. **Document deviations.** Every deviation must include requirement ID, reason, impact, proposed alternative, data/security implications, testing implications, and owner approval status.
15. **Do not expose secrets or personal data.** Never commit credentials, API keys, gateway secrets, private storage URLs, signing keys, authentication tokens, or production customer data. Mask sensitive values in logs and support tooling.

### 0.2 Definition of done

A page or workflow is complete only when:

- Its responsive layout, navigation, loading, empty, error, success, and permission-denied states are implemented.
- Every action calls a real backend operation and reflects the persisted result.
- Client-side and server-side validation exist where relevant.
- Authentication, authorization, and ownership checks are enforced server-side.
- Data access is scoped to the authenticated customer or authorized staff role.
- Relevant activity is audited, especially financial, entitlement, account-security, and support actions.
- Pagination, sorting, and filtering are server-side for potentially large result sets.
- Accessibility, keyboard behaviour, and screen-reader semantics are verified.
- Unit, integration, permission, workflow, regression, and end-to-end tests pass.
- No critical console errors, broken links, placeholder data, dead controls, or unresolved critical security findings remain.

### 0.3 Requirement priority labels

- **P0 — Must:** Required for the initial production launch.
- **P1 — Should:** Important for a polished initial product; may follow P0 if explicitly planned.
- **P2 — Later:** Useful expansion, but must not delay the initial commercial release unless the owner promotes it.
- **Decision required:** Must not be implemented as a fixed business rule until the owner approves the decision.

---

# 1. Product definition and panel boundaries

## 1.1 Product vision

Build a premium, easy-to-use digital bookstore where customers can discover ebooks, understand exactly what they are buying, complete a low-friction payment, and immediately access the ebooks they purchased. Customers should be able to manage their own library, reading progress, downloads, orders, preferences, and support requests without contacting the business for routine tasks.

The platform is initially designed as an owner-operated ebook store. The owner manages the catalogue and commerce through the Admin Panel. Customers use the Customer Panel. Employees who resolve customer-facing operational issues use the Support/Operations Panel.

## 1.2 The three-panel architecture

| Panel | Primary users | Main responsibility | Included in this document? |
|---|---|---|---|
| Customer Panel | Visitors and registered customers | Discover, purchase, read, download, and manage purchased ebooks | Yes |
| Owner/Admin Panel | Owner and authorized administrators | Operate the business, catalogue, finance, marketing, content, settings, and governance | No — companion specification |
| Support/Operations Panel | Support agents, senior agents, operations staff | Resolve customer, order, payment, access, and support issues within limited permissions | Yes |

The platform must not create additional author, publisher, affiliate, vendor, seller, or partner portals in the current scope.

## 1.3 Customer Panel includes two experiences

### Public bookstore

Accessible without signing in. Includes the home page, product discovery, catalogue, book details, author/collection pages, policies, cart, and allowed checkout steps.

### Signed-in customer workspace

Accessible after authentication. Includes the customer dashboard, personal ebook library, reader, reading progress, downloads, orders, invoices, wishlist, reviews, notifications, account settings, privacy controls, and support history.

Both experiences must share the same visual identity, navigation conventions, design tokens, and customer account identity.

## 1.4 Support/Operations Panel boundary

Support/Operations is an internal staff workspace, not a second customer-facing account area and not a reduced copy of the Admin Panel. It provides the minimum information and actions required to resolve customer issues.

Support staff may be able to inspect customer/order/payment/entitlement status and initiate approved workflows. They must not have unrestricted control over catalogue publication, prices, gateway secrets, tax configuration, global settings, staff permissions, or unrestricted financial operations.

## 1.5 Initial business-model planning assumption

The owner has not yet formally confirmed single-vendor versus marketplace operation. The safest implementation baseline is **single-vendor: the platform owner is the seller and controls the ebook catalogue**. This is a planning assumption, not a final commercial decision.

Do not build third-party seller onboarding, vendor dashboards, royalty wallets, revenue splits, seller settlements, author submissions, or marketplace order splitting. Keep domain boundaries clean enough that future marketplace expansion is not needlessly blocked, but do not build marketplace complexity now.

## 1.6 Product non-goals for the current scope

- Physical goods, shipping, delivery addresses, warehouse stock, or courier tracking.
- Third-party seller or publisher self-service.
- Affiliate/partner portals.
- Native iOS/Android applications.
- A full accounting/ERP replacement.
- Guaranteed prevention of screenshots, screen recording, copying, or redistribution.
- Arbitrary executable code inside customer-submitted content or CMS content.
- Automatic refunds or irreversible account actions without approved rules and safeguards.

---

# 2. Product principles and modern customer expectations

These principles guide the implementation. They are not permission to add fashionable features that create unnecessary complexity.

1. **Mobile-first, not mobile-only.** The storefront, checkout, library, reader, and support request flows must work comfortably on narrow screens and also scale to desktop/tablet.
2. **Make the next action obvious.** Visitors should quickly understand how to find a book, inspect it, buy it, and access it. Signed-in customers should quickly find their library and continue reading.
3. **Show the real product before purchase.** Provide meaningful descriptions, format information, sample previews where supplied, clear pricing, and access/download conditions.
4. **Do not force account creation too early.** A guest checkout option is recommended to reduce friction, but the exact purchase-identity and account-claim rules require owner approval. If guest checkout is enabled, design a secure way to attach the confirmed purchase to a customer account.
5. **No shipping fiction.** These are digital products. Do not ask for shipping address, shipping method, delivery date, or courier details. Collect billing/tax details only where required by approved payment and tax policy.
6. **Immediate, trustworthy fulfilment.** After payment is verified, create the customer entitlement and show where to access the ebook. Do not rely on the browser returning from the payment gateway as proof of payment.
7. **Persist the customer's place.** Save reader position and preferences so the customer can resume later and, where supported, across devices.
8. **Accessible by design.** Keyboard users, screen-reader users, customers with low vision, and customers using zoom or reduced-motion settings must be able to use core flows.
9. **Performance is part of UX.** The homepage, catalogue, product pages, and account/library should be fast, stable, and responsive on ordinary mobile connections.
10. **Respect customer privacy.** Collect only data needed to provide the service, explain data use, give appropriate account/data controls, and avoid invasive tracking.
11. **Never pretend digital protection is absolute.** Time-limited links and personalized watermarking can deter casual sharing, but cannot guarantee prevention of screenshots or copying after a file is downloaded.
12. **Do not overwhelm.** Start with a clear catalogue and simple customer dashboard. Advanced preferences and optional features should not obstruct the primary purchase/read tasks.

## 2.1 Research and standards used

The requirements below are informed by the current references in Section 24. In particular:

- Baymard's checkout research highlights persistent usability issues in ecommerce checkout and the importance of clear account selection, forms, payment methods, error handling, and order confirmation.
- WCAG 2.2 is the accessibility baseline. Target WCAG 2.2 AA; use generous touch targets (44 CSS pixels as a design target where practical), clear focus indicators, proper labels, keyboard access, and contrast.
- Google Core Web Vitals guidance gives useful field-performance targets: LCP within 2.5 seconds, INP under 200 milliseconds, and CLS below 0.1 at the 75th percentile.
- W3C EPUB 3.3 is the current EPUB publication specification, and EPUB Reading Systems 3.3 defines reading-system conformance requirements. EPUB Accessibility 1.1 defines accessibility conformance and discoverability metadata.
- OWASP ASVS 5.0.0 and the OWASP Cheat Sheet Series provide a security verification baseline for authentication, authorization, file uploads, and sensitive account flows.

Do not claim that compliance has been achieved solely because a library or framework is used. Validate the actual implementation.

---

# 3. Open product decisions and safe defaults

The following items have not been fully confirmed. The implementation agent must preserve flexibility and must not invent a permanent rule.

| Decision ID | Decision | Recommended baseline for design | Approval needed before production? |
|---|---|---|---|
| DEC-01 | Single-vendor or marketplace | Plan for single-vendor owner-operated store | Yes — confirm commercial model |
| DEC-02 | Primary payment gateway | Provider-neutral payment adapter; evaluate an India-capable gateway such as Razorpay only after approval | Yes |
| DEC-03 | Currency | INR as the initial display/settlement candidate, with currency stored explicitly per order | Yes |
| DEC-04 | Tax/GST | Configurable tax engine; do not hardcode rates or tax treatment | Yes, with qualified tax review |
| DEC-05 | Guest checkout | Offer guest checkout if the owner approves; verify purchase email and provide secure account-claim flow | Yes |
| DEC-06 | Customer registration | Self-registration is the natural customer experience; admin-created accounts may be supported by the Admin Panel | Confirm account policy |
| DEC-07 | Reading and download | Design for online reading and controlled downloads, configurable per ebook | Yes — decide enabled modes and file formats |
| DEC-08 | Ebook formats | Plan for PDF and EPUB; validate actual format support and accessibility metadata | Yes — approve supported formats for launch |
| DEC-09 | Refund rules | Owner-configurable policy with manual review; no automatic universal promise | Yes — define eligibility, time window, and access revocation |
| DEC-10 | Customer reviews | Verified-purchase reviews are recommended; define moderation and eligibility | Yes |
| DEC-11 | Offline reading | Do not cache full purchased files offline by default; evaluate later with explicit security/licensing rules | Yes |
| DEC-12 | Languages | Design localization-ready; launch language(s) to be confirmed | Yes |
| DEC-13 | Customer support channels | In-app ticket/email baseline; live chat/phone only if selected and staffed | Yes |
| DEC-14 | Account deletion/retention | Provide request workflow; apply approved legal/financial record retention and anonymization rules | Yes, with legal review |
| DEC-15 | Gift purchases / multiple licences | Excluded from initial scope unless approved | Yes |
| DEC-16 | Subscription access | Excluded from initial scope unless separately approved | Yes |

### 3.1 Rules for handling unresolved decisions

- Use configuration/feature flags only when the setting has a meaningful owner-facing control and defined default. Do not create a large settings surface for hypothetical features.
- If an unresolved decision blocks a workflow, implement a safe inactive state and document the dependency. Do not quietly assume the decision.
- Place all product-owner approvals in an ADR/decision log with date, decision maker, decision, rationale, and affected requirements.
- Keep the user interface honest. Do not show a payment method, refund promise, book format, or offline feature that is not actually enabled.

---

# 4. Users, roles, and access boundaries

## 4.1 Customer roles

| Role/state | Description | Access |
|---|---|---|
| Visitor | Unauthenticated person browsing the storefront | Public catalogue, public pages, sample previews, cart, and approved guest checkout |
| Registered Customer | Authenticated customer account | Own profile, orders, entitlements, library, downloads, reading progress, wishlist, reviews, notifications, and support tickets |
| Restricted Customer | Customer account under a documented restriction | Only permitted account functions; restriction reason and support route shown where appropriate |

A customer must never access another customer's records by changing an identifier in a URL, request body, query string, or API call.

## 4.2 Support/Operations roles

| Role | Intended capability | Must not be allowed by default |
|---|---|---|
| Support Agent | View assigned/allowed tickets, customer-safe account context, order status, entitlement status, and respond to customers | Change prices, publish ebooks, edit gateway secrets, approve own restricted refunds, manage admins, or alter tax settings |
| Senior Support Agent | All agent actions plus reassignment, escalation, approved exception handling, and review of lower-level work | Global owner settings, staff permission administration, unrestricted refunds |
| Operations Manager | Operational queues, assignment, SLA monitoring, order/access investigations, and approved operational actions | Gateway secrets, owner-only settings, unrestricted financial reporting, or access beyond assigned business need |
| Support Administrator (optional) | Manage support categories, macros, queues, and support team configuration | Owner-level product/finance/security control |

The owner may choose to grant one person multiple roles. The application must still enforce each active role and permission explicitly.

## 4.3 Permission model

Use explicit permission identifiers, for example:

- `support.ticket.view`
- `support.ticket.reply`
- `support.ticket.assign`
- `support.ticket.close`
- `support.customer.view_limited`
- `support.order.view`
- `support.payment.view_limited`
- `support.entitlement.view`
- `support.entitlement.revoke_request`
- `support.refund.create_request`
- `support.refund.approve`
- `support.account.security_action`
- `support.export.allowed_data`
- `support.settings.manage`

The backend must apply deny-by-default authorization and validate permission on every protected request. The permission to view a record does not automatically grant permission to modify it.

## 4.4 Support access should be purpose-limited

- Support users should see only the personal information required to resolve the ticket.
- Mask full payment instrument details, authentication secrets, reset tokens, and gateway credentials.
- Do not expose raw ebook storage paths or unrestricted original files to support agents.
- Any privileged action must record the staff actor, reason, timestamp, affected record, previous state, and resulting state.
- Use step-up authentication for sensitive operations such as issuing refunds, changing customer email, or revoking access, where policy requires it.

---

# 5. Customer Panel information architecture and sitemap

## 5.1 Public bookstore navigation

Recommended desktop primary navigation:

- Logo / Home
- Explore Books
- Categories (menu)
- Collections / Bestsellers
- Offers (only when active)
- Search
- Wishlist (if signed in; otherwise sign-in prompt)
- Account / Sign In
- Cart

Do not place every category or CMS page in the primary navigation. Use a concise menu and a footer for secondary/legal links.

## 5.2 Customer account navigation

Recommended account navigation:

- Overview
- My Library
- Continue Reading
- My Orders
- Wishlist
- Downloads
- Reviews
- Notifications
- Help & Support
- Profile & Preferences
- Security & Privacy

On mobile, use a clear account menu or bottom navigation for the most frequent actions. Do not force a desktop sidebar into a narrow viewport.

## 5.3 Complete public sitemap

| Page ID | Page | Priority | Authentication |
|---|---|---:|---|
| CP-PUB-001 | Homepage | P0 | Public |
| CP-PUB-002 | Explore / All Ebooks | P0 | Public |
| CP-PUB-003 | Category Listing | P0 | Public |
| CP-PUB-004 | Search Results | P0 | Public |
| CP-PUB-005 | Ebook Detail | P0 | Public |
| CP-PUB-006 | Author Profile | P1 | Public |
| CP-PUB-007 | Collection / Bestseller / New Release | P1 | Public |
| CP-PUB-008 | Offers / Promotions | P1 | Public; only active offers shown |
| CP-PUB-009 | Cart | P0 | Public/session |
| CP-PUB-010 | Checkout | P0 | Guest or signed-in subject to decision |
| CP-PUB-011 | Payment Result / Order Confirmation | P0 | Order-bound secure access |
| CP-PUB-012 | Login | P0 | Public |
| CP-PUB-013 | Registration | P0 | Public |
| CP-PUB-014 | Email Verification | P0 | Token/session-bound |
| CP-PUB-015 | Forgot / Reset Password | P0 | Public with secure token |
| CP-PUB-016 | About | P1 | Public |
| CP-PUB-017 | Contact | P1 | Public |
| CP-PUB-018 | FAQ / Help Centre | P1 | Public |
| CP-PUB-019 | Terms and Conditions | P0 | Public |
| CP-PUB-020 | Privacy Policy | P0 | Public |
| CP-PUB-021 | Refund Policy | P0 | Public |
| CP-PUB-022 | Cookie / Tracking Preferences (where applicable) | P1 | Public |
| CP-PUB-023 | 404 / Not Found | P0 | Public |
| CP-PUB-024 | 500 / Temporary Error | P0 | Public |

## 5.4 Complete signed-in customer sitemap

| Page ID | Page | Priority | Authentication |
|---|---|---:|---|
| CP-CUS-001 | Customer Overview | P0 | Required |
| CP-CUS-002 | My Library | P0 | Required |
| CP-CUS-003 | Library Book Details / Access | P0 | Required; entitlement check |
| CP-CUS-004 | Online Reader | P0 if reader is enabled | Required; entitlement check |
| CP-CUS-005 | Reading Progress / Continue Reading | P0 | Required |
| CP-CUS-006 | My Downloads | P0 if downloads enabled | Required; entitlement check |
| CP-CUS-007 | My Orders | P0 | Required |
| CP-CUS-008 | Order Details | P0 | Required; own order only |
| CP-CUS-009 | Invoice / Receipt View | P0 where supported | Required; own order only |
| CP-CUS-010 | Wishlist | P1 | Required |
| CP-CUS-011 | My Reviews | P1 | Required |
| CP-CUS-012 | Notifications | P1 | Required |
| CP-CUS-013 | Support Centre / My Tickets | P0 | Required |
| CP-CUS-014 | Create Support Request | P0 | Required |
| CP-CUS-015 | Ticket Conversation | P0 | Required; own ticket only |
| CP-CUS-016 | Profile | P0 | Required |
| CP-CUS-017 | Password / Sign-in Methods | P0 | Required |
| CP-CUS-018 | Active Sessions | P1 | Required |
| CP-CUS-019 | Reading Preferences | P1 | Required |
| CP-CUS-020 | Communication Preferences | P1 | Required |
| CP-CUS-021 | Privacy / Data Requests | P1 | Required |
| CP-CUS-022 | Account Deactivation / Deletion Request | P1 | Required; policy-controlled |

---

# 6. Global Customer Panel layout and design system

## 6.1 Visual direction

Use a premium editorial bookstore feel rather than a generic product marketplace template. Book covers, typography, readable descriptions, calm whitespace, and a clear hierarchy should dominate. The design must remain accessible and must not sacrifice readability for decoration.

Recommended qualities:

- Strong, consistent typography hierarchy.
- Spacious book cards with legible titles and author names.
- Consistent book-cover ratio and safe image cropping.
- Clear prices and purchase buttons.
- Neutral backgrounds with one restrained brand accent.
- Consistent buttons, forms, dialogs, toasts, badges, cards, tabs, menus, and empty states.
- Optional light/dark reader themes; do not require the entire storefront to be dark.
- Motion used sparingly and disabled/reduced when the user prefers reduced motion.

Do not copy a competitor's copyrighted branding or exact layout. Use familiar interaction patterns but create a distinct brand identity.

## 6.2 Global desktop header

The public header should contain:

1. Brand logo linking to home.
2. Primary navigation.
3. Search trigger or integrated search field.
4. Wishlist entry where supported.
5. Cart icon with item count.
6. Account/sign-in entry.

Header behaviour:

- Sticky header is optional; if used, it must not obscure focused controls or content.
- Search should be easy to find.
- Cart item count must reflect actual session/server cart state.
- The account control must clearly distinguish sign-in from account navigation.
- Menus must support pointer, keyboard, touch, Escape, and focus return.

## 6.3 Mobile navigation

- Logo and compact actions at the top.
- Search easily discoverable.
- Menu with clear labels, not icon-only controls without accessible names.
- Cart and account access remain easy to reach.
- Avoid horizontal scrolling for main page content.
- Keep sticky bottom purchase controls on product detail pages only where they do not cover content or accessibility focus.

## 6.4 Standard page pattern

Most catalogue and account pages should use a consistent structure:

1. Breadcrumb where helpful.
2. Page heading and a concise explanation.
3. Main action(s), if relevant.
4. Search/filter/sort controls, if relevant.
5. Main content (cards, list, table, reader, or form).
6. Pagination or load-more control for large collections.
7. Clear loading, empty, error, and success states.

## 6.5 Global interaction rules

- Buttons must have clear action labels (for example, “Add to cart”, “Continue to payment”, “Open book”).
- Do not use vague labels such as “Submit” where a meaningful action name is possible.
- Disable a button only when the reason is clear and communicated.
- Prevent duplicate checkout submission with server idempotency and client feedback.
- Preserve form values after recoverable validation errors.
- Confirm destructive actions and explain their consequence.
- Use inline validation near the relevant field and a summary for long forms.
- Toasts are for brief feedback; important order/payment information must remain visible on the page.
- Provide retry paths for recoverable network and file-loading errors.

---

# 7. Detailed public storefront page specifications

## 7.1 CP-PUB-001 — Homepage (P0)

**Goal:** Help a new visitor understand what the store sells and find a relevant ebook quickly.

### Page structure

1. Header with navigation, search, account, and cart.
2. Hero section with a clear value proposition and one primary CTA (for example, “Explore ebooks”).
3. Featured categories, limited to a manageable number.
4. Bestselling ebooks (only when supported by real sales data).
5. New releases (only when real publication dates exist).
6. Curated collections / editor picks (owner-managed in Admin Panel).
7. Active promotion or offer section (only if an eligible campaign exists).
8. Short trust/value section explaining secure access, supported formats, and customer help without making unsupported guarantees.
9. Newsletter signup only if email marketing is configured and consent is captured appropriately.
10. Footer with legal, help, contact, social, and brand links.

### Behaviour

- Homepage sections should be managed through the Admin Panel's section-based homepage builder.
- Hide sections with no active content; do not show empty placeholder carousels.
- Book cards link to canonical ebook detail routes.
- Featured/bestseller logic must be explicit. Do not label a book “Bestseller” from arbitrary or fabricated values.
- The hero CTA must navigate to the relevant catalogue/collection.
- Preserve public SEO metadata and structured data where applicable.

### Acceptance criteria

- A first-time visitor can reach Explore and a book detail page without registering.
- All displayed book prices and statuses match current published catalogue data.
- No unpublished, archived, or restricted ebook is publicly discoverable unless specifically configured as preview-only.
- Mobile layout has no clipped controls or horizontal overflow.

## 7.2 CP-PUB-002 — Explore / All Ebooks (P0)

**Goal:** Let visitors browse the complete published catalogue.

### Page structure

- Heading and catalogue description.
- Result count based on the active filters.
- Search within catalogue.
- Filters: category, author, format, language, price range, and availability/offer only if those attributes are actually stored and meaningful.
- Sort: relevance (when search query exists), newest, title A–Z, price low-high, price high-low, popularity/bestseller only with defined data.
- Responsive ebook grid/list.
- Pagination or accessible load-more interaction.

### Ebook card fields

- Cover image with meaningful alt text or decorative treatment when the title is already adjacent.
- Title.
- Author.
- Price and sale price if active.
- Format badges only for actually available formats.
- Rating only when real, moderated review data exists and minimum display rules are met.
- Optional discount label only when computed from valid current pricing.
- CTA: open details; quick-add only if the product's purchase rules allow it and the UI explains the action.

### Filter behaviour

- Filters must combine predictably.
- Active filters appear as removable chips or an equivalent visible summary.
- Provide “Clear all”.
- Query/filter state should be represented in the URL where useful so results can be shared/bookmarked.
- Changing filters must not reset the user to an unrelated route.
- Empty results must explain that no books match and offer filter reset or a broader search.

## 7.3 CP-PUB-003 — Category Listing (P0)

Show category title, description, optional banner, child categories, and relevant ebooks.

Requirements:

- Support nested category navigation where the taxonomy contains children.
- Breadcrumbs must show the hierarchy accurately.
- Do not create category pages with thin/empty content for SEO.
- Preserve canonical URLs and avoid duplicate-index pages caused by arbitrary filter combinations.
- Display only currently published/available ebooks.

## 7.4 CP-PUB-004 — Search Results (P0)

**Search should be forgiving and useful.**

Search across approved public fields:

- Ebook title and subtitle.
- Author name.
- Category and tags.
- Description/keywords where indexing quality supports it.

Requirements:

- Debounce live suggestions; allow explicit Enter/search action.
- Search suggestions must be keyboard accessible.
- Show a clear result count and query.
- Highlight query matches only where safe and useful; never inject raw HTML from the query.
- Support typo tolerance/fuzzy matching only if the search service can do so reliably.
- Search should have a helpful zero-result state with clear spelling/filter suggestions.
- Do not expose draft, private, archived, or restricted products through search endpoints.
- Track search terms in privacy-conscious analytics; avoid storing unnecessary personal data.

## 7.5 CP-PUB-005 — Ebook Detail (P0)

**Goal:** Give a visitor enough reliable information to decide whether to purchase.

### Recommended layout

Left/primary visual area:
- Ebook cover.
- Additional preview images/pages when supplied.
- “Preview sample” action when a sample file exists.

Right/primary purchase area:
- Title and subtitle.
- Author link.
- Rating/review count only if real and approved.
- Price, sale price, and discount presentation.
- Format(s) supplied and purchasable.
- Access modes available for this product (online reader, download, or both).
- Main CTA: Add to Cart / Buy Now.
- Wishlist control.
- Clear disclosure that the product is digital and no physical item is shipped.

Below the purchase area:
- Full description.
- What the reader will learn/receive, if supplied by owner.
- Table of contents or sample preview, if supplied.
- Format and compatibility details.
- Language, page count, publication date, ISBN, and file-size information only when known and accurate.
- Author information.
- Reviews and rating distribution when enabled.
- Related ebooks / curated collection.
- Refund/access policy link.

### Purchase rules

- If unpublished, unavailable, or disabled, do not show a working purchase CTA.
- If an ebook is already owned by the signed-in customer, replace purchase CTA with “Open in My Library” unless repeat purchase/gifting has been approved.
- Do not reveal ownership status to another user.
- If a product has different access modes, explain them before purchase.
- Sale prices must be computed server-side from active pricing rules and must not be trusted from the browser.
- The page must show currency and any applicable tax disclosure according to the approved tax/pricing policy.

### Preview rules

- Only use a preview asset explicitly approved for public access.
- Do not expose the original full ebook file through a guessed preview URL.
- Preview pages/sections must be bounded and served through a public preview service or a separate approved asset.
- Sample preview must work on mobile and keyboard/screen reader where supported.

## 7.6 CP-PUB-006 — Author Profile (P1)

Although the product is not a marketplace, owner-managed author profiles are allowed as catalogue content.

Show:
- Author name and image (if available).
- Biography.
- Published ebooks by that author.
- Optional external links selected by the owner.

Do not expose private author contact information or create an author login portal.

## 7.7 CP-PUB-007 — Collections / Bestsellers / New Releases (P1)

A collection is a curated storefront grouping, not a separate product or seller.

Show:
- Collection title, description, banner, and products.
- Owner-defined order or rule-driven inclusion.
- Pagination for large collections.
- Empty/expired state that does not expose an inactive campaign as live.

## 7.8 CP-PUB-008 — Offers / Promotions (P1)

- List only active, customer-eligible offers.
- Explain valid dates, eligible products/categories, minimum order, maximum discount, and exclusions when relevant.
- Do not promise an offer that the checkout will reject.
- Coupon codes should be clearly distinguished from automatically applied discounts.
- If there are no active promotions, hide the navigation link or show a deliberate no-offers state rather than fake promotions.

## 7.9 CP-PUB-016 to CP-PUB-022 — Informational and policy pages (P0/P1)

About, Contact, FAQ, Terms, Privacy, Refund Policy, and Cookie/Tracking Preferences must be editable through the Admin Panel's content management system.

Rules:

- Legal/policy text must be owner-approved; the coding agent must not invent legal commitments.
- Display current effective date and version where appropriate.
- Terms/Privacy/Refund links must be accessible from the footer and checkout where relevant.
- Contact form submission must use server-side validation, anti-abuse controls, and a real notification/ticket destination.
- Do not collect a phone number unless there is a justified support/payment need and the purpose is explained.
- Cookie/tracking controls must reflect actual trackers. Do not show a consent banner that claims granular control while scripts ignore it.

---

# 8. Cart and checkout workflows

## 8.1 CP-PUB-009 — Cart (P0)

### Cart line item

- Cover thumbnail.
- Ebook title and author.
- Format/access summary where relevant.
- Current price.
- Discount/offer details if applied.
- Remove action.
- Save to wishlist action for signed-in customers.

### Cart summary

- Subtotal.
- Discount.
- Applicable tax/fees only if configured and correctly calculated.
- Total payable.
- Coupon entry (if enabled).
- Continue shopping.
- Continue to checkout.

### Cart rules

- Digital ebooks do not require shipping quantity or shipping options.
- Default to one entitlement per ebook per customer/cart unless multiple licences/gifting has been approved.
- Duplicate additions should merge or show that the product is already in the cart.
- Price and discount must be revalidated by the server before checkout.
- If a product becomes unavailable or its price changes, explain the change and require the customer to review before payment.
- Cart persistence for anonymous visitors may use a secure session/cart token; do not put sensitive purchase data in a publicly readable token.
- For signed-in users, merge guest cart into account cart only under a documented deterministic rule.

## 8.2 CP-PUB-010 — Checkout (P0)

**Goal:** Complete a digital purchase with as few fields and decisions as possible, while meeting payment, security, and tax requirements.

### Recommended steps

1. Identify customer: sign in, create account, or continue as guest if guest checkout is approved.
2. Confirm email address or verify the account/guest purchase identity as required by policy.
3. Enter only necessary billing/tax fields.
4. Choose an available payment method from the configured gateway.
5. Review the complete order summary and policies.
6. Accept required terms where legally/business-required.
7. Continue to payment.

A single-page or short-step checkout may be used. The chosen layout must be usability-tested on mobile and must preserve entered values after recoverable errors.

### Digital-only checkout requirements

- Do not ask for shipping address, shipping method, delivery slot, or courier selection.
- Collect name/email and other billing data only as required by the chosen gateway/tax configuration.
- Mark optional versus required fields clearly.
- Explain why a phone number is requested if required.
- Show the final total, currency, discounts, taxes/fees, and the exact ebooks being purchased before the customer pays.
- Do not hide mandatory charges until after payment initiation.
- Make guest checkout prominent if enabled; do not force registration before the customer can understand the purchase.

### Guest checkout and account linking (decision required)

If guest checkout is approved:

- The customer must provide an email address and verify possession through an approved secure mechanism before the entitlement can be claimed or accessed.
- Do not attach a purchase to an account based only on an unverified email string.
- After successful payment, guide the guest to create/claim an account or securely access the order using a short-lived, single-use, purpose-limited link.
- Avoid exposing order details or download links to anyone who merely knows an order number.
- Define the behaviour when an existing account uses the same email address.

## 8.3 Coupon validation

- Coupon validation is authoritative on the server.
- Validate status, date/time, usage limits, per-customer limits, minimum order, maximum discount, product/category eligibility, and stacking policy.
- Display a clear error for expired, invalid, already-used, or ineligible coupons without exposing internal fraud/risk rules.
- Revalidate immediately before order/payment creation.
- Coupon application/removal must update totals consistently.

## 8.4 Payment initiation and result handling

Payment must be implemented through a provider adapter. Do not couple customer UI components directly to one gateway's response shape.

Required flow:

1. Customer submits checkout.
2. Backend revalidates cart, product publication state, price, discount, tax, customer identity, and total.
3. Backend creates a pending order/payment attempt with an idempotency key.
4. Backend creates the gateway payment session/order using server-held credentials.
5. Customer completes payment through the gateway's approved hosted/embedded interface.
6. Gateway sends a signed server-to-server webhook.
7. Backend verifies webhook authenticity, event identity, amount, currency, order mapping, and current payment state.
8. Backend processes the event idempotently.
9. Only after trusted payment confirmation does the order become paid and entitlements get issued.
10. Customer is redirected to an order result page which fetches the actual server-side order/payment state.

**Critical:** A success redirect, frontend callback, browser query string, screenshot, or client-side status is not proof of payment. Never issue an ebook entitlement solely because the browser says payment succeeded.

### Payment state handling

The customer result page must support:

- `processing`: payment result is not yet confirmed; explain that the customer should not pay again immediately and provide a safe refresh/check-status action.
- `paid`: show order confirmation and library access.
- `failed`: explain that payment was not completed and offer a safe retry when the payment/order policy permits.
- `cancelled`: explain that the payment flow was cancelled.
- `expired`: payment session/order expired; allow a new attempt only through a safe server-side flow.
- `unknown/manual_review`: display a neutral message and a support route; do not imply a successful purchase.

Do not create two paid orders from repeated clicks, browser refreshes, retries, or duplicate webhooks.

## 8.5 CP-PUB-011 — Payment result / order confirmation (P0)

For a verified successful purchase, show:

- Confirmation heading.
- Order number/reference.
- Date/time.
- Purchased ebook list.
- Amount and currency.
- Payment status.
- “Open My Library” / “Start Reading” primary action.
- Receipt/invoice access if available.
- Support link for access problems.

For pending/failed states, do not show success styling or claim the ebooks are owned. Show a safe next step and preserve the order reference.

---

# 9. Authentication, registration, and account recovery

## 9.1 CP-PUB-012 — Login

Fields:
- Email address or approved account identifier.
- Password.
- Show/hide password control.
- Remember-this-device option only if session policy supports it.

Actions:
- Sign in.
- Forgot password.
- Create account.
- Optional passkey/social sign-in only after owner approval and full security design.

Requirements:
- Use generic error messages that do not reveal whether an account exists.
- Rate-limit repeated attempts and apply risk-based protection.
- Use secure session cookies (HttpOnly, Secure, SameSite as appropriate) or an equivalently secure session architecture.
- Rotate session identifiers after authentication and privilege changes.
- Do not store passwords in plaintext or reversible encryption.
- Support password managers and browser autofill; use correct input autocomplete attributes.

## 9.2 CP-PUB-013 — Registration

Fields should be minimal:
- Name/display name if required.
- Email.
- Password if password authentication is selected.
- Confirmation only if required by the form pattern; avoid needless repeated fields.
- Required legal acceptance/consent with links and recorded policy version where applicable.

Requirements:
- Explain why each requested field is needed.
- Verify email before granting sensitive account access where policy requires.
- Apply anti-abuse protection without making genuine users solve excessive challenges.
- Do not activate an account solely because an email address was typed.
- Avoid account enumeration in registration and recovery responses.

## 9.3 CP-PUB-014 — Email verification

- Use cryptographically random, single-use, time-limited tokens.
- Store a hash of the token where appropriate; do not log the raw token.
- Resend must be rate-limited.
- Expired links must have a clear resend path.
- Verification must be bound to the intended account/email and purpose.

## 9.4 CP-PUB-015 — Forgot/reset password

- Use single-use, time-limited reset tokens.
- Return a generic response whether or not the email exists.
- Rate-limit requests and attempts.
- Notify the account owner after a successful reset.
- Invalidate outstanding reset tokens and consider invalidating active sessions according to security policy.
- Never email or reveal the existing password.

## 9.5 Sign-in sessions and optional passkeys

Passkeys/WebAuthn may be added as an optional sign-in method after the primary authentication flow is stable. Do not make passkeys mandatory in the initial release without an explicit product decision and account recovery plan.

Customer security page should list active sessions/devices to the extent supported by the session architecture and offer “Sign out other sessions” where safely implemented.

---

# 10. Customer dashboard and account pages

## 10.1 CP-CUS-001 — Customer Overview (P0)

**Goal:** A customer should immediately know what they own and what to do next.

### Page sections

- Friendly greeting.
- Continue Reading card (most recently opened entitled ebook, if available).
- Recently Added to Library.
- My Library count.
- Recent order summary.
- Wishlist preview.
- New notifications.
- Open support ticket summary.

Do not show business-owner metrics, other customers' data, or irrelevant financial analytics.

## 10.2 CP-CUS-002 — My Library (P0)

This is a core destination and should be easy to access from the main account navigation.

### Layout

- Page title and short explanation.
- Search within owned ebooks.
- Filters: All, In Progress, Not Started, Completed, Download Available, Reader Available (only relevant capabilities).
- Sort: Recently Added, Recently Opened, Title, Author.
- Responsive ebook card/grid or list.

### Library card

- Cover.
- Title and author.
- Purchase/added date where useful.
- Reading progress bar/percentage if tracking is enabled and supported.
- Status: Not Started / In Progress / Completed (definitions must be consistent).
- Primary CTA: Continue Reading or Open Book.
- Secondary action: Download if enabled.

### Entitlement rule

A book appears only if the authenticated customer has a valid entitlement or an explicitly approved free-access entitlement. A purchase record alone is not the only authorization check; the backend must validate the current entitlement status and applicable policy.

## 10.3 CP-CUS-003 — Library Book Details / Access (P0)

Show:
- Cover, title, author, description.
- Purchase/order reference and date (where useful).
- Available formats and access modes.
- Continue Reading / Start Reading.
- Download options if allowed.
- Reading progress and last opened timestamp if enabled.
- Support link for access/download issues.

Do not display private internal file IDs, object-storage keys, signed storage paths, admin notes, or payment gateway secrets.

## 10.4 CP-CUS-004 — Online Reader (P0 if reader enabled)

The reader is a primary product experience. It must be implemented as a focused reading surface, not a product-detail page with a file embedded casually.

### Supported content

- EPUB: plan around W3C EPUB 3.3 and reading-system expectations. Validate EPUB container/package/navigation documents and resource handling. Support reflowable EPUB first; fixed-layout EPUB support must be explicitly tested and documented.
- PDF: use a secure browser-based PDF viewer strategy. Test text selection/search, zoom, page navigation, mobile behaviour, keyboard access, and accessibility limitations.
- Other formats: do not advertise until a reader/download path is actually implemented and tested.

### Reader layout

Header/toolbar:
- Back to My Library.
- Book title.
- Table of contents toggle (where supported).
- Previous/next chapter or page controls.
- Progress indicator.
- Reading settings.
- Bookmark action.
- Help/report issue action.

Reading area:
- Readable column width for reflowable text.
- Clear line height and paragraph spacing.
- Responsive layout.
- Fullscreen/focus mode optional.
- No disruptive popups, advertisements, or promotional widgets within the reading text.

### Reading settings

For reflowable EPUB:
- Font family choices from a small accessible set.
- Font size.
- Line spacing.
- Paragraph spacing/margins where supported.
- Light/sepia/dark reading themes with adequate contrast.
- Page/scroll mode if supported by the chosen reading engine.

For PDF:
- Zoom in/out.
- Fit width/page.
- Page navigation.
- Text search if text layer exists.
- Fullscreen if accessible.

### Reader features

P0:
- Open entitled book.
- Table of contents/navigation where format supports it.
- Persist reading position.
- Resume from last position.
- Adjust basic typography/zoom.
- Handle load, format, and access errors with a useful recovery route.

P1:
- Bookmarks.
- Highlights/annotations if supported and privacy-reviewed.
- Cross-device reading progress synchronization.
- Keyboard shortcuts with a visible/help reference.
- Reading session history.

P2:
- Reading goals, reminders, notes export, offline mode, advanced annotation sync.

### Progress semantics

- Store format-specific location data (for example EPUB CFI/location or a normalized reader position and PDF page/scroll position).
- Do not store only a percentage if a stable format location is available.
- Progress writes must be debounced/batched to avoid excessive writes but must not lose the latest position on normal navigation.
- Handle multiple devices with a deterministic conflict rule (for example latest valid position by timestamp); do not silently jump a customer's position unpredictably.
- “Completed” must use a defined threshold or customer action; do not infer completion from a single open.

### Reader protection and honesty

- Verify entitlement server-side before issuing reader access.
- Use short-lived signed reader sessions or equivalent controlled access.
- Do not expose origin storage URLs or public unrestricted file URLs.
- Consider watermark overlays or personalized rendered exports if approved.
- Do not claim screenshots, screen recording, browser developer tools, or all forms of copying are technically impossible.
- Avoid techniques that break accessibility or legitimate assistive technology.

## 10.5 CP-CUS-005 — Reading Progress / Continue Reading (P0)

This may be a dashboard section and/or a filtered library view rather than a separate complex page. It should show:
- Book cover/title.
- Last read location/progress.
- Last opened timestamp.
- Continue Reading action.

Do not show progress for books that have been revoked or are no longer accessible; explain the access state without leaking internal reasons.

## 10.6 CP-CUS-006 — My Downloads (P0 if downloads enabled)

Show ebooks for which download access is currently valid.

Download flow:
1. Customer selects a format/download action.
2. Backend authenticates customer and verifies entitlement, product download policy, current account state, and any configured limit.
3. Backend records an authorized download event.
4. Backend creates a short-lived signed download URL or streams the file through an authorized delivery service.
5. Customer receives the file or a clear error/retry path.

Requirements:
- Never place permanent public file URLs in HTML or API responses.
- Never trust a customer-supplied file ID without ownership/entitlement checks.
- Apply rate limits and abuse controls.
- Make download limits, if any, visible to customers in plain language before purchase and in My Downloads.
- Avoid counting a request as a completed download if delivery failed before transfer began; define attempted/started/completed events separately.
- Preserve a customer-friendly way to retry legitimate failed downloads.
- If a download limit is exhausted, show a support path; support may investigate but must not silently bypass owner policy.
- Watermarking is optional and must be approved. If used, explain the use of customer/order identifiers in policy text and avoid exposing unnecessary personal information in a shared copy.

## 10.7 CP-CUS-007 — My Orders (P0)

Table/list fields:
- Order reference.
- Date.
- Items summary.
- Total and currency.
- Payment status.
- Order/refund status.
- Action: View Details.

Requirements:
- Sort newest first by default.
- Pagination or incremental loading.
- Filter by date/status if useful.
- Only return orders owned by the authenticated customer.
- Use a stable public order reference that does not expose sequential database IDs.

## 10.8 CP-CUS-008 — Order Details (P0)

Show:
- Order reference and date.
- Ebook line items.
- Subtotal, discount, tax/fees, total, currency.
- Payment method label and status (never card/bank secrets).
- Refund status, if applicable.
- Invoice/receipt link if generated.
- Entitlement fulfilment status.
- Support action for payment or access issue.

Do not show internal fraud/risk notes, gateway secrets, raw webhook data, staff-only comments, or another customer's information.

## 10.9 CP-CUS-009 — Invoice / Receipt (P0 where supported)

- Generate from canonical order/tax records, not client-side values.
- Display accurate seller identity, invoice number, date, buyer/billing fields, line items, totals, tax breakdown, and currency as approved by finance/tax policy.
- Provide accessible HTML view and downloadable PDF where supported.
- Never invent GST values or legal invoice fields. Validate with qualified tax/finance review before production.

## 10.10 CP-CUS-010 — Wishlist (P1)

- Add/remove a published ebook.
- Show current price and availability.
- Move to product detail/cart.
- If a book becomes unpublished or unavailable, explain its status and do not offer purchase.
- Wishlist is private to the customer by default.
- If visitors can wishlist without an account, define secure session persistence/merge behaviour; otherwise ask them to sign in clearly.

## 10.11 CP-CUS-011 — My Reviews (P1)

- Show submitted reviews and moderation state.
- Allow editing/removal only under the approved policy.
- Distinguish verified-purchase status only when the backend can verify it.
- Do not allow customers to review books they have not purchased unless the owner explicitly approves open reviews.
- Explain that reviews may be moderated for abuse, spam, or policy violations.

## 10.12 CP-CUS-012 — Notifications (P1)

Notification categories:
- Orders and payment.
- Library/access.
- Account/security.
- Support ticket updates.
- Marketing (only with appropriate consent/preferences).

Requirements:
- Separate essential service notifications from optional promotional messages.
- Provide unread/read state and mark-all-read.
- Avoid showing sensitive data in push/browser previews by default.
- Do not use notification read state as proof that an email was delivered.

## 10.13 CP-CUS-016 — Profile (P0)

- Display name.
- Email with verified status.
- Phone only if collected and required.
- Billing profile/address where appropriate and supported.
- Update profile with server-side validation.
- Email change requires re-authentication and verification of the new address, with notification to the old address according to security policy.
- Do not allow a profile update to change the identity attached to historical orders without a controlled process.

## 10.14 CP-CUS-017/018 — Security and sessions (P0/P1)

- Change password.
- View sign-in methods supported by the account.
- Sign out current session.
- View/manage active sessions where supported.
- Sign out other sessions.
- Show security notifications such as password or email changes.
- Re-authenticate before sensitive changes.
- Never show passwords, reset tokens, session tokens, or full authentication secrets.

## 10.15 CP-CUS-019 — Reading preferences (P1)

Persist reader settings where supported:
- Theme.
- Font size/family.
- Line spacing.
- Margins/column width.
- Reading mode.

Preference saving should be unobtrusive and should not overwrite a user's current settings with defaults on every session.

## 10.16 CP-CUS-020 — Communication preferences (P1)

- Essential account/order/security messages cannot be disabled where required to deliver the service, but explain why.
- Marketing email/newsletter consent must be separate and explicit where required.
- Store consent timestamp, source, policy/version, and withdrawal timestamp.
- Unsubscribe must work and must not require login if the relevant rules allow one-click unsubscribe.

## 10.17 CP-CUS-021/022 — Privacy and account deletion requests (P1)

Provide a request workflow for:
- Downloading available personal data.
- Correcting personal data.
- Deleting/deactivating an account.
- Contacting privacy support.

Do not delete financial/order records that must legally be retained. Define an approved process for anonymizing personal data and revoking sessions/entitlements. The final deletion/retention policy requires legal review for applicable jurisdictions.

---

# 11. Customer support experience inside the Customer Panel

This is part of the Customer Panel, not a fourth panel.

## 11.1 CP-CUS-013 — Support Centre / My Tickets (P0)

Show:
- Create new support request.
- Open tickets.
- Waiting for customer response.
- Resolved/closed tickets.
- Ticket reference, subject, category, status, last update.

Customers can see only tickets they created or are explicitly authorized to access.

## 11.2 CP-CUS-014 — Create Support Request (P0)

Fields:
- Category: order/payment, missing library ebook, reader issue, download issue, account/security, refund request, other.
- Related order selector (only own orders).
- Related ebook selector (only public or owned product context as appropriate).
- Subject.
- Description.
- Optional attachment subject to safe upload validation.

Requirements:
- Pre-fill known order/book context when the customer enters support from an order/book page.
- Do not ask the customer to retype details already safely available.
- Explain expected response expectations only if a real support SLA has been configured.
- Confirmation shows a ticket reference and what happens next.

## 11.3 CP-CUS-015 — Ticket Conversation (P0)

- Chronological conversation.
- Clear distinction between customer and support replies.
- Attachment previews/downloads only after authorization and safety checks.
- Reply box.
- Add attachment.
- Close/reopen action only according to ticket policy.
- Show status and last updated timestamp.

Internal notes must never be returned by the customer-facing ticket API.

---

# 12. Support/Operations Panel information architecture

## 12.1 Overall layout

Use a focused internal workspace with:

- Left sidebar: queue/navigation.
- Top bar: global ticket/customer/order search, alerts, agent profile and help.
- Main content: queue/list or active ticket.
- Optional contextual side panel: customer, order, payment, entitlement, and activity summary.

Support UI should prioritize speed, accuracy, context, and clear handoffs. Do not overload agents with owner-level analytics or settings.

## 12.2 Support sidebar

1. Overview
2. My Queue
3. Unassigned
4. All Tickets (permission-controlled)
5. Pending Customer
6. Escalated
7. Resolved / Closed
8. Customer Lookup
9. Order Lookup
10. Payment / Access Investigations
11. Knowledge Base / Macros
12. Reports (limited, role-controlled)
13. Team / Queue Settings (senior/manager only)

The sidebar must show only permitted destinations, while backend authorization remains authoritative.

## 12.3 Support/Operations sitemap

| Page ID | Page | Primary purpose | Default role |
|---|---|---|---|
| SO-001 | Support Overview | Queue and operational summary | All support roles |
| SO-002 | My Queue | Tickets assigned to current agent | Agent+ |
| SO-003 | Unassigned Queue | Tickets waiting for assignment | Senior/Manager |
| SO-004 | All Tickets | Search and manage authorized tickets | Permission-controlled |
| SO-005 | Ticket Detail Workspace | Investigate and respond to a ticket | Agent+ |
| SO-006 | Customer Lookup | Find limited customer account context | Agent+ |
| SO-007 | Customer Support Profile | Orders, library/access and ticket summary | Agent+ limited |
| SO-008 | Order Lookup | Find and inspect an order | Agent+ |
| SO-009 | Order Investigation | Payment/fulfilment timeline | Agent+ limited |
| SO-010 | Payment Investigation | Investigate payment result and provider reference | Senior/Finance-approved |
| SO-011 | Entitlement / Download Investigation | Investigate ebook access/delivery issue | Agent+ limited |
| SO-012 | Refund Request Workspace | Create/review approved refund requests | Permission-controlled |
| SO-013 | Knowledge Base | Internal guidance and customer-safe help articles | Agent+ |
| SO-014 | Reply Macros | Approved response templates | Senior/Manager manage; Agent use |
| SO-015 | Escalation / Handoff | Transfer cases to appropriate team | Agent+ |
| SO-016 | Support Reports | Queue/SLA/quality operational metrics | Manager/Analyst permission |
| SO-017 | Support Team / Queue Settings | Assignment and categories | Manager/Admin only |
| SO-018 | Agent Profile / Availability | Agent details and queue presence if used | Own profile / manager |
| SO-019 | Support Audit History | Review support actions | Senior/Manager/Audit role |

---

# 13. Detailed Support/Operations page specifications

## 13.1 SO-001 — Support Overview

### Summary cards

- My open tickets.
- Unassigned tickets (only if role permits).
- Tickets awaiting customer response.
- Escalated tickets.
- Overdue tickets, only if a real SLA is configured.
- Average first response/resolution time, with calculation definition and date range.

### Main sections

- My assigned queue.
- Priority/aged tickets.
- Escalation alerts.
- Recent updates.
- Team load for managers.

Do not present fake SLA warnings. If no SLA policy exists, show ticket age and status rather than claiming a breach.

## 13.2 SO-002 — My Queue

Columns:
- Ticket ID.
- Customer name (masked or minimized where appropriate).
- Subject.
- Category.
- Priority.
- Status.
- Related order (if applicable).
- Last updated.
- Waiting duration.

Controls:
- Search.
- Filter by status/category/priority/date.
- Sort by oldest, newest, priority, waiting time.
- Open ticket.
- Claim/release/reassign only with permission.

## 13.3 SO-003 — Unassigned Queue

For senior agents/managers:
- Show unassigned tickets.
- Filter by category, priority, age, language, and issue type where these attributes exist.
- Assign to a specific agent or queue.
- Bulk assignment only with confirmation and audit.
- Avoid unfairly overloading one agent; show queue load to managers if supported.

## 13.4 SO-004 — All Tickets

A unified search/list across tickets the staff member is allowed to see.

- Do not allow unrestricted search by arbitrary personal data for every role.
- Apply server-side pagination and filters.
- Keep internal and customer-facing notes distinct.
- Show ticket status and assignment clearly.

## 13.5 SO-005 — Ticket Detail Workspace

This is the primary support screen.

### Layout

Main conversation area:
- Ticket subject and ID.
- Customer message timeline.
- Customer attachments.
- Reply composer.
- Internal note composer (visually distinct and never sent to customer).
- Canned response/macro picker.
- Attachment upload.
- Send/reply action.

Side context panel:
- Customer identity summary.
- Related order(s).
- Related ebook(s).
- Payment status (limited, verified status only).
- Entitlement/access status.
- Related tickets.
- Assigned agent/queue.
- Priority and category.

Ticket controls:
- Assign/reassign.
- Change priority.
- Change category.
- Set status.
- Escalate.
- Link/unlink order/book with permission and confirmation.
- Close/resolve with resolution reason.

### Reply requirements

- Customer reply and internal note must use separate API actions and separate data fields/types.
- Show a strong visual difference between public replies and internal notes.
- Prevent sending an internal note to the customer due to a wrong toggle or ambiguous button.
- Preserve drafts after transient errors.
- Avoid duplicate replies on repeated clicks using idempotency.
- Show sent/delivered state only where provider data supports it.

## 13.6 SO-006/007 — Customer lookup and support profile

Support staff can search using approved identifiers such as customer ID, exact email, or order reference, subject to permissions and audit.

Show only necessary context:
- Customer ID and display name.
- Verified email state; mask email where full value is not needed.
- Account status.
- Joined date.
- Recent orders summary.
- Library entitlement summary.
- Recent tickets.
- Customer-facing account/security status needed for support.

Do not show:
- Passwords or password hashes.
- Reset/verification tokens.
- Full card/bank information.
- Gateway secrets.
- Full private ebook files.
- Unrelated browsing history or invasive tracking data.
- Unnecessary personal or sensitive profile fields.

Any account suspension, email change, security reset, or sensitive account intervention must use a specific permission, require a reason, and create an audit record. Support agents should not impersonate customers by default.

## 13.7 SO-008/009 — Order lookup and investigation

### Order summary

- Public order reference.
- Customer reference.
- Order creation time.
- Items.
- Currency and totals.
- Payment state.
- Refund state.
- Entitlement state.

### Timeline

- Order created.
- Payment attempt created.
- Gateway event received/verified.
- Order paid/failed/cancelled/expired.
- Entitlement created/revoked.
- Notification queued/sent/failed, where supported.
- Refund requested/approved/processed/failed.

Support staff may explain status and identify where a workflow stopped. They must not directly change payment status or fabricate a successful transaction.

## 13.8 SO-010 — Payment Investigation

Show only the minimum payment data needed:
- Internal payment attempt ID.
- Gateway/provider name.
- Gateway transaction/reference ID, if available.
- Amount/currency.
- Current verified status.
- Created/updated timestamps.
- Webhook processing state and safe event reference.
- Reconciliation status if available.

Mask secrets and sensitive payer details. Raw webhook payloads should not be exposed by default; if a senior technical support permission needs diagnostic access, redact secrets and personal data.

Support actions:
- Refresh/reconcile status through a safe server-side action if designed.
- Create finance escalation.
- Attach internal investigation note.
- Link relevant gateway event ID.

Forbidden actions by default:
- Manually mark paid.
- Edit amount/currency.
- Change gateway credentials.
- Replay arbitrary webhook payloads.
- Issue refund without explicit refund permission and workflow.

## 13.9 SO-011 — Entitlement / Download Investigation

Show:
- Entitlement ID and current status.
- Customer and ebook reference.
- Source order.
- Access modes available.
- Download policy/limit summary (not private file URL).
- Download attempts/events and outcome.
- Reader access/session errors where safely recorded.
- Last successful access event.

Support may:
- Retry a safe fulfilment operation if the backend provides an idempotent retry action.
- Create an entitlement-repair request for authorized operations staff.
- Explain current access policy.
- Escalate malware/format/file integrity issues.

Do not expose storage keys, permanently signed links, or original files to ordinary support agents. A support action must not grant access to a customer who lacks a valid entitlement unless a separately approved exception workflow exists.

## 13.10 SO-012 — Refund Request Workspace

A customer refund request and a payment-provider refund are distinct records/stages.

Workflow:
1. Customer submits a request or an authorized staff member records a request.
2. System links the request to the original order/payment.
3. Staff checks the owner-approved refund policy and records evidence/reason.
4. Request is assigned for approval if the agent lacks approval permission.
5. Authorized reviewer approves/rejects with a reason.
6. Backend calls the gateway refund API using server credentials and an idempotency key.
7. Provider confirmation/webhook updates refund state.
8. Order financial status and access-entitlement policy are updated according to approved rules.
9. Customer is notified of the actual state.

Refund statuses should distinguish:
- requested
- under_review
- approved
- rejected
- submitted_to_gateway
- processing
- succeeded
- failed
- cancelled (only where provider/workflow permits)

The support UI must never display “Refunded” merely because an agent clicked approve. Approval and provider settlement/confirmation are separate states.

## 13.11 SO-013 — Knowledge Base

Internal knowledge base for agents:
- Troubleshooting steps.
- Payment issue runbooks.
- Download and reader issue guides.
- Account recovery steps.
- Refund policy guidance.
- Escalation rules.

Articles should have title, category, version, owner, status, updated date, and searchable content. Only authorized roles may publish or edit internal operational guidance.

A separate customer-facing Help Centre may expose selected public articles. Never expose internal-only notes or escalation instructions.

## 13.12 SO-014 — Reply Macros

- Create approved response templates.
- Use safe variables such as customer first name, order reference, ticket reference, and support signature.
- Preview fully interpolated message before sending.
- Escape/untrusted input safely to prevent injection.
- Version and audit macro changes.
- Do not include a macro that promises a refund, access, timeline, or policy exception unless that promise is approved and accurate.

## 13.13 SO-015 — Escalation / Handoff

Support can escalate to:
- Senior support.
- Operations manager.
- Finance/owner through the Admin Panel's approved process.
- Technical/infrastructure queue if available within the internal role system.

Escalation must include reason, issue summary, current evidence, customer expectation, and requested next action. The customer-facing ticket should remain coherent across assignment changes.

## 13.14 SO-016 — Support Reports

Operational metrics may include:
- Ticket volume by category.
- Open/closed ticket counts.
- First response time.
- Resolution time.
- Reopen rate.
- Escalation rate.
- Ticket age distribution.
- Customer satisfaction only if actively collected and statistically described.

Define clock rules, working hours, business timezone, paused states, and whether time to customer response is excluded before publishing SLA metrics. Support reports must not expose full customer data to users without report permissions.

## 13.15 SO-017/018/019 — Team, profile, audit

- Team settings: queue categories, assignment rules, working hours, SLA settings, macro permissions.
- Agent profile: display name, notification preference, availability/presence if used.
- Audit history: ticket assignment changes, status changes, private note/reply event, customer account actions, refund actions, entitlement-repair actions, exports, and permission changes.

Do not record raw authentication tokens, payment secrets, full bank/card data, or unnecessary private content in audit logs.

---

# 14. End-to-end customer workflows

## 14.1 Workflow A — Visitor discovers and purchases an ebook

1. Visitor opens homepage.
2. Visitor browses categories, collections, search, or recommendations.
3. Visitor opens an ebook detail page.
4. Visitor inspects description, price, format, access modes, and sample if available.
5. Visitor selects Add to Cart or Buy Now.
6. Cart validates current product availability and price.
7. Visitor proceeds to checkout.
8. Visitor signs in, creates account, or uses guest checkout if approved.
9. Checkout collects the minimum required billing/payment details.
10. Backend revalidates totals, coupon, availability, and identity.
11. Backend creates a pending order and payment attempt.
12. Gateway payment flow completes.
13. Backend verifies signed provider event/response and idempotently marks the payment.
14. Backend creates the entitlement.
15. Backend queues transactional confirmation notification.
16. Customer sees the authoritative order result.
17. Customer opens My Library and starts reading/downloads according to product policy.

Failure handling:
- If payment is pending, do not create entitlement yet.
- If payment fails, preserve enough order/cart context for a safe retry.
- If payment succeeds but entitlement creation fails, use a durable retry/outbox/repair mechanism and alert operations; do not ask the customer to pay again.
- If the notification fails, the library entitlement should still exist and be visible after login.

## 14.2 Workflow B — Registered customer purchases an ebook

Same as Workflow A, but the signed-in customer identity is already verified. The backend still checks account state, current product status, price, ownership/duplicate purchase rules, and payment state.

If the customer already owns the ebook, show “Open in My Library” unless repeat purchase/gift purchase is explicitly supported.

## 14.3 Workflow C — Guest checkout purchase claim (only if approved)

1. Guest provides email.
2. Backend creates a pending guest purchase identity/order.
3. Guest completes payment.
4. Backend verifies payment.
5. Backend creates a claim-pending entitlement or securely links it to a verified identity based on approved policy.
6. Customer verifies email using a single-use time-limited token/link.
7. Customer creates/claims account.
8. Entitlement becomes available in My Library.
9. Reused/expired claim tokens are rejected and can be safely reissued after rate-limited verification.

Never allow someone to claim an order merely by entering the order number or knowing the purchaser's email address.

## 14.4 Workflow D — Customer resumes reading

1. Customer signs in.
2. Customer opens My Library.
3. Library shows Continue Reading for entitled books with saved progress.
4. Customer selects a book.
5. Backend verifies entitlement and reader availability.
6. Reader loads the correct format and saved position.
7. Customer reads and adjusts accessible reader preferences.
8. Reader periodically persists position/bookmark updates.
9. Customer exits or changes device.
10. Next session retrieves the latest valid position and resumes.

## 14.5 Workflow E — Customer downloads an ebook

1. Customer opens My Library or My Downloads.
2. Customer selects a supported format.
3. Backend verifies authentication, account state, entitlement, product policy, and current download limit.
4. Backend records download authorization/event.
5. Backend creates short-lived delivery URL or streams the file.
6. Browser receives file.
7. Download outcome is recorded to the extent technically measurable.
8. If failed, customer receives a retry/support route.

## 14.6 Workflow F — Customer reports missing ebook after payment

1. Customer opens order detail and selects “I can't access this ebook”.
2. Support ticket is prefilled with order reference and relevant ebook.
3. Support agent opens the ticket and sees limited order/payment/entitlement context.
4. Agent verifies actual payment state and entitlement state.
5. If payment is pending/failed, agent explains the verified state and escalates if needed.
6. If payment is paid but entitlement is missing, agent creates/initiates the approved idempotent fulfilment repair workflow.
7. Backend confirms entitlement creation.
8. Customer receives a response and can access the book.
9. Ticket is resolved with an audit timeline.

## 14.7 Workflow G — Customer has a failed download

1. Customer reports download failure from the book page or My Downloads.
2. Support checks authorization/entitlement, format status, file health, delivery logs, and limits.
3. If a transient delivery failure occurred, support may trigger a safe retry/reissue action if authorized.
4. If the ebook file is corrupted/missing, escalate to operations/admin to replace/repair the source file.
5. Customer is notified only when a real state change has occurred.

## 14.8 Workflow H — Customer requests a refund

1. Customer opens order and chooses refund/help request.
2. UI shows the approved policy and eligibility information without guaranteeing approval.
3. Customer provides reason and submits request.
4. Backend creates refund request linked to order/payment.
5. Support investigates and records evidence.
6. Authorized role approves/rejects.
7. If approved, backend submits provider refund.
8. Provider result is verified and stored.
9. Entitlement access is changed only according to the approved refund-access policy.
10. Customer sees actual refund state and receives notification.

## 14.9 Workflow I — Account recovery

1. Customer requests password reset.
2. UI displays generic acknowledgement.
3. Backend sends a single-use time-limited link if account matches.
4. Customer sets a new password.
5. Existing sessions are handled according to policy.
6. Customer receives security notification.
7. If recovery fails, customer can open a support ticket, but support cannot view/set the customer's existing password.

---

# 15. Shared domain model and data ownership

The agent must compare this logical model with the existing repository before introducing duplicate tables. Names are conceptual and may be mapped to the established naming convention.

## 15.1 Customer and identity entities

| Entity | Important fields / purpose |
|---|---|
| Customer | id, display_name, status, created_at, updated_at |
| CustomerEmail | customer_id, normalized_email, verified_at, is_primary |
| CustomerProfile | customer_id, optional profile/billing fields, locale |
| CustomerConsent | customer_id, consent_type, policy_version, granted_at, withdrawn_at, source |
| AuthCredential | customer_id, credential_type, secure credential metadata (never plaintext password) |
| Session | customer_id, session metadata, created_at, expires_at, revoked_at |
| CustomerPreference | customer_id, preference_key, preference_value, updated_at |

## 15.2 Catalogue and commerce entities

These may already be defined by the Admin Panel specification:

- Ebook/Product.
- EbookFormat/DigitalAsset.
- PublicPreviewAsset.
- Author (content metadata only; no author portal).
- Category and CategoryRelation.
- Collection and CollectionItem.
- Price/PriceRule.
- Coupon/Promotion.
- Cart and CartItem.
- Order and OrderItem.
- PaymentAttempt and PaymentEvent.
- RefundRequest and RefundTransaction.
- Invoice/Receipt.

Do not duplicate catalogue, order, or payment records in a separate customer database just to support the frontend.

## 15.3 Entitlement and reader entities

| Entity | Important fields / purpose |
|---|---|
| Entitlement | id, customer_id or claim_identity_id, product_id, order_item_id, status, granted_at, revoked_at, source |
| EntitlementPolicySnapshot | entitlement_id, access modes/limits granted at purchase time, policy version |
| DownloadEvent | id, entitlement_id, asset_version_id, status, timestamp, safe metadata |
| ReaderSession | id, customer_id, entitlement_id, expires_at, revoked_at |
| ReadingProgress | customer_id, entitlement_id, format, location_data, percentage, last_opened_at, updated_at |
| Bookmark | customer_id, entitlement_id, format/location, label, created_at |
| Annotation (P1) | customer_id, entitlement_id, location, content, privacy flags, created_at, updated_at |

Store the access policy snapshot so a later product configuration change does not silently rewrite the purchased customer's rights unless the approved terms/policy permit it.

## 15.4 Support entities

| Entity | Important fields / purpose |
|---|---|
| SupportTicket | id, public_reference, customer_id, subject, category, priority, status, assigned_agent_id, created_at, updated_at |
| TicketMessage | id, ticket_id, actor_id, visibility (customer/internal), message_body, created_at |
| TicketAttachment | id, ticket_id, message_id, secure_asset_id, scan_status, visibility |
| TicketAssignmentEvent | ticket_id, from_agent/queue, to_agent/queue, actor, reason, timestamp |
| TicketStatusEvent | ticket_id, old_status, new_status, actor, timestamp, reason |
| SupportMacro | id, title, body/template, status, version |
| KnowledgeArticle | id, title, body, visibility, version, status |
| SupportSLA (if enabled) | policy, working hours, response/resolution targets, pause conditions |
| SupportAuditEvent | actor, action, target, reason, timestamp, safe context |

## 15.5 Data ownership rules

- Customer identity is managed by the shared identity service/domain.
- Orders/payments/refunds are owned by the commerce/finance domain shared with Admin.
- Entitlements are owned by the digital access domain and read by Customer and Support.
- Reading progress/preferences are owned by the customer reading domain.
- Tickets/messages are owned by the support domain.
- Staff permissions are centrally managed; Support UI must not maintain an independent weaker permission database.
- The Customer Panel must not read or write database tables directly from browser code.

---

# 16. State machines and business invariants

## 16.1 Order lifecycle (illustrative; align to existing Admin model)

Possible states:
- `pending_payment`
- `paid`
- `payment_failed`
- `cancelled`
- `partially_refunded`
- `refunded`
- `manual_review`

Allowed transitions must be explicit. Do not allow a customer or support agent to transition an order directly to `paid`.

## 16.2 Payment lifecycle

Possible states:
- `created`
- `requires_action`
- `processing`
- `succeeded`
- `failed`
- `cancelled`
- `expired`
- `unknown`

Payment events are append-only evidence. Use idempotency and provider reference uniqueness to prevent duplicate state changes.

## 16.3 Entitlement lifecycle

Possible states:
- `claim_pending` (guest flow only, if approved)
- `active`
- `suspended`
- `revoked`
- `expired` (only if access expiry is a configured product policy)

Invariants:
- A paid, fulfilment-eligible order item creates one entitlement according to product purchase rules.
- Duplicate payment webhooks do not create duplicate entitlements.
- A failed/pending payment does not create an active entitlement.
- Only authorized backend workflows may revoke/suspend an entitlement.
- A refund affects entitlement only according to the approved refund-access rule.
- Every change has an audit record and reason where applicable.

## 16.4 Ticket lifecycle

Suggested states:
- `new`
- `open`
- `pending_customer`
- `pending_internal`
- `escalated`
- `resolved`
- `closed`

Define which roles may reopen tickets, whether customer replies reopen resolved tickets, and how archived tickets behave. Do not let an agent close a ticket without an appropriate resolution code if the workflow requires one.

---

# 17. API and integration boundaries

This document does not dictate a specific framework or transport. Use the repository's established API conventions, but keep the following boundaries.

## 17.1 Customer-facing API capability groups

- Public catalogue: list/search/category/collection/detail/preview.
- Customer identity: register/login/logout/verify/recovery/profile/session.
- Cart: create/read/update/remove/apply coupon.
- Checkout: calculate quote/create order/create payment attempt/retrieve result.
- Orders: list own orders/read own order/invoice.
- Library: list own entitlements/read access metadata.
- Reader: create authorized reader session/read progress/update progress/bookmarks.
- Downloads: request authorized download/record outcome.
- Wishlist: list/add/remove.
- Reviews: list eligible reviews/create/update own review.
- Notifications: list/read/mark read.
- Support: create/list own tickets/read own thread/add customer reply/add safe attachment.
- Privacy: request export/correction/deletion workflow.

## 17.2 Support-facing API capability groups

- Queue/list/search authorized tickets.
- Read ticket and permitted contextual records.
- Reply publicly.
- Add private internal note.
- Assign/reassign/escalate/change status.
- Lookup customer/order within permission and purpose limits.
- Inspect payment state and safe event metadata.
- Inspect entitlement/download diagnostics.
- Create refund request or invoke approval workflow only with permission.
- Trigger approved idempotent fulfilment repair.
- Read support metrics/knowledge base/macros.

## 17.3 API invariants

- Customer APIs derive customer identity from the authenticated session/token. Never trust `customer_id` supplied by the browser to scope private requests.
- Support APIs validate staff role, action permission, record scope, and action reason where required.
- Use opaque public references; do not expose sequential primary keys as authorization mechanisms.
- Use server-side pagination and validated filter/sort allowlists.
- Validate payload size, type, and schema.
- Apply rate limits to login, search suggestions, coupon attempts, checkout creation, download requests, support creation, and other abuse-prone endpoints.
- Use idempotency keys for payment/order creation, webhook processing, refund submissions, and retryable fulfilment operations.
- Return safe error codes/messages; do not leak stack traces, file paths, secrets, or internal fraud rules.
- Version APIs if the existing architecture uses versioning; do not introduce incompatible shapes without migration planning.

## 17.4 Suggested event-driven integration

Use reliable asynchronous events/outbox patterns for operations that must survive transient failures, such as:

- `payment.succeeded`
- `order.paid`
- `entitlement.grant.requested`
- `entitlement.granted`
- `notification.requested`
- `download.started`
- `download.completed`
- `refund.requested`
- `refund.succeeded`
- `ticket.created`
- `ticket.escalated`

The exact event names may follow repository conventions. Events should have unique IDs, version/schema, occurrence time, aggregate reference, and safe metadata. Consumers must be idempotent. Do not put passwords, full payment details, full file contents, or secrets into events.

---

# 18. Security, privacy, and digital content protection

## 18.1 Security baseline

Use OWASP ASVS 5.0.0 as a verification reference and relevant OWASP Cheat Sheets for authentication, authorization, file uploads, sessions, and data protection. Select a risk-appropriate verification level with the security reviewer; do not claim formal certification without a real assessment.

## 18.2 Authentication controls

- Secure password hashing with a modern approved password-hashing algorithm and parameters.
- Multi-factor authentication for staff accounts; strongly recommended/required for privileged support and admin roles.
- Rate limiting and brute-force detection.
- Secure session lifecycle, rotation, expiration, and revocation.
- CSRF protection for cookie-authenticated state-changing requests.
- Generic login/recovery errors to reduce account enumeration.
- Email verification and secure address change.
- Password reset tokens single-use and time-limited.
- No plaintext passwords or credentials in logs.

## 18.3 Authorization controls

- Deny by default.
- Validate permission on every API request.
- Enforce customer ownership at object level.
- Enforce staff role and record scope at object level.
- Separate customer and staff auth audiences/clients or equivalent session contexts.
- Do not trust a frontend role claim without server-side validation.
- Prevent insecure direct object reference (IDOR) on order, invoice, ticket, entitlement, download, and reader endpoints.
- Test cross-customer and cross-role access explicitly.

## 18.4 Digital file upload and storage

For admin-managed ebook files and support attachments:

- Use allowlisted extensions and verify actual file signatures/content; do not trust browser Content-Type.
- Generate server-side storage names; do not use user filenames as object keys.
- Enforce size, decompression, page-count, resource, and processing limits.
- Store protected original ebook files outside the public web root or in private object storage.
- Use malware scanning/sandboxing where available and appropriate.
- Parse/convert files in isolated workers with resource limits.
- Keep public sample assets separate from protected originals.
- Do not execute scripts/content from uploaded assets in the application's origin.
- Apply safe Content-Disposition, Content-Type, and content security policies.
- Quarantine suspicious files and show clear internal status.

Support attachments must have size/type restrictions, safe retrieval authorization, and malware scanning before staff/customer access.

## 18.5 Protected ebook access

- Never make purchased source files anonymously public.
- Every reader session and download request must validate entitlement and current policy.
- Use short-lived signed URLs, authenticated proxy streaming, or equivalent controls.
- Avoid logging signed URLs or tokens.
- Rate-limit downloads and reader session creation.
- Revoke sessions on account security events and entitlement revocation where appropriate.
- Use personalized watermarking only if approved, with data minimization and transparent policy.
- Do not market the system as “uncrackable” or “100% piracy-proof”.

## 18.6 Privacy controls

- Data minimization: collect only required information.
- Purpose limitation: use customer data for stated service purposes.
- Retention: define durations and deletion/anonymization workflows.
- Consent: separate required service notices from optional marketing consent.
- Logging: mask or pseudonymize personal identifiers where possible.
- Exports: scope and authorize customer data exports.
- Support visibility: purpose-limited customer data access and audit.
- Avoid using private reading history for marketing/recommendations without appropriate transparency and controls.

## 18.7 Payment security

- Prefer a provider-hosted checkout/secure components to reduce direct handling of card data.
- Never store CVV or raw card credentials.
- Keep API secrets server-side in a secrets manager/environment configuration.
- Verify webhook signatures and event identity.
- Check amount/currency/order mapping before marking paid.
- Use idempotency keys and replay protection.
- Reconcile provider status where event ordering/duplication is possible.
- Keep payment and refund audit trails.
- The chosen provider and applicable PCI obligations require a separate integration/security review.

---

# 19. Accessibility, internationalization, SEO, and performance

## 19.1 Accessibility target

Target WCAG 2.2 AA for all core journeys, including:
- Browse and search.
- Product detail and sample preview.
- Cart and checkout.
- Login and registration.
- My Library.
- Reading controls.
- Downloads.
- Support ticket creation/conversation.

Requirements:
- Correct semantic headings and landmarks.
- Keyboard access to all controls.
- Visible, unobscured focus indicators.
- Programmatic labels and meaningful error association.
- Sufficient text/non-text contrast.
- Avoid colour-only status communication.
- Touch target design goal of approximately 44 by 44 CSS pixels where practical; do not fall below WCAG 2.2 AA requirements without a justified exception.
- Respect reduced-motion preferences.
- Screen-reader names for icon-only controls.
- Accessible dialogs, menus, tabs, pagination, and toasts.
- Reader accessibility must be tested separately for each supported ebook format/engine.

## 19.2 Ebook accessibility

- Preserve EPUB accessibility metadata where available.
- Make format/accessibility information discoverable on product pages when known.
- Avoid stripping semantic structure, headings, reading order, or alternative text when processing EPUB content.
- Provide accessible reader controls and keyboard navigation.
- PDF accessibility depends on the uploaded PDF's structure; do not claim an untagged PDF has become accessible merely because it opens in a viewer.
- Allow the customer to report an inaccessible/corrupted ebook.

## 19.3 Internationalization

- Store timestamps in UTC and render in the customer's chosen/local timezone where appropriate.
- Store currency code with every monetary value; do not assume all monetary values are INR in the data model.
- Use locale-aware date/number/currency formatting.
- Avoid concatenating translated strings in code.
- Support Unicode names and multilingual search where relevant.
- Keep a canonical language/locale field for catalogued ebook metadata.
- Initial supported languages/currencies require owner approval.

## 19.4 SEO and public discoverability

Public pages should support:
- Unique title and meta description.
- Canonical URLs.
- Open Graph/social preview metadata.
- Sitemap entries for public indexable products/categories/authors/collections.
- Structured product data only when fields are accurate and match the visible page.
- Correct index/noindex rules for account, cart, checkout, private, search parameter, and internal pages.
- Clean stable slugs.
- Useful alt text for content images.

Do not expose private files, customer pages, internal support URLs, or order references to search engines.

## 19.5 Performance targets

Use field data where available and target Google's “good” Core Web Vitals at the 75th percentile:
- LCP: 2.5 seconds or less.
- INP: less than 200 milliseconds.
- CLS: less than 0.1.

Implementation guidance:
- Optimized responsive cover images and lazy loading below the fold.
- Reserve image dimensions to avoid layout shift.
- Use CDN/caching for public assets and public catalogue data where safe.
- Do not cache private customer data across users.
- Keep JavaScript bundles appropriate to the route; do not load the full reader bundle on every storefront page.
- Paginate catalogue/library/order/ticket lists.
- Avoid N+1 queries and unbounded API results.
- Measure mobile performance on realistic network/device profiles.

## 19.6 Progressive Web App (optional P1/P2)

An installable PWA may be considered to improve app-like access. It is not a launch blocker. If implemented:
- Provide a valid web app manifest and install experience where browser support exists.
- Cache public app shell/static assets safely.
- Do not cache protected ebooks or private API responses in shared browser caches by default.
- Explain offline limitations honestly.
- Do not claim offline reading works unless license, encryption, expiry, revocation, and device-storage rules have been designed and tested.

---

# 20. Notifications and communications

## 20.1 Transactional messages

Potential email/in-app notifications:
- Account verification.
- Password reset/security event.
- Order confirmation.
- Payment pending/failed/succeeded state (only verified/authoritative states).
- Ebook added to library/access available.
- Refund request received.
- Refund approved/rejected/processing/completed/failed.
- Support ticket created/replied/resolved.

## 20.2 Notification rules

- Payment/entitlement email is not the sole delivery mechanism; My Library is authoritative after sign-in.
- Failed email delivery must not roll back a verified payment or entitlement.
- Use an outbox/retry mechanism for important transactional notifications.
- Avoid duplicate notifications from repeated webhooks.
- Do not include permanent ebook download URLs in email.
- Use short-lived secure links only where necessary and only after identity/entitlement checks.
- Promotional notifications require appropriate consent/preferences.

## 20.3 Customer communication history

Customer may see relevant communications in Notifications and Support. Support may see the safe service history required for troubleshooting. The Admin Panel remains the owner control point for templates and delivery providers.

---

# 21. Analytics and product instrumentation

Collect only analytics needed to improve the product and operate the service. Use an approved privacy/consent model.

## 21.1 Customer funnel events

Examples:
- `catalogue_viewed`
- `search_submitted`
- `search_no_results`
- `ebook_viewed`
- `sample_opened`
- `add_to_cart`
- `remove_from_cart`
- `checkout_started`
- `payment_attempt_created`
- `payment_result_verified`
- `order_paid`
- `entitlement_granted`
- `library_opened`
- `reader_opened`
- `reading_progress_saved`
- `download_requested`
- `download_completed`
- `support_ticket_created`

Use pseudonymous identifiers where possible. Do not put email, passwords, payment details, full reader content, or private ticket message text into analytics event payloads.

## 21.2 Definitions

- Product view: server/client event after a product detail page is meaningfully loaded.
- Add-to-cart: backend confirms the cart item was added.
- Checkout started: a valid checkout session/order draft exists.
- Purchase conversion: verified paid order, not button click or redirect.
- Library activation: customer opens an entitled ebook.
- Download completion: only report completion if the delivery infrastructure can measure it reliably; otherwise label it “download started/requested”.

## 21.3 Do not build dark patterns

Do not use fake scarcity, fake countdowns, fabricated review counts, hidden fees, pre-checked promotional consent, misleading discount percentages, or intentionally confusing unsubscribe/account deletion controls.

---

# 22. UI component and interaction inventory

The design system should provide reusable, accessible components:

### Public storefront
- Header/navigation and mobile menu.
- Search input and accessible suggestion list.
- Ebook card and ebook grid/list.
- Cover image with fallback.
- Price/discount display.
- Rating/review summary.
- Filter panel and active filter chips.
- Sort selector.
- Breadcrumb.
- Pagination/load more.
- Cart drawer/page and order summary.
- Coupon input.
- Checkout form fields.
- Payment method selector.
- Confirmation/status panel.
- Preview viewer.
- Footer and policy links.

### Customer account
- Account layout/navigation.
- Library book card/list.
- Reading progress component.
- Continue-reading card.
- Reader toolbar, TOC, settings drawer, bookmark control.
- Download action/state.
- Order list and order detail summary.
- Invoice view/download.
- Wishlist control.
- Review form/list.
- Notification list.
- Profile/security form.
- Support ticket list/conversation/composer.

### Support workspace
- Queue navigation.
- Ticket table/list.
- Priority/status badges.
- Ticket timeline/conversation.
- Public reply composer.
- Internal note composer (distinct styling and action).
- Customer/order context panel.
- Assignment/escalation dialog.
- Refund request workflow panel.
- Entitlement/download diagnostic view.
- Macro/knowledge article selector.
- Audit timeline.

Every component must support appropriate loading, empty, error, disabled, focus, hover, active, and responsive states. Keep visual conventions consistent across all three panels, while making staff-only controls unmistakably internal.

---

# 23. Phased implementation roadmap and acceptance criteria

## Phase 0 — Repository audit and architecture alignment

Deliverables:
- Repository inventory.
- Route map for the three panels.
- Existing authentication/authorization diagram.
- Existing database/domain model map.
- Existing design system/component inventory.
- Integration inventory.
- Gap/risk report against Admin and Customer/Support specifications.
- Approved implementation plan and decision log.

Exit criteria:
- No coding begins on major workflows until the owner can review the plan and unresolved blocking decisions are identified.

## Phase 1 — Customer storefront foundation (P0)

Build:
- Public layout/header/footer.
- Homepage.
- Explore/catalogue.
- Category pages.
- Search.
- Ebook detail.
- Public legal/help pages.
- Responsive/error/empty states.

Acceptance:
- Public pages render real published catalogue data.
- Search and filters work with server-side validation/pagination.
- Unpublished/protected ebooks are not exposed.
- SEO metadata/canonical/indexing rules are present.
- Mobile and accessibility checks pass.

## Phase 2 — Customer identity and account foundation (P0)

Build:
- Registration/login/logout.
- Email verification.
- Password recovery/change.
- Secure sessions.
- Customer overview.
- Profile/security.

Acceptance:
- Customer ownership is enforced on every protected route/API.
- Account enumeration and brute-force protections are tested.
- Session lifecycle is tested.
- No sensitive account data leaks into client logs or URLs.

## Phase 3 — Cart, checkout, payment and order lifecycle (P0)

Build:
- Cart.
- Checkout.
- Payment adapter integration.
- Payment result page.
- Order history/detail.
- Invoice/receipt where approved.
- Webhook verification/idempotency.

Acceptance:
- No order is considered paid from a frontend redirect alone.
- Duplicate clicks/webhooks do not create duplicate paid orders.
- Failed and pending paths are clear.
- Amount/currency/order mapping is server-verified.
- Gateway sandbox and failure tests pass.

## Phase 4 — Digital library and fulfilment (P0)

Build:
- Entitlement service integration.
- My Library.
- Library book details.
- Download authorization/delivery if enabled.
- Fulfilment retry/repair mechanism.

Acceptance:
- Paid eligible order items create correct entitlements exactly once.
- Customer can only access own valid entitlements.
- Protected original files are never public.
- Failed notification does not block access.

## Phase 5 — Reader and reading progress (P0/P1 subject to format decision)

Build:
- EPUB reader support based on chosen library/engine.
- PDF viewing if enabled.
- Reading settings.
- Reading progress persistence/resume.
- Reader entitlement validation.
- Accessibility testing per supported format.

Acceptance:
- Only entitled customer can open content.
- Customer can resume from saved location.
- Supported format limitations are documented.
- Keyboard/screen reader/zoom/mobile behaviour is tested.

## Phase 6 — Customer self-service (P1)

Build:
- Wishlist.
- Reviews.
- Notifications.
- Communication preferences.
- Privacy requests.
- Account sessions.

Acceptance:
- Customer can manage only own records.
- Marketing consent can be withdrawn.
- Reviews obey eligibility/moderation policy.
- Privacy requests create trackable workflows.

## Phase 7 — Support/Operations foundation (P0)

Build:
- Separate staff route/layout/client boundary.
- Staff authentication and role/permission enforcement.
- Support overview.
- My Queue/unassigned/all-ticket lists.
- Ticket details/conversation.
- Customer lookup (limited).
- Order lookup and investigation.
- Internal notes/public replies.

Acceptance:
- Customer accounts cannot open staff routes.
- Agents cannot read internal content they are not permitted to access.
- Internal notes never appear in customer APIs/emails.
- All staff actions are audited.

## Phase 8 — Support workflows and operations (P0/P1)

Build:
- Payment investigation.
- Entitlement/download investigation.
- Refund request and approval workflow.
- Escalation/handoff.
- Knowledge base/macros.
- Limited support reporting.

Acceptance:
- Staff cannot manually forge payment success.
- Refund approval and gateway completion are separate states.
- Privileged actions require correct permission/reason.
- Idempotent recovery actions do not duplicate entitlements/refunds.

## Phase 9 — Optimization and release hardening

Build/verify:
- Performance monitoring.
- Accessibility audit.
- SEO technical audit.
- Security testing and ASVS-aligned review.
- Backup/recovery integration verification.
- Rate-limit and abuse tests.
- Browser/device matrix.
- Error monitoring and alerting.
- User acceptance testing.

Exit criteria:
- No open critical/high security defects.
- All P0 acceptance criteria pass.
- Payment, entitlement, reader, download, and support workflows are tested end to end.
- Admin, Customer, and Support use the same canonical order/payment/entitlement state.

---

# 24. QA and test matrix

## 24.1 Customer access-control tests

- Visitor cannot read private customer pages.
- Customer A cannot read Customer B's order, invoice, ticket, entitlement, reader session, bookmark, or download.
- Changing route IDs/query IDs/request body IDs does not bypass ownership.
- Expired/revoked entitlement cannot open the reader or download file.
- A customer with a failed/pending payment cannot access the purchased file.
- Suspended/restricted account behaviour matches the approved policy.

## 24.2 Commerce/payment tests

- Cart price changes between add-to-cart and checkout.
- Expired/invalid coupon.
- Duplicate coupon use.
- Payment success webhook delivered once and multiple times.
- Invalid webhook signature.
- Valid signature but wrong amount/currency/order reference.
- Browser reports success but webhook/provider state is pending/failed.
- Gateway timeout and delayed result.
- Customer closes browser after paying.
- Payment succeeds but entitlement service temporarily fails.
- Email delivery fails after successful purchase.
- Refund approved but provider refund remains processing/fails.
- Partial refund, full refund, and repeat refund attempt.

## 24.3 Reader/download tests

- Valid EPUB 3.3 file.
- Invalid/corrupt EPUB.
- Unsupported fixed-layout EPUB behaviour.
- PDF with text layer.
- Scanned/image-only PDF limitations.
- Large file/resource-limit handling.
- Expired signed URL.
- Reused download token.
- Download limit reached.
- Customer tries another customer's asset identifier.
- Revocation while reader session is active.
- Cross-device progress conflict.
- Offline cache does not leak protected content between accounts.

## 24.4 Support authorization tests

- Customer cannot access staff workspace.
- Agent cannot access owner/admin-only pages or APIs.
- Agent cannot view another ticket outside scope without permission.
- Agent cannot perform refund approval if not permitted.
- Agent cannot mark order/payment paid.
- Internal note is never exposed to customer.
- Support attachment access validates ticket/role.
- Sensitive lookup/export actions are audited.
- Revoked staff session loses access promptly.

## 24.5 UX/accessibility/performance tests

- Keyboard-only navigation for key journeys.
- Screen-reader labels and reading order.
- Focus visibility and focus restoration after dialogs.
- Contrast and zoom/reflow.
- Reduced motion.
- Mobile viewport and touch targets.
- Checkout error recovery and preservation of entered values.
- LCP/INP/CLS field/lab measurements.
- Low-bandwidth image loading.
- Empty catalogue/library/order/ticket states.
- Network failure and retry states.

---

# 25. Final launch checklist

## Customer Panel

- [ ] Public catalogue, category, search, and ebook detail pages work with real data.
- [ ] Pricing and format/access disclosures are accurate.
- [ ] Cart and checkout are usable on mobile.
- [ ] Guest/account checkout policy has owner approval and is correctly implemented.
- [ ] Payment status is verified server-side.
- [ ] Order confirmation is authoritative.
- [ ] My Library displays only valid entitlements.
- [ ] Reader/download access checks entitlement server-side.
- [ ] Reading progress persists if reader is enabled.
- [ ] Account recovery and sessions are secure.
- [ ] Support request flow works and reveals only customer-visible messages.
- [ ] Privacy, terms, and refund policy pages are owner-approved.
- [ ] Accessibility and performance checks pass.

## Support/Operations Panel

- [ ] Staff authentication is distinct from customer access context.
- [ ] Role/permission model is enforced server-side.
- [ ] Ticket queues and assignment work.
- [ ] Public replies and internal notes are separate.
- [ ] Customer/order/payment/entitlement context is limited and accurate.
- [ ] Support cannot forge payment success.
- [ ] Refund workflow reflects provider-confirmed state.
- [ ] Access repair is idempotent and audited.
- [ ] Staff actions and exports are audited.
- [ ] Sensitive fields and secrets are masked.

## Cross-panel consistency

- [ ] Admin, Customer, and Support use the same customer identity.
- [ ] All panels use the same canonical order and payment state.
- [ ] All panels use the same entitlement state.
- [ ] Refund changes are reflected consistently.
- [ ] Product publication/access changes are reflected consistently.
- [ ] Notifications do not create inconsistent purchase truth.
- [ ] No extra portal/panel has been created outside the approved three-panel scope.

---

# 26. Agent handoff / mandatory first response

Before implementing, the coding agent must return:

1. **Repository assessment:** current framework, application boundaries, routes, domain modules, database, auth, integrations, existing tests, and reusable components.
2. **Three-panel route proposal:** exact proposed route groups for Customer, Admin, and Support; clearly show which routes already exist and which are new.
3. **Shared-domain diagram:** customer identity, product, cart, order, payment, refund, entitlement, reader progress, and support ticket relationships.
4. **Security boundary diagram:** visitor/customer/staff/admin roles, authentication context, and server-side permission checks.
5. **Decision register:** every unresolved item from Section 3, with recommendation and impact. Do not silently choose.
6. **Milestone plan:** implementation sequence, dependencies, migration strategy, test plan, rollback plan, and acceptance criteria.
7. **Risk list:** payment race conditions, entitlement duplication, file access, guest purchase claiming, refund/access coupling, customer data leakage, support privilege escalation, and reading-engine limitations.

Only after that assessment should implementation begin according to the approved milestones.

---

# 27. References and standards

The following sources inform the UX, accessibility, performance, digital publishing, and security requirements. The agent should re-check the live standards before a production release because technical specifications can be updated.

1. **Baymard Institute — Checkout UX: 10 Pitfalls and Best Practices (2026 update).** Research on checkout usability across desktop, mobile, and apps, including account selection, form design, payment, and error handling.  
   https://baymard.com/research-articles/current-state-of-checkout-ux
2. **Baymard Institute — Ecommerce Cart & Checkout Usability Research.** Research topics include guest checkout/account creation, payment flow, validation, field design, order review, and order confirmation.  
   https://baymard.com/research/checkout-usability
3. **W3C — Web Content Accessibility Guidelines (WCAG) 2.2.** Accessibility standard; target Level AA for core journeys.  
   https://www.w3.org/TR/WCAG22/
4. **Google Search Central — Understanding Core Web Vitals.** LCP, INP, and CLS definitions and good-experience thresholds.  
   https://developers.google.com/search/docs/appearance/core-web-vitals
5. **W3C — EPUB 3.3.** Digital publication format specification.  
   https://www.w3.org/TR/epub-33/
6. **W3C — EPUB Reading Systems 3.3.** Reading-system conformance requirements.  
   https://www.w3.org/TR/epub-rs-33/
7. **W3C — EPUB Accessibility 1.1.** Accessibility conformance and discoverability requirements for EPUB publications.  
   https://www.w3.org/TR/epub-a11y-11/
8. **OWASP — Application Security Verification Standard (ASVS) 5.0.0.** Security verification requirements for web applications and services.  
   https://asvs.owasp.org/
9. **OWASP — Authentication Cheat Sheet.** Authentication and session-security guidance.  
   https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
10. **OWASP — Authorization Cheat Sheet.** Least privilege, deny-by-default, and permission validation on every request.  
    https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
11. **OWASP — File Upload Cheat Sheet.** File validation, storage, malware scanning, safe naming, and upload threat guidance.  
    https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html
12. **OWASP — Email Validation and Verification Cheat Sheet.** Account verification, recovery-token, and account-enumeration protections.  
    https://cheatsheetseries.owasp.org/cheatsheets/Email_Validation_and_Verification_Cheat_Sheet.html
13. **MDN — Making Progressive Web Apps Installable.** PWA installability basics; relevant only if PWA is approved for a later phase.  
    https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable

---

**End of specification.**  
**Owner approval required:** business model, payment gateway, currency/tax policy, guest checkout/account claiming, supported ebook formats/access modes, refund/access policy, languages, support channels, offline reading, and data retention.
