# 02 — Master Design System

**Version:** 0.1 proposal  
**Status:** Directional baseline; exact gallery references, palette and font selection require owner approval  
**Design inspiration:** User-selected Dribbble `book-landing-page` gallery: https://dribbble.com/tags/book-landing-page

## 1. Design objective

Create one recognizable digital-bookstore brand expressed through three purpose-built interfaces:

1. **Customer Panel:** expressive, editorial, book-cover-led, welcoming and conversion-conscious.
2. **Owner/Admin Panel:** precise, information-rich, calm and efficient.
3. **Support/Operations Panel:** task-focused, legible, fast to scan and optimized for case resolution.

All three share design tokens, typography principles, iconography, interaction patterns and brand voice. They must not share identical page layouts: a bookstore landing page and an operational ticket queue have different jobs.

The Dribbble gallery is an inspiration source, not a complete specification. It contains many different visual directions. Do not copy one shot wholesale, assume all shots share the same style, or invent a final brand identity. The owner should select 2–4 concrete reference shots before high-fidelity design sign-off.

## 2. Proposed visual language

### 2.1 Customer-facing direction

Use a premium editorial bookstore aesthetic:

- Strong typographic hierarchy and art-directed headlines.
- Large, high-quality ebook-cover imagery treated as the primary product asset.
- Distinctive hero compositions rather than a generic centered headline with three cards.
- Editorial grid layouts, controlled asymmetry and generous whitespace.
- Curated shelves/collections with clear hierarchy.
- Strong product detail pages with clear purchase actions and factual access information.
- Subtle background texture or paper-like surface only when it does not harm contrast or performance.
- Motion used for purposeful transitions, image reveal or state feedback; no animation that blocks shopping or reading.

### 2.2 Admin direction

Translate the same brand into an operational interface:

- Clear, persistent navigation and page titles.
- Compact but readable data tables.
- Useful KPI cards with definitions and date ranges.
- Strong status hierarchy and predictable filters.
- No decorative hero blocks that waste workspace.
- Dense content should still have clear grouping, alignment and spacing.

### 2.3 Support direction

Use a focused case-management workspace:

- Queue/navigation area, ticket list and ticket detail/context area where desktop space permits.
- Clearly distinguish customer-visible replies from internal notes.
- Prominent status, priority, assignee and SLA/age indicators only if those policies are defined.
- Keep dangerous or privileged actions separated and confirmed.
- On narrow viewports, collapse to a single-column navigation flow rather than compressing three columns.

## 3. Provisional design tokens

The tokens below are starting values for design exploration. They are not final brand approvals. The final palette must be validated against actual book-cover imagery, WCAG contrast and selected Dribbble references.

### 3.1 Colour roles

| Token | Provisional value | Use |
|---|---|---|
| `color.canvas` | `#F7F5F0` | Warm bookstore background |
| `color.surface` | `#FFFFFF` | Cards, forms and content surfaces |
| `color.surface-muted` | `#EEEAE2` | Subtle panels and secondary backgrounds |
| `color.text-primary` | `#20231F` | Main text |
| `color.text-secondary` | `#646960` | Secondary text; verify contrast before use |
| `color.border` | `#DEDAD1` | Dividers and control borders |
| `color.brand-primary` | `#244B3D` | Provisional deep green accent/action |
| `color.brand-hover` | `#183A2F` | Hover/active accent |
| `color.brand-tint` | `#E6EEE9` | Soft selected/brand surface |
| `color.accent` | `#C27B45` | Restrained warm accent; not for long text |
| `color.success` | `#24764A` | Successful state |
| `color.warning` | `#9A621B` | Warning state |
| `color.danger` | `#B33C36` | Destructive/error state |
| `color.info` | `#356A9A` | Informational state |

Do not encode meaning through colour alone. Pair semantic colours with labels/icons and ensure text/background contrast. Avoid using the accent colour for every button or every decorative element.

### 3.2 Typography

Typography is a two-role system: expressive display type for selected editorial moments and a highly legible interface sans-serif for navigation, body text, forms, tables and reader controls.

Candidate pairings to test, not final selections:

- Display: `DM Serif Display`, `Fraunces`, or another distinctive licensed/web-safe editorial face.
- Interface/body: `Inter`, `Manrope`, or another highly legible sans-serif.
- Reader: user-selectable serif/sans-serif system stack plus adjustable font size and line height.

Rules:

- Self-host only fonts the project is licensed to use; document font licences and subsets.
- Provide system fallbacks and avoid layout shifts while fonts load.
- Do not use display serif for dense tables, small labels or operational controls.
- Use a consistent type scale and line-height system; avoid arbitrary per-page font sizes.

Proposed scale (rem; adjust after visual testing):

| Token | Size | Primary usage |
|---|---:|---|
| `type.display-xl` | 3.5rem | Large storefront hero headline, desktop only |
| `type.display-lg` | 2.75rem | Editorial section hero |
| `type.h1` | 2.25rem | Page title |
| `type.h2` | 1.75rem | Section title |
| `type.h3` | 1.375rem | Card/detail subheading |
| `type.body-lg` | 1.125rem | Intro copy and book descriptions |
| `type.body` | 1rem | Standard body text |
| `type.body-sm` | 0.875rem | Secondary text |
| `type.label` | 0.75rem | Compact labels; not for long content |

### 3.3 Spacing and layout

Use a consistent 4px base spacing scale. Suggested tokens: `4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96` CSS pixels.

- Main content should use a shared responsive container with a documented maximum width.
- Storefront sections may use wider full-bleed art direction, but text measure should remain readable.
- Admin and Support should prioritize usable work area over decorative margins.
- Use consistent vertical rhythm between headings, descriptions, filters, tables and actions.

Proposed breakpoint starting points (not device-specific hard rules):

| Name | Width | Behaviour |
|---|---:|---|
| `sm` | 640px | Small mobile / large phone layout changes |
| `md` | 768px | Tablet navigation and grid changes |
| `lg` | 1024px | Desktop sidebar and multi-column workspaces |
| `xl` | 1280px | Wide data/table layouts |
| `2xl` | 1536px | Wider content container; cap line length |

Use content-driven responsive breakpoints if a layout needs them. Test at 320px, 360px, 390px, 768px, 1024px, 1280px and 1440px widths, plus zoom/reflow.

### 3.4 Shape, elevation and borders

- Small controls: 8px radius.
- Cards and grouped surfaces: 12–16px radius.
- Large media/hero surfaces: 16–24px only where visually appropriate.
- Borders: subtle 1px neutral border for grouping and focusable controls.
- Shadows: restrained and consistent; do not use shadows as the only way to indicate hierarchy.
- Focus indicators: always visible and sufficiently contrasting.

## 4. Panel-specific layout standards

### 4.1 Customer Panel

**Public bookstore**

- Header: brand mark, main navigation, search entry, account, wishlist/cart where included.
- Homepage: editorial hero, category discovery, curated collection shelves, featured/new/bestseller sections, trust/help content and footer.
- Catalogue: filter/sort controls, clear result count, responsive grid/list options where useful.
- Ebook detail: large cover, title, author, format, language, edition, clear price, concise benefits, sample/preview, purchase CTA and access/refund terms.
- Cart/checkout: focused steps, price breakdown, discount visibility, payment status and accessible error recovery.
- Mobile: sticky purchase action only when it does not obscure content or accessibility controls; ensure cart and checkout are comfortable on narrow screens.

**Signed-in customer workspace**

- Library-first navigation with clear tabs/filters for all, reading, downloaded or archived content where those states exist.
- Book cards must distinguish owned books from books not owned.
- Continue-reading cards show reliable progress only when the reader supports it.
- Account settings must be organized, not mixed with discovery navigation.

**Reader**

- Keep reading area visually quiet.
- Controls: contents, progress, font family/size, line spacing, margins, theme, bookmarks and search where supported by format.
- Reader themes: paper, sepia and night as a proposed preference set.
- Persist reading preferences and progress only according to the documented privacy and sync policy.
- Never promise features unsupported by the actual format/reader engine.

### 4.2 Owner/Admin Panel

- Persistent collapsible sidebar on desktop; accessible drawer/navigation on mobile.
- Top bar with page context, global search/command palette if implemented, notifications, store preview and admin profile.
- Each management page: breadcrumb, heading, descriptive context, primary action, filters, summary metrics if meaningful, data table/list, pagination and empty/loading/error states.
- Separate destructive actions from routine edit actions.
- Data table density may be compact but text and hit targets must remain accessible.
- Analytics must show metric definition, date range, timezone and comparison where relevant.

### 4.3 Support/Operations Panel

- Desktop may use a three-region layout: queue/navigation, ticket list, conversation and customer/order context.
- Keep ticket response composer persistent and distinguish reply from internal note.
- Surface customer/order/payment state as read-only context unless a permission-checked action is explicitly available.
- Use priority/status text plus icons/colour, not colour alone.
- Provide keyboard shortcuts only when discoverable, non-conflicting and configurable.

## 5. Component inventory and behaviour

Create a shared component library with documented props/states. Reuse components, not panel-specific business logic.

### 5.1 Foundation

- Buttons: primary, secondary, tertiary/ghost, destructive, icon-only.
- Text inputs, textareas, select/combobox, date picker, checkbox, radio, switch and file upload.
- Form field: label, helper text, validation message and required state.
- Badge/status indicator.
- Tooltip, popover, dropdown and command palette.
- Dialog/modal and side drawer.
- Tabs, breadcrumbs, pagination and stepper.
- Toast/inline alert and confirmation pattern.
- Skeleton, spinner, empty state and error state.
- Accessible table, filter chips, sort controls and bulk-action bar.

### 5.2 Commerce

- Ebook cover/card with defined image aspect ratio and fallback.
- Price display with original/sale price only when a real discount exists.
- Format/access badge (PDF, EPUB, online reader/download) only when accurate.
- Product rating only where verified reviews exist.
- Add-to-cart/buy action with disabled, loading, success and failure states.
- Cart line item and checkout price summary.
- Order status/payment status as separate indicators.

### 5.3 Reader

- Reader toolbar and overflow menu.
- Table of contents navigation.
- Reading progress indicator.
- Bookmark control and bookmark list.
- Typography and theme preferences.
- Search in book if supported.
- Loading, invalid-file, unavailable-content and offline/error states.

### 5.4 Support

- Ticket row/card, queue filter, assignment selector, priority/status control.
- Conversation message and internal note (visually unmistakable distinction).
- Attachment control with upload progress and safe file constraints.
- Customer/order context panel.
- Escalation and handoff action.

## 6. Interaction standards

- Every visible control has a real action or is not rendered in production.
- Provide immediate feedback for state transitions without claiming success before server confirmation.
- Preserve user input on recoverable failures.
- Disable duplicate submissions while a request is pending; backend idempotency is still required.
- Confirm destructive or privileged actions and explain their consequence.
- Use optimistic UI only for safe, reversible actions; financial, entitlement and permission changes require server-confirmed state.
- Use inline validation and accessible error summaries for long forms.
- Use skeletons for stable content layout; use spinners for small local operations.
- Do not trap users in blocking overlays or autoplay motion/audio/video.
- Respect `prefers-reduced-motion` and provide non-animated alternatives.

## 7. Accessibility requirements

Target WCAG 2.2 Level AA. Minimum design/QA expectations:

- Full keyboard operation with logical focus order.
- Visible focus and skip links where relevant.
- Semantic headings, landmarks, buttons and form labels.
- Appropriate accessible names for icon-only controls.
- Contrast verified for text, controls, borders and focus indicators.
- Do not rely on colour alone for statuses or charts.
- Reflow at narrow widths and high zoom without loss of functionality.
- Accessible dialogs, menus, tables, error messages and live updates.
- Touch targets meet WCAG 2.2 target-size criteria or documented exceptions.
- Reader controls are keyboard-accessible and exposed to assistive technology.
- Test with automated tools plus manual keyboard and screen-reader checks.

## 8. Content and imagery rules

- Use real licensed ebook covers and product media; never fabricate book covers for real products.
- Use responsive image formats, dimensions, lazy loading below the fold and appropriate priority for the hero/LCP image.
- Set stable image aspect ratios to avoid layout shift.
- Provide meaningful alt text when an image conveys information; decorative images use empty alt text.
- Avoid text baked into images where the text should be translated, responsive or accessible.
- Do not use stock images as substitutes for actual book covers on real product pages.
- Keep marketing imagery, labels and offer dates truthful and synchronized with backend data.

## 9. Responsive and performance standards

- Customer Panel: mobile-first; shopping, account, library and reader flows work on phone, tablet and desktop.
- Admin Panel: desktop-optimized; high-value workflows remain functional on tablet/mobile with adapted navigation and table behavior.
- Support Panel: desktop-optimized; single-column adaptation on mobile; no clipped ticket composer or inaccessible customer context.
- Use responsive images, font loading strategy, route-level code splitting where appropriate and avoid unnecessary client JavaScript.
- Target good Core Web Vitals at the 75th percentile for eligible real-user sessions: LCP ≤ 2.5 seconds, INP ≤ 200 milliseconds, CLS ≤ 0.1. Monitor by panel/device; treat as measured targets, not a guarantee.

Performance thresholds reference: [Google web.dev — Core Web Vitals](https://web.dev/articles/vitals). The numeric thresholds are field targets, not a promise that every visit will meet them.

## 10. Design QA checklist

- [ ] Selected reference shots and signed-off brand direction are documented.
- [ ] Tokens are implemented centrally; no arbitrary per-page colour values.
- [ ] Typography, spacing, radii, borders and shadows are consistent.
- [ ] All three panels feel related but serve distinct tasks.
- [ ] Customer storefront is visually distinctive and product-led.
- [ ] Admin pages prioritize data clarity and action hierarchy.
- [ ] Support workspace prioritizes ticket resolution and clear internal/customer messages.
- [ ] Every component has hover, focus, active, disabled, loading and error states as appropriate.
- [ ] Mobile, tablet, desktop, zoom and keyboard states are reviewed.
- [ ] Contrast and accessibility checks are recorded.
- [ ] No dead buttons, fake charts, placeholder customer data or decorative-only controls remain.

## 11. Design decisions still requiring owner approval

1. Exact 2–4 Dribbble reference shots.
2. Final brand name, logo, wordmark and favicon.
3. Final palette (provisional warm-paper/ink/green/amber direction is not locked).
4. Final display and interface typefaces after license and rendering checks.
5. Whether Admin and Support provide a dark mode at first launch.
6. Final density choice for tables and operational dashboards.
7. Whether Customer storefront uses a persistent desktop sidebar (not recommended by default) or conventional top navigation.
8. Final reader appearance controls and supported reader features after format/engine selection.

Do not treat provisional token values as brand approval. Build them as centralized tokens so they can be changed without rewriting components.
