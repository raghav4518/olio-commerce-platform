# Frontend / Backend Handoff — FOLIO Stitch Export Integration

## 1. Supplied UI reference

The product owner supplied a Google Stitch export containing 83 individual HTML screens plus `folio_design_system.md`. See:
- `frontend-reference/SCREEN-INVENTORY.csv`
- `frontend-reference/folio_design_system.md`
- `frontend-reference/FRONTEND-EXPORT-TECHNICAL-OBSERVATIONS.md`
- `backend/15-FRONTEND-SCREEN-TO-BACKEND-MAPPING.md`

Panel distribution:
- Customer: 51 screens (includes public pages, authentication, commerce, account/library/reader, customer support and legal pages).
- Owner/Admin: 18 screens.
- Support/Operations: 14 screens.

## 2. Design identity to preserve

The design-system file is the authoritative visual reference for this export. Key visual rules include:
- Quiet-luxury editorial digital bookstore.
- Cormorant Garamond for literary/display typography and Inter for UI/body copy.
- Warm-neutral monochrome surfaces; primary charcoal `#202321`, page surface `#FAFAF9`, hero surface `#F1F1F0`, secondary surface `#F6F6F4`, white `#FFFFFF`, outline `#DEDEDC`.
- 1320px max-width / 1440px desktop canvas, 60px desktop margin; 390px mobile frame with 20px horizontal margin.
- Restrained 2–8px radii, hairline borders, limited shadows and deliberate motion.
- Customer-facing terminology uses **Bag** rather than Cart, **Preview** rather than Demo, and **Explore Books**.
- Admin and Support preserve the same brand language but may use denser, task-oriented layouts.
- WCAG 2.2 AA is the design target; status cannot rely on colour alone.

Read the copied full design-system document rather than relying only on this summary.

## 3. Export is not the running application

Do not treat the exported HTML as a deployable SPA. It contains one static HTML file per screen and references a Tailwind CDN script, external Google fonts/Material Symbols and remote image URLs. It has legacy relative href values in several pages and demo/simulation affordances such as a payment failure link. It has no package.json, runtime router, API client, app-level state, backend, or executable persistence.

Use it to reproduce the intended screen appearance and discover UI controls. Build proper components, routes, state, validation, API integration and responsive breakpoints in the existing repository.

## 4. Screen states must be data-driven

Every exported page must be implemented using real backend data or an explicitly declared state. At minimum create loading, empty, error, unauthorized, success, pending and unavailable states where relevant. A screen must never invent a paid order, successful payment, entitlement, customer, financial metric or support outcome.

## 5. Terminology and sample values

The designs show `₹` pricing, UPI ID input, NetBanking, and example order/ticket references. These are UI evidence for India-oriented payment UX and support scenarios, not final legal/market/gateway approval. Keep currency and payment method availability configurable. Do not hard-code sample customer names, prices, order IDs or payment references into production.

## 6. Three-panel boundary

- Customer pages never expose internal controls, internal notes, staff data or privileged operations.
- Owner/Admin pages use explicit granular permissions and audit logs.
- Support pages expose only the minimum customer/order/payment/entitlement context needed for approved support tasks.
- No fourth panel or seller/author portal is authorized. Public author profile pages are editorial catalog content.
