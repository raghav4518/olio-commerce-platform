---
name: "FOLIO Editorial Design System"
description: "Quiet-luxury digital bookstore design system combining high-contrast editorial serif typography, tactile neutral surfaces, and museum-grade book presentation."
---

## Overview

FOLIO is a premium digital ebook bookstore built with the restraint and authority of an independent literary publishing house. The interface prioritizes quiet luxury, generous negative space, and tactile 3D book-cover presentation over promotional clutter.

- **Brand Personality:** Cultured, understated, trustworthy, and approachable.
- **Target Audience:** Discerning readers, designers, thinkers, and lifelong learners seeking curated digital editions.
- **Design Style:** Editorial Minimalism — monochromatic warm-stone and alabaster surfaces, crisp hairline rules, high-contrast serif headlines (`Cormorant Garamond`), and clean geometric sans-serif UI copy (`Inter`).

## Colors

The interface chrome is strictly monochromatic and warm-neutral so book-cover artwork provides the only chromatic richness. Dark charcoal (`#202321`) anchors primary actions, while subtle slate (`#777F86`) marks quiet metadata and secondary states.

```yaml
colors:
  primary: "#202321"
  on-primary: "#FFFFFF"
  surface: "#FAFAF9"
  on-surface: "#171717"
  surface-hero: "#F1F1F0"
  surface-secondary: "#F6F6F4"
  surface-white: "#FFFFFF"
  on-surface-muted: "#5C5D5A"
  secondary-typography: "#777875"
  outline: "#DEDEDC"
  accent: "#777F86"
  on-accent: "#141414"
  success: "#2F5E44"
  on-success: "#FFFFFF"
  error: "#8C2D2D"
  on-error: "#FFFFFF"
```

## Typography

A disciplined dual-font pairing separates literary voice from commerce utility: **Cormorant Garamond** carries editorial headlines, section titles, and book titles with classical contrast, while **Inter** handles navigation, body prose, pricing (`tabular-nums`), and form controls with crisp legibility.

```yaml
typography:
  display-lg:
    fontFamily: "Cormorant Garamond"
    fontSize: 56px
    fontWeight: 500
    lineHeight: 62px
    letterSpacing: -0.5px
  display-lg-mobile:
    fontFamily: "Cormorant Garamond"
    fontSize: 32px
    fontWeight: 500
    lineHeight: 38px
    letterSpacing: -0.25px
  headline-lg:
    fontFamily: "Cormorant Garamond"
    fontSize: 40px
    fontWeight: 500
    lineHeight: 46px
    letterSpacing: -0.3px
  headline-lg-mobile:
    fontFamily: "Cormorant Garamond"
    fontSize: 28px
    fontWeight: 500
    lineHeight: 34px
    letterSpacing: -0.2px
  headline-md:
    fontFamily: "Cormorant Garamond"
    fontSize: 30px
    fontWeight: 500
    lineHeight: 36px
    letterSpacing: -0.15px
  title-lg:
    fontFamily: "Cormorant Garamond"
    fontSize: 24px
    fontWeight: 600
    lineHeight: 30px
    letterSpacing: 0px
  title-md:
    fontFamily: "Cormorant Garamond"
    fontSize: 20px
    fontWeight: 600
    lineHeight: 26px
    letterSpacing: 0px
  title-sm:
    fontFamily: "Inter"
    fontSize: 15px
    fontWeight: 600
    lineHeight: 22px
    letterSpacing: 0px
  body-lg:
    fontFamily: "Inter"
    fontSize: 17px
    fontWeight: 400
    lineHeight: 28px
    letterSpacing: 0px
  body-md:
    fontFamily: "Inter"
    fontSize: 15px
    fontWeight: 400
    lineHeight: 24px
    letterSpacing: 0px
  body-sm:
    fontFamily: "Inter"
    fontSize: 13px
    fontWeight: 400
    lineHeight: 20px
    letterSpacing: 0.1px
  label-lg:
    fontFamily: "Inter"
    fontSize: 14px
    fontWeight: 500
    lineHeight: 20px
    letterSpacing: 0.15px
  label-md:
    fontFamily: "Inter"
    fontSize: 12px
    fontWeight: 500
    lineHeight: 16px
    letterSpacing: 0.3px
  label-sm:
    fontFamily: "Inter"
    fontSize: 11px
    fontWeight: 500
    lineHeight: 14px
    letterSpacing: 0.5px
```

## Layout & Spacing

Desktop uses a **Fixed-Max-Width Grid** (`max-w-[1320px]`) inside a `1440px` canvas with `60px` outer page margins and a 12-column grid (`24px` gutters). Mobile (`390px`) uses a **Fluid Grid** with `20px` horizontal margins and generous vertical rhythm (`56px`–`72px` between sections). All spacing derives from a `4px`/`8px` base unit.

```yaml
spacing:
  gutter-xs: 4px
  gutter-sm: 8px
  gutter-md: 16px
  gutter-lg: 24px
  gutter-xl: 32px
  section-sm: 48px
  section-md: 72px
  section-lg: 96px
  page-margin: 20px
  page-margin-desktop: 60px
```

## Elevation & Depth

Hierarchy is established primarily through **tonal surface shifts** (`#FAFAF9` main ground, `#F1F1F0` hero stage, `#F6F6F4` secondary bands, `#FFFFFF` pure cards/drawers) and crisp `1px` hairline borders (`#DEDEDC`). Interface cards avoid heavy drop shadows. Only physical 3D book covers and floating modal/drawer overlays cast soft, directional ambient shadows (`0 24px 48px -12px rgba(23, 23, 23, 0.14)`).

## Shapes

Corners are architectural and restrained — crisp `2px`–`4px` radii on buttons, inputs, and book-cover frames preserve the precision of printed matter, avoiding overly soft, generic SaaS pill cards. Full circular radii (`9999px`) are reserved strictly for small icon badges and carousel pagination dots.

```yaml
rounded:
  none: 0px
  sm: 2px
  DEFAULT: 4px
  md: 6px
  lg: 8px
  full: 9999px
```

## Motion & Micro-interaction

Motion is deliberate, tactile, and unhurried. Interactive controls respond immediately (`120ms`), while book-cover hover lifts and navigation/cart drawers glide with a smooth, damped deceleration curve. Nothing bounces, pulses, or moves unpredictably under a customer's cursor. `prefers-reduced-motion` is respected across all transitions.

```yaml
motion:
  state-change:
    duration: 120ms
    delay: 0ms
    timingFunction: ease-out
  book-lift:
    duration: 280ms
    delay: 0ms
    timingFunction: cubic-bezier(0.22, 1, 0.36, 1)
  surface-enter:
    duration: 300ms
    delay: 0ms
    timingFunction: cubic-bezier(0.2, 0, 0, 1)
  surface-exit:
    duration: 180ms
    delay: 0ms
    timingFunction: cubic-bezier(0.3, 0, 1, 1)
```

## Voice & Content Strategy

- **Tone of voice:** Quietly authoritative, warm, and literate — like an independent bookshop curator. Never pushy, hyped, or SaaS-jargony.
- **Capitalisation:** Title Case for primary editorial headlines and book titles; Sentence case for supporting descriptions, body copy, and navigation items.
- **Terminology:** Use *Bag* (not Cart), *Library* (not Dashboard), *Preview* (not Demo), *Curated* (not Trending AI).
- **Button labels:** Clear verb-led or destination-led labels (*Explore Books*, *Add to Bag*, *Read Sample*, *Subscribe*).
- **Honesty & Restraint:** Never fabricate awards, reader counts, or ratings. Show ratings only when genuine data exists; clearly label review placeholders during design.
- **Numbers:** Always use `tabular-nums` for prices (`$18.00`), cart counts, and reading metadata.

## Do's and Don'ts

- **Do** keep at least 35% of the viewport as breathing room around the 3D hero book trio and editorial headings.
- **Do** maintain strict WCAG 2.2 AA contrast (`4.5:1` for body text, `3:1` for large text and UI controls) using `on-surface` (`#171717`) and `on-surface-muted` (`#5C5D5A`).
- **Don't** use neon gradients, glassmorphic blur, `rounded-2xl` generic cards, decorative numbered badges, or fabricated social proof.
- **Don't** shift book positions unpredictably on hover when a user tries to click them; use subtle elevation or scale only.

## Components

Component implementations live as individual HTML sticker sheet files in
`components/`. Each file contains the canonical, styled HTML components with
`<!-- COMPONENT: name -->` comment markers. Read each file before use.

- **Button** → `@components/button.html`: `button-primary`, `button-icon`, `button-with-icon`
- **Card** → `@components/card.html`: `card-elevated`, `card-action`, `card-interactive`, `card-interactive-2`
- **Drawer** → `@components/drawer.html`: `drawer-left`
- **Input Field** → `@components/input-field.html`: `input-field-text`, `input-field-icon`
- **Modal** → `@components/modal.html`: `modal-standard`, `modal-alert`, `modal-drawer`
- **Admin Kit** → `@components/admin-kit.html`: KPI cards, data tables, status pills, role-permission badges, refund-review modals
- **Support Kit** → `@components/support-kit.html`: SLA timers, ticket list rows, customer-visible vs. private internal note cards, role-restricted action guards, empty/filtered states

## Button & Input Standards

- **Primary Actions:** `bg-primary text-on-primary rounded font-label text-label-sm` (Admin/Support) or `text-label-md` (Customer), with `shrink-0 whitespace-nowrap` on compact header/card actions.
- **Secondary Actions:** `border border-outline bg-surface-white text-on-surface rounded`, transitioning to `hover:bg-surface-secondary` or `hover:border-primary`.
- **Destructive / Escalation Actions:** `border border-error text-error bg-surface-white` for escalations/refund triggers, separated from routine reply or save controls.
- **Restricted Actions:** Disabled state (`disabled cursor-not-allowed bg-surface-hero border-outline text-on-surface-muted`) paired with explicit role-requirement helper text (e.g., *Requires Owner/Admin approval*).
- **Inputs & Selects:** `bg-surface-white border border-outline rounded font-body text-body-sm focus:border-primary focus:ring-0` with explicit `<label for="...">` associations.

## Table & Chart Standards

- **Tables:** Wrapped in `border border-outline rounded bg-surface-white overflow-x-auto` with `bg-surface-secondary border-b border-outline` headers (`uppercase tracking-wider font-label text-label-sm text-on-surface-muted`), `divide-y divide-outline` rows, and `tabular-nums` on all prices, IDs, dates, and SLA timers.
- **Charts & Metrics:** Editorial monochrome bars/sparklines using `primary` (`#202321`), `accent` (`#777F86`), and `success`/`error` only for threshold or compliance states. Avoid decorative 3D or multi-hue charts.

## Cross-Panel Status Dictionary

| Record | Canonical UI States | Notes / Confirmation Required |
| :--- | :--- | :--- |
| **Ebook** | `Draft`, `Scheduled`, `Published`, `Unpublished`, `Archived` | Drafts and Unpublished titles must never appear in the Customer storefront. |
| **Payment** | `Pending`, `Successful` (*Paid* / *Captured*), `Failed`, `Cancelled`, `Refunded`, `Partially Refunded` | Pending payments must never display as Successful or unlock library downloads. |
| **Order** | `Pending`, `Confirmed`, `Completed`, `Cancelled`, `Refund Pending`, `Refunded` | Confirm whether digital-only orders transition immediately from `Confirmed` to `Completed` upon entitlement sync. |
| **Support Ticket** | `Open`, `Pending` (*Pending Customer*), `Escalated`, `Resolved`, `Reopened` | Confirm policy window for customer/agent `Reopened` transitions after `Resolved`. |
| **Customer Account** | `Active` (*Verified*), `Inactive`, `Suspended` | Confirm whether `Suspended` revokes existing library downloads or only blocks new purchases. |

## Navigation Conventions

- **Customer Panel:** Top editorial header (`Explore`, `Collections`, `About`, `Help`, search, bag, account) + mobile drawer; Account area uses a left sidebar on desktop and horizontal pill nav on mobile.
- **Owner/Admin Panel:** Persistent `256px` left sidebar (`Overview`, `Catalog`, `Orders`, `Customers`, `Finance`, `Marketing`, `Website`, `Analytics`, `Support`, `Notifications`, `Team`, `Security`, `Settings`) + sticky top breadcrumb bar + slide-over drawer below `lg:`.
- **Support & Operations Panel:** Persistent `256px` left sidebar (`Dashboard`, `Ticket Inbox` with nested `My Assigned` / `Unassigned` / `Escalated`, `Customers`, `Orders & Payments`, `Ebook Access Issues`, `Knowledge Base`, `Saved Replies`, `Notifications`, `Team Performance`, `Reports`, `Profile & Settings`) + global search, availability toggle, and full 11-link mobile drawer.

## Responsive & Accessibility Rules

- **Breakpoints:** Audited at **Desktop (`1440px`)**, **Tablet (`768px`)**, and **Mobile (`390px`)**. Multi-pane workspaces (such as Ticket Conversation) collapse into an explicit pane switcher below `xl:` rather than stacking unreadably.
- **Contrast & Touch Targets:** Strict WCAG 2.2 AA contrast (`#171717` and `#5C5D5A` on `#FAFAF9`/`#FFFFFF`/`#F6F6F4`), `min-h-[40px]`–`44px` interactive targets on mobile, and visible focus states.
- **Multi-Cue Status & Privacy:** Never rely on color alone for priority, status, or SLA urgency. Private internal notes in Support must always combine a `2px` `border-primary` frame, `bg-surface-hero` fill, a lock icon, and an explicit *Private Internal Note* label.

