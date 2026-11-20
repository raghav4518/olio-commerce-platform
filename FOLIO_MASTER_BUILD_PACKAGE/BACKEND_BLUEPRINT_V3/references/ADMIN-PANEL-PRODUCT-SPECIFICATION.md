# Ebook Commerce Platform — Admin Panel Product & Engineering Specification

**Document type:** Product Requirements Document (PRD) + Admin UX Specification + Functional Acceptance Criteria  
**Version:** 1.0  
**Status:** Draft for product approval and implementation planning  
**Scope:** Owner/admin web application for an independently operated ebook commerce platform  
**Primary audience:** Product owner, UX/UI designer, frontend agent, backend agent, QA agent, security reviewer  
**Implementation rule:** This document defines what must be built and how it must behave. It does not authorize an agent to invent unresolved business rules or silently remove requirements.

---

## 0. Instructions to the implementation agent

Read this document in full before changing code. Treat it as the source of truth for the admin panel unless the product owner explicitly approves a change.

### 0.1 Required working method

1. **Inspect before implementing.** Inspect the existing repository, framework, routes, design system, authentication, database schema, API conventions, tests, and deployment setup. Do not assume a greenfield project.
2. **Report the existing state.** Before implementation, produce a concise inventory of what already exists, what is reusable, what is incomplete, and what conflicts with this specification.
3. **Create an implementation plan.** Break work into small phases and page/module tickets. Identify dependencies, data-model changes, migrations, integrations, and risks.
4. **Do not guess unresolved business decisions.** Use the open-decision register in Section 4. Build configurable foundations where practical, but do not activate a business model or policy that the owner has not selected.
5. **Do not build decorative-only UI.** Every visible control must perform a real, permission-checked action, or it must be clearly marked as unavailable/not yet implemented and omitted from production navigation. No fake charts, fake metrics, placeholder success messages, or dead buttons.
6. **Enforce security on the server.** Hiding a menu item or button is not authorization. Every API/action must independently authenticate the user and enforce role/permission and object-level access.
7. **Preserve financial and customer records.** Do not hard-delete paid orders, payment attempts, refund records, invoices, audit records, or customer purchase entitlements. Use archive, anonymization, retention, or an explicitly approved legal deletion workflow.
8. **Use real state and consistent definitions.** All dashboard metrics, tables, charts and exports must be calculated from persisted data using documented definitions and a consistent timezone/date-range policy.
9. **Use safe defaults.** Destructive actions require confirmation; sensitive actions require re-authentication or step-up verification where appropriate; payment state must be provider-verified; file access must be private by default.
10. **Test each phase.** Add unit, integration, permission, workflow, regression and end-to-end tests. Do not claim completion merely because the page renders.
11. **Document deviations.** Any deviation from this specification must be recorded with: requirement ID, reason, impact, proposed alternative, migration implications, security implications, and owner approval status.
12. **Do not expose secrets.** Never commit credentials, API keys, private storage URLs, webhook secrets, signing keys, or production customer data.

### 0.2 Definition of “done”

A module is complete only when:

- Its pages, navigation, loading/empty/error/success states and responsive behavior are implemented.
- Its actions are connected to real backend operations.
- Input validation exists on both client and server.
- Access is enforced through permissions on the server.
- Changes that require audit are recorded.
- Data is paginated and filtered server-side where appropriate.
- Relevant tests pass, including unauthorized and invalid-state tests.
- Relevant documentation and environment variables are documented.
- No critical console errors, broken links, dead buttons, mock data in production paths, or unresolved security findings remain.

---

# 1. Product definition and scope

## 1.1 Product vision

Build a premium, reliable, owner-operated admin control centre for a digital bookstore that sells ebooks. The owner should be able to operate the business from the admin panel without routine direct database access or source-code changes.

The admin panel is responsible for managing:

- Ebook catalogue and digital files.
- Authors, categories, tags and curated collections.
- Customer accounts and customer support.
- Orders, payment records, refunds and invoices.
- Digital reading/download access and purchase entitlements.
- Coupons, campaigns and storefront content.
- Analytics and reports.
- Staff accounts, roles and permissions.
- Security, audit history, integrations and store settings.

## 1.2 Initial business-model assumption

The previously discussed design has not yet confirmed whether the business will be single-vendor, multi-vendor or hybrid.

**Safe planning assumption for the first implementation:** the platform owner is the seller and publishes the ebooks. Do not implement third-party seller onboarding, seller wallets, vendor commissions, vendor settlements or multi-vendor order splitting unless the owner explicitly approves a marketplace model.

The architecture may avoid decisions that make future marketplace expansion impossible, but do not add marketplace complexity to the initial release without approval.

## 1.3 Admin panel is not the storefront

This document specifies the administration application. The public storefront and customer account application are separate product surfaces. The admin panel may provide preview links and manage content shown on those surfaces, but the implementation agent must not treat an admin page as a substitute for the customer-facing page.

## 1.4 Primary users

- **Owner / Super Admin:** owns the store and has full administrative control.
- **Operations Manager:** oversees catalogue and order operations.
- **Catalog Manager:** creates and maintains ebooks and related metadata.
- **Finance Manager:** reviews payment transactions, refunds, invoices and financial reports.
- **Marketing Manager:** manages promotions, collections and campaign content.
- **Support Agent:** handles customer enquiries and support tickets.
- **Analyst:** read-only access to approved analytics and reports.

A single person may initially hold several roles. The system must nevertheless enforce permissions by user and action, not assume that every administrator is the owner.

## 1.5 Non-goals for initial release unless separately approved

- Third-party seller marketplace operations.
- Physical inventory, shipping, delivery tracking or warehouse management.
- Full accounting/ERP replacement.
- Native mobile applications.
- Fully unrestricted visual page builder with arbitrary code execution.
- Automatic AI-generated pricing or automatic refunds.
- DRM guarantees that claim to prevent all copying or screenshots.

---

# 2. Product principles

1. **Owner control:** routine business operations must be possible through the admin UI.
2. **Clarity over density:** show the most important information first; use tabs, filters and detail pages for complexity.
3. **Consistent interaction:** use the same page header, table, form, status, confirmation and notification patterns throughout.
4. **Data integrity:** preserve purchase, payment, refund, invoice and entitlement history.
5. **Secure by default:** private files, least-privilege access, verified payments and auditable sensitive actions.
6. **Real metrics:** never display fabricated metrics or silently mix different financial definitions.
7. **Recoverability:** drafts, version history, backups and safe rollback should exist where appropriate.
8. **Accessible and responsive:** desktop-first admin UX must still be usable on tablet and mobile for essential tasks.
9. **Operational transparency:** display meaningful status, timestamps, processing history and actionable error explanations.
10. **Extensibility without premature complexity:** use clear modules and boundaries; do not split into microservices without a demonstrated need.

---

# 3. Global application shell and navigation

## 3.1 Layout

Use a persistent application shell:

- Left sidebar with grouped, collapsible navigation.
- Top bar with global search, quick-create menu, notifications, help, storefront preview and administrator profile.
- Main content workspace.
- Breadcrumbs on nested pages.
- Optional contextual right drawer for quick inspection or editing.

The sidebar and top bar should remain consistent across all admin routes.

## 3.2 Proposed sidebar structure

1. **Overview**
   - Dashboard
   - Activity feed
   - Alerts
2. **Catalog Management**
   - All ebooks
   - Add new ebook
   - Categories
   - Authors
   - Collections
   - Digital files
   - Reviews
3. **Orders & Commerce**
   - All orders
   - Refunds
   - Abandoned carts (only if cart tracking is implemented)
   - Order settings
4. **Customers**
   - All customers
   - Customer segments
   - Customer groups
5. **Payments & Finance**
   - Transactions
   - Payment methods / gateways
   - Refund management
   - Invoices / credit notes
   - Tax configuration
   - Financial reports
6. **Marketing & Promotions**
   - Coupons
   - Discount rules
   - Campaigns
   - Affiliate management (later / disabled unless approved)
   - Referral programs (later / disabled unless approved)
   - Email campaigns (later)
7. **Website & Content**
   - Homepage builder
   - Banners
   - Navigation menus
   - Landing pages
   - Static pages
   - Blog / editorial content (optional)
   - SEO settings
   - Footer settings
8. **Analytics & Reports**
   - Analytics overview
   - Sales analytics
   - Revenue analytics
   - Product analytics
   - Customer analytics
   - Conversion analytics
   - Download analytics
   - Reading analytics (only if reader tracking is implemented and disclosed)
9. **Customer Support**
   - Support tickets
   - Contact messages
   - FAQs
   - Support settings
10. **Notifications & Communication**
    - Email templates
    - Notification templates
    - Delivery logs
    - Notification preferences
11. **Team & Permissions**
    - Admin users
    - Roles and permissions
    - Login sessions
12. **Security & Audit**
    - Audit logs
    - Login history
    - Security alerts
    - Access policies
    - API keys
13. **Store Settings**
    - General
    - Branding
    - Domain
    - Localization
    - Currency
    - Checkout
    - Payments
    - Digital delivery
    - File storage
    - Taxes
    - Customer accounts
    - Email
    - Integrations
    - SEO
    - Backup and recovery
    - Maintenance mode

### Navigation rules

- Only show modules the logged-in admin is authorized to access.
- Hiding a menu item is a UX behavior, not a security mechanism.
- Keep the active page and expanded sidebar group visually clear.
- On narrow screens, use an accessible slide-over navigation drawer.
- Show a badge/count only when it is meaningful (e.g. pending tickets), and do not display misleading stale counts.
- Every sidebar item must navigate to a real implemented route. Do not ship placeholder routes that lead to blank pages.

## 3.3 Top bar

Required elements:

- **Global search:** search ebooks, authors, categories, customers, orders and tickets according to permission. Results must be permission-filtered.
- **Quick Create:** shortcuts for new ebook, category, author, collection, coupon and campaign, limited by permission.
- **Notifications:** payment failures, suspicious events, pending refund approvals, support ticket assignments and operational warnings.
- **Help:** internal documentation/help links.
- **Storefront preview:** open the public site in a new tab, preferably with a preview/draft token when previewing unpublished content.
- **Admin profile:** profile, security settings, active sessions and logout.
- **Command palette:** keyboard shortcut `Ctrl+K` / `⌘K` where supported, with navigation and permitted actions.

## 3.4 Global page template

All list/management pages should use a consistent structure:

1. Breadcrumb.
2. Page title and concise explanation.
3. Primary action (e.g. Add ebook).
4. Optional secondary actions (Import, Export, Preview).
5. KPI summary where meaningful.
6. Search and filters.
7. Data table or card/list view.
8. Pagination and page-size selector.
9. Empty, loading, error and no-results states.

All detail pages should use:

1. Breadcrumb and record title/identifier.
2. Status badge and primary actions.
3. Summary panel.
4. Tabs or clearly separated sections.
5. Activity/audit timeline where relevant.
6. Related records.

All forms must provide:

- Visible labels and helpful descriptions.
- Required/optional indicators.
- Inline validation and server validation.
- Unsaved-change warning when leaving a dirty form.
- Clear Save draft / Save changes / Publish actions as applicable.
- A meaningful success or failure message.
- Disabled/loading states during requests to prevent duplicate submissions.

## 3.5 Design system requirements

The exact visual design is to be finalized separately, but the implementation must establish reusable tokens/components for:

- Typography scale and font weights.
- Spacing scale.
- Surface/background, border, text and focus colors.
- Primary, secondary, destructive and quiet buttons.
- Status badges (draft, published, pending, paid, failed, refunded, archived, suspended, etc.).
- KPI cards.
- Data tables.
- Search/filter controls.
- Date range selector.
- Tabs.
- Drawers and dialogs.
- Toasts/alerts.
- Form fields, uploaders and rich-text editor.
- Charts with accessible labels and tooltips.
- Empty/loading/error states.

Provide light and dark themes only if the owner confirms both are wanted. Do not build two themes as a substitute for a coherent design system.

---

# 4. Unresolved product decisions — do not guess

The owner has not yet selected answers to the following questions. The agent must keep them visible in planning and must not quietly treat recommendations as approved decisions.

| ID | Decision | Recommended default for planning | Status |
|---|---|---|---|
| DEC-01 | Business model: single-vendor, marketplace or hybrid | Single-vendor for Release 1; keep future extension possible | Unconfirmed |
| DEC-02 | Dashboard complexity | Advanced dashboard with configurable widgets | Unconfirmed |
| DEC-03 | Theme | Light and dark themes with user preference | Unconfirmed |
| DEC-04 | Ebook publishing workflow | Owner can publish directly; configurable approval workflow for staff roles | Unconfirmed |
| DEC-05 | Customer registration | Both self-registration and admin-created accounts | Unconfirmed |
| DEC-06 | Ebook access | Online reader and downloads, subject to file format support and access rules | Unconfirmed |
| DEC-07 | Initial payment provider | Select after business/legal/technical review; Razorpay is a candidate, not a confirmed choice | Unconfirmed |
| DEC-08 | Initial currencies | INR as launch currency is a likely assumption, but must be approved | Unconfirmed |
| DEC-09 | Tax configuration | Configurable tax engine; exact Indian tax treatment reviewed by qualified tax professional | Unconfirmed |
| DEC-10 | Support channels | Admin ticketing plus email; live chat not assumed | Unconfirmed |
| DEC-11 | Blog/editorial module | Optional Release 2 | Unconfirmed |
| DEC-12 | Affiliates/referrals/subscriptions | Exclude from initial release; decide later | Unconfirmed |
| DEC-13 | Staff roles | Start with Owner and staff roles listed in this document; owner approves final role matrix | Unconfirmed |
| DEC-14 | Refund policy | Configurable policy; owner approves customer-facing rules and staff approval thresholds | Unconfirmed |
| DEC-15 | File formats | PDF and EPUB are initial candidates; validate actual reader/storage support | Unconfirmed |
| DEC-16 | Watermarking | Optional configurable feature; do not market as complete DRM | Unconfirmed |

**Rule:** If a decision blocks implementation, create a decision request and continue with independent work. Do not ask the same question repeatedly if its answer is already documented. If no answer is available, implement only a reversible, configurable foundation and clearly mark the feature as pending approval.

---

# 5. Module 01 — Overview dashboard

## 5.1 Purpose

Give the owner an accurate, actionable overview of store performance and operational issues.

## 5.2 Page: Dashboard

### Header

- Greeting and admin name.
- Date range: Today, yesterday, last 7 days, last 30 days, last 90 days, month-to-date, year-to-date and custom range.
- Compare-to-previous-period toggle where meaningful.
- Export dashboard/report action.
- Quick Create menu.

### KPI cards

- Gross sales.
- Net sales/revenue (precise definition must be documented and consistent).
- Paid orders.
- Average order value.
- Registered customers.
- Published ebooks.
- Refund value.
- Payment success rate.

### Charts and panels

- Revenue trend: daily/weekly/monthly granularity based on selected period.
- Orders trend.
- Sales by category.
- Best-selling ebooks by units and revenue.
- Recent orders.
- Recent customers.
- Pending refund requests.
- Failed or delayed payment alerts.
- Unresolved support tickets.
- Ebook publication/draft activity.
- Operational alerts (storage threshold, email delivery issue, webhook failures, backup failures, abnormal download activity).

### Dashboard behavior

- Widgets must use actual API data and support loading/empty/error states.
- Date filtering must apply consistently to all time-based widgets.
- Counts such as total customers/published ebooks must clearly indicate whether they are current totals or period-specific additions.
- Charts must be accessible without relying on color alone.
- Clicking a KPI or chart element should navigate to a relevant filtered report/list page.
- Provide a way to hide/reorder widgets only if the feature is implemented fully and preference is persisted.

## 5.3 Dashboard metric definitions

Do not use ambiguous labels without definitions. At minimum:

- **Paid orders:** orders with a verified successful payment, excluding test orders and voided transactions according to the finalized order policy.
- **Gross sales:** sum of eligible item/subtotal amounts before refunds; clearly state treatment of discounts, taxes and fees.
- **Refunds:** sum of successfully completed refund transactions, not merely requested refunds.
- **Net sales:** gross sales minus completed refunds and any other explicitly defined adjustments. Taxes and payment gateway fees must be separately reported unless the owner approves another accounting definition.
- **Average order value:** gross eligible paid order amount divided by eligible paid order count, with treatment of refunds documented.
- **Payment success rate:** successful payment attempts divided by eligible payment attempts for the selected period; exclude or separately report abandoned/unsubmitted attempts.
- **Product sales:** distinguish units sold from number of orders containing the product.

Use the store's configured reporting timezone for date grouping. Store timestamps in UTC and convert for display/reporting.

---

# 6. Module 02 — Catalog management

## 6.1 Page: All ebooks

### Header and actions

- Page title: Ebooks.
- Total count and status counts.
- Add New Ebook.
- Import (only when supported by a validated import format).
- Export (respect permissions and privacy rules).

### Search and filters

- Keyword search: title, subtitle, product identifier/ISBN, author.
- Category and subcategory.
- Author.
- Status: draft, review/pending approval, scheduled, published, archived.
- Format: PDF, EPUB, other supported format.
- Price range.
- Created/updated/published date.
- Sales range.

### Table columns

- Selection checkbox.
- Cover thumbnail.
- Ebook title and unique product ID.
- Author.
- Category.
- Current price and sale price if applicable.
- Units sold.
- Revenue (permission-restricted if necessary).
- Status.
- Last updated.
- Row actions.

### Row actions

- View/preview.
- Edit.
- Duplicate.
- Publish/unpublish (permission and workflow dependent).
- Archive.
- View sales.
- Delete only if safe and no historical/financial/entitlement dependency exists. Prefer archive over deletion.

### Bulk actions

- Change category.
- Change status where permitted.
- Archive.
- Export selected.
- Delete only after dependency validation and explicit confirmation.

### Requirements

- Server-side search, filtering, sorting and pagination.
- Stable sorting and query parameters preserved on navigation where practical.
- Do not load the entire catalogue into the browser.
- Bulk actions must report per-record success/failure; a partial failure must not be presented as total success.

## 6.2 Page: Add new ebook — multi-step wizard

The form should be divided into steps. Preserve draft data between steps and allow safe continuation later.

### Step 1 — Basic information

Fields:

- Title (required).
- Subtitle (optional).
- Short description.
- Full description (sanitized rich text/Markdown; no arbitrary executable HTML).
- Primary author (required, unless owner approves another policy).
- Additional contributors (optional).
- Language.
- ISBN or internal identifier (optional, unique where provided).
- Publication date (optional).
- Edition/version label (optional).
- Publisher/imprint (optional).
- Product status: draft by default.

Validation:

- Trim whitespace.
- Enforce reasonable length limits.
- Prevent duplicate identifiers where uniqueness is required.
- Sanitize rich text on the server.

### Step 2 — Cover and digital files

Fields/actions:

- Cover image upload.
- Optional alternate/social image.
- Main ebook file upload (required before publication if product is to be delivered as a file).
- Supported format indicator.
- Optional sample/preview file or preview pages.
- File version label.
- File size and detected MIME type.
- Upload progress and retry handling.

Security and validation:

- Validate extension, MIME type and actual file signature where feasible.
- Enforce configurable file-size limits.
- Scan uploads for malware using an appropriate service/tool before making them available.
- Store original files in private object storage, never a public web directory.
- Do not expose permanent storage paths or bucket keys to the browser.
- Generate cover thumbnails/previews separately from original protected files.
- Reject executable content and unsupported archive types unless explicitly approved.
- Use resumable/retryable uploads for large files where supported.

### Step 3 — Classification

Fields:

- Primary category (required).
- Secondary categories (optional).
- Subcategories.
- Tags/keywords.
- Genre/subject labels.
- Curated collections.
- Related/recommended ebooks.

### Step 4 — Pricing and offers

Fields:

- Base price.
- Sale price (optional).
- Sale start/end time (optional).
- Currency (must follow store currency policy).
- Tax class or tax category (if tax engine configured).
- Coupon eligibility.
- Bundle eligibility.
- Optional compare-at/reference price only if legally and accurately represented.

Validation:

- No negative values.
- Sale price must be less than base price unless a documented business rule permits otherwise.
- Sale end must be later than sale start.
- Currency must be supported.
- Price must use decimal-safe monetary representation; do not use binary floating-point for stored financial amounts.

### Step 5 — Digital access rules

Configurable fields:

- Online reading enabled/disabled.
- Download enabled/disabled.
- Download limit (optional; define whether it is lifetime, per file version or per entitlement).
- Access duration/expiry (optional; default should not expire without an approved business policy).
- Watermarking enabled/disabled (if implemented).
- Preview availability.
- Number of allowed devices (only if device model is implemented and clearly described).
- File revision policy for existing buyers.

Do not claim that watermarking prevents copying. Provide honest descriptions of protection limits.

### Step 6 — SEO and discovery

Fields:

- URL slug (unique, editable with redirect creation if changed after publication).
- SEO title.
- Meta description.
- Social sharing image.
- Search keywords.
- Indexing preference, subject to platform SEO rules.

### Step 7 — Preview and publish

Show a storefront preview including:

- Cover.
- Title/subtitle.
- Author.
- Description.
- Price/discount.
- Category and metadata.
- Sample preview action if enabled.
- Availability/access labels.

Actions:

- Save draft.
- Submit for review (only if approval workflow enabled).
- Publish now (only if permission and approval policy permit).
- Schedule publication (only if scheduler exists and has been tested).

Publication checklist:

- Required metadata complete.
- Valid cover.
- Required digital file present and scanned.
- Valid price/currency.
- At least one category.
- Slug unique.
- Access configuration valid.
- No unresolved validation errors.

## 6.3 Page: Ebook details/edit workspace

Tabs:

1. **Overview:** status, product identifiers, current price, sales summary, created/updated/published dates.
2. **Content:** title, description, cover, author, metadata.
3. **Digital files:** file list, versions, upload/replace, file integrity/status.
4. **Pricing:** base/sale pricing, schedule, tax class.
5. **Access:** reader/download policy, preview, access duration, limits.
6. **SEO:** slug, metadata, social image, redirects.
7. **Sales:** units, orders, revenue, refunds related to this ebook.
8. **Reviews:** ratings and moderation.
9. **Activity:** audit history and publication changes.

File replacement must not silently invalidate existing buyers. A policy must define whether an earlier buyer receives the new file version, retains the old version, or receives a controlled update. Until approved, preserve existing buyer entitlement and record the file version purchased/available.

## 6.4 Page: Categories

List view:

- Name.
- Parent category.
- Slug.
- Number of active ebooks.
- Display order.
- Status.
- Updated date.
- Actions.

Create/edit fields:

- Name (required).
- Slug (unique).
- Description.
- Parent category.
- Thumbnail/banner.
- Display order.
- SEO title/meta description.
- Active/inactive.

Rules:

- Support a controlled hierarchy.
- Prevent a category from being its own parent or descendant.
- Warn when deleting/archiving a category with products or child categories.
- Provide reassignment or deactivation flow before removal.
- Category URLs must remain stable; create redirects when a published slug changes.

## 6.5 Page: Authors

List columns:

- Profile image.
- Name.
- Number of associated ebooks.
- Units/revenue (permission restricted).
- Status.
- Updated date.

Fields:

- Full name (required).
- Biography.
- Profile photo.
- Website/social links.
- Optional country/location.
- Associated ebooks.
- SEO fields.
- Visibility status.

Rules:

- Do not assume an author is a vendor or payee.
- Removing an author must not delete associated ebooks; require reassignment or preserve an archived author record.

## 6.6 Page: Collections

Collections are curated storefront groupings, distinct from categories.

Fields:

- Collection name and slug.
- Description.
- Cover/banner.
- Manual product selection or rule-based selection.
- Display order.
- Start/end schedule.
- Active/inactive.
- SEO metadata.

Rule-based collections must show the rule and a preview of matching products before publication. Avoid hidden rules that make products appear/disappear unexpectedly.

## 6.7 Page: Digital file library

Provide a searchable view of uploaded assets and their relationships to ebooks.

Show:

- File display name.
- Associated ebook/product.
- File type and size.
- Version.
- Upload status and malware scan status.
- Storage status.
- Uploaded by/date.
- Integrity checksum where supported.

Actions:

- View metadata.
- Retry failed processing.
- Replace file through a versioned workflow.
- Quarantine/remove an unsafe file.

Never provide an unrestricted public file URL. File deletion must check product/entitlement dependencies.

## 6.8 Page: Reviews and ratings

List:

- Rating.
- Review title/body.
- Ebook.
- Customer.
- Submission date.
- Moderation status.

Actions:

- Approve/publish.
- Reject/hide.
- Flag for review.
- Respond if public responses are supported.

Rules:

- Do not edit a customer's review content as though the customer wrote the revised content.
- Record moderation reason and admin identity.
- Define whether only verified purchasers may review; owner must approve policy.
- Provide abuse/spam reporting and moderation audit history.

---

# 7. Module 03 — Orders and commerce

## 7.1 Page: All orders

Filters:

- Order ID.
- Customer name/email.
- Date range.
- Payment status.
- Order/fulfilment status.
- Amount range.
- Product/ebook.
- Payment provider/method.

Columns:

- Order ID.
- Customer.
- Item count.
- Subtotal.
- Discount.
- Tax.
- Total.
- Payment status.
- Order status.
- Created date.
- Actions.

Requirements:

- Order list and payment list are separate concepts.
- One order may have multiple payment attempts and multiple partial refunds.
- Do not infer payment success from a browser redirect or client-side checkout result.
- Provide server-side pagination, filtering and export.

## 7.2 Page: Order details

Sections:

1. Order summary and current status.
2. Customer details.
3. Purchased items with item-level price, discount, tax and quantity.
4. Totals breakdown.
5. Payment attempts and provider references.
6. Refunds/credit notes.
7. Digital fulfilment and entitlement status.
8. Customer invoice/receipt.
9. Internal notes.
10. Event timeline.

Admin actions, permission-controlled:

- Resend order confirmation/receipt.
- Resend access instructions.
- View customer profile.
- Start refund flow.
- Review/reconcile payment status.
- Suspend/reinstate a digital entitlement with required reason and audit entry.
- Add internal note.

Do not provide a generic “mark as paid” action. If manual payment reconciliation is required, it must be a separate controlled process with verified evidence, restricted permissions, reason, approval policy and immutable audit event.

## 7.3 Page: Order settings

Configurable settings, subject to owner approval:

- Order ID format/prefix (only if safe to change).
- Guest checkout policy.
- Account creation behavior.
- Confirmation email behavior.
- Order status transitions.
- Test-order mode (must be clearly separated from live commerce).

## 7.4 Page: Abandoned carts

Only include if the storefront persists cart sessions and the required consent/privacy policy is approved.

Show:

- Cart/customer/session identifier.
- Cart contents and estimated amount.
- Last activity.
- Recovery status.
- Campaign/contact status.

Do not expose anonymous session data beyond its intended purpose. Do not email customers without an approved consent/legal basis and compliant unsubscribe handling.

---

# 8. Module 04 — Customers

## 8.1 Page: All customers

KPI cards:

- Total registered customers.
- New customers in selected period.
- Customers with at least one paid order.
- Returning purchasers.
- Suspended accounts.

Filters:

- Name/email/customer ID.
- Registration date.
- Account status.
- Paid order count.
- Lifetime spend.
- Last active date.
- Customer segment.

Columns:

- Customer ID.
- Name.
- Email.
- Paid orders.
- Lifetime spend.
- Purchased ebook count.
- Registration date.
- Last activity.
- Status.

## 8.2 Page: Customer profile

Header:

- Name, verified email status and account status.
- Customer ID.
- Joined date.
- Actions allowed by permission.

Tabs:

1. Overview.
2. Orders.
3. Library/entitlements.
4. Download history.
5. Reading activity (only where enabled, necessary and disclosed).
6. Support tickets.
7. Activity.
8. Security/session information (restricted).

Overview metrics:

- Paid order count.
- Lifetime spend using documented definition.
- Refund total.
- Number of active ebook entitlements.
- Last purchase date.

Admin actions:

- Edit permitted profile fields.
- Suspend/reactivate account with reason.
- Initiate secure password reset (never view/change the customer's existing password).
- View orders and entitlements.
- Add internal note.
- Handle data export/deletion requests through the approved privacy workflow.

## 8.3 Customer segments

Allow saved rule-based segments. Example rule:

`Paid order count >= 3 AND lifetime paid spend >= configured amount`.

Segment builder requirements:

- Clear AND/OR logic.
- Preview count before saving.
- Save/edit/archive segment.
- Show why a customer qualifies where feasible.
- Segment membership updates based on current data.
- Marketing use must obey consent, unsubscribe and applicable privacy requirements.

## 8.4 Customer groups

If customer groups are implemented separately from segments, define the difference:

- **Groups:** manually assigned or administratively managed labels/tiers.
- **Segments:** dynamic groups based on rules.

Do not implement two overlapping features with indistinguishable behavior.

---

# 9. Module 05 — Payments and finance

## 9.1 Page: Transactions

Transaction types:

- Payment attempt.
- Successful payment/capture.
- Failed payment.
- Void/reversal where provider supports it.
- Refund.
- Chargeback/dispute where provider supports it.
- Settlement/reconciliation record where available.

Columns:

- Internal transaction ID.
- Provider transaction/payment ID.
- Order ID.
- Customer.
- Amount/currency.
- Method.
- Provider.
- Transaction type.
- Status.
- Created/updated timestamp.

Filters:

- Date range.
- Provider.
- Method.
- Type.
- Status.
- Order/customer reference.
- Amount range.

Requirements:

- Keep payment attempts distinct from orders.
- Store provider responses safely; redact sensitive data.
- Verify webhook signatures.
- Make webhook processing idempotent.
- Reconcile asynchronous payment states.
- Preserve original transaction records even when refunds occur.

## 9.2 Page: Payment methods / gateways

For each configured provider:

- Provider name and environment (test/live).
- Enabled/disabled state.
- Supported methods/currencies.
- Connection/health status.
- Last webhook received.
- Last reconciliation status.
- Test connection action where supported.
- Credential setup/update flow.

Security:

- Secrets must be encrypted at rest using an appropriate secrets-management mechanism.
- Show masked credentials only; never return full secrets to frontend after saving.
- Restrict configuration changes to owner/approved finance/admin permission.
- Require re-authentication/step-up confirmation for sensitive changes.
- Never log secrets.

## 9.3 Page: Refund management

Statuses:

- Requested.
- Under review.
- Approved.
- Rejected.
- Submitted to gateway.
- Processing.
- Succeeded.
- Failed.
- Cancelled where provider supports it.

List filters:

- Refund ID.
- Order ID.
- Customer.
- Amount.
- Request date.
- Status.
- Reason.

Refund detail:

- Original order and payment.
- Eligible refundable amount.
- Previously refunded amount.
- Requested amount.
- Reason and evidence.
- Staff requester/reviewer.
- Gateway response/reference.
- Entitlement impact.
- Complete status timeline.

Workflow:

1. Verify that the original payment is eligible for refund.
2. Calculate remaining refundable amount.
3. Validate full/partial refund rules.
4. Check staff permission and approval threshold.
5. Confirm the request and present its consequences.
6. Submit using an idempotency key.
7. Record provider response.
8. Update status only from a verified provider result or controlled reconciliation.
9. Apply the owner-approved digital access policy.
10. Create audit entry and notify the appropriate parties.

Do not automatically revoke ebook access on refund unless the owner-approved refund policy says to do so. The entitlement change must be explicit, logged and explainable.

## 9.4 Page: Invoices and credit notes

Features:

- Invoice sequence/numbering configuration.
- Store legal/business details.
- Customer billing details as collected.
- Tax breakdown.
- PDF generation/download.
- Resend invoice.
- Credit note for eligible refund/adjustment.
- Invoice history.

Do not silently regenerate an issued invoice with changed values. Use corrected/replacement documents or credit-note workflows consistent with the approved accounting policy.

## 9.5 Page: Tax configuration

Features:

- Configurable tax regions/rules.
- Product tax classes/categories.
- Inclusive/exclusive price behavior.
- Tax rates and effective dates.
- Customer tax/billing fields where required.
- Tax reports.

The product owner must obtain qualified tax advice for applicable Indian tax/GST treatment. The software must support configuration and traceability, not invent legal conclusions.

## 9.6 Page: Financial reports

- Sales summary.
- Tax summary.
- Refund summary.
- Gateway transaction summary.
- Reconciliation/settlement report where supported.
- Coupon/discount impact.
- Export to CSV/XLSX/PDF only when implemented and permission-controlled.

Clearly distinguish sales revenue, taxes collected, discounts, refunds, provider fees, settlements and cash received. Do not label gross sales as profit.

---

# 10. Module 06 — Marketing and promotions

## 10.1 Page: Coupons

Fields:

- Coupon code (unique; case/whitespace normalization defined).
- Name/internal description.
- Discount type: percentage or fixed amount.
- Discount value.
- Maximum discount for percentage coupons.
- Minimum cart subtotal.
- Start/end timestamps.
- Total use limit.
- Per-customer use limit.
- Applicable ebooks/categories/collections.
- Excluded products/categories.
- Customer eligibility: all, selected segment, selected group, first purchase, etc. (only supported conditions).
- Stackability/combination rules.
- Active/inactive state.

Validation:

- Discount value cannot be negative or invalid.
- Percentage cannot exceed approved maximum (normally 100%).
- Fixed discount cannot exceed eligible subtotal unless explicit rule supports it.
- End date must be after start date.
- Limits must be positive integers where provided.
- Validate eligibility again on the server at checkout; never trust the frontend cart calculation.

Coupon list should show code, type/value, usage, date range, status, and actions.

## 10.2 Page: Discount rules

Provide rules for automated discounts separate from coupon codes. Rules should have:

- Priority/order.
- Eligibility conditions.
- Applicable products/categories.
- Discount action.
- Start/end time.
- Combination behavior.
- Active state.
- Conflict/preview explanation.

Do not allow conflicting rules to produce unpredictable totals. Define deterministic precedence and test it.

## 10.3 Page: Campaigns

Fields:

- Campaign name and internal objective.
- Start/end schedule and timezone.
- Target audience/segment.
- Included ebooks/collections.
- Landing page.
- Banner/content creative.
- Coupon linkage (optional).
- Email/notification linkage (only if approved and implemented).
- Status: draft, scheduled, active, completed, cancelled.

Campaign detail should show:

- Configuration.
- Preview.
- Schedule and status.
- Sales/orders attributed to campaign where tracking is reliable.
- Campaign performance report.

## 10.4 Affiliate and referral programs — later/optional

Do not build into Release 1 unless the owner confirms. Future requirements may include:

- Affiliate applications and review.
- Unique tracking links/codes.
- Attribution windows and rules.
- Commission calculation and reversal rules.
- Fraud/self-referral checks.
- Payout records/approval.
- Referral rewards and limits.

If built, keep affiliate promotion and marketplace seller functionality separate. Affiliates promote products; they are not automatically product owners or payees under the marketplace model.

## 10.5 Email campaigns — later/optional

Requires:

- Consent/eligibility rules.
- Audience selection.
- Template preview.
- Test send.
- Schedule/cancel.
- Unsubscribe handling.
- Bounce/complaint suppression.
- Delivery logs and campaign reporting.

Do not send marketing messages to all customers by default.

---

# 11. Module 07 — Website and content management

## 11.1 Homepage builder

Use a structured section-based builder for Release 1/2, not arbitrary code injection.

Supported section types should be explicitly defined, such as:

- Hero banner.
- Featured categories.
- Bestseller carousel/grid.
- New arrivals.
- Editor's picks.
- Curated collection.
- Promotional offer.
- Testimonials/reviews (moderated source only).
- Newsletter signup (only with consent handling).
- Text/image editorial section.

For each section:

- Enable/disable.
- Reorder.
- Edit heading/subheading.
- Select product/category/collection source.
- Configure layout and item limit within approved design tokens.
- Configure CTA/destination.
- Configure desktop/mobile image if relevant.
- Schedule/publish where scheduler exists.

Builder requirements:

- Draft and published versions.
- Preview in desktop/tablet/mobile sizes.
- Save without publishing.
- Publish confirmation.
- Validation for broken/missing product references.
- Version history and rollback for published changes where feasible.
- No arbitrary JavaScript or unsafe HTML input.

## 11.2 Banner management

Fields:

- Internal banner name.
- Desktop image.
- Mobile image (optional).
- Heading and subtitle.
- CTA label and destination.
- Placement/position.
- Start/end schedule.
- Alt text/accessibility description.
- Active/inactive status.

Validate destination URLs and prevent unsafe schemes. Show preview before publication.

## 11.3 Static page management

Pages may include About, Contact, Privacy Policy, Terms, Refund Policy, FAQ and custom informational pages.

Fields:

- Title.
- Slug.
- Content (sanitized editor).
- SEO title/meta description.
- Social image.
- Draft/published status.
- Publish schedule if supported.
- Revision/version history.

Legal pages must not be auto-written or silently changed by the software agent. The owner should provide/approve final legal wording.

## 11.4 Navigation and footer management

- Main menu items and nesting.
- Footer columns/links.
- Social links.
- Legal links.
- Custom destinations.
- Visibility and display order.
- Preview.

Prevent circular menu nesting, invalid destinations and links to unpublished content unless preview is intended.

## 11.5 Landing pages

Optional campaign/SEO landing pages using approved templates and content sections. Define template limits and avoid custom-code execution.

## 11.6 Blog/editorial content — optional Release 2

- Posts, categories, tags and author attribution.
- Draft/review/published/scheduled status.
- Featured image and alt text.
- Slug and SEO metadata.
- Preview and revision history.
- Related ebooks/collections.

---

# 12. Module 08 — Analytics and reports

## 12.1 General behavior

Every analytics page should have:

- Date range selector.
- Comparison period toggle where valid.
- Relevant filters.
- Clear metric definitions.
- Charts and tables with accessible labels.
- Export action, permission-controlled.
- Last data refresh timestamp.
- Loading, empty and error states.

Use the reporting timezone configured for the store. Financial amounts must retain currency precision. Do not combine currencies in totals unless converted with a documented, timestamped exchange-rate methodology.

## 12.2 Sales analytics

Metrics:

- Paid orders.
- Units sold.
- Gross order value.
- Discounts.
- Refunds.
- Net sales (definition shown).
- Average order value.
- Orders by time, category and product.

## 12.3 Revenue analytics

Show a financial bridge:

- Item subtotal.
- Discounts.
- Tax collected.
- Payment gateway fees (if available).
- Successful refunds.
- Net sales/revenue according to an approved definition.
- Settled amount where provider data supports it.

Never call sales revenue “profit” unless cost and expense inputs have been implemented and the formula is explicit.

## 12.4 Product analytics

- Top products by units.
- Top products by revenue.
- Low-selling products.
- Product page views (if analytics tracking is implemented).
- View-to-purchase conversion (with clearly defined attribution).
- Category performance.
- Frequently purchased together (requires enough data and a defined method).

## 12.5 Customer analytics

- New vs returning purchasing customers.
- Customer purchase frequency.
- Repeat purchase rate.
- Customer lifetime value using a documented formula/time window.
- Cohort/retention analysis if sufficient history exists.
- Customer segment performance.

## 12.6 Conversion analytics

Only implement metrics if events are collected reliably and privacy obligations are met:

- Store visits/sessions.
- Product page views.
- Add-to-cart.
- Checkout started.
- Payment initiated.
- Purchase completed.

Clearly define funnel denominators, sessionization and bot filtering. Do not invent visitor counts when only order data is available.

## 12.7 Download analytics

- Successful download events.
- Failed/expired access attempts.
- Per-product download counts.
- Per-entitlement counts.
- Suspicious patterns/threshold alerts.

Store only the minimum event data necessary. Avoid collecting unnecessary device fingerprints or sensitive browsing data.

## 12.8 Reading analytics

Optional and subject to privacy disclosure. Possible aggregated metrics:

- Reader opens.
- Reading sessions.
- Aggregate progress distribution.
- Completion estimate if a defensible definition exists.

Do not expose detailed personal reading behavior to staff without explicit need and permission. Do not use reading analytics as an excuse for invasive surveillance.

---

# 13. Module 09 — Customer support

## 13.1 Support ticket list

Columns:

- Ticket ID.
- Customer.
- Subject.
- Category.
- Priority.
- Assigned agent.
- Status.
- Last activity.
- Created date.

Filters:

- Status: new, open, awaiting customer, awaiting internal, resolved, closed.
- Priority.
- Category.
- Assigned agent.
- Date range.

## 13.2 Ticket detail

- Conversation thread with sender and timestamps.
- Customer context and related orders.
- Attachments with safe file handling.
- Internal notes clearly distinguished from customer-visible replies.
- Assignee and priority.
- Status transitions.
- Response editor.
- Resolution summary.
- Audit timeline.

Never accidentally expose internal notes to customers. Validate attachment types and scan uploads. Record outbound delivery status where supported.

## 13.3 Contact messages and FAQ

- Contact form submissions list/detail.
- Spam/report status.
- Assignment and resolution.
- FAQ create/edit/reorder/publish.
- Public support information/settings.

---

# 14. Module 10 — Notifications and communication

## 14.1 Template manager

Initial templates:

- Account welcome/registration.
- Email verification.
- Password reset.
- Order confirmation.
- Payment success.
- Payment failure.
- Ebook access instructions.
- Refund requested/approved/failed/completed.
- Support ticket received/replied/resolved.
- Promotional message (only with approved marketing consent).

Template fields:

- Internal name.
- Channel/provider.
- Subject (for email).
- Body/content.
- Supported placeholders and validation.
- Active/inactive state.
- Locale/language if multilingual support is enabled.

Requirements:

- Preview with safe sample data.
- Send test to authorized test destination.
- Validate unknown placeholders.
- Version history.
- Avoid exposing raw secrets or sensitive user information in templates/logs.

## 14.2 Delivery logs

Show:

- Message ID.
- Recipient (masked where appropriate).
- Template.
- Channel.
- Created/sent/delivered timestamps.
- Provider status.
- Failure reason (redacted/safe).
- Retry state.

Retry must be idempotent or create a traceable new delivery attempt.

## 14.3 Preferences

Allow owner to configure transactional notifications separately from marketing notifications. Critical security and payment communications must not be disabled accidentally by a general marketing toggle.

---

# 15. Module 11 — Team and permissions

## 15.1 Admin users list

Columns:

- Name.
- Email/login identity.
- Assigned roles.
- Account status.
- Last login.
- MFA status.
- Created date.

Actions:

- Invite admin.
- Edit profile/role assignment.
- Suspend/reactivate.
- Revoke sessions.
- Resend invitation.

Do not allow the final active Owner/Super Admin account to be removed or demoted without a safe ownership-transfer process.

## 15.2 Role model

Suggested starting roles:

| Role | Intended scope |
|---|---|
| Owner / Super Admin | Complete platform administration |
| Operations Manager | Catalogue operations, orders and customer operations |
| Catalog Manager | Ebooks, digital files, authors, categories, collections and reviews |
| Finance Manager | Transactions, refunds, invoices, finance reports and reconciliation |
| Marketing Manager | Coupons, campaigns, content, SEO and approved customer segments |
| Support Agent | Customer lookup, tickets and limited order visibility |
| Analyst | Read-only access to approved reports |

The owner must be able to approve the final role matrix. Do not assume every role above is required for the first deployment.

## 15.3 Granular permissions

Support permission dimensions such as:

- View/list.
- View sensitive details.
- Create.
- Edit.
- Publish/approve.
- Archive/delete.
- Export.
- Refund/financial action.
- Configure integration.
- Manage users/roles.

Permissions should be scoped by resource and action. Examples:

- `catalog.ebooks.view`
- `catalog.ebooks.create`
- `catalog.ebooks.edit`
- `catalog.ebooks.publish`
- `orders.view`
- `payments.transactions.view`
- `refunds.request`
- `refunds.approve`
- `finance.reports.export`
- `customers.sensitive.view`
- `team.roles.manage`
- `settings.gateway.manage`
- `security.audit.view`

These are illustrative identifiers; use a consistent naming convention in the actual codebase.

## 15.4 Separation of duties

Where practical, support configurable approval thresholds for high-impact actions (e.g. large refunds, gateway configuration changes, role escalation). Do not force two-person approval for every action if it prevents normal operations, but allow the owner to enable it for defined high-risk actions.

---

# 16. Module 12 — Security and audit

## 16.1 Audit logs

Record security- and business-critical actions, including:

- Admin login/logout and failed login events.
- MFA changes and recovery events.
- Admin invitation, suspension and role/permission changes.
- Ebook create/edit/publish/archive and file replacement.
- Price/discount changes.
- Order status/reconciliation changes.
- Refund requests, approvals and gateway submissions.
- Entitlement grant/revoke/restore.
- Customer suspension and sensitive profile changes.
- Gateway, storage, email and integration configuration changes.
- Export of sensitive or financial information.
- Security policy changes.

Audit record fields:

- Actor ID and role snapshot where appropriate.
- Action/event name.
- Target resource type and ID.
- Timestamp in UTC.
- Result (success/failure).
- Request/correlation ID.
- Redacted IP/device context where permitted.
- Before/after values for selected fields where safe.
- Reason/approval reference for sensitive actions.

Audit logs should be append-only from the application perspective and protected from ordinary administrators editing/deleting them. Never store passwords, access tokens, full payment card data or secrets in audit records.

## 16.2 Login history and sessions

- Current admin's active sessions.
- Recent login history.
- Device/browser summary where reasonable.
- Approximate IP/location only where lawful and necessary.
- Revoke current/other sessions.
- Suspicious login alerts.

## 16.3 Security controls

- MFA support for admin accounts; strongly recommended/required for Owner and Finance roles.
- Rate limiting for login, password reset, OTP and sensitive actions.
- Secure session cookies and session rotation after authentication/privilege changes.
- CSRF protection where applicable.
- Secure headers and restrictive content security policy.
- Server-side permission checks.
- Object-level authorization.
- Input validation and output encoding.
- Safe rich-text sanitization.
- Dependency and vulnerability scanning.
- Secret management and rotation process.
- Backup encryption and access control.
- Security event monitoring.

## 16.4 API keys

- Create/revoke key.
- Restrict scopes and environment.
- Show secret only once at creation.
- Store hashed or securely managed secret representation as appropriate.
- Record last-used timestamp and scope.
- Rate limit and allow revocation.

Do not create long-lived broad-scope API keys by default.

---

# 17. Module 13 — Store settings

Settings should be grouped into clear pages with save/apply behavior. Avoid a single unmanageable settings form.

## 17.1 General settings

- Store name.
- Store description.
- Contact email.
- Support phone (optional).
- Business/legal display details.
- Reporting timezone.
- Default locale/language.

## 17.2 Branding

- Primary logo.
- Alternate/dark logo where needed.
- Favicon.
- Brand colors and approved theme tokens.
- Typography selection from safe approved fonts.
- Social sharing defaults.

## 17.3 Domain settings

- Storefront domain status.
- Admin domain/subdomain if separate.
- SSL/TLS status (read-only provider information where possible).
- Canonical URL.
- Redirect configuration subject to safety validation.

Do not allow arbitrary redirects to unsafe destinations.

## 17.4 Localization and currency

- Default timezone.
- Date/time format.
- Number format.
- Default currency.
- Supported currencies if multi-currency is approved.
- Locale/language settings.

Changing currency must not rewrite historical transaction amounts. Store each order/transaction with its original currency and amount.

## 17.5 Checkout settings

- Guest checkout policy.
- Account creation behavior.
- Required customer fields.
- Billing address requirements.
- Coupon behavior.
- Checkout terms/consent configuration.

Only collect personal data that is needed for legitimate operational/legal purposes.

## 17.6 Digital delivery settings

- Reader availability.
- Download availability.
- Default download limits.
- Default access duration.
- Signed-link expiry duration.
- Watermark policy.
- File replacement/version policy.
- Suspicious download thresholds.

Product-level settings may override defaults only where permitted. Clearly show inherited vs overridden configuration.

## 17.7 File storage settings

- Storage provider selection/integration state.
- Private bucket/container status.
- Upload size limits.
- Allowed formats.
- Malware scanning status.
- Processing/transcoding status.
- Retention/cleanup policy.

Changing storage provider requires a migration plan and must not break existing file references.

## 17.8 Customer account settings

- Self-registration enabled/disabled.
- Email verification requirement.
- Password policy.
- Social login providers if approved.
- Account deletion/data export workflow.
- Session/device controls.

## 17.9 Email settings

- Provider.
- Sender name/address.
- Connection health.
- Test email action.
- Webhook/delivery status if supported.

Credentials are never displayed in full after storage.

## 17.10 Integrations

List integration cards for configured services (payment, storage, email, analytics, malware scanning, support, etc.). Each card should show:

- Provider.
- Connected/disconnected status.
- Environment.
- Last successful check.
- Last error (safe and redacted).
- Configure/test/disconnect action, permission-controlled.

## 17.11 SEO settings

- Default SEO title and description.
- Default social image.
- Sitemap status/regeneration.
- Robots/indexing controls.
- Canonical URL behavior.
- Redirect list.

Do not allow accidental global no-index in production without a clear warning and confirmation.

## 17.12 Backup and recovery

- Last backup timestamp/status.
- Backup schedule/retention where supported.
- Backup encryption status.
- Restore-point inventory.
- Recovery procedure link/runbook.
- Restore action only for highly privileged users and preferably through a separately secured operational procedure.

The UI must not claim a successful backup merely because a job was queued. Show actual completion and verification status.

## 17.13 Maintenance mode

- Enable/disable maintenance mode.
- Public maintenance message.
- Optional scheduled window.
- Admin bypass rules, securely implemented.
- Preview.

Require confirmation and record audit entry. Maintenance mode must not prevent critical background payment webhooks from being processed unless the architecture explicitly guarantees safe deferred processing.

---

# 18. Core business workflows and state transitions

## 18.1 Ebook publication lifecycle

Suggested states:

`Draft → Pending Review (optional) → Scheduled (optional) → Published → Archived`

Rules:

- Draft is not visible/purchasable on public storefront.
- Pending Review is visible only to authorized admins/reviewers.
- Scheduled publication requires a reliable scheduler and configured timezone.
- Published ebooks must pass validation.
- Archived products are no longer purchasable but remain in historical orders and customer libraries according to the approved entitlement policy.
- Unpublishing a product must not delete buyer entitlements or historical records.

## 18.2 Payment lifecycle

Payment states depend on the gateway, but the internal model should distinguish at least:

`Created/Initiated → Pending → Succeeded | Failed | Cancelled | Expired`

Refunds are separate records and must not overwrite the original payment state. Webhook events may arrive late, duplicated or out of order; processing must be idempotent and state transitions must be validated.

## 18.3 Order lifecycle

A suggested internal order model:

`Pending Payment → Paid → Fulfilment Complete`

Alternative terminal or exception states may include cancelled, partially refunded, refunded or payment review. The agent must document exact state transitions and prevent invalid transitions.

For digital products, fulfilment means the entitlement has been created/activated successfully, not that a physical package has shipped.

## 18.4 Purchase-to-entitlement workflow

1. Customer checks out.
2. Backend creates a pending order and payment attempt.
3. Payment provider confirms success through verified callback/webhook or trusted provider query.
4. Backend atomically/idempotently records payment success and marks the order paid.
5. Backend creates one entitlement per eligible purchased ebook (or an equivalent line-item entitlement model).
6. Customer library reflects the entitlement.
7. Customer receives receipt/access notification.
8. All relevant events are recorded for audit and reconciliation.

Do not grant an entitlement based solely on a client redirect, query parameter or unverified request.

## 18.5 Refund-to-entitlement workflow

1. Refund request is created.
2. Remaining refundable amount and policy are validated.
3. Appropriate staff member reviews/approves.
4. Gateway refund is submitted idempotently.
5. Verified result updates refund record.
6. Approved entitlement policy is applied (retain, suspend or revoke access).
7. Customer and finance records are updated.
8. Audit record and notification are created.

## 18.6 File replacement workflow

1. Admin uploads a new file version.
2. System validates and scans it.
3. Admin sees a preview/metadata result.
4. New version is stored privately.
5. Admin selects which buyer-access policy applies.
6. New version becomes active only after successful processing.
7. Previous version is retained for the configured period/policy.
8. All impacted entitlements and changes are auditable.

---

# 19. Data and domain model requirements

The exact schema depends on the existing stack. The implementation agent must create a database ERD and migration plan before finalizing tables.

At minimum, the domain model should account for:

- AdminUser.
- Role.
- Permission.
- AdminRoleAssignment.
- Customer.
- CustomerSegment / CustomerGroup.
- Author.
- Category (hierarchical).
- Tag.
- Collection.
- Ebook/Product.
- ProductPrice / PriceSchedule where needed.
- ProductFile / FileVersion.
- ProductAccessPolicy.
- ProductReview.
- Cart / CartItem (if cart is persisted).
- Order.
- OrderItem.
- PaymentAttempt / PaymentTransaction.
- Refund.
- Invoice / CreditNote.
- Entitlement / CustomerLibraryItem.
- DownloadEvent.
- ReaderProgress / ReadingSession (if reader is implemented).
- Coupon.
- DiscountRule.
- Campaign.
- HomepageSection / Banner / ContentPage / MenuItem.
- SupportTicket / TicketMessage / TicketAttachment.
- NotificationTemplate / NotificationDelivery.
- AuditLog.
- AdminSession / SecurityEvent.
- IntegrationConfiguration (secrets stored securely, not as ordinary plaintext settings).
- StoreSetting.

### Domain rules

- Use immutable unique identifiers (UUID/ULID or the project's established safe identifier strategy).
- Store monetary amounts in integer minor units or a proper decimal type, with currency recorded. Never use binary floating-point for monetary storage/calculation.
- Store timestamps in UTC; display using configured timezone.
- Use database transactions for payment/order/entitlement transitions where appropriate.
- Use unique constraints and idempotency keys to prevent duplicate purchases, webhook handling, refund submission and entitlement creation.
- Use foreign keys/referential integrity where supported.
- Use soft-delete/archive only where it makes sense; do not apply soft delete blindly to financial/audit records.
- Track created_at, updated_at and actor IDs for administrative changes where useful.
- Define indexes for common filters/searches and monitor query performance.
- Decide retention/anonymization rules for personal data with legal guidance.

---

# 20. Permission matrix — initial proposal

This is a starting matrix, not final owner approval. The implementation must use a permission system that can support finer-grained control.

| Capability | Owner | Operations | Catalog | Finance | Marketing | Support | Analyst |
|---|---:|---:|---:|---:|---:|---:|---:|
| View dashboard | Yes | Yes | Limited | Yes | Yes | Limited | Yes |
| Manage ebooks | Yes | Limited | Yes | No | Limited | No | No |
| Publish ebooks | Yes | Optional | Optional | No | No | No | No |
| Manage categories/authors | Yes | Limited | Yes | No | Limited | No | No |
| View all orders | Yes | Yes | No | Yes | Limited | Limited | Read-only aggregate |
| View payment details | Yes | Limited | No | Yes | No | Masked/limited | Aggregate only |
| Request refund | Yes | Optional | No | Yes | No | Optional | No |
| Approve refund | Yes | Optional/threshold | No | Optional/threshold | No | No | No |
| Manage customers | Yes | Yes | No | Limited | Segment-limited | Limited | Aggregate only |
| Manage coupons/campaigns | Yes | No | No | No | Yes | No | Read-only reports |
| Manage website content | Yes | No | Limited | No | Yes | No | No |
| View analytics | Yes | Yes | Product-only | Finance-only | Marketing-only | Limited | Yes |
| Manage support tickets | Yes | Yes | No | No | No | Yes | No |
| Manage admins/roles | Yes | No | No | No | No | No | No |
| Configure payment gateway | Yes | No | No | Optional | No | No | No |
| View audit logs | Yes | Limited | Limited | Limited | Limited | No | No |
| Change security settings | Yes | No | No | No | No | No | No |

**Critical:** “Limited” must be translated into explicit resource/action permissions. Do not implement a vague role name without defining the actual permissions.

---

# 21. Global non-functional requirements

## 21.1 Security

- Authentication must use a well-maintained, secure approach appropriate to the chosen stack.
- Require strong authentication for admin accounts; MFA strongly recommended and should be required for privileged roles unless owner approves a documented exception.
- Enforce server-side authorization on every route/action/API.
- Apply object-level authorization to prevent IDOR/insecure direct object reference vulnerabilities.
- Rate-limit authentication and sensitive endpoints.
- Protect sessions and cookies using secure, HTTP-only, same-site settings appropriate to deployment.
- Use CSRF protections where applicable.
- Validate input and encode output.
- Sanitize rich content.
- Protect against SQL/NoSQL injection, XSS, SSRF, CSRF, path traversal, file upload abuse and privilege escalation.
- Keep secrets out of source code, client bundles, logs and error messages.
- Use HTTPS in deployed environments.
- Maintain dependency vulnerability scanning and patch process.

## 21.2 Digital file security

- Original ebooks must be private at rest.
- Use server-authorized, short-lived signed download URLs or a secure streaming endpoint.
- Verify current entitlement and access policy on each download/reader access.
- Avoid permanent public object URLs.
- Apply rate limits and download limits server-side.
- Log relevant download events without collecting unnecessary personal data.
- Scan files before publication.
- Consider watermarking only as a deterrent; never promise that it makes copying impossible.

## 21.3 Reliability and performance

- Paginate/filter large tables server-side.
- Avoid N+1 database queries.
- Use indexed query paths for common filters.
- Use background jobs for file processing, email, report generation and other long-running work.
- Provide job state/retry visibility to admins where operationally useful.
- Handle duplicate and delayed webhook events.
- Provide structured logging and correlation IDs.
- Add health checks and monitoring.
- Define backups and test restoration; a backup job is not proven until restore is tested.

## 21.4 Accessibility

Target WCAG 2.2 AA where practical:

- Keyboard navigation.
- Visible focus states.
- Semantic labels and headings.
- Dialog focus management.
- Sufficient contrast.
- Screen-reader labels for icon-only actions.
- Accessible tables and form errors.
- Charts with textual summaries or data tables.
- Do not use color as the only indicator of status.

## 21.5 Responsive behavior

Desktop is the primary admin experience. Tablet/mobile must still support core tasks:

- Review dashboard alerts.
- Search/view orders and customers.
- View ebook information.
- Respond to support tickets.
- Review/approve configured actions where secure.

Dense tables may transform into horizontally scrollable tables or responsive row cards, but must remain usable. Do not hide important actions without a discoverable alternative.

## 21.6 Observability

- Structured application logs.
- Error monitoring.
- Request/correlation IDs.
- Payment webhook health.
- Background job health.
- File upload/scan processing health.
- Email delivery status where provider supports it.
- Backup completion and restore verification.
- Security alerts for anomalous activity.

---

# 22. Error, empty, loading and success states

Every page must be designed for more than the happy path.

## 22.1 Loading

- Use skeletons or clear progress indicators.
- Prevent duplicate submissions while a request is pending.
- Preserve filters and form values when safe after recoverable errors.

## 22.2 Empty states

Examples:

- No ebooks yet: explain how to add the first ebook and show the Add Ebook action if permitted.
- No orders for selected dates: explain that no records match the period and offer date/filter reset.
- No search results: show the query and clear-filter action.
- No customers: explain registration behavior and avoid implying a system failure.

## 22.3 Error states

- Explain what failed in user-friendly language.
- Do not expose stack traces, secret values, database details or internal tokens.
- Provide a retry action only when retry is safe.
- For payment/refund ambiguity, show “status pending verification” rather than falsely reporting success/failure.
- Log technical details securely with a correlation ID.

## 22.4 Success states

- Confirm the actual persisted result.
- Show next useful action (e.g. View ebook, View order, Preview page).
- Do not show success before the server confirms completion.

---

# 23. Search, filtering, sorting and export standards

All major list pages must support relevant server-side search/filter/sort/pagination.

- Search must be permission-aware.
- Filter state should be represented in URL query parameters where practical so pages can be bookmarked/shared internally without leaking sensitive information.
- Sorting must use an allowlist of supported columns.
- Pagination must have stable ordering to avoid duplicate/missing rows when data changes.
- Exports must match active filters and respect permissions.
- Exports containing personal, financial or security data require stricter permissions and audit logging.
- Large exports should run as background jobs and provide secure, expiring download access.
- Do not expose unrestricted database exports to ordinary staff roles.

---

# 24. Testing requirements

## 24.1 Unit tests

Test:

- Price/discount calculation.
- Coupon validity and limits.
- Tax calculation based on configured rules.
- Status transition rules.
- Permission evaluation.
- Refund remaining-balance calculation.
- Metric calculations.
- Slug uniqueness/normalization.
- File policy validation.

## 24.2 Integration tests

Test:

- Admin authentication and MFA flow.
- Role/permission enforcement on APIs.
- Ebook create/edit/publish/archive.
- File upload, scan, private storage and download authorization.
- Order creation and payment verification.
- Duplicate/out-of-order payment webhook handling.
- Idempotent entitlement creation.
- Full and partial refunds.
- Refund failure and retry.
- Customer suspension/reactivation.
- Notification template delivery pipeline.
- Content draft/publish/preview.
- Export permission and audit events.

## 24.3 End-to-end tests

Minimum scenarios:

1. Owner logs in and views dashboard.
2. Owner creates a category and author.
3. Catalog manager creates a draft ebook with valid files.
4. Authorized user publishes the ebook according to policy.
5. Unauthorized user cannot publish or change price.
6. Owner views an order and its payment record.
7. Verified successful payment creates a customer entitlement exactly once.
8. Unverified/failed payment does not create entitlement.
9. Customer support agent can view permitted customer/order details but cannot see payment secrets or issue unapproved refunds.
10. Finance user requests/processes refund according to approval policy.
11. Refund updates records only after provider-confirmed status.
12. Customer access follows the approved refund entitlement policy.
13. Admin content editor saves a draft and previews it without publishing.
14. A staff member cannot access a restricted admin route by entering its URL directly.
15. Audit logs record sensitive admin actions.
16. Search, filter, sorting, pagination and export work on representative data volumes.

## 24.4 Security tests

At minimum test:

- Horizontal/vertical privilege escalation.
- IDOR across ebook, customer, order and ticket IDs.
- CSRF/XSS/input sanitization.
- File upload MIME/signature bypass and path traversal.
- Rate limit bypass and brute-force protection.
- Session invalidation and logout.
- Secret leakage in API responses/logs/client bundle.
- Duplicate webhook and replay behavior.
- Refund overpayment/double-refund prevention.
- Unentitled digital file access.

## 24.5 Visual/UX QA

- Test all pages at desktop, tablet and mobile breakpoints.
- Test keyboard-only navigation.
- Test empty/loading/error/success states.
- Verify every button and menu item.
- Verify long titles, large tables, large numbers and missing images.
- Confirm all destructive actions require appropriate confirmation.

---

# 25. Delivery phases and recommended build order

The build order may change after repository inspection, but dependencies should generally follow this sequence.

## Phase 0 — Discovery and architecture validation

Deliverables:

- Repository and stack audit.
- Existing feature inventory.
- Gap analysis against this document.
- Architecture proposal.
- Data model/ERD proposal.
- Permission model.
- UI design system proposal.
- Decision log for unresolved items.
- Phased implementation tickets.

No major feature implementation until this discovery is complete and reviewed.

## Phase 1 — Admin foundation

- Admin authentication.
- Admin shell and navigation.
- Role/permission framework.
- Audit logging foundation.
- Global form/table/dialog/toast components.
- Settings framework.
- Error handling and observability.

## Phase 2 — Catalogue

- Categories.
- Authors.
- Ebook list/create/edit/detail.
- Private file storage/upload pipeline.
- File versioning.
- Publication workflow.
- Collections.

## Phase 3 — Orders, payments and entitlements

- Orders list/detail.
- Payment attempts/transactions.
- Verified provider integration.
- Webhook idempotency.
- Entitlement fulfilment.
- Refund workflow.
- Invoice/receipt foundation.

## Phase 4 — Customers and support

- Customer list/profile.
- Customer groups/segments as approved.
- Support tickets.
- Customer/account controls.

## Phase 5 — Store content and marketing

- Homepage section manager.
- Banners/static pages/navigation/footer.
- Coupons/discount rules.
- Campaign management.
- Notification templates.

## Phase 6 — Analytics and finance reports

- Dashboard metrics.
- Sales/revenue/product/customer analytics.
- Financial reporting and exports.
- Download/reading analytics where approved.

## Phase 7 — Advanced operations

- Affiliate/referral/subscription modules if approved.
- Advanced fraud controls.
- Advanced automation.
- Additional integrations.

## Phase 8 — Production hardening

- Security review and penetration testing.
- Load/performance testing.
- Backup/restore drill.
- Disaster recovery runbook.
- Monitoring and alert verification.
- Accessibility and cross-browser QA.
- Production go-live checklist.

---

# 26. Global acceptance checklist

The admin panel is not ready for release until all applicable items below are satisfied.

## Product completeness

- [ ] Every approved sidebar page exists and works.
- [ ] No unapproved or unfinished module is exposed as complete.
- [ ] All page actions work end-to-end.
- [ ] The owner can operate the approved business workflows without direct database edits.
- [ ] Customer-facing effects of admin changes are previewable or clearly explained.

## Data integrity

- [ ] Orders, payments, refunds, invoices and entitlements remain historically consistent.
- [ ] Duplicate webhook events cannot duplicate payments or entitlements.
- [ ] Refunds cannot exceed eligible captured amounts.
- [ ] Product archiving does not erase purchase history.
- [ ] File replacement follows a defined version policy.
- [ ] Monetary calculations use safe decimal/minor-unit handling.

## Security

- [ ] Every admin API/action is permission checked server-side.
- [ ] Direct URL access cannot bypass permissions.
- [ ] Admin sessions and authentication are secure.
- [ ] Sensitive settings and exports require elevated permissions.
- [ ] Private ebook files cannot be accessed without valid entitlement.
- [ ] Secrets are not exposed in frontend, logs or source control.
- [ ] Audit logs capture required sensitive actions.

## UX and accessibility

- [ ] Consistent design system across modules.
- [ ] Loading, empty, error and success states exist.
- [ ] Destructive actions are confirmed.
- [ ] Keyboard navigation and visible focus work.
- [ ] Responsive layouts are usable.
- [ ] All icons/buttons have accessible labels.

## Operations

- [ ] Background jobs have status/retry visibility where appropriate.
- [ ] Payment webhook health can be monitored.
- [ ] Backup success and recovery are verifiable.
- [ ] Errors have correlation IDs and useful logs.
- [ ] Exports are secure and expire where appropriate.
- [ ] Runbooks exist for gateway failure, storage failure, refund failure and restore.

## Quality

- [ ] Unit tests pass.
- [ ] Integration tests pass.
- [ ] E2E critical flows pass.
- [ ] Permission/security tests pass.
- [ ] No critical/high unresolved security vulnerabilities.
- [ ] No dead buttons, broken navigation or production mock data.
- [ ] Documentation and deployment configuration are updated.

---

# 27. Required agent deliverables before declaring implementation complete

The implementation agent must provide:

1. **Repository assessment:** existing architecture, reusable components, technical debt and gaps.
2. **Approved decision register:** list of product decisions confirmed by owner and unresolved items.
3. **Admin sitemap:** all routes and permission requirements.
4. **Page specification:** page structure, data fields, actions, validations and states.
5. **Architecture diagram:** frontend, backend, database, storage, payment gateway, email and background jobs.
6. **ERD and data dictionary:** entities, relationships, key constraints and indexes.
7. **Permission matrix:** role/action/resource mapping, enforced by backend.
8. **API specification:** endpoints/actions, payloads, response/error formats and authorization requirements.
9. **Threat model:** assets, trust boundaries, misuse cases and mitigations.
10. **Test report:** unit, integration, E2E, security and visual QA results.
11. **Deployment/runbook:** environment variables, migrations, secrets, backups, restore and monitoring.
12. **Known limitations:** anything not implemented, deferred or requiring owner decision.

---

# 28. Final implementation guardrails

- Do not start with the dashboard alone and leave the underlying workflows as mock data.
- Do not build every sidebar item as a separate disconnected mini-application. Use consistent domain models, shared components and module boundaries.
- Do not create an unnecessarily complex microservice architecture for the initial store. A well-structured modular monolith is the default proposal unless repository constraints or measured scale justify another design.
- Do not let frontend state be the source of truth for prices, discounts, payment status, permissions or digital access.
- Do not trust a payment success redirect; verify with the provider.
- Do not expose ebook files through public permanent URLs.
- Do not promise that watermarking or browser viewing prevents screenshots or copying.
- Do not hard-delete records required for financial, legal, audit or customer entitlement history.
- Do not invent tax/legal policies. Make them configurable and require owner/professional approval.
- Do not send marketing messages without the required consent and unsubscribe controls.
- Do not expose customer reading activity beyond the approved privacy model.
- Do not claim a feature is complete until its workflow, backend checks, error handling and tests are complete.
- Do not silently change this document. Record and explain any proposed change for owner review.

**End of specification.**
