# Frontend Export — Technical Inspection Findings

**Source archive:** `stitch_folio_digital_bookstore_ux 2.17.19 PM(1).zip` (uploaded 20 November 2026)  
**Inspection scope:** Archive contents and HTML structure; this is not a live browser test of a running website.

## 1. Archive structure

- 168 ZIP entries including directory entries.
- 84 actual files: 83 `code.html` files and one `folio_design_system.md`.
- 83 screen HTML files are distributed across 51 Customer, 18 Owner/Admin and 14 Support/Operations screens.
- Desktop and mobile exports exist for selected Customer screens, including homepage, explore, search results, login, register and ebook reader. The design system specifies desktop 1440px, tablet 768px and mobile 390px verification.

## 2. Runtime characteristics

The export is a set of static screen references, not an integrated frontend project:
- No `package.json`, lockfile, build configuration, router, reusable local component source tree, application API client, test runner or frontend environment template was found.
- All 83 HTML screens reference `https://cdn.tailwindcss.com?plugins=forms,container-queries`.
- External references include Google Fonts for Cormorant Garamond and Inter, Material Symbols, and 15 distinct `lh3.googleusercontent.com` image URLs.
- 23 form elements were counted in the supplied HTML files.
- Some pages use legacy relative links referencing older exported filenames; convert these into the actual app router.
- Certain demo actions, including a checkout simulation/failure link, are reference-only and must not survive as fake production payment behaviour.

## 3. Implementation consequences

1. Rebuild the screens within the repository's chosen framework and router.
2. Convert repeated headers, sidebars, book cards, tables, forms, dialogs, status pills, navigation, and empty/error states into reusable components.
3. Remove Tailwind CDN runtime dependence; use the repository build pipeline and a pinned compatible dependency.
4. Load fonts in a way that meets the deployment/performance and licensing requirements; provide fallback fonts.
5. Download/replace remote design images with owned, licensed, stable assets. Do not assume Google-hosted design preview image URLs are production-grade or commercially licensed.
6. Implement true API-driven interactions; do not treat static HTML links/buttons as functioning business logic.
7. Preserve the named route concepts in the screen inventory but align actual paths with repository conventions.
8. Use the exact supplied design-system reference for visual tokens unless the owner approves a deliberate update.

## 4. Observed business and terminology signals

- Customer design uses “Bag”, “Add to Bag”, “Explore Books”, and “Preview”.
- Checkout sample contains rupee price formatting and UPI ID / NetBanking choices.
- Payment success, pending and failed states are separate screens.
- The support workspace contains a sample UPI payment issue and an EPUB 403 download issue.
- Desktop and mobile reader screens exist.
- Admin includes catalog, ebook editor/details, categories/collections, orders/refunds, customers, finance, marketing/coupons, website content, analytics, support oversight, notifications, team/permissions, security/audit and settings.
- Support includes inbox/workspace, customer lookup/profile, order/payment investigation, ebook-access troubleshooting, knowledge base, saved replies, notifications, performance, reports and profile settings.

These signals help map workflows but do not constitute approval of launch currency, payment provider, ebook formats, refund policy, tax rules, authentication choices or operating jurisdiction.
